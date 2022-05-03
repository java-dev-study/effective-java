**Item 05 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라**

```
public class SpellChecker{
    private static final Lexicon dictionary = ...;

    private SpellChecker(); //객체 생성 방지

    public static boolean isValid(String word){}
    public static List<String> suggestions(String typo){}
}
```

위 예시에서 SpellChecker가 여러 사전을 사용할 수 있도록 변경 방법은?

1. final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가
   => 문제점 : 어색하고, 오류를 내기 쉬우며, 멀티 스레드 환경에서 쓸 수 없음, 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음
   => 보완 : 인스턴스를 생성할 때, 생성자에 필요한 자원을 넘겨주는 방식으로 바꿔줄 수 있음

```
public class SpellChecker{
    private static final Lexicon dictionary = ...;

    public SpellChecker(Lexion dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word){}
    public static List<String> suggestions(String typo){}
}
```

장점
1. 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동함
2. 불변을 보장하여 여러 클라이언트가 의존 객체들을 공유할 수 있음


이 패턴의 변형 => 생성자에 자원 팩터리를 넘겨주는 방식

팩터리? 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체 즉, 팩터리 메서드 패턴 예시)Supplier<T>

의존 객체 주입 단점?
의존성이 수천개나 되는 큰 프로젝트네서는 코드를 어지럽게 만들기도 한다.  => Dagger,Guice,Spring 같은 의존 객체 주입 프레임워크를 사용하면 이런 어려움을 해소할 수 있음


