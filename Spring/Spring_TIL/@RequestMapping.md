# @RequestMapping

- dispatcher servlet으로 들어온 모든 요청을 처리하기 위해 제공하는 handler Mapping 4가지
  - BeanNameUrIHandlerMapping
  - ControllerClassNameHandlerMapping
  - SimpleUrIHandlerMapping
  - DefaultAnnotationHandlerMapping
- 위 4가지 중 RequestMapping은 DefaultAnnotationMapping에 속함
  - 컨트롤러 선택시 사용
  - 클래스 레벨 기준에서 메서드 레벨로 세분화



### 속성

- consumes
- headers
- method

- name
- params
- path
- produces
- value



### [consumes](https://yang1650.tistory.com/133)

- 요청 컨텐츠 제한
- 요청헤더가 Content-Type를 제한하여 지정할 수 있다.
- 만약 consume에 없는 요청이 들어올 경우 415 error를 표시한다
  - 415 : Unsupported Media Type



### headers 

- 헤더 조건을 지정하는 옵션
- 원하는 옵션 값을 지정하거나 배제할 수 있다.
  - My-Header=myValue또는 My-Header!=myValue

```java
 @RequestMapping(value = "/head", headers = {"content-type=text/plain", "content-type=text/html"})
```



### method

- 원하는 메서드를 지정할 수 있다
- RequestMethod.GET, Request.PUT 등등
- 요즘에는 @RequestMapping에 옵션 값을 지정하지 않고 @GetMapping 등 메서드에 맞춰 애노테이션을 지정하는 것이 좋다

### produces

- request에서 JSON을 요구하는 경우 produces 속성을 사용한다
- request에서는 accept로 원하는 MediaType을 지정해서 요청한다
  - Accept 헤더는 요청을 보낼 때 서버에 이런 타입의 데이터를 보내줬으면 좋겠다고 명시할 때 사용 [참고링크](https://www.zerocho.com/category/HTTP/post/5b3ba2d0b3dabd001b53b9db)
- 만약 없을 경우 415 error를 표시한다



### params 

- 원하는 파라미터를 지정하여 처리할 수 있다.
- myParam=myValue 또는 myParam!=myValue와 같이 지정해주면 된다.



### path, value

- path와 value는 동일하다고 보면된다.(@aliasFor)
- 메소드 호출하기 위한 표현식 또는 경로를 지정할 경우 실행하기 위해 사용
- 가장 일반적으로 많이 사용하는 옵션



### name

- path, value와는 다르게 # 구분자를 통해 controller명과 name에 지정된 값으로 메소드 호출이 가능

- [예시 코드](https://stackoverflow.com/questions/61152283/what-are-the-use-cases-of-the-name-property-of-the-requestmapping-annotation)

  ```java
  @Controller
  @ResponseBody
  @RequestMapping(name = "AdminController")
  class BlogController {
  
      @RequestMapping(name="BlogComments", path="blog/{blog}/comments/{page}")
      public List<Comment> listBlogComments(@PathVariable Blog blog, @PathVariable Long page) {
          ...
      }
  }
  ```

  ```html
  <a href="${s:mvcUrl('AdminController#BlogComments').arg("1","123").build()}">Get Person</a>
  
  ```

  

