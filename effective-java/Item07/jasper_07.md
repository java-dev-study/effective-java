## Item 07. 다 쓴 객체 참조를 해제하라.

- 어떤 객체에 대한 레퍼런스가 남아있다면 해당 객체는 가비지 컬렉션의 대상이 되지 않는다.

- 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야 한다.
    - 예) 스택, 캐시, 리스너 또는 콜백

- 참조 객체를 null 처리하는 일은 예외적인 경우이며 가장 좋은 방법은 유효 범위 밖으로 밀어내는 것이다.

### NullPointerException

- NullPointerException을 만나는 이유
    - 메소드에서 Null을 리턴하기 때문에 && null 체크를 하지 않았기 때문에

- 메소드에서 적절한 값을 리턴할 수 없는 경우에 선택할 수 있는 대안
    - 예외를 던진다
    - null을 리턴한다.
    - Optional을 리턴한다

### WeakHashMap

> 더 이상 사용하지 않는 객체를 GC 할때 자동으로 삭제해주는 Map

- WeakHashMap의 key 타입은 wrapper 타입 사용(Long, Integer, String 등 제외)

- 레퍼런스
    - Strong, soft, weak, phantom