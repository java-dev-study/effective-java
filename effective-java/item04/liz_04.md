**Item 04 인스턴스화를 막으려거든 private 생성자를 사용하라**

인스턴스화를 막는 방법 : private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```
public class UtilityClass{
    //기본 셍상자가 만들어지는 것을 막는다.(인스턴스화 방지용)
    private UtilityClass(){
        throw new AsserionError();
    }
}
```

- 명시적 생성자가 private이라 클래스 바깥에서는 접근할 수 없음
- 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아줌
- 상속을 불가능하게 하는 효과