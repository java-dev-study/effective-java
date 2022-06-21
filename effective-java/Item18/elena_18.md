## **상속보단 컴포지션**

**1. 상속**
- 코드를 재사용하는 수단
- 다른 패키지의 구체 클래스를 상속하는 일은 위험
- 구현 상속 : 클래스가 다른 클래스를 확장하는 것
- 메서드 호출과 달리 **캡슐화를 깨뜨림**
---
**2. 컴포지션**
- 기존 클래스 확장
- 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조\
==> 기존 클래스가 새로운 클래스의 구성요소에서 쓰임 : **컴포지션(composition)**
- 메소드 재정의 등의 문제 해결 가능한 방법
- 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환 : **전달(forwarding method)**
- 새로운 클래스는 기존 클래스의 내부 구현 방식에서 벗어남
- 기존 클래스에 새로운 메서드가 추가되더라도 영향을 받지 않음

`래퍼 클래스`
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;
  
  public InstrumentedSet(Set<E> s) {
    super(s);
  }
  
  @Override public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

`재사용할 수 있는 전달 클래스`
```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;
  public ForwardingSet(Set<E> s) { this.s = s; }
  
  public void clear()                         { s.clear(); }
  public boolean contains(Object o)           { return s.contains(o); }
  public boolean isEmpty()                    { return s.imEmpty(); }
  public int size()                           { return s.size(); }
  public Iterator<E> iterator()               { return s.iterator(); }
  public boolean add(E e)                     { return s.add(e); }
  public boolean remove(Object o)             { return s.remove(o); }
  public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
  
  ... //중략
}
```
---
**3. 래퍼 클래스**
- 콜백(callback) 프레임워크와는 어울리지 않는다는 점 외에는 단점의 거의 없음
---
**4. 정리**
- 컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 것
> 1. API가 내부 구현에 묶임
> 2. 그 클래스의 성능도 영원히 제한됨
> 3. 클라이언트가 노출된 내부에 직접 접근할 수 있음
> 4. 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수 있음
