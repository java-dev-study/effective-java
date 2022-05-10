## item07 다 쓴 객체 참조를 해제하라

JVM    
: 운영체제의 메모리 영역에 접근하여 메모리를 관리하는 프로그램.  
-> 메모리 관리, Garbage Collector 수행

Garbage Collector   
: 동적으로 할당한 메모리 영역 중 사용하지 않는 영역을 탐지하여 해지하는 기능   

Stack: 정적으로 할당한 메모리 영역   
-> 원시 타입의 데이터가 값과 함께 할당. Heap영역에 생성된 Object 타입의 데이터의 참조 값 할당.   

Heap: 동적으로 할당한 메모리 영역.   
모든 Object 타입의 데이터가 할당. Heap영역의 Object를 가리키는 참조 변수가 Stack에 할당.   

<정적>   
```java
int result = 6
int tmp = 12
int param = 4
```
<img width="286" alt="image" src="https://user-images.githubusercontent.com/62540133/167291998-ed570240-b551-4df5-a114-16f2ff314e5d.png">
   
<동적>
```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> listArgument = new ArrayList<>();
        listArgument.add("yaboong");
        listArgument.add("github");

        print(listArgument);
    }
```   
<img width="657" alt="image" src="https://user-images.githubusercontent.com/62540133/167292051-7cf161d4-e91b-4437-947e-d99e7e7960c9.png">.  

(unreachable object).  
```java
String url = "https://"
url = "https://yaboong.github.io"
```
<img width="637" alt="image" src="https://user-images.githubusercontent.com/62540133/167292323-df419a61-e344-44f4-a028-d2ec1e528de6.png">.    

(책의 stack 예제)   
```java
public class Stack {
    private Object[] elements; // (0) 직접 관리하려는 저장소 pool
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {...}

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size]; // (1) 메모리 누수 발생
    }
}
```
![image](https://user-images.githubusercontent.com/62540133/167296293-bcb29a31-db43-4e92-a277-536b9f3580ca.png)
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
-> 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항상 메모리 누수에 주의해야 한다.    

(캐시)   
Key-Value 값을 저장하기 위해 HashMap 이 자주 사용된다. 하지만 HashMap은 GC에 의해 수집되지 않아서 많은 메모리를 소유하게 될 수 있다.   
즉, HashMap에 Key가 null 이어도 이 객체가 GC에 의해서 사라지지 않고 계속 가지고 있다는 뜻이다.   
```java
@DisplayName("HashMap 테스트")
  @Test
  void HashMapTest() throws InterruptedException {
      //given
      Map<Foo, String> map = new HashMap<>();

      Foo key = new Foo();
      map.put(key, "1");

      //when
      key = null;
      System.gc();

      TimeUnit.SECONDS.sleep(5);

      assertFalse("HashMap 은 참조를 끊어도 Map이 비어있지 않다.", map.isEmpty());
  }

  @DisplayName("WeakHashMap 테스트")
  @Test
  void test2() throws InterruptedException {
      //given
      Map<Foo, String> map = new WeakHashMap<>();

      Foo key = new Foo();
      map.put(key, "1");

      //when
      key = null;
      System.gc();

      TimeUnit.SECONDS.sleep(5);

      assertTrue("WeakHashMap은 참조를 끊으면 GC에 의해 MAP이 비어진다", map.isEmpty());
  }
```
Java의 참조 유형에는 크게 4가지가 있습니다.   
참조 유형에 따라 GC 실행 대상여부, 시점이 달라집니다.   

1. Strong References (강한 참조)

2. Soft References (소프트 참조)

3. Weak References (약한 참조)

4. Phantom References (팬텀 참조)   

(강한 참조)        
<img width="419" alt="image" src="https://user-images.githubusercontent.com/62540133/167298543-404fcfec-73e5-47a4-97a4-fa43797c5ce1.png">     
-> g 변수가 참조를 가지고 있는 한, Gfg객체는 GC의 대상이 되지 않습니다      

(소프트 참조).   
<img width="625" alt="image" src="https://user-images.githubusercontent.com/62540133/167298922-1f256d2e-65ba-468d-9cc1-9be62984a890.png">     
-> 대상 객체를 참조하는 경우가 softReference 객체만 존재하는 경우 gc의 대상이 된다.      
단, jvm의 메모리가 부족한 경우에만 힙영역에서 제거되고 메모리가 부족하지 않다면 굳이 제거하지 않습니다.      


(약한 참조)      
<img width="653" alt="image" src="https://user-images.githubusercontent.com/62540133/167298524-9d3fd8fd-b7e3-42f0-9d1e-daffec097df6.png">    
-> 대상 객체를 참조하는 경우가 WeakReferences 객체만 존재하는 경우 GC의 대상이 됩니다.    
다음 GC 실행시 무조건 힙 메모리에서 제거됩니다.      

(팬텀 참조).   
어렵고 잘 쓰이지 않는다고 하여 개인적으로 맡깁니다!     

<백그라운드 쓰레드>      
개념적으로만 쓰레드를 실행할 때 가장 오래된 캐시를 찾아서 3초마다 계속 반복하는 백그라운드 쓰레드를 실행.     
(issue에서 설명)   
   
<LinkedHashMap>.  
: 
예시) map의 크기가 3보다 커지면 가장 오래된 데이터 제거   
```java
final int capacity = 3;
Map<Integer, Integer> map = new LinkedHashMap<Integer, Integer>() {
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > capacity;
    }
};
```
-> 사이즈가 3보다 커지면 가장 오래된 데이터를 지우고 새 값으로 대체. (return true이면 )
* size가 3일때 출력   
```java
map.put(1, 1);
map.put(2, 2);
map.put(3, 3);
System.out.println(Arrays.toString(map.keySet().toArray()));
```
output:   
[1, 2, 3].  
   
* size가 3이 넘어갈 때   
```java
map.put(1, 1);
map.put(2, 2);
map.put(3, 3);
map.put(4, 4);
System.out.println(Arrays.toString(map.keySet().toArray()));
```
output:   
[2,3,4].  
   
-> 따라서 일정한 크기를 정해놓은 캐시로 사용하는데 적합하다.(알아서 오래된 데이터를 지워준다.)
   
   
(리스너 콜백)
   
콜백 : 이벤트가 발생하면 특정 메서드를 호출해 알려줍니다(1개)

리스너 : 이벤트가 발생하면 연결된 리스너(핸들러)에게 이벤트를 전달합니다(n개)

예시 여러 서비스(리스너) 한 리포지토리(콜백)



  
  
