## **finalizer와 clenar사용을 피하라**

**1. 객체 소멸자**
1) finalizer
- 예측 불가능
- 상황에 따라 위험
- 오동작, 낮은 성능, 이식성
- 자바 9 → 사용 자제 API로 지정, cleaner를 그 대안으로 소개
\n
2) cleaner
- finalizer보다는 덜 위험
- 여전히 예측 불가능
- 느림
- 일반적으로 불필요

> cf. C++의 `파괴자(destructor)`\
> 특정 객체와 관련된 자원을 회수하는 보편적인 방법\
> 비메모리 자원을 회수하는 용도로도 사용
---
**2. 단점**
1. 즉시 수행된다는 보장이 없음
> - finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없음
> - 가비지 컬렉터 알고리즘에 따라 천차만별
> - finalizer의 경우 자원 회수가 제멋대로 지연될 수 있음
> - cleaner의 경우 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있음\
> => 제때 실행되어야 하는 작업은 절대 할 수 없음

2. 수행 여부 조차 보장하지 않음
> - 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해선 안 됨
> - System.runFinalizersOnExit & Runtime.runFinalizersOnExit => 실행을 보장해 주나 심각한 결함이 있음
> finalizer 동작 중 발생한 예외는 무시
> 처리할 작업이 남았더라고 그 순간 종료

3. 성능 문제
> - finalizer는 가비지 컬렉터의 효율을 떨어뜨림\
> => 안전망 형태로만 사용하면 빨라짐

4. finalizer 공격 노출
> - 심각한 보안 문제
> - 생성자나 직렬화 과정에서 예외 발생 시 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있음
> - 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지도 않음
> => 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언
---
**3. 대안**
- AutoCloseable을 구현하고, 인스턴스를 다 쓰고 나면 close 메서드 호출
- close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록
- 다른 메서드는 이 필드를 검사하여 객체가 닫힌 후 부르면 IllegalStateException을 던짐
---
**4. 적절한 활용**
1. 안전망 역할
> - 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비
> - 자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공\
> ex) FileInputStream, FileOutputStream, ThreadPoolExecutor

2. 네이티브 피어(native peer)와 연결된 객체에서 사용
cf) native peer : 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
> - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못함
> - 즉, 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못 함
> - 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 finalizer와 cleaner 사용
---
**5. cleaner**
1. 방(room) 자원을 수거하기 전에 반드시 청소(clean)해야 함
2. Room 클래스는 AutoCloseable 구현
```java
public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create();
  
  //청소가 필요한 자원 | 절대 Room을 참조해서는 안 됨
  private static class State implements Runnable {
    int numJunkPiles; //방(Room) 안의 쓰레기 수
    
    State(int numJunkPiles) {
      this.numJunkPiles = numJunkPiles;
    }
    
    //close 메서드나 cleaner가 호출됨
    @Override public void run() {
      System.out.println("방 청소");
      numJunkPiles = 0;
    }
  }
  
  //방의 상태 | cleanable과 공유
  private final State state;
  
  //cleanable 객체 | 수거 대상이 되면 방을 청소
  private final Cleaner.Cleanable cleanable;
  
  public Room(int numJunkPiles) {
    state = new State(numJunkPiles);
    cleanable = cleaner.register(this, state);
  }
  
  @Override public void close() {
    cleanable.clean();
  }
}
```
- State는 Runnable을 구현, 그 안의 run 메서드는 cleanable에 의해 딱 한 번만 호출
- cleanable 객체는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻음
- State 인스턴스가 Room 인스턴스를 참조할 경우 `순환참조`가 생겨 가비지 컬렉터가 Room 인스턴스를 회수할 기회가 오지 않음
- State를 정적 중첩 클래스로 구현함으로 인해 자동으로 바깥 객체의 참조를 갖게 됨
- 람다 역시 바깥 객체의 참조를 갖기 쉬우니 사용하지 않는 것이 좋음\
↓
잘 짜인 클라이언트 코드
```java
public class Adult {
  public static void main(String[] args) {
    try (Room myRoom = new Room(7)) {
      System.out.println("안녕~");
    }
  }
}
```
↓
절대 방 청소를 하지 않는 코드
```java
public class Teenager {
  public static void main(String[] args) {
    new Room(99);
    System.out.println("아무렴");
  }
}
```
---
**6. cleaner 명세**
> System.exit을 호출할 때의 cleaner 동작은 구현하기 나름이다. 청소가 이뤄질지는 보장하지 않는다.
