### 불필요한 객체 생성을 피하라



#### 문자열 객체 생성

```java
String strObject1 = new String("TEST");
String strObject2 = new String("TEST");
String strObject3 = new String("TEST");
```

문자열 s1, s2, s3는 모두 "TEST"라는 문자열을 갖지만 각각 참조하는 주소가 다르기 때문에 메모리 낭비가 발생한다.

![image-20220510081120835](C:\Users\UserK\AppData\Roaming\Typora\typora-user-images\image-20220510081120835.png)

new 연산자를 통해 문자열 객체를 생성하는 경우 메모리의 Heap 영역에 할당되어 총 3개의 인스턴스가 생긴 것을 알 수 있다.



```java
String strObject1 = "TEST";
String strObject2 = "TEST";
String strObject3 = "Java"
```

위 코드는 하나의 인스턴스를 사용한다. 나아가 이 방식을 사용한다면 같은 JVM안에서 "TEST" 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함을 보장한다.

![image-20220510081309143](C:\Users\UserK\AppData\Roaming\Typora\typora-user-images\image-20220510081309143.png)

반면, 리터럴을 이용한 경우 String Pool 영역에 할당되어 strLiter1과 strLiteral2는 같은 문자열을 참조하고 있는 것을 알 수 있다.

##### String Pool

String Pool은 Heap의 저장 영역이다. 

JVM은 과도한 String 객체의 수를 줄이고자 문자열 리터럴이 생성될 때마다 String Pool을 체크하여 이미 String Pool에 존재하는 경우 해당 인스턴스에 대한 참조가 반환된다. 존재하지 않을 경우 새로운 인스턴스가 String Pool에 생성된다.



#### 정적 팩터리 메서드 사용

생성자는 호출할 때마다 새로운 객체를 만들지만, 정적 팩터리 메서드를 이용하면 불필요한 객체 생성을 피할 수 있다.

```java
Boolean b = new Boolean("true");
```

위 코드는 문자열을 매개변수로 받는 생성자를 통해 Boolean 인스턴스를 만들고 있다. Boolean은 true 혹은 false만 존재하는데, 매번 인스턴스를 만드는 것은 메모리 낭비가 된다. 따라서 정적 팩터리 메서드인 Boolean.valueOf()를 사용하는 것이 좋다.

```java
Boolean b = Boolean.valueOf("true");
```



#### 캐싱하여 재사용

생성 비용이 크면 캐싱하여 재사용하는 것이 좋지만, 우리가 만드는 객체의 비용을 알 수 없다. 

예를 들어 주어진 문자열이 유효한 로마 숫자인지 확인하는 메서드를 작성하고 싶다면, 다음과 같이 정규 표현식을 활용하는 것이 가장 쉽다.

```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

이 방식의 문제는 String.matches 메서드를 사용하는 데 있다. 이 메서드가 내부에서 만드는 정규 표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 되는데, 해당 정규 표현식이 반복해서 사용되는 빈도가 높아질수록 동일한 Pattern 인스턴스가 생성되고 버려지는 비용이 커진다.

따라서, Pattern 인스턴스를 미리 캐싱해 두고, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용하는 것이 좋다.

```java
public class RomanNumerals {

    private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    public static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```



#### 어댑터

불변 객체를 사용한다면 재사용해도 안전하다. 그러나 불변 객체를 재사용한다는 직관에 반대되는 경우가 있다.

어댑터(View)는 실제 작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 객체다. 어댑터는 뒷단 개체만 관리하면 되므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.



예를 들어 Map 인터페이스의 keySet 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰를 반환한다. keyset 메서드를 호출할 때마다 새로운 Set 인스턴스가 만들어지리라 착각할 수 있지만, 사실은 매번 같은 Set 인스턴스를 반환한다.

```java
@DisplayName("어댑터 패턴이 적용된 keySet 테스트")
@Test
void name() {
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "첫번째");
    map.put(2, "두번째");
    map.put(3, "세번째");

    // keySet은 view의 역할을 한다.
    Set<Integer> keySet = map.keySet();
    assertThat(keySet).contains(1, 2, 3);

    // 따라서 뒷단 객체인 map이 기능을 담당하고 있다.
    map.remove(3);
    assertThat(keySet).doesNotContain(3);

    // view인 keySet에서 기능을 동작하려 하면 exception 이 발생한다.
    assertThatThrownBy(() -> keySet.add(3))
        .isInstanceOf(UnsupportedOperationException.class);
}
```



#### 오토박싱

오토박싱은 프로그래머가 기본 타입과 래퍼 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 오토박싱은 기본 타입과 래퍼 타입의 구분을 흐려줄 뿐 완전히 없애주는 것은 아니다.

```java
public static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

로직상 문제는 없지만 성능적으로 매우 비효율적인 코드다. 

sum은 Long 타입이고, i는 long 타입이다. 반복문을 돌면서 i가 sum에 더해질 때마다 새로운 Long 인스턴스가 만들어진다. 때문에 래퍼 타입보다는 기본 타입을 사용하고 의도치 않은 오토박싱이 사용되지 않도록 주의해야 한다.