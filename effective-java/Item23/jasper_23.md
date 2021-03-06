## Item 23

> 태그 달린 클래스보다는 클래스 계층 구조를 활용하라

```java
class Figure {

    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE :
                return this.length * this.width;
            case CIRCLE :
                return Math.PI * (this.radius * this.radius);
            default :
                throw new AssertionError(shape);
        }
    }

}

```


위의 코드 처럼 태그 달린 클래스에는 단점이 한가득이다. 

- 열거 타입 선언, 태그 필드, switch 문 등 쓸데 없는 코드가 많다.

- 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다.

- 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
    - 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다(쓰지 않는 필드를 초기화하는 불필요한 코드가 늘어난다.)

- 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는 데 컴파일러가 도와줄 수 있는 건 별로 없다. 
    - 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러날 뿐이다.

- 또 다른 의미를 추가하려면 코드를 수정해야 한다.

- 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.


> 태그 달린 클래스는 장황하고, 오류 내기 쉽고, 비효율적이다. 클래스 계층 구조를 어설프게 흉내낸 아류일 뿐이다.


#### 태그 달린 클래스를 클래스 계층 구조로 바꾸는 방법

- 가장 먼저 계층구조의 루트(root)가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
    - 위 코드에서 area가 이러한 메서드에 해당한다.

- 그런 다음 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.

- 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.

- Figure 클래스에서는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 그 결과 루트 클래스에는 추상 메서드인 area 하나만 남게 된다.

- 다음으로, 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.
    - Figure를 확장한 원(Circle) 클래스와 사각형(Rectangle) 클래스를 만들면 된다.

- 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다.
    - 원에는 반지름(radius)을, 사각형에는 길이(length)와 너비(width)를 넣으면 된다.

- 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다. 

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    double area() {
        return Math.PI * (this.radius * this.radius);
    }

}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return this.length * this.width;
    }

}

```

위의 코드 처럼 클래스 계층 구조는 태그 달린 클래스의 단점을 모두 날려버린다.

간결하고 명확하며 쓸데없는 코드도 모두 사라졌다. 각 의미를 독립죈 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다. 살아 남은 필드들은 모두 final이다.
각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다. 실수로 빼먹은 case문 때문에 런타임 오류가 발생할 일도 없다.

루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 타입이 의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 또 특정 의미만 매개변수로 받을 수 있다.

또한, 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연셩은 물론 컴파일타임 타입 검사 능력을 높여준다는 장점도 있다.

> 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층 구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층 구조로 리팩터링하는 걸 고민해보자.