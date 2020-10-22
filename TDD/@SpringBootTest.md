# @SpringBootTest

[참고주소:TOAST 기술블로그](https://meetup.toast.com/posts/124)

[참고주소:갓대희의 작은공간](https://goddaehee.tistory.com/211)



### 개요 

- 실제 운영환경에서 사용되는 클래스를 통해 통합테스트 진행

- Spring-test에서 제공하는 @ContextConfiguration의 발전된 기능
- 반드시 @Runwith(SpringRunner.class)를 붙여줘야한다
  - SpringRunner 클래스는 SpringJunit4ClassRunner를 상속받은 클래스이다

### 빈 관련 설정

- classes 속성을 지정해주면 @Configuration 설정이 된 클래스가 빈으로 생성되며 지정하지 않으면 애플리케이션 상 모든 빈이 생성된다



### Properties 

- 프로퍼티 설정을 통해 설정관련 내용을 지정할 수 있다.



### 트랜잭션

- Transaction 어노테이션을 함께 사용하면 테스트 끝날 대 rollbback 처리
- RANDOM_PORT, DEFINED_PORT로 테스트할 경우 실제 테스트 서버는 별도의 스레드에서 수행되기 때문에 롤백되지 않음

