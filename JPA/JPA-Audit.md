# 	JPA Auditing

참고 블로그 : https://umanking.github.io/jpa-audit
https://velog.io/@conatuseus/2019-12-06-2212-%EC%9E%91%EC%84%B1%EB%90%A8-1sk3u75zo9

- Spring Data JPA는 누가 값을 언제 변경했고 언제 추가했는지 확인하는 기본 값 설정 관련 애노테이션을 제공한다(Audit : 감사)
  - @CreateDate
  - @LastModifDate
  - @CreateBy
  - @LastModifiedBy

- 설정은 다음과 같이 진행된다.

  - 기본 어플리케이션 클래스에 @Enable 추가

  ```java
  @SpringBootApplication
  @EnableJpaAuditing
  public class BuskingbuskingApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(BuskingbuskingApplication.class, args);
  	}
  	
  }
  ```

  - 개별 클래스에 Listener로 등록하는 방법도 있지만 공통으로 사용하기 때문에 Super 클래스로 등록한다

  ```java
  @Getter
  @MappedSuperclass
  @EntityListeners(AuditingEntityListener.class)
  public class AuditingEntity {
  
      @CreatedDate
      private LocalDateTime createdDate;
  
      @LastModifiedDate
      private LocalDateTime modifiedDate;
  
      @CreatedBy
      @ManyToOne
      private Member createdBy;
  
      @LastModifiedBy
      @ManyToOne
      private Member modifiedBy;
  }
  ```

  - 개별 엔티티 클래스에 상속해준다

  ```java
  public class Notice extends AuditingEntity {
  
  }
  ```

  

