## Item19

> 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 상속을 고려한 설계와 문서화란?

- 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남김.

- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 함.

- 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수도 있다.
    - 이때, 호출되는 메서드가 재정의 가능 메서드라면 그 사실을 호출하는 메서드의 API 설명에 적시해야 한다.
    - 덧붙여서 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.

- 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.

- 클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야만 한다.

### 상속용 클래스 설계

- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.

- 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.

- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.


```java

// 상위 클래스
public class Super {

    public Super() {
        overrideMe();
    }

    public void overrideMe() {

    }

}

// 하위 클래스
public final class Sub extends Super {

    //초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        Instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}


```

상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출한다. 

> private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

- Cloneable과 Serializable 인터페이스의 clone과 readObject을 구현할 때 따르는 제약도 생성자와 비슷하다.
    - 즉, clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다. 


### 상속을 금지하는 방법

- 클래스를 final로 선언.

- 모든 생성자를 private나 package-private로 선언하고 정적 팩터리르 만들어주는 방법.
    - 정적 팩터리 방법은 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 준다.