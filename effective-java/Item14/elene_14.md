## **Comparable 구현**

**1. compareTo**
- Comparable 인터페이스 메서드
- 단순 동치성 비교 + 순서 비교
- 제너릭
- Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻함
==> `Arrays.sort(a);`로 정렬
- String이 Comparable을 구현 → 중복 제거 + 알파벳순으로 출력
```java
public class WordList {
  public static void main(String[] args) {
    Set<String> s = new TreeSet<>();
    Collections.addAll(s, args);
    System.out.println(s);
  }
}
```
```java
public interface Comparable<T> {
  int compareTo(T t);
}
```
---
**2. compareTo 메서드 규약**
> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.\
> 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.\
> 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
> - Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.\
> (따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다)
> - Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
> - Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
> - 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. (x.compareTo(y) == 0) == (x.eqauls(y))여야 한다.\
> Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다.\
> `"주의: 이 클래스의 순서는 equlas 메서드와 일관되지 않는다."`

1) 첫 번째 규약
- 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 함
2) 두 번째 규약
- 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 함
3) 세 번째 규약
- 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 함
- compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것
- compareTo의 순서와 equals의 결과가 일관되지 않은 클래스도 동작은 함\
==> 단, 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스에 정의된 동작과 엇박자를 낼 것
---
**3. compareTo 메서드 작성 요령**
- 타입을 인수로 받는 제너릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해짐\
==> 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻 (인수의 타입이 잘못됐다면 컴파일 자체가 되지 않음)
- null을 인수로 넣어 호출하면 NullPointerException을 던져야 함
- 실제로도 인수(이 경우 null)의 멤버에 접근하려는 순간 이 예외가 던져질 것
---
**4. compareTo 메서드**
- 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출
- 자바가 제공하는 비교자를 사용한 compareTo
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
}
```
- Java 7 : compareTo 메서드에서 관계 연산자 <와 >를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니 추천 X
- 핵실 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해야 함
```java
public int compareTo(PhoneNumber pn) {
  int result - Short.compare(areaCode, pn.areaCode);  //가장 중요한 필드
  if(result == 0) {
    result = Short.compare(prefix, pn.prefix);  //두 번째로 중요한 필드
    if(result == 0) {
      result = Short.compare(lineNum, pn.lineNum);  //세 번째로 중요한 필드
    }
  }
}
```
---
**5. 보조 생성 메서드**
- long과 double용 : comparingInt와 thenComparingInt의 변형 메서드
- short : int용 버전 사용
- float : double용 사용
- 객체 참조용 비교자 생성 메서드\
→ comparing 정적 메서드 2개 다중 정의\
└ 1) 키 추출자를 받아서 그 키의 자연적 순서 비교
└ 2) 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받음\
→ thenComparing 인스턴스 메서드 3개 다중 정의\
└ 1) 비교자 하나만 인수로 받아 그 비교자로 부차 순서를 정함
└ 2) 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조 순서를 정함
└ 3) 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받음

---
**6. compareTo나 compare**
- `값의 차`를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo나 compare 메서드와 마주할 것
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
}
```
└ 사용 X → 정수 오버플로 or 부동소수점 계산 방식에 따른 오류를 낼 수 있음
- 해결방안 2가지
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2,hashCode());
  }
};
```
```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
