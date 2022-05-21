## Item 10

> equals는 일반 규약을 지켜 재정의하라.

### 1. equals를 재정의 하지 않는 것이 최선

- 각 인스턴스가 본질적으로 고유하다.
    - 싱글톤, enum 

- 인스턴스의 '논리적 동치성'을 검사할 필요가 없다.

- 상위 클래스에서 재정의한 equals가 하위 클래스에도 적절하다.

- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.


---


### 2. equals를 재정의해야 할 때

- 객체 식별성(Object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않을 때.

- 주로 값 클래스들이 여기에 해당. -> VO (value object)
    - 값 클래스라 해도, 같이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 됨.

---


### 3. equals 규악

#### 3.1 반사성(reflexivity)

> null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.

- 객체는 자기 자신과 같아야 함.

#### 3.2 대칭성(symmetry)

> null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.

- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

#### 3.3 추이성(transitivity)

> null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.

#### 3.4 일관성(consistency) 

> null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다(불변 객체)

- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministc) 계산만 수행.

#### 3.5 null-아님

> null이 아닌 모든 참조 값 x에 대해. x.equals(null)은 false 이다.

---

### 4. equals 구현 방법

- == 연산자를 사용해 자기 자신의 참조인지 확인한다.

- instanceof 연산자로 올바른 타입인지 확인한다.

- 입력된 값을 올바른 타입으로 형변환 한다.

- 입력 객체와 자기 자신의 대응되는 핵심 필드가 일치하는지 확인한다.


---

### 5. 주의 사항

- equals를 재정의 할 때 hasCode도 반드시 재정의 하자.

- 너무 복잡하게 해결하지 말자.

- Object가 아닌 타입의 매개변수를 받는 equals 메서드는 선언하지 말자.