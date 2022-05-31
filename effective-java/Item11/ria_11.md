### equals를 재정의하려거든 hashCode도 재정의하라

#### hashCode 규약

- equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야 한다.

- 두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다.
- 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 해시 테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.

##### HashMap 동작 방법

HashMap은 Map을 구현한 객체 중 하나로  Key-value 형태로 데이터를 저장한다.

![image-20220531195448941](/Users/eunkyung/Library/Application Support/typora-user-images/image-20220531195448941.png)

위 그림은 해시 함수를 사용하여 키를 해시값으로 매핑하고, 해시값을 인덱싱하여 데이터를 키와 함께 버킷에 저장한다. key-value가 1:1로 매핑되어 있어 삽입, 삭제, 검색 과정에서 평균적으로 O(1)의 시간복잡도를 가진다.

- HashMap에 데이터를 넣는 방법

  - ```java
    HashMap<Integer, String> map = new HashMap<>();
    map.put(1,"하나");
    map.put(2,"둘");
    map.put(3,"셋");
    map.put(4,"넷");
    ```

- HashMap에서 값을 가져오는 방법

  - ```java
    String val = map.get(2).toString();
    ```

#### hashCode 구현 방법

```java
@Override 
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

1. 하나의 핵심 필드 해쉬값을 계산해서 result 값을 초기화 한다.

2. 기본 타입은 Type.hashCode / 참조 타입은 해당 필드의 hashCode / 배열은 모든 원소를 재귀적으로 위의 로직을 적용하나, Arrays.hashCode

   result = 31 * result + 해당 필드의 hashCode 계산값

3. result를 리턴한다.

#### hashCode 구현 대안

- 구아바 or Lombok의 equalsAndHashCode
- Objects 클래스의 hash 메서드
- 캐싱을 사용해 불변 클래스의 해시 코드 계산 비용을 줄일 수 있다.

#### 주의할 것

- 지연 초기화 기법을 사용할 때 스레드 안전성을 신경써야 한다.
- 성능 때문에 핵심 필드를 해시코드 계산할 때 빼면 안된다.
- 해시코드 계산 규칙을 API에 노출하지 않는 것이 좋다.