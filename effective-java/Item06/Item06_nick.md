# 아이템 6 <불필요한 객체 생성을 피하라>   
똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 편이 나을 때가 많다.   
-> 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 말자.    
왜?? 하나를 더 생성하는 것은 불필요한 메모리 낭비 + 성능상의 이슈.     
예시1) String    
```java
// bad example - 호출될 때마다 새로운 인스턴스 생성
String s = new String("bad example"); // Heap 영역에 존재

// good example - 하나의 인스턴스를 사용
String s = "good example"; // String constant pool 영역에 존재
```

<img width="685" alt="image" src="https://user-images.githubusercontent.com/62540133/167284908-e93140b2-a721-4534-b675-0d3f17f944c4.png">   
literal로 string 객체를 생성하면 string pool에 그 값이 있는지 확인해 그 값이 있다면 같은 값을 참조하도록 한다.   
만약 값이 없다면 string pool에 추가한다.   
하지만 new 연산자로 생성한 string 객체는 같은 값이 string pool에 있더라도, heap 영역 내 별도의 객체를 가리키게 된다.   
   
예시2) 정적 팩터리 매서드를 제공하는 불변 클래스 사용 - boolean.  
boolean class.  
<img width="547" alt="image" src="https://user-images.githubusercontent.com/62540133/167285120-1917af47-5f98-45ed-9e9b-9e6e271ebc2d.png">   
만약 true나 false를 return할 경우가 있으면 새롭게 객체를 생성하는 것이 아니라 이미 클래스내에서 만들어진 boolean 객체를 사용   
   
<boolean 좋지 않은 예> - Boolean(String) 생성자
```java
public Boolean(String s) {
        this(parseBoolean(s));
    }
    
public static boolean parseBoolean(String s) {
        return ((s != null) && s.equalsIgnoreCase("true"));
    }
```   
호출할 때 마다 새로운 생성자를 생성   

<boolean 좋은 예> - Boolean.valueOf(String) 팩토리 메서드   
```java
public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }
```
호출시에도 새로운 객체가 아닌 위에서 만들어진 true, false 객체를 사용   
* 예시에서는 불변 객체를 다루고 있지만 만약 가변 객체라 해도 사용 중에 변경 되지 않을 것임을 안다면 재사용 할 수 있다.

예시3) 생성 비용이 아주 비싼 객체 - 정규표현식   
-> '비싼 객체'가 반복해서 필요하다면 캐싱해서 재사용하자! 

[캐싱이란?]   
* 캐시   
: 컴퓨터 과학에서 데이터나 값을 미리 복사해 놓는 임시 장소. 캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다.    

캐싱 -> 캐시라는 작업을 하는 행위(행동)

<정규표현식 좋지 않은 예>   
```java
// 정규 표현식을 활용해 유효한 로마 숫자인지 확인하는 메서드
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
-> 정규 표현식용 pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 가비지 컬렉션의 대상이 된다.    
-> 그런데 pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높다. 
* 유한 상태 머신
: 상태의 수가 유한한 기계
<img width="358" alt="image" src="https://user-images.githubusercontent.com/62540133/167286355-4ecd126a-3357-4272-bd6a-75552fada03e.png">   
상단 경로 혹은 하단 경로를 탈 수 있다. 상단 경로는 a+ // 하단 경로는 'b' 다음에 'c'혹은 'd'   

<정규표현식 캐싱하여 개선>   
```java
public class RomanNumerals {

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
-> 빈번이 호출 되는 상황에서 6.5배 정도 빨라졌다.

예시4) 어댑터(view).  
어댑터 : 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체.   
- map에 적용   
어댑터의 역할 keySet // 뒷단 객체는 Map.   
Map의 view역할을 keySet이 하여 Set으로 보여준다.
```java
   @Test
    void name() {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "첫번째");
        map.put(2, "두번째");
        map.put(3, "세번째");

        // keySet 은 view 의 역할을 한다.
        Set<Integer> keySet = map.keySet();
        assertThat(keySet).contains(1, 2, 3);

        // 따라서 뒷단 객체인 map 이 기능을 담당하고 있다.
        map.remove(3);
        assertThat(keySet).doesNotContain(3);

```
-> keySet을 호출할 때마다 새로운 Set인스턴스가 만들어지다고 생각을 할 수도 있지만 사실은 매번 같은 Set인스턴스를 반환한다.   

<Map interface keyset>
<img width="654" alt="image" src="https://user-images.githubusercontent.com/62540133/167286959-3bec9cbd-408c-4523-ad25-59b38d097102.png">.  
   
<구현체 hashmap>
```java
public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```
-> adapter 어떠한 map이 오더라도 뒷단 객체가 keyset을 잘 구현을 하기만 하면 된다. 

예시5) 오토박싱   
오토박싱 : auto boxing은 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.    
기본타입과 그에 대응하는 박싱된 기본타입의 구분을 흐려주지만 완전히 없애주진 않는다. 성능에서 좋아진다 볼 수 없다.   
```java
private static long sum(){

    Long sum = 0L;

    for (long i = 0; i< Integer.MAX_VALUE; i++) {
        sum += i;
    }

    return sum;
}
```
sum 변수를 long이 아닌 Long으로 선언하여 불필요한 인스턴스가 sum += i 연산이 이루어질 때마다 생성되는 것이다. 단순히 sum 타입을 long으로 변경해주어도 성능이 개선된다.   

[추가]   
* 객체 생성은 비싸니 피해야 한다 X
-> jvm에서는 너무 무거운 객체가 아닌 이상 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.   
ex) 데이터 베이스같은 경우에는 생성 비용이 워낙 비싸기 때문에 재사용을 하지만 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 떨어뜨린다. 
