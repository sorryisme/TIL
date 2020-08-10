# @ControllerAdvice

Controller를 실행함에 따라 도움을 제공하기 위한 애노테이션

스프링 5.x 버전에서는 다음과 같이 정의한다

> Specialization of [`@Component`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html) for classes that declare [`@ExceptionHandler`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html), [`@InitBinder`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/InitBinder.html), or [`@ModelAttribute`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ModelAttribute.html) methods to be shared across multiple `@Controller` classes.

즉 공통적으로 처리하기 위해 간편하게 제공하는 애노테이션이다.


### @ExceptionHandler

```java
@ControllerAdvice("com.sorry.buskingbusking.controller")
public class CommonExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String handlerRuntimeException(){
        return "error/commonException";
    }
}
```

- 위와 같이 Runtime으로 전체로 묶어주는 것보다 좀 더 디테일하게 잡아주는 것이 좋다고 판단된다.(예: 권한 에러 발생 등)
- [참고 사이트](https://www.slipp.net/questions/600)

### @InitBinder

- validator, Formatter, Converter 등 다양한 설정이 가능하다

  ```java
  @InitBinder
  protected void initBinder(WebDataBinder binder) {
  		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
      dateFormat.setLenient(false); 
      binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false)); 
    
      SimpleDateFormat dateFormat2 = new SimpleDateFormat("yyyy-MM-dd") {
        @Override
        public java.util.Date parse(String source) throws ParseException {
          java.util.Date date = super.parse(source);
          return new java.sql.Date(date.getTime());
        }
      };
  		
    	dateFormat2.setLenient(false);
  		binder.registerCustomEditor(java.sql.Date.class,new CustomDateEditor(dateFormat2, false)); 
  		CustomDateEditor editor = new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd hh:mm"), true);
      binder.registerCustomEditor(Date.class, editor);
  }
  ```

  - 학원에서 배웠던 방식으로 "yyyy-MM-dd"를 Date 객체로 변환하는 역할을 한다.

### @ModelAttribute

```java
@ControllerAdvice
public class GlobalControllerAdvice {

    @ModelAttribute
    public User user(@AuthenticationPrincipal User user) {
        return user;
    }
}
```

- 글로벌하게 ModelAttribute를 사용할 때 편리하다
- 매번 Controller에 모두 작성하지 않아도 다음과 같이 사용자정보를 전역적으로 확인할 수 있다.



[참고 블로그](http://wonwoo.ml/index.php/post/2208)