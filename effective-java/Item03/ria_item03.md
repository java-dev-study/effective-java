### Private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴이란 오직 하나의 인스턴스를 생성할 수 있는 클래스를 말한다. 한 번 생성된 인스턴스를 공유하기 때문에 메모리 낭비를 방지할 수 있으나, 멀티 스레드 환경에서 동시성 문제가 발생할 수 있다.

> 싱글턴을 만드는 방법

#### public static 멤버가 final 필드인 방식
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {}
    
    public void speak() {
    	System.out.println("elvis");
    }
}
```
private 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출되어 전체 시스템에서 유일한 인스턴스임을 보장한다. 단, AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있기 때문에 두 번째 객체가 생성될 때 예외를 던져 막을 수 있다.

- 장점
  - 해당 클래스가 싱글턴임이 API에 명백히 드러남
  - 간결함

#### 정적 팩터리 메서드를 public static로 제공하는 방식
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {}
    
    public static Elvis getInstance() {
    	return INSTANCE;
    }
    
    public void speak() {
    	System.out.println("elvis");
    }
}
```
- 장점
  - API를 바꾸지 않고도 싱글턴이 아니게 변경 가능
  - 정적 팩터리를 제네릭 싱글턴 팩터리로 변경 가능
  	-> 동일한 인스턴스를 다른 타입으로 반환
  - 정적 팩터리의 메서드 참조를 공급자로 사용 가능
  
#### 열거 타입을 사용하는 방식
```java
public enum Elvis {
	INSTANCE;
    
    public void speak() {
    	System.out.println("elvis");
    }
}
```
싱글턴을 구현하는 가장 바람직한 방법으로 위의 두 방식에 비해 간결하고, 리플렉션 공격에도 안전하다.
단, 클래스를 상속받을 수 없다는 것은 주의해야 한다.