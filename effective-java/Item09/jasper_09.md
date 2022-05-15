## Item09

> try-finally 보다 try-with-resouces를 사용하라.

- try-finally는 더이상 최선의 방법이 아니다. (자바7부터)

- try-with-resources를 사용하면 코드가 더 짧고 분명하다.

- 만들어지는 예외 정보도 훨씬 유용하다.

```java

public class BadBufferedReader extends BufferedReader {

    public BadBufferedReader(Reader in, int sz) {
        super(in, sz);
    }

    public BadBufferedReader(Reader in) {
        super(in);
    }

    @Override
    public String readLine() throws IOException {
        throw new CharConversionException();
    }

    @Override
    public void close() throws IOException {
        throw new StreamCorruptedException();
    }
}

/**
 * try-finally 구문일 경우에는 CharConversionException이 스택 트레이스에 보이지 않지만,
 * try-with-resource 구문일 경우에는 최초 발생한 CharConversionException 과 Suppressed인 StreamCorruptedException 둘 다 스택 트레이스에 보여진다.
 */
public class TopLine {
    
    static String firstLineOfFile(String path) throws IOException {
        try(BufferedReader br = new BadBufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }

    public static void main(String[] args) throws IOException {
        System.out.println(firstLineOfFile("pom.xml"));
    }
}



```
