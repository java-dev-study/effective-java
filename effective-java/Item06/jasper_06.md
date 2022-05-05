## Item06

> 불필요한 객체 생성을 피하라.

#### 문자열

- 사실상 동일한 객체라서 매번 새로 만들 필요가 없다.

- new String("자바")을 사용하지 않고 문자열 리터럴 ("자바")를 사용해 기존에 동일한 문자열을 재사용하는 것이 좋다

```java

String hello = "hello";

// 이 방법은 권장하지 않습니다.
String hello2 = new String("hello");
String hello3 = "hello";

System.out.println(hello == hello2); //false
System.out.println(hello.equals(hello2)); //true
System.out.println(hello == hello3); //true
System.out.println(hello.equals(hello3)); //true

```

#### 정규식, Pattern

- 생성 비용이 비싼 객체라서 반복해서 생성하기 보다, 캐싱하여 재사용하는 것이 좋다.

```java

// 값비싼 객체를 재사용해 성능을 개선한다. (32쪽)
public class RomanNumerals {
    // 코드 6-1 성능을 훨씬 더 끌어올릴 수 있다!
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"); //Pattern 인스턴스는, 한번 쓰고 버려져셔 곧바로 가비지 컬렉션의 대상
    }

    // 코드 6-2 값비싼 객체를 재사용해 성능을 개선한다.
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }

    public static void main(String[] args) {
        boolean result = false;
        long start = System.nanoTime();
        for (int j = 0; j < 100; j++) {
            //TODO 성능 차이를 확인하려면 xxxSlow 메서드를 xxxFast 메서드로 바꿔 실행해보자.
            result = isRomanNumeralSlow("MCMLXXVI");
        }
        long end = System.nanoTime();
        System.out.println(end - start);
        System.out.println(result);
    }
}

```

#### 오토박싱 (auto boxing) 

- 기본 타입(int)을 그에 상응하는 박싱된 기본 타입(Integer)으로 상호 변환해주는 기술.

- 기본 타입과 박싱된 기본 타입을 섞어서 사용하면 변환하는 과정에서 불필요한 객체가 생성될 수 있다.

```java

public class Sum {
    private static long sum() {
        // TODO Long을 long으로 변경하여 실행해 보세요.
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        long x = sum();
        long end = System.nanoTime();
        System.out.println((end - start) / 1_000_000. + " ms.");
        System.out.println(x);
    }
}

```


> "객체 생성은 비싸니 피하라"는 뜻으로 오해하면 안된다. 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권함.
