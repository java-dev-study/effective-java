## **clone 재정의는 주의해서 진행하기**

**1. Coneable**
- 믹스인 인터스페이스(mixin interface)
- clone 메서드 선언 위치 : Object, protected
- Object의 protected 메서드인 clone의 동작 방식을 결정
- 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경
- Object 명세
> - 이 객체의 본사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.\
> `x.clone() != x`\
> 또한 다음 식도 참이다.\
> `x.clone().getClass() == x.getClass()`\
> 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.\
> 한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.\
> `x.clone().equals(x)`\
> 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.\
> 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.\
> `x.cone().getClass() == x.getClass()`
> 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다.\
> 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

- **생성자 연쇄(constructor chaining)** 와 비슷한 메커니즘 (강제성이 없다는 점만 제외하고)
---
**2. Cloneable 구현**
- 제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속
- super.clone : 원본의 완벽한 복제본
- 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태\
ex) PhoneNumber 클래스
- 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는 게 좋음
```java
@Override public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```
└ 이 메서드가 동작하려면 PhoneNumber의 클래스 선언에 Cloneable을 구현한다고 추가해야 함\
└ try-catch 블록으로 감싼 이유 : Object의 clone 메서드가 검사 예외(checked exception)인 CloneNotSupportedException을 던지도록 선언되었기 때문

---
**3. Stack 복제**
- 클래스가 가변 객체를 참조하는 순간 재앙
```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFUALT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
   
   public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
   }
   
   public Object pop() {
    if(size == 0) {
      throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null;  //다 쓴 참조 해제
    return result;
   }
   
   //원소를 위한 공간을 적어도 하나 이상 확보한다.
   private void ensureCapacity() {
    if(elements.length == size) {
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
   }
}
```
- clone 메서드는 사실상 생성자와 같은 효과를 냄
- clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 함
- 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해 주는 것
```java
@Override public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements == elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```
- elements 필드가 fianl일 경우 작동하지 않음 : final 필드에는 새로운 값을 할당할 수 없기 때문
- Cloneable 아키텍처는 **가변 객체를 참조하는 필드는 final로 선언하라**는 일반 용법과 충돌 → 일부 필드에서 final 한정자를 제거해야 할 수도 있음
---
**4. 해시테이블용 clone**
- 성능을 위해 java.util.LinkedList 대신 직접 수현한 경량 연결 리스트 사용
```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }
  ... //나머지 코드는 생략
}
```
- 버킷 배열의 clone을 재귀적으로 호출
```java
@Override public HashTable clone() {
  try {
    HashTable result = (HashTable) super.clone();
    result.buckets = buckets.clone();
    return result;
  } catch(CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```
- 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생김\
==> 이를 해결하기 위해 각 버킷을 구성하는 연결 리스트를 복사해야 함
```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
    
    //이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
    Entry deepCopy() {
      return new Entry(key, value, next == null ? null : next.deepCopy());
    }
  }
  
  @Ovveride public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for(int i = 0; i < buckets.length; i++) {
        if(buckets[i] != null) {
          result.buckets[i] = buckets[i].deepCopy();
        }
      }
      result result;
    } catch(CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
  ... //나머지 코드는 생략
}
```
- Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출
- 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비 → 리스트가 길면 스택 오버플로를 일으킬 위험이 있음\
==> deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정
```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for(Entry p = result; p.next != null; p = p.next) {
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  }
  result result;
}
```
---
**5. clone 구현**
- Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 함
- 그렇지 않은 상황에서는 **복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공**할 수 있음\

`복사 생성자 : 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자`
```java
public Yum(Yum yum) { ... };
```
`복사 팩터리 : 복사 생성자를 모방한 정적 팩터리`
```java
public static Yum newInstance(Yum yum) { ... };
```
