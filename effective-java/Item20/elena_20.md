## **추상 클래스보다는 인터페이스를 우선**

**1. 다중 구현 메커니즘**
- 자바가 제공하는 다중 구현 메커니즘 : 인터페이스, 추상 클래스
- 자바 8부터 인터페이스도 `디폴트 메서드(default method)`를 제공
- 차이
> 1. 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점
> 2. 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급
---
**2. 인터페이스**\
(1) **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있음**
> 1. 인터페이스가 요구하는 메서드 추가
> 2. 클래스 선언에 implements 구문 추가
> - 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려움

(2) **믹스인(mixin) 정의에 안성 맞춤**
> - 믹스인 : 클래스가 구현할 수 있는 타입, 원래의 `주된 타입` 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 줌
> ex) Comparable
> 추상 클래스로는 믹스인을 정의할 수 없음 (기존 클래스에 덧씌울 수 없기 때문)

(3) **계층구조가 없는 타입 프레임워크를 만들 수 있음**
```java
/*
* 가수 인터페이스와 작곡가 인터페이스
*/

public interface Singer {
  AudioClip sing(Song s);
}

public interface Songwriter {
  Song compose(int chartPosition);
}
```

```java
/*
* 타입을 인터페이스로 정의하면 가수 클래스가 Singer와 Songwriter 모두를 구현해도 문제가 되지 않음
* 심지어 Singer와 Songwriter 모두를 확장하고 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있음
*/
public interface SingerSongWriter extends Singer, Songwriter {
  AudioClip strum();
  void actSensitive();
}
```

(4) **래퍼 클래스 관용구와 함께 사용하면 기능을 향상시키는 안전하고 강력한 수단이 됨**

---
**3. 디폴트 메서드**
- 인터페이스의 메서드 중 구현 방법이 명백한 것
- 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 `@implSpec` 자바독 태그를 붙여 문서화해야 함

`제약`
- 많은 인터페이스가 equals와 hashCode 같은 Object의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안 됨
- 인터페이스는 인스턴스 필드를 가질 수 없음
- public이 아닌 정적 멤버를 가질 수 없음 (단, private 정적 메서드는 예외)
- 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없음
---
**4. 추상 골격 구현(skeletal implementation) 클래스**
- 인터페이스와 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법
- 인터페이스 : 타입 정의, 필요 시 디폴트 메서드 몇 개도 함께 제공
- 골격 구현 클래스 : 나머지 메서드들까지 구현

└ **`템플릿 메서드 패턴`**
- 관례상 인터페이스 이름이 `Interface`라면 골격 구현 클래스의 이름은 `AbstractInterface`

`AbstractList 골격 구현`
```java
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);
  
  // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 사용 가능하다.
  // 더 낮은 버전을 사용한다면 <Integer>로 수정
  return new AbstractList<>() {
    @Override public Integer get(int i) {
      return a[i];  //오토박싱
    }
    
    @Override public Integer set(int i, Integer val) {
      int oldVal = a[i]; 
      a[i] = val; //오토언박싱
      return oldVal;  //오토박싱
    }
    
    @Override public int size() {
      return a.length;
    }
  };
}
```

└ int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 `어댑터(Adapter)`이기도 함

- 추상 클래스처럼 구현을 도와 줌
- 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서 자유로움
- 우회적으로 사용 가능 \
: 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스 정의 & 각 메서드 호출을 내부 클래스의 인스턴스에 전달

└ **`다중 상속(simulated multiple inheritance)`**

---
**5. 골격 구현 작성**
1. 인터페이스에서 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정 (추상 메서드가 될 것)
2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드들을 모두 디폴트 메서드로 제공
3. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면,
\이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성
4. 필요하면 public이 아닌 필드와 메서드를 추가해도 됨

- `골격 구현`은 기본적으로 상속해서 사용하는 걸 가정
---
**6. 단순 구현(simple implementation)**
- 골격 구현의 작은 변종
- ex) AbstractMap.SimpleEntry
- 상속을 위해 인터페이스를 구현한 것
- 추상 클래스가 아님
- 즉, 동작하는 가장 단순한 구현
