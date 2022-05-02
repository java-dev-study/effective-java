## **private 생성자를 사용하여 인스턴스화 막기**
> - 정적 멤버만 담은 Utility Class는 인스턴스로 만들어 쓰려고 설계한 것이 아님
> - 그러나, 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만듦
> - 사용자는 이 생성자가 자동 생성된 것인지 구분 불가
> - 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없음
> - private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있음
> - 상속을 불가능하게 하는 효과도 있음

---
## **추상 클래스로 인스턴스화 막기**
```java
public class UtilClass {
	//보통 Util Class를 만들 경우 static method를 만듦
	public static String getName() {
		return "Elena";
	}
	
	//Class에 abstract를 붙이면 1차적으로는 instance 생성을 막을 수 있긴 함
	//그러나 어떤 클래스가 UtilClass를 상속받을 경우엔 instance 생성이 가능해짐
	static class AnotherClass extends UtilClass {
		
	}
	
	public static void main(String[] args) {
		AnotherClass anotherClass = new AnotherClass();
	}
}
```
---
## **private 생성자로 인스턴스화 막기**
```java
public class UtillClassPrivate {
	// UtilClass.java에서와 같이 의미없는 instance를 만들 가능성을 배제하기 위해 Item 04에서 private 생성자를 추가하라고 하는 것
	// 상속도 막을 수 있음
	// → 상속 시에는 암묵적으로 부모의 생성자를 상속받게 되는데 private으로 만들어져 있기 때문에 호출이 막혔기 때문에 상속이 막힘
	
	private UtillClassPrivate() {
		throw new AssertionError();
	}
	
	public static String getName() {
		return "Elena";
	}
	
	public static void main(String[] args) {
		
	}
}
```
