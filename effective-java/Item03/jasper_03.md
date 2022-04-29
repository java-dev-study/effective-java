## Item03

> private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

### 싱글턴이 사용되는 예시.

#### 1.  함수와 같은 무상태(stateless) 객체
-  Stateless Object란? 인스턴스 변수가 없는 객체

```java

public class Car {
    void Car() {
        System.out.println("I`m car!");
    }
}

---
// 아래와 같은 컴파일 타임에 정의되어있고 변경되지 않는 상수를 가지는 경우도 Statless Object
public class Car {
    static final String CAR_MESSAGE = "I`m car!";

    void car() {
        System.out.println(CAR_MESSAGE);
    }
}

---

/*
    Stateless Object != Immutable Object 서로 다른 개념
    객체지향 프로그래밍에서 Immutable Object는 상태를 바꿀 수 없는 객체.
    아래 코드와 같이 불변 객체는 상태가 한번 지정되면 바뀔 수 없지만 컴파일 시점(static)에 값이 
    정의되는 것이 아니어서 Stateless Object라고 할수는 없다.
*/

public class Car {
    private String message;

    public Car(String message) {
        this.message = message;
    }
    
    public void printMessage() {
        System.out.println(this.message);
    }
}

```

#### 2. 설계상 유일해야 하는 시스템 컴포넌트

- Ex) 게임 서비스에서 화면 밝기나, 언어 등 설정에 관련된 객체들.
- 이런 객체들이 여러개 생성될 경우(충돌) 애플리케이션에 문제가 될 수 있음.

---

### 싱글턴을 만드는 첫번째 방법 

> private 생성자 + public static final 필드

```java

public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I`m outta here!");
    }
}

public class Client {

    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }

}

```

#### 장점과 단점

- 장점
    - 간결하고 싱글턴임을 API에 들어낼 수 있다.(자바 독)
- 단점
    - 싱글톤을 사용하는 클라이언트 테스트하기 어려워진다.
    - 리플렉션으로 private 생성자를 호출할 수 있다.
    - 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.


#### 리플렉션 예제 코드

```java

public class ElvisReflection {

    public static void main(String[] args) {
        try {
            Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
            defaultConstructor.setAccessible(true); //private 생성자에 접근할 수 있게 된다.
            Elvis elvis1 = defaultConstructor.newInstance();
            Elvis elvis2 = defaultConstructor.newInstance();
            System.out.println(elvis1==elvis2); //싱글턴이 깨지게 된다.
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}

```

#### 역직렬화 에제 코드

```java

// 역직렬화로 여러 인스턴스를 만들기
public class ElvisSerialization {

    public static void main(String[] args) {
        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
            out.writeObject(Elvis.INSTANCE);
        } catch (IOException e) {
            e.printStackTrace();
        }

        try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
            Elvis elvis3 = (Elvis) in.readObject();
            System.out.println(elvis3 == Elvis.INSTANCE);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

#### 리플렉션, 역직렬화에도 싱글턴을 보장할 수 있는 방법

```java

public class Elvis implements IElvis, Serializable {

    public static final Elvis INSTANCE = new Elvis();
    private static boolean created;

    private Elvis() {
        //리플렉션 방어 코드
        if (created) {
            throw new UnsupportedOperationException("can`t be created by constructor.");
        }
        created = true;
    }

    @Override
    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I`m outta here!");
    }

    @Override
    public void sing() {
        System.out.println("I`ll have a blue~ Christmas without you~");
    }

    // 역직렬화 시 새로운 인스턴스가 아닌 재정의한 readResolve 메소드가 실행됨.
    private Object readResolve() {
        return INSTANCE;
    }

}

```

---

### 싱글턴을 만드는 두번째 방법

private 생성자 + 정적 팩터리 메서드

```java

public class Elvis {

    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public static void main(String[] args) {
        Elvis evlis = Elvis.getInstance();
        evlis.leaveTheBuilding();
    }
}

```

#### 장점과 단점

- 장점
    -  API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
    - 정적 팩터리 메서드를 제네릭 싱글턴 팩터리로 만들 수 있다.
    - 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.
- 단점
    - 첫번째 방법과 동일

#### 제네릭 싱글턴 팩터리 예제 코드

```java

public class MetaElvis<T> {

    private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();

    private MetaElvis() { }

    // 동일한 인스턴스를 각각 다른 타입으로 사용할 수 있다.
    @SuppressWarnings("unchecked")
    public static <E> MetaElvis<E> getInstance() { return (MetaElvis<E>) INSTANCE; }

    public void say(T t) {
        System.out.println(t);
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public static void main(String[] args) {
        MetaElvis<String> elvis1 = MetaElvis.getInstance();
        MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
        System.out.println(elvis1);
        System.out.println(elvis2);
        elvis1.say("hello");
        elvis2.say(100);
        // 타입은 다르기 때문에 == 비교는 할 수 없다.
    }
}

```

#### 공급자(Supplier) 예제 코드

```java

public class Concert {

    public void start(Supplier<Singer> singerSupplier) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }

    public static void main(String[] args) {
        Concert concert = new Concert();
        concert.start(Elvis::getInstance);
    }
}

```

---

### 싱글턴을 만드는 세번째 방법

열거타입

- 가장 간결한 방법이며 직렬화와 리플렉션에도 안전하다.
- 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
- 단, enum은 상속을 할 수 없다.(다른 인터페이스를 구현하도록 선언할 수 있다)

```java

// enum 상수는 new 생성자를 통해 생성할 수 없다.
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("hello enum");
    }

    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}

```