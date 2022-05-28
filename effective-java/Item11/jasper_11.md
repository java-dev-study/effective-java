## Item 11

> equals를 재정의하려거든 hashCode도 재정의하라

### hashCode 규약

- equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야 한다.(변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다.)

- 두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다.

- 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 해시 테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.

---

### hascode가 재정의 되어 있지 않을 경우

- 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킴.

```java

Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)); // null 반환

```

- hashcode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 null을 리턴한다.

- 두 인스턴스를 같은 해시 버킷에 담았더라도 get 메소드는 여전히 null를 반환하는데, HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어 있기 때문이다.

---

### hashcode 재정의

- 롬복의 @EqualsAndHashCode 사용하자..

---

### hashcode 재정의 시, 주의점

- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.

- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

---

### 쓰레드 안전

- 가장 안전한 방법은 여러 쓰레드 간에 공유하는 데이터가 없는 것

- 공유하는 데이터가 있다면

    - Synchronization

    - ThreadLocal

    - 불변 객체 사용

    - Synchronized 데이터 사용

    - Concurrent 데이터 사용 (여러 쓰레드에서 동시 접근이 가능한 컬렉션)