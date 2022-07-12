## **태그 달린 클래스보다는 클래스 계층구조를 활용하라**

**1. 태그 달린 클래스**

- 두 가지 이상의 의미 표현이 가능하면서 현재 표현하는 의미를 태그 값으로 알려 주는 클래스

`↓ 원과 사각형을 표현하는 태그 달린 클래스`
```java
class Figure {
  enum Shape { RECTANGLE, CIRCLE };
  
  // 태그 필드 - 현재 모양을 나타냄
  final Shape shape;
  
  // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰임
  double length;
  double width;
  
  // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰임
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

**단점**
1. 열거 타입 선언, 태그 필드, switch문 등 **쓸데없는 코드 多**
2. 여러 구현이 한 클래스에 혼합되어 있어 **가독성이 나쁨**
3. 다른 의미를 위한 코드도 언제나 함께 해서 **메모리가 많이 사용됨**
4. 필드들을 final로 선언하려면 해당 의미에 **쓰이지 않는 필드들까지 생성자에서 초기화**
5. 다른 의미를 추가하려면 **코드 수정 필요**\
`ex) 삼각형(TRIANGE)을 추가하게 되면, 열거 타입 수정 + 삼각형용 생성자 추가 + switch문 추가 등의 작업이 필요`
6. 인스턴스 타입만으로는 **현재 나타내는 의미를 알 길이 없음**

==> **태그 달린 클래스는 장황 + 오류를 내기 쉬움 + 비효율적**이며 계층구조를 어설프게 흉내낸 아류일 뿐

---
**2. 계층 구조**

**계층 구조로 바꾸는 방법**
1. 계층구조의 루트(root)가 될 추상 클래스 정의
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언 → `Figure 클래스에서는 area가 해당`
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스에 작성\
`cf) Figure 클래스에서는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없음`\
`→ 루트 클래스에는 추상 메서드는 area 하나만 남게 됨`
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의 → `Figure를 확장한 원(Circle) 클래스와 사각형(Rectangle) 클래스`
6. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드를 넣음 → `원에는 반지름(radius), 사각형에는 길이(length)와 너비(width)`
7. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현

`↓ 태그 달린 클래스를 클래스 계층구조로 변환`
```java
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
  final double radius;
  
  Circle(double radius) {
    this.radius = radius;
  }
  
  @Override
  double area() {
    return Math.PI * (radius * radius);
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
    return length * width;
  }
}
```

**결과**
- 간결하고 명확
- 쓸데없는 코드 제거
- 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드 제거
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 구현했는지 컴파일러가 확인
- 실수로 빼먹은 case 문 때문에 런타임 오류가 발생하지도 않음
- 루트 클래스의 코드 수정 없이 독립적으로 계층구조를 확장하고 함께 사용 가능
- 유연성 및 컴파일타임 타입 검사 능력 향상

`+ 클래스 계층구조에서라면 아래와 같이 정사각형이 사각형의 특별한 형태임을 간단하게 반영 가능`
```java
class Square extends Rectangle {
  Square(double side) {
    super(side, side);
  }
}
```
