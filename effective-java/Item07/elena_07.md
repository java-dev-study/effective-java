## **다 쓴 객체 참조 해제하기**

**1. 문제 코드**
```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    return elements[--size];
  }
  
  /**
   * 원소를 위한 공간을 적어도 하나 이상 확보한다.
   * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
   */
  private void ensureCapacity() {
    if (elements.length == size) {
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```
- **메모리 누수** 문제가 있는 코드
> 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않음\
> 이 스택이 그 객체들의 **다 쓴 참조(obsolete reference, 앞으로 다시 쓰지 않을 참조)** 를 여전히 가지고 있기 때문\
> elements 배열의 '활성 영역' 밖의 참조들이 모두 이에 해당 (활성 영역은 인덱스가 size보다 작은 원소들로 구성)
- 가비지 컬렉션 활동과 메모리 사용량이 늘어나게 됨 → 성능 저하
- 디스크 페이징, OutOfMemoryError 발생 가능성도 O
---
**2. 해결 방안**
```java
public Object pop() {
  if (size == 0) {
    throw new EmptyStackException()l
  }
  Object result = elements[--size];
  elements[size] = null;  //다 쓴 참조 해제
  return result;
}
```
- 해당 참조를 다 썼을 때 null 처리(참조 해제)
- null 처리한 참조를 사용하려 하면 NullPointerException을 던질 수 있음
　
- 그러나, null 처리하는 일은 **예외적인 경우**여야 함
- 가장 좋은 방법은 **그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것**
---
**3. 주의점**
- 자기 메모리를 직접 관리하는 클래스라면 항시 메모리 누수에 주의
- 캐시도 메모리 누수를 일으키는 주범
> WeakHashMap 사용\
> 다 쓴 엔트리는 그 즉시 자동으로 제거될 것\
> 백그라운드 스레드 활용\
> LinkedHashMap : removeEldestEntry 메서드 사용
- 리스너(listner) 혹은 콜백(callback)도 메모리 누수의 주범
> 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거
