### Item 22

> 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴트를 참조할 수 있는 타입 역할을 한다.

달리 말해, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.

#### 상수 인터페이스 

- 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스.

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다.

- java.io.ObjectStreamConstants

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/838a7408-6057-4f4b-9263-09a124910f4a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-07-05_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.08.37.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220704%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220704T160936Z&X-Amz-Expires=86400&X-Amz-Signature=c58d5a86a4990eb7b1583768ae9758ddf6a0a3bf397c9ded8e0c1915500293af&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-07-05%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%25201.08.37.png%22&x-id=GetObject)


![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a2e96b8a-e362-4303-a5c6-81b40b3c7520/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-07-05_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.09.05.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220704%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220704T161000Z&X-Amz-Expires=86400&X-Amz-Signature=7ba5f43d7afa5695b273f5c547afe3369c505876a0daffd792d6cefc56d4127f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-07-05%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%25201.09.05.png%22&x-id=GetObject)

### 더 좋은 방안

- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가.

    - Integer, Double의 MIN_VALUE, MAX_VALUE

- 열거 타입

- 유틸리티 클래스
    - 정적 임포트 

> 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.