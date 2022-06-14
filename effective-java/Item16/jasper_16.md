## Item 16

> public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

```java

class Point {
    public double x;
    public double y;
}


```

해당 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.

API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

따라서 필드을 모두 private으로 바꾸고 public 접근자(getter)를 추가한다.

```java

class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return this.x; }
    public double getY() { return this.y; }
}


```

public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.

그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.

> public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을때도 있다.

