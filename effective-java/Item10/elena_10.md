# **일반 규약 지켜 equals 재정의하기**

**1. 문제 회피**
- equals 메서드 재정의의 문제를 회피하는 가장 쉬운 방법은 재정의를 하지 않는 것
- 아래 中 하나에 해당한다면 재정의 X
> 1. 각 인스턴스가 본질적으로 고유\
> `동작하는 개체를 표현하는 클래스`\
> `ex) Thread`
> 2. 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없음
> 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞음
> 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없음\
> `equals가 실수로라도 호출되는 경우 방지`
> ```java
> @Override public boolean equals(Object o) {
>   throw new AssertionError();
> }
---
**2. 재정의해야 할 때**
- 객체 식별성이 아닌 논리적 동치성을 확인해야 함
- **상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때**
- 값 클래스가 이에 해당 (Integer, String : 값을 표현하는 클래스)
---
**3. equals 재정의 규약**
- Object 명세에 적힌 규약
> equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
> - **반사성(reflexivity)** : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.\
> `객체는 자기 자신과 같아야 한다는 뜻`
> - **대칭성(symmetry)** : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.\
> `두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻`
> - **추이성(transitivity)** : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 ture면 x.equals(z)도 true다.\
> `첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻`
> - **일관성(consistency)** : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)르 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.\
> `두 객체가 같다면 어느 하나 혹은 두 객체 모두가 수정되지 않는 한 앞으로도 영원히 같아야 한다는 뜻`
> - **null-아님** : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.\
> `모든 객체가 null과 같지 않아야 한다는 뜻`

- equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응하는지 알 수 없음 
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않음
---
**4. 신뢰할 수 없는 자원**
- equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 됨
- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 함
---
**5. equals 메서드 구현 방법**
1) == 연산자를 사용해 입력이 자기 자신의 참조인지 확인
> - 자기 자신이면 true를 반환\
> - 비교 작업이 복잡한 상황일 때 값어치를 함
2) instanceof 연산자로 입력이 올바른 타입인지 확인
> - 어떤 인터페이스는 자신을 구현한 클래스끼리도 비교할 수 있도록 equals 규약을 수정하기도 함
> - Set, List, Map, Map.Entry
3) 입력을 올바른 타입으로 형변환
4) 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사
> - 모든 필드가 일치하면 true, 하나라도 다르면 false를 반환
---
**5. equals 구현 후**
1. 대칭적인가?
2. 추이성이 있는가?
3. 일관적인가?
---
**6. 주의사항**
1. equals를 재정의할 땐 hashCode도 반드시 재정의
2. 복잡하게 해결 X
3. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언 X
