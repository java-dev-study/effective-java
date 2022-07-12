### 태그 달린 클래스보다는 클래스 계층 구조를 활용하라

**태그 달린 클래스는 장황하고, 오류 내기 쉽고, 비효율적이다.**

```java
/**
 * 태그 달린 클래스
 */
public class Figure {
    enum Shape {
        RECTANGLE, CIRCLE
    }

    // 태그 필드
    final Shape shape;

    // 사각형
    double length;
    double width;

    // 원
    double radius;

    // 원 모양 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형 모양 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

#### 태그 달린 클래스를 클래스 계층 구조로 변경하는 방법

1. 계층 구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.

```java
/**
 * 클래스 계층 구조
 */
abstract class Figure2 {
    abstract double area();
}

class Circle extends Figure2 {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure2 {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

클래스 계층 구조로 변경할 경우, 가독성이 높아지고 효율적인 코드를 작성할 수 있다. 또한, 루트 클래스의 코드를 건드리지 않고도 독립적으로 계층 구조를 확장하고 함께 사용할 수 있다.