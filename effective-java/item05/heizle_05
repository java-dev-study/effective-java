1. 개요 
    
    : 서비스의 사이즈가 커지면 의존하는 객체들이 늘어나기 쉽다. 따라서 해당 서비스 내에서 다른 서비스나 자원을 사용할 경우 직접 객체들을 생성하여 사용하는 것이 아니라, 의존을 주입(Dependency Injection)받아 사용하는 것이 좋다. 
    
2. 의존성 (Depencency)
: In object oriented programming, an object 'A' is said to be dependent on another object 'B' if object 'A' contains 'B' as a data member. This means that the execution of 'A' is dependent on 'B' and thus 'A' has a dependency on 'B'.
한 객체가 다른 객체를 사용할 때 의존성이 있다고 한다. 예를 들어 Store 객체가 Pencil 객체를 사용하고 있는 경우에 우리는 Store 객체가 Pencil 객체에 의존성이 있다고 표현한다.

```java
public class Store {

	private Pencil pencil;

}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/056b7877-9a82-48aa-af81-561fe4e39ade/Untitled.png)

그리고 두 객체 간의 관계(의존성)를 맺어주는 것을 의존성 주입이라고 하며 생성자 주입(****Constructor Injection****), 필드 주입**(Field Injection)**, 수정자 주입**(Setter Injection)**등 다양한 주입 방법이 있다. Spring 4 부터는 생성자 주입을 강력이 권장하고 있다. (출처 : [https://mangkyu.tistory.com/150](https://mangkyu.tistory.com/150))
