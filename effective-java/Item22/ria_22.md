### 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 하며, 오직 이 용도로만 사용해야 한다.

#### 상수 인터페이스

메서드없이 static final 필드로만 구성된 인터페이스를 말한다.

```java
public interface CarConstants {
    public static final String INPUT_CAR_NAME_MESSAGE = "경주할 자동차 이름을 입력하세요.(이름은 쉼표(,) 기준으로 구분)";
    public static final String INPUT_ROUND_MESSAGE = "시도할 회수는 몇회인가요?";
}
```

위 코드는 상수 인터페이스 안티패턴으로 인터페이스를 잘못 사용한 예다.

- 클래스 내부에서 사용하는 상수는 내부 구현에 해당되므로, 상수 인터페이스를 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위이다.

#### 상수 공개 방법

1. 클래스나 인터페이스 자체에 추가

   - 특정 클래스나 인터페이스와 강하게 연관된 상수인 경우

     ```java
     public final class Integer extends Number implements Comparable<Integer> {
     	...
     	@Native public static final int   MIN_VALUE = 0x80000000;
     
     	@Native public static final int   MAX_VALUE = 0x7fffffff;
         ...
     }
     ```

2. 열거 타입으로 정의

   ```java
   public enum Food {
     BUGER,
     PIZZA,
     CHICKEN
   }
   ```

3. 유틸리티 클래스에 추가

   ```java
   public class CarConstants {
       public static final String INPUT_CAR_NAME_MESSAGE = "경주할 자동차 이름을 입력하세요.(이름은 쉼표(,) 기준으로 구분)";
       public static final String INPUT_ROUND_MESSAGE = "시도할 회수는 몇회인가요?";
   
       private CarConstants() {
           // 인스턴스화 방지
       }
   }
   ```

   - 만약, 유틸리티 클래스 사용이 빈번할 경우 정적 임포트로 클래스 이름 생략이 가능하다.

     ```java
     import static item22.CarConstants.*;
     
     public class Main {
         String printInputRoundMessage(){
             return INPUT_ROUND_MESSAGE;
         }
         
         String displayInputCarNameMessage(){
             return INPUT_CAR_NAME_MESSAGE;
         }
      ...   
     }
     ```

     