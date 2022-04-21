## 아이템1

> 생성자 대신 정적 팩터리 메서드를 고려하라.

---

### 정적 팩터리 메서드와 GoF 팩터리 메서드의 차이

1. 정적 팩터리 메서드 : 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드.

```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.True : Boolean.FALSE;
}
```

2. GoF 팩터리 메서드 : 구체적으로 어떤 인스턴스를 만들지는 서브 클래스가 정한다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8aa3b475-57ee-4100-add2-db40107793a6/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-04-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.54.14.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220421%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220421T125519Z&X-Amz-Expires=86400&X-Amz-Signature=9155e763872298dc88960852c1fd4441ef671cbd1f711b574183f204d4ab34c8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-04-21%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.54.14.png%22&x-id=GetObject)

Ex) Ship을 만드는 로직. (Ship에는 BlackShip, WhiteShip 등이 있음)

    1. 유효성 검사
    2. Ship에 로고를 새긴다.
    3. Ship에 color를 입힌다.
    4. 유저한테 email을 날린다.

```java
// OCP(개방 폐쇄 원칙), SRP(단일 책임원칙) 위배
public static Ship orderShip(String name, String email) {
        // validate
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("배 이름을 지어주세요.");
        }
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("연락처를 남겨주세요.");
        }

        prepareFor(name);

        Ship ship = new Ship();
        ship.setName(name);

        // Customizing for specific name
        if (name.equalsIgnoreCase("whiteship")) {
            ship.setLogo("\uD83D\uDEE5️");
        } else if (name.equalsIgnoreCase("blackship")) {
            ship.setLogo("⚓");
        }

        // coloring
        if (name.equalsIgnoreCase("whiteship")) {
            ship.setColor("whiteship");
        } else if (name.equalsIgnoreCase("blackship")) {
            ship.setColor("black");
        }

        // notify
        sendEmailTo(email, ship);

        return ship;
    }
```

```java
// ShipFactory -> interface
default Ship orderShip(String name, String email){
        validate(name, email);
        prepareFor(name);
        Ship ship = createShip(); //하위 클래스에서 구현
        sendEmailTo(email, ship);
        return ship;
    }

Ship createShip();

// BlackShipFactory -> 구체 클래스
public class BlackShipFactory implements ShipFactory {
    @Override
    public Ship createShip() {
        return new BlackShip();
    }
}
```


### 정적 팩터리 메서드의 장점

1. 이름을 가질 수 있다.
- 반환될 객체의 특성을 쉽게 묘사.
2. 호출 될때마다 인스턴스를 새로 생성하지는 않아도 된다.
- 불필요한 객체 생성을 피할 수 있다. -> (생성 비용이 큰) 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

```java

static HelloService of(String lang) {
    if (lang.equals("ko")) {
        return new KoreanHelloService();
    } else {
         return new EnglishHelloService();
    }
}

```

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

```java

ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);

Optional<HelloService> helloServiceOptional = loader.findFirst();

helloServiceOptional.ifPresent(h -> {
    System.out.println(h.hello());
});

```
