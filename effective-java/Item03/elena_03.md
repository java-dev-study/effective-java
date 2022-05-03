## **싱글턴 보증하기**

**싱글턴(Singletone)**
> - 인스턴스를 오직 하나만 생성할 수 있는 클래스
> - ex) 무상태(stateless) 객체(함수), 시스템 컴포넌트(객체나 설계상 유일)
> - 클래스를 싱글턴으로 만들 경우 이를 사용하는 클라이언트를 테스트하기 어려움
---
**싱글턴 만드는 방식**
1. public static 멤버가 final
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  
  public void leaveTheBuilding() { ... }
}
```
- private 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출됨
- 권한이 있는 클라이언트의 경우 리플렉션 API(AccessibleObject.setAccessible)을 사용해 private 생성자 호출 가능 (두 번째 객체가 생성될 때 예외 처리)
- 해당 클래스가 싱글턴임이 명백히 드러남
- 간결
  
2. 정적 팩터리 메서드를 public static 멤버로 제공
  ```java
  public class Elvis {
    privaet static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTACNE; }
    
    public void leaveTheBuilding() { ... }
  }
  ```
  - getInstance는 항상 같은 객체의 참조를 반환
  - 리플렉션을 통한 예외는 동일 적용
  - 싱글턴이 아니게 변경하기에 용이
  - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들기 용이
　
3. 열거 타입 선언
```java
public enum Elvis {
  INSTANCE;
  
  public void leaveTheBuilding() { ... }
}
```
- 간결
- 직렬화 용이
- 제2의 인스턴스가 생기는 일을 완벽히 차단
