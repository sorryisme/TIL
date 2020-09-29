# HandlerMethodArgumentResolver

[참고주소](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html)

[참고주소2](https://reflectoring.io/spring-boot-argumentresolver/)

- 인자값에 대한 값을 매핑해줄 때 사용하는 인터페이스

- PathVariable은 PathVariableMethodArgumentResolver를 통해 구현된다

  ```java
  public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
  		implements BeanFactoryAware, InitializingBean {
      ... 중략
          
          
  	private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
  		
          List<HandlerMethodArgumentResolver> resolvers = new ArrayList<>(30);
  
  		// Annotation-based argument resolution
  		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
  		resolvers.add(new RequestParamMapMethodArgumentResolver());
  		resolvers.add(new PathVariableMethodArgumentResolver());
  		resolvers.add(new PathVariableMapMethodArgumentResolver());
  		resolvers.add(new MatrixVariableMethodArgumentResolver());
  		resolvers.add(new MatrixVariableMapMethodArgumentResolver());
  		resolvers.add(new ServletModelAttributeMethodProcessor(false));
  		resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  		resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
  		resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
  		resolvers.add(new RequestHeaderMapMethodArgumentResolver());
  		resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
  		resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
  		resolvers.add(new SessionAttributeMethodArgumentResolver());
  		resolvers.add(new RequestAttributeMethodArgumentResolver());
  
  		// Type-based argument resolution
  		resolvers.add(new ServletRequestMethodArgumentResolver());
  		resolvers.add(new ServletResponseMethodArgumentResolver());
  		resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  		resolvers.add(new RedirectAttributesMethodArgumentResolver());
  		resolvers.add(new ModelMethodProcessor());
  		resolvers.add(new MapMethodProcessor());
  		resolvers.add(new ErrorsMethodArgumentResolver());
  		resolvers.add(new SessionStatusMethodArgumentResolver());
  		resolvers.add(new UriComponentsBuilderMethodArgumentResolver());
  
  		// Custom arguments
  		if (getCustomArgumentResolvers() != null) {
  			resolvers.addAll(getCustomArgumentResolvers());
  		}
  
  		// Catch-all
  		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
  		resolvers.add(new ServletModelAttributeMethodProcessor(true));
  
  		return resolvers;
  	}
  }
  ```




- 만약 새롭게 만든 argumentResolver를 적용하고 싶다면 상속받아 처리할 수도 있다.

  ```java
  public class TestRequestMappingHandlerAdapter extends RequestMappingHandlerAdapter{
  	private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
  	    List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver> ();
          
  	    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
  	    resolvers.add(new RequestParamMapMethodArgumentResolver());
  	    resolvers.add(new PathVariableMethodArgumentResolver());
  	    resolvers.add(new ServletModelAttributeMethodProcessor(false));
  	    resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters()));
  	    resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters()));
  	    resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
  	    resolvers.add(new RequestHeaderMapMethodArgumentResolver());
  	    resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
  	    resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
  
  	    resolvers.add(new ServletRequestMethodArgumentResolver());
  	    resolvers.add(new ServletResponseMethodArgumentResolver());
  	    resolvers.add(new HttpEntityMethodProcessor(getMessageConverters()));
  	    
         	//다음과 같이 추가
          resolvers.add(this.testArgumentResolver);
  	    
  	    resolvers.add(new RedirectAttributesMethodArgumentResolver());
  	    resolvers.add(new ModelMethodProcessor());
  	    resolvers.add(new MapMethodProcessor());
  	    resolvers.add(new ErrorsMethodArgumentResolver());
  	    resolvers.add(new SessionStatusMethodArgumentResolver());
  	    resolvers.add(new UriComponentsBuilderMethodArgumentResolver());
  
  	    if (getCustomArgumentResolvers() != null) {
  	      resolvers.addAll(getCustomArgumentResolvers());
  	    }
  
  	    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
  	    resolvers.add(new ServletModelAttributeMethodProcessor(true));
  
  	    return resolvers;
  	  }
  
  }
  
  ```

  

>resultArguement

```java
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

}
```

- NativeWebRequest를 통해서 HttpServletRequest를 얻을 수 있다

  ```java
  HttpServletRequest req = (HttpServletRequest) webRequest.getNativeRequest();
  ```



>supportsParameter

```java
public boolean supportsParameter(MethodParameter parameter) {}
```

- 현재 파라미터를 resolver에서 지원할지 boolean으로 체크
- 파라미터를 검증하는 용도로 사용(조건 미충족 시 다음 resolver로 넘어감)



참고 블로그 : https://velog.io/@kingcjy/Spring-HandlerMethodArgumentResolver%EC%9D%98-%EC%82%AC%EC%9A%A9%EB%B2%95%EA%B3%BC-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC