### equals는 일반 규약을 지켜 재정의하라

> equals를 재정의하지 않는 것이 최선

#### equals를 재정의할 필요가 없는 상황

- 각 인스턴스가 본질적으로 고유하다.
  - Singleton, Enum
- 인스턴스의 "논리적 동치성"을 검사할 필요가 없다.
  - 여기서 말하는 논리적 동치성은 "값 자체"를 의미하며, 문자열의 경우 논리적 동치성을 검사할 필요가 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 적절하다.
  - Set의 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 쓴다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
  - Public class의 경우 equals 메서드를 호출할 가능성이 있지만, private class인 경우 외부에서 equals 메서드를 호출할 일이 없다.

#### equals 규약

- 반사성 : A.equals(A) == true
  - 자기 자신과 비교했을 때 같아야 한다.
- 대칭성 : A.equals(B) = B.equals(A)
- 추이성 : A.equals(B) && B.equals(C), A.equals(C)
- 일관성 : A.equals(B) == A.equals(B)
  - 두 객체가 같다면, 앞으로도 영원히 같아야 한다.
- Null-아님 : A.equals(null) == false

#### equals 구현 방법

1. == 연산자를 이용해 자기 자신의 참조인지 확인한다.

2. Instance 연산자로 올바른 타입인지 확인한다.

3. 입력된 값을 올바른 타입으로 형변환 한다.

4. 입력 객체와 자기 자신의 대응되는 핵심 필드가 일치하는지 확인한다.

   - 기본 타입은 == 연산자로 비교하고, 참조 타입은 equals 메서드로 비교한다.

   - 단, float와 double 타입은 Float.compare(float, float)와 Double.compare(double, double)로 비교하는데, 특수한 부동소수 값을 다루기 때문이다.

#### equals 재정의 주의사항

- equals를 재정의할 때 hashCode도 반드시 재정의하자
- 너무 복잡하게 해결하지 말자
- Object가 아닌 타입의 매개변수를 받는 equals 메서드는 선언하지 말자.