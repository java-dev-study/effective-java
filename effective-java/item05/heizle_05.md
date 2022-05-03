Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
=============
### 1. 개요 
    
    > 서비스의 사이즈가 커지면 의존하는 객체들이 늘어나기 쉽다.
    따라서 해당 서비스 내에서 다른 서비스나 자원을 사용할 경우 직접 객체들을 생성하여 사용하는 것이 아니라, 의존을 주입(Dependency Injection)받아 사용하는 것이 좋다. 
    
### 2. 의존성 (Depencency)

	> In object oriented programming, an object 'A' is said to be dependent on another object 'B' if object 'A' contains 'B' as a data member. 
	This means that the execution of 'A' is dependent on 'B' and thus 'A' has a dependency on 'B'.
	한 객체가 다른 객체를 사용할 때 의존성이 있다고 한다. 
	예를 들어 Store 객체가 Pencil 객체를 사용하고 있는 경우에 우리는 Store 객체가 Pencil 객체에 의존성이 있다고 표현한다.

```java
public class Store {

	private Pencil pencil;

}
```

	> 그리고 두 객체 간의 관계(의존성)를 맺어주는 것을 의존성 주입이라고 하며 생성자 주입(Constructor Injection****), 필드 주입**(Field Injection)**, 수정자 주입**(Setter Injection)**등 다양한 주입 방법이 있다. 
	Spring 4 부터는 생성자 주입을 강력이 권장하고 있다.
	
### 3. 정적 유틸리티 클래스 및 싱글톤의 한계

	> 사용하는 자원에 따라 동작이 달라지는 클래스 에서는 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다. 
	대신, 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다. 
	인스턴스를 생성할 때, 필요한 자원 (의존 객체 ex. Lexicon)을 클래스가 자체 생성하는 것이 아니라 주입해준다. 
	인스턴스의 불변성을 보장해주며, 다양한 종류의 자원(의존 객체)을 사용할 수 있게 된다. 
	이를통해 클래스의 유연성(flexible), 재사용성(reuesable), 테스트 용이성(testable)이 높아진다.

### 4. 자원 팩토리 (Factory Method pattern)
	- 생성자에 자원 팩토리를 넘겨주는 방식을 적용할 수 있다. 즉, 팩토리 메서드 패턴을 구현하는 것이다.
	> Supplier<T> is perfectly suited for representing factories
	methods that take this interface should typically constrain the factory’s type parameter with a bounded wildcard type (Item 31)
	the client should be able to pass in a factory that requires any subtype of a specified type
	
	
