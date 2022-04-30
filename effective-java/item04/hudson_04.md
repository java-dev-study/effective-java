## 인스턴스화를 막으려거든 private 생성자를 사용하라.

1. 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.
2. 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
3. private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
4. 생성자에 주석으로 인스턴스화 불가한 이유를 설명하는 것이 좋다.
5. 상속을 방지할 때도 같은 방법을 사용할 수 있다.


---

### 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.
Utility성 클래스들은 정적 메서드만을 가지고 있다.
불필요한 인스턴스 생성은 메모리를 낭비할 수 있다.

인스턴스 생성을 방지하는 불완전한 방법 - 추상 클래스
    
``` java

  public abstract class UtilityClass {
    public UtilityClass() {
      System.out.println("Construction");
    }
    
    public static String hello() {
      return "hello";
    }
    
    public static void main(String[] args) {
      String hello = UtilityClass.hello();
    }
  }

```

### 추상 클래스를 만드는 것으로는 인스턴스화를 막을 수 없다. 
    - 서브클래스를 인스턴스화할 수 있다.
    

### private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
    - 서브클래스가 부모의 기본 생성자를 호출할 수 없기 때문에 오류가 발생한다.
    - 클래스 내부에서 인스턴스를 생성할 수 있기 때문에 기본 생성자에 Error를 발생시키도록 하자.
    
        
``` java

  public abstract class UtilityClass {
    private UtilityClass() {
      throw new AssertionError();
    }
    
    public static String hello() {
      return "hello";
    }
    
    public static void main(String[] args) {
      String hello = UtilityClass.hello();
      
      UtilityClass utilityClass = new UtilityClass(); // AssertionError 발생
    }
  }

```

> Exception과 Error의 차이 
> ![image](https://user-images.githubusercontent.com/31056938/166100470-03cf728e-5894-4847-a642-20d24639e448.png)
> 오류와 예외의 상속 관계
> 
> Exception(예외)
> 예외는 실행 도중 중단될 정도로 큰 문제가 아닐 때 발생하는 것으로
> 
> Checked Exception, Unchecked Exception 두 종류의 예외가 존재하는데
> 
> Checked Exception는 실행하기 전에 예측 가능한
> SQLException이나 FileNotFoundException 등을 말한다
> 
> Unchecked Exception는 실행하고 난 후에 알 수 있는
> ArrayIndexOutOfBoundException, NullPointerException 등을 말한다
> 
> Error(에러)
> Error의 경우에는 런타임에서 실행 시 발생되며 전부 예측 불가능한 Unchecked Error에 속한다
> 
> Exception과 다르게 에러가 발생할 경우 코드를 고치지 않고서는 해결이 불가능하며
> 예로는 StackOverflowError나 OutOfMemoryError가 있다

### 생성자에 주석으로 인스턴스화 불가한 이유를 설명하는 것이 좋다.
        
``` java

  public abstract class UtilityClass {
    
    /**
    * 이 클래스는 인스턴스를 만들 수 없습니다.
    */
    private UtilityClass() {
      throw new AssertionError();
    }
    
  }

```
### 상속을 방지할 때도 같은 방법을 사용할 수 있다.
