## **try-with-resources 사용하기**

**1. 자원 닫기**
- close 메서드를 호출해 직접 닫아줘야 하는 자원
- InputStream, OutputStream, java.sql.Connection
- 자원이 제대로 닫힘을 보장하는 try-finally
```java
static String firstLinOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FiledReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}
```
자원을 하나 더 사용한다면\
↓
```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while((n = in.read(buf) >= 0) {
        out.write(buf, 0, n);
      }
    } finally  {
      out.close();
    }
  } finally {
    in.close();
  }
}
```
1) 예외는 try 블록과 finally 블록 모두에서 발생할 수 있음
2) readLine 메서드에서 예외를 던지게 된다면, close 메서드도 실패할 것
3) 두 번째 예외가 첫 번째 예외를 집어삼키고, 스택 추적 내역에 첫 번째 예외에 관한 정보가 남지 않아 디버깅이 어려움
---
**2. try-with-resources**
- 해당 자원이 AutoCloseable 인터페이스를 구현해야 함
- void를 반환하는 close 메서드 하나만 정의된 인터페이스
```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
       OutputStream out = new FileOutputStream(dst)) {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >=0) {
        out.write(buf, 0, n);
      }
  }
}
```
1) 짧고 읽기 수월
2) 보여질 예외 하나만 보존되고 다른 예외가 숨겨질 수 있음
3) 스택 추적 내역에 '숨겨졌다(suppressed)'는 꼬리표를 달고 출력됨\

catch절과 함께 사용\
↓
```java
static String firstLinOfFile(String path, String defaultVal) {
  try (BufferedReader br = new BufferedReader(
          new FileReader(path))) {
    return br.readLine();
  } catch (IOException e) {
    return defaultVal;
  }
}
```
