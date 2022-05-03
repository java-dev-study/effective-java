## **의존 객체 사용하기**
- 맞춤법 검사기를 이용한 예제

**1. 정적 유틸리티**
```java
import java.util.List;

public class SpellCheckerStaticUtil {
	// static utility로 구현 (Item 04)
	
	// SpellCheckerStaticUtil가 Lexicon 타입의 dictionary를 사용하고 있음
	// 한국 사전으로 고정이 되어 있고, 이를 바꾸려면 코드를 계속해서 바꿔야 함
	
	// isValid와 suggestions에서 사용해야 하기 때문에 static으로 선언
	private static final Lexicon dictionary = new KoreanDictionary();
	
	private SpellCheckerStaticUtil() {	//instance로 만들 필요가 없음
		
	}
	
	public static boolean isValid(String word) {
		throw new UnsupportedOperationException();
	}
	
	public static List<String> suggestions(String typo) {
		throw new UnsupportedOperationException();
	}
	
	public static void main(String[] args) {
		SpellCheckerStaticUtil.isValid("hello");
	}
}

interface Lexicon {}

class KoreanDictionary implements Lexicon {}
```
- 문제점 : 유연하지 않으며 테스트하기 어려움
---
**2. 싱글턴**
```java
import java.util.List;

public class SpellCheckerSingletone {
	// 하나의 사전만 사용할 경우엔 괜찮으나, 다른 나라의 언어도 사용하고자 할 경우 유연하지 않음

	private final Lexicon dictionary = new KoreanDictionary();
	
	private SpellCheckerSingletone() {	//instance로 만들 필요가 없음
		
	}
	
	public static final SpellCheckerSingletone INSTANCE = new SpellCheckerSingletone() {};
	
	public boolean isValid(String word) {
		throw new UnsupportedOperationException();
	}
	
	public List<String> suggestions(String typo) {
		throw new UnsupportedOperationException();
	}
	
	public static void main(String[] args) {
		SpellCheckerSingletone.INSTANCE.isValid("hello");	//싱글통 instance를 통해 method 사용
	}
}
```
- 문제점 : 유연하지 않으며 테스트하기 어려움
---
**3.의존 객체**
```java
import java.util.List;
import java.util.Objects;

public class SpellChecker {
	private final Lexicon dictionary;	//dictionary에 따라 SpellChecker의 언어가 달라질 것 → 의존성을 손쉽게 바꿔 낄 수 있음
	
	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}
	
	public boolean isValid(String word) {
		throw new UnsupportedOperationException();
	}
	
	public List<String> suggestions(String typo) {
		throw new UnsupportedOperationException();
	}
	
	public static void main(String[] args) {
		SpellCheckerSingletone.INSTANCE.isValid("hello");	//싱글통 instance를 통해 method 사용
	}
}
```
---
**4. Supplier**
```java
import java.util.List;
import java.util.Objects;
import java.util.function.Supplier;

public class SpellCheckerSupplier {
	//Lexicon을 제공받는 Supplier에 Lexicon 타입을 받는 방법
	
	private final Lexicon dictionary;
	
	public SpellCheckerSupplier(Supplier<Lexicon> dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary.get());
	}
	
	public boolean isValid(String word) {
		throw new UnsupportedOperationException();
	}
	
	public List<String> suggestions(String typo) {
		throw new UnsupportedOperationException();
	}
	
	public static void main(String[] args) {
		// 이와 같이 의존성으로 사용하면, Test 할 때에는 Test용 Dictionary를 만들어서 Unit Test가 가능함
		Lexicon lexicon = new Lexicon();
		SpellCheckerSupplier spellChecker = new SpellCheckerSupplier(new Supplier<Lexicon>() {
			@Override
			public Lexicon get() {
				// TODO Auto-generated method stub
				return lexicon;
			}
		});
		//SpellCheckerSupplier spellChecker = new SpellCheckerSupplier(() -> lexicon);
		spellChecker.isValid("hello");
	}
}
```
