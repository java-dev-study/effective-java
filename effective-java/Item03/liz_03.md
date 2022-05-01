**Item 03 private 생성자나 열거 타입으로 싱글턴임을 보증하라**

싱글턴 : 인스턴스를 오직 **하나**만 생성할 수 있는 클래스

예1)  무상태(stateless) 객체

object가 instance fields가 없거나, instance fields가 있더라도 그 값이 절대 변하지 않는다면 그것을 무상태 객체라고 한다.

```
class Stateless {
    void test() {
        System.out.println("Test!");
    }
}
```

```
class Stateless {
    //No static modifier because we're talking about the object itself
    final String TEST = "Test!";

    void test() {
        System.out.println(TEST);
    }
}
```
아래  object는 state가 있다. 그러므로 무상태 객체는 아니지만 state가 딱 한번 set되고 절대 변하지않는다면 이것을 immutab이라고 부른다.

```
class Immutable {
    final String testString;

    Immutable(String testString) {
        this.testString = testString;
    }

    void test() {
        System.out.println(testString);
    }
}
```

예2) 설계상 유일해야 하는 시스템 컴포넌트

싱글톤의 문제점 :  이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. => 싱글턴 인스턴스를 mock구현으로 대체할 수 없기때문에

싱글톤을 만드는 방식 

두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다.

1. piublic static final 필드 방식의 싱글턴
```
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis(){...}
}
```
- public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됨 
=> 하지만 AccessibleObject.setAccessible(true)로 private 생성자를 호출 할 수 있다.

이러한 공격을 방어하려면 두번째 객체가 생성되려할 때 예외를 던지면 된다. -> 어떻게?

장점
- 해당 클래스가 싱글턴임이 API에 드러남
- 간결함

2. 정적 팩터리 방식의 싱글톤
```
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis(){...}
    public status Elvis getInstance() { return INSTANCE;  }
}
```
장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음
- 제너릭 싱글턴 팩터리로 만들 수 있음
- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있음(Elvis::getInstance -> Supplier<Elvis> )

제너릭 싱글톤 팩터리  예제

```
public class GenericFactoryMethod {
public static final Set EMPTY_SET = new HashSet();

    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
}

```
```
@Test public void genericTest() { 
    Set<String> set = GenericFactoryMethod.emptySet(); 
    Set<Integer> set2 = GenericFactoryMethod.emptySet(); 
    Set<Elvis> set3 = GenericFactoryMethod.emptySet(); 
    set.add("ab"); 
    set2.add(123); 
    set3.add(Elvis.INSTANCE); 
    String s = set.toString(); 
    System.out.println("s = " + s); 
}

```
`
s = [ab, item3.Elvis@3439f68d, 123]
`

여러 타입으로 내부 객체를 받아도 에러가 나지 않음 -> 큰 유연성을 제공

싱글톤 클래스의 직렬화

싱글톤 클래스는 직렬화 가능한 클래스가 되기 위해 Serializable 인터페이스를 구현(implements) 하는 순간, 싱글톤 클래스가 아닌 상태가 된다. 직렬화를 통해 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환되기 때문이다.

```
import java.io.Serializable;

public final class MySingleton implements Serializable { // Serializable
	private static final MySingleton INSTANCE = new MySingleton();

	private MySingleton() {
	}

	public static MySingleton getINSTANCE() {
		return INSTANCE;
	}
}
```

이를 직렬화/ 역직렬화 해보면

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class SerializationTester {

	public byte[] serialize(Object instance) {
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		try (bos; ObjectOutputStream oos = new ObjectOutputStream(bos)) {
			oos.writeObject(instance);
		} catch (Exception e) {
			// ... 구현 생략
		}
		return bos.toByteArray();
	}

	public Object deserialize(byte[] serializedData) {
		ByteArrayInputStream bis = new ByteArrayInputStream(serializedData);
		try (bis; ObjectInputStream ois = new ObjectInputStream(bis)) {
			return ois.readObject();
		} catch (Exception e) {
			// ... 구현 생략
		}
		return null;
	}

	public static void main(String[] args) {
		MySingleton instance = MySingleton.getINSTANCE();
		SerializationTester serializationTester = new SerializationTester();
		byte[] serializedData = serializationTester.serialize(instance);
		MySingleton result = (MySingleton)serializationTester.deserialize(serializedData);
		System.out.println("instance == result : " + (instance == result));
		System.out.println("instance.equals(result) : " + (instance.equals(result)));
	}
}
```

결과는 
```

instance == result : false
instance.equals(result) : false
```

이럴 때 readResolve 메서드를 사용하면 된다. 이 메서드를 직접 정의하여 역직렬화 과정에서 만들어진 인스턴스 대신에 기존에 생성된 싱글톤 인스턴스를 반환하도록 하면 된다.

만일 역직렬화 과정에서 자동으로 호출되는 readObject 메서드가 있더라도 readResolve 메서드에서 반환한 인스턴스로 대체된다. readObject 메서드를 통해 만들어진 인스턴스는 가비지 컬렉션 대상이 된다.

```
import java.io.Serializable;

public final class MySingleton implements Serializable {
	private static final MySingleton INSTANCE = new MySingleton();

	private MySingleton() {
	}

	public static MySingleton getINSTANCE() {
		return INSTANCE;
	}

    // readResolve 메서드를 정의한다.
	private Object readResolve() {
        // 싱글턴을 보장하기 위함!
		return INSTANCE;
	}
}
```


3. 열거 타입 방식의 싱글턴

```
pulic enum Elvis {
    INSTANCE;
}
```

대부분 상황에서는 우너소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법

장점
- public 필드 방식과 비슷하지만 더 간결
- 추가 노력 없이 직렬화 가능
- 제 2의 인스턴스가 생기는 일을 막아줌

단점
- 만들려는 싱글턴이 enum 외의 클래스를 상속해야 한다면 이방법은 사용할 수 없음