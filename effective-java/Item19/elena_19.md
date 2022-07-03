## **상속을 고려해 설계하고 문서화하고, 그러지 않았다면 상속 금지**

**1. 상속을 고려한 설계와 문서화**
- 재정의 가능 메서드를 호출할 수 잇는 모든 상황을 문서로 남겨야 함
> 1. 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 함
> 2. 호출되는 메서드가 재정의 가능 메서드라면 그 사실을 호출하는 메서드의 API 설명에 적시
> 3. 어떤 순서로 호출하는지
> 4. 호출 결과가 이어지는 처리에 어떤 영향을 주는지

- API 문서의 메서드 설명 끝에서 Implementation Requirements로 시작하는 절 = 내부 동작 방식을 설명하는 곳
- 메서드 주석에 `@impleSpec` 태그를 붙여주면 자바독 도구가 생성해 줌\
`ex) java.util.AbstractCollection`
---
**2. 효율적인 하위 클래스 생성**
- `@implSpec`
> 1. 선택사항으로 남겨져 있음
> 2. 활성화하려면 명령줄 매개변수 `-tag "implSpec:Implementation Requirements:"` 지정

- 내부 메커니즘을 문서로 남기는 것이 끝이 아님
- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 **훅(hook)** 을 잘 선별하여 **protected** 메서드 형태로 공개해야 효율적인 하위 클래스를 어려움 없이 만들 수 있음\
`ex) java.util.AbstractList의 removeRange 메서드`
---
**3. protected 노출 메서드 결정**
- 예측 후 실제 하위 클래스를 만들어 시험
- protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 한 적어야 함
- 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의
- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어 보는 것이 **유일**
---
**4. 상속을 허용하는 클래스 제약 조건**
1. 상속용으로 설게한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증
2. 상속하려는 사람을 위해 덧붙은 설명은 군더더기
3. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 됨

`3 규칙 위반 코드`
```java
/*
* 상위 클래스
*/
public class Super {
  // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출
  public Super() {
    overrideMe();
  }
  
  public void overrideMe() {
  }
}
```

```java
/*
* 하위 클래스
*/
public final class Sub extends Super {
  // 초기화되지 않은 final 필드, 생성에서 초기화된다.
  private final Instant instant;
  
  Sub() {
    instant = Instant.now();
  }
  
  // 재정의 가능 메서드, 상위 클래스의 생성자가 호출한다.
  @Override public void overrideMe() {
    System.out.println(instant);
  }
  
  public static void main(String[] args) {
    Sub sub = new Sub();
    sub.overrideMe();
  }
}
```

└ 상위 클래스의 생성자는 하위 클래스의 생성자 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출

---
**5. 그 외 일반적 구체 클래스**
- 전통적으로 이런 클래스는 final도 아니고 상속용으로 설계되거나 문서화되지도 않았으나, 클래스에 변화가 생길 때마다 하위 클래스를 오동작하게 만들 수 있음
- **해결 방법** : 상속용으로 설계하지 않은 클래스는 상속을 금지

`상속 금지 방법`
1. 클래스를 final로 선언
2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리 만들기

- 그러나 상속을 금지하면 사용하기에 상당히 불편
- 상속을 꼭 허용해야할 경우 : 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기기
---
**cf.**
- 클래스 동작을 유지하면서 재정의 가능 메서드를 사용하느 코드를 제거할 수 있는 기계적인 방법
> 1. 각각의 재정의 가능 메서드는 자신의 본문 코드를 private '도우미 메서드'로 옮기기
> 2. 이 도우미 메서드를 호출하도록 수정
> 3. 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 