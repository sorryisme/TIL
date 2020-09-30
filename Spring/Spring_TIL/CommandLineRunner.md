# CommandLineRunner

- https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html

- 스프링 2.x 대에 제공되는 functional Interface

- SpringBootApplication에 포함되면 실행과 동시에 원하는 값을 미리 넣어둘 수 있다.

- 또는 다음과 같이 인터페이스를 구현하여 처리할 수도 있다.

  ```java
  import org.springframework.boot.*;
  import org.springframework.stereotype.*;
  
  @Component
  public class MyBean implements CommandLineRunner {
  
      public void run(String... args) {
          // Do something...
      }
  
  }
  ```

  - 참고 URL https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/



- SpringApplication 내 Bean을 생성하는 방식으로 구현하였다.

  ```java
  	@Bean
  	public CommandLineRunner createCode(CommonCodeRepository commonCodeRepository) throws Exception{
  		return (args)->{
  
  				String[] locArray = {"홍대","건대","대학로","신촌","한강공원"};
  				List<String> locList = new ArrayList<>(Arrays.asList(locArray));
  
  				locList.stream().forEach(arr->{
  					CommonCode code = CommonCode.builder().codeName("LocCode")
  															.codeDesc(arr)
  															.regDt(LocalDateTime.now())
  															.build();
  					commonCodeRepository.save(code);
  
  				});
  
  				String[] genreArray = {"노래","랩","춤","마술","행위예술","피아노"};
  				List<String> genreList = new ArrayList<>(Arrays.asList(genreArray));
  
  				genreList.stream().forEach(arr->{
  					CommonCode code = CommonCode.builder().codeName("GenreCode")
  														.codeDesc(arr)
  														.regDt(LocalDateTime.now())
  														.build();
  					commonCodeRepository.save(code);
  				});
  
  		};
  	}
  ```

- 정리 잘된 블로그 : https://jeong-pro.tistory.com/20