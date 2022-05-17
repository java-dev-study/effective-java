### try-finally보다는 try-with-resources를 사용하라

자원 회수는 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어지며, 안전망으로 finalizer를 제공하지만 finalizer 사용을 지속적으로 지양하고 있다.

```java
static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

위 코드는 try-finally로 자원을 회수하는 코드이다. 자원을 하나만 사용한다면 크게 문제되지 않지만, 만약 자원을 하나 이상 사용하게 된다면 어떻게 될까?

```java
static void copy(String src, String dst) throws IOException {
        FileInputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
                }
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```

첫 번째 코드에 비해 복잡해진 것을 알 수 있다. 만약, try 블록과 finally 블록에서 예외가 발생한다면 두 번째 예외가 첫 번째 예외를 집어삼켜 디버깅에 어려움을 겪을 수 있다.
이에 대한 대안으로 JDK 1.7부터 try-with-resources문이 추가되었으며, 이를 사용하기 위해서는 AutoCloseable을 구현해야 한다.

```java
static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
```

```java
static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             FileOutputStream out = new FileOutputStream(dst)) {
            ;
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
```
위 코드는 try-with-resources문으로 개선한 코드이다. 
try-with-resources문은 짧고 읽기 수월할 뿐 아니라 문제를 파악하기도 훨씬 좋다. 또한 예외 발생 시 스택 추적 내역에 숨겨져 디버깅하기에도 수월하다.
