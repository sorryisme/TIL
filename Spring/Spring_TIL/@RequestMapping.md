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



### produces

- request에서 JSON을 요구하는 경우 produces 속성을 사용한다
- request에서는 accept로 원하는 MediaType을 지정해서 요청한다
  - Accept 헤더는 요청을 보낼 때 서버에 이런 타입의 데이터를 보내줬으면 좋겠다고 명시할 때 사용 [참고링크](https://www.zerocho.com/category/HTTP/post/5b3ba2d0b3dabd001b53b9db)
- 만약 없을 경우 415 error를 표시한다



#### 정리중



