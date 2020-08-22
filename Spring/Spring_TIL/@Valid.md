## @Valid

- 스프링을 통해서 유효성 검토를 할 수 있다.



#### 의존성 추가

- gradle에 다음과 같이 의존성을 추가한다

  ```groovy
  implementation 'org.springframework.boot:spring-boot-starter-web'
  ```

- 만약 개별적으로 validation 의존성을 추가하고 싶다면 다음 의존성을 추가한다

  ```groovy
  implementation 'org.springframework.boot:spring-boot-starter-validation'
  ```

  

#### 유효성 애노테이션 추가

- DTO에 다음과 같이 validation 체크 애노테이션을 추가

```java
 @NotNull(message = "email은 입력되어야합니다.")
 private String email;
```



#### Controller @Valid 추가

- Cotnroller 커맨드 객체 앞에 @Valid를  붙여준다
- 또한 **BindingResult**를 통해 결과가 에러인 경우 다시 form으로 리턴한다

```java
@PostMapping("/")
public String checkPersonInfo(@Valid PersonForm personForm, BindingResult bindingResult) {

  if (bindingResult.hasErrors()) {
    return "form";
  }

  return "redirect:/results";
}
```



#### Thymeleaf 에러 결과 표시

- 에러발생 시 발생된 에러 결과를 form 페이지에 표시한다

```html
<form id='formData' th:action="@{/member/signUp}" th:object="${memberDTO}" method="post" enctype="multipart/form-data">

<td th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Name Error</td>
```



참고 사이트 : https://spring.io/guides/gs/validating-form-input/

