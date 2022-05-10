## **불필요한 객체 생성 피하기**

1. 
```java
String s = new String("bikini");
```
> 실행될 때마다 String 인스턴스를 새로 만듦

```java
String s = "bikini";
```
> 하나의 String 인스턴스를 사용\
> 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장됨
---
2.
- 정적 팩터리 메소드를 사용해 불필요한 객체 생성 피하기 가능
- Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메서드 사용
```java
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|d?C{0, 3})"
          + "(X[CL]|L?X{0, 3})(I[XV]|V?I{0, 3})$");
}
```
> matches : 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않음\
> 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 되기 때문

↓
```java
public class RomanNumerals {
  private static fina Pattern ROMAN = Pattern.compile(
          "^(?=.)M*(C[MD]|d?C{0, 3})"
          + "(X[CL]|L?X{0, 3})(I[XV]|V?I{0, 3})$");
          
  static boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
  }
}
```
> 정적 초기화 과정에서 직접 생성해 캐싱\
> 추후 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용
---
3.
**오토박싱(auto boxing)**
- 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 상호 변환해 주는 기술
- 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아님
```java
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    sum += i;
  }
  
  return sum;
}
```
> 박싱된 기본 타입보다는 기본 타입을 사용\
> 의도치 않은 오토박싱이 숨어들지 않도록 주의
---
4.
- 단순히 객체 생성을 피하고자 자체 객체 풀(pool)을 만드는 것은 피하기
- 방어적 복사와 대조적
