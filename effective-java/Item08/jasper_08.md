## Item 08

> finalizer와 cleaner 사용을 피하라

- finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

- finalizer와 cleaner는 실행되지 않을 수 있다.

- finalizer 동작 중에 예외가 발생하면 정리 작업이 처리되지 않을 수도 있다.

- finalizer와 cleaner는 심각한 성능 문제가 있다.

- finalizer는 보안 문제가 있다.

- 반납할 자원이 있는 클래스는 AutoCloseable을 구현하고 클라이언트에서 close()를 호출하거나 try-with-resource를 사용해야 한다.

    - client가 try-with-resource를 사용하지 않은 경우 cleaner를 통해 안전망을 둘 수 있다.


---

### Finalizer 공격

```java

public class Account {

    private String accountId;

    public Account(String accountId) {
        this.accountId = accountId;

        if (accountId.equals("�black")) {
            throw new IllegalArgumentException("black list 계정입니다");
        }

    }

   // 계좌 이체 메소드
    public void transfer(BigDecimal amount, String to) {
        System.out.printf("transfer %f from %s to %s\n", amount, accountId, to);
    }

}

class AccountTest {

    @Test
    void 정상_로직() {
       // 생성자 호출 시 IllegalArgumentException이 발생.
        Account account = new Account("�black");
        account.transfer(BigDecimal.valueOf(1000000), "hello");
    }

}

```   
하지만 finalizer 공격을 통해서 정상적으로 계좌 이체 메소드를 호출할 수 있다.

```java

//Account 상속
public class BrokenAccount extends Account {

    public BrokenAccount(String accountId) {
        super(accountId);
    }

   //finalize 오버라이딩
    @Override
    protected void finalize() throws Throwable {
        this.transfer(BigDecimal.valueOf(1000000), "hello");
    }

}

class AccountTest {

    @Test
    void finalizer_공격()  throws InterruptedException  {
      
        Account account = null;
        try {
              account = new BrokenAccount("black");
        } catch (Exception e) {
              // 예외를 잡고 GC를 수행시킨다면?
        }

       System.gc();
       Thread.sleep(3000L);
   
     }

}

```


---

### AutoClosable

```java

public class AutoClosableIsGood implements AutoCloseable{

    private BufferedReader reader;

    public AutoClosableIsGood(String path) {
        try {
            this.reader = new BufferedReader(new FileReader(path));
        } catch (FileNotFoundException e) {
            throw new IllegalArgumentException(path);
        }
    }

    /**
     * close 메소드는 멱등성을 유지 시켜야 함.
     */
    @Override
    public void close() {
        try {
            reader.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

```