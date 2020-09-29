# DispatcherServlet

참고 링크 : https://jeong-pro.tistory.com/225

참고 링크 : http://wonwoo.ml/index.php/post/2308

> SpringServletContainerInitializer.class

```java
@Override
	public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
        
        ... 중략
            
        for (WebApplicationInitializer initializer : initializers) {
            initializer.onStartup(servletContext);
        }
    }
```

- Bean으로 등록된 ServletContext를 onStartup을 통해서 순차적으로 초기화하는 과정을 거친다.
- Web.xml을 사용하지 않는다면 WebApplcationInitializer를 구현하면 DispatcherServlet을 생성할 수 있다.



> Web.xml

```xml
<web-app>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

- DispatcherServlet(FrontController)는 Controller, ViewResolver, HandlerMapping 같은 스프링 Bean 구성
- WebApplication에는 모든 서블릿이 공유할 수 있는 Service, Repository를 구성

```xml
<servlet>
	<servlet-name>app</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/app/*</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>json</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>json</servlet-name>
    <url-pattern>/json/*</url-pattern>
</servlet-mapping>
```

- 다음과 같이 Dispatcher 서블릿을 여러 개 구성할 수 있다.



## 핵심 Bean

### HandlerMapping

- Request에 맞는 컨트롤러 선택하기 위한 인터페이스
- 가장 많이 사용하는 구현체는 RequestMappingHandlerMapping(@RequestMapping 지원)



### HandlerAdapter

- HandlerMapping에 결정된 정보로 메서드 호출 후 request 전달 및 response를 생성하는 인터페이스

- RequestMappingHandlerAdapter를 가장 많이 사용 (RequestMappingHandlerMapping 매칭)

  > RequestMappingHandlerAdapter

  ```java
  public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
  		implements BeanFactoryAware, InitializingBean {
  	
      ...중략
          
  	public RequestMappingHandlerAdapter() {
  		StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter();
  		stringHttpMessageConverter.setWriteAcceptCharset(false);  // see SPR-7316
  
  		this.messageConverters = new ArrayList<HttpMessageConverter<?>>(4);
  		this.messageConverters.add(new ByteArrayHttpMessageConverter());
  		this.messageConverters.add(stringHttpMessageConverter);
  		this.messageConverters.add(new SourceHttpMessageConverter<Source>());
  		this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());
  	}
      
      ... 중략 
          
      public void setArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
  		if (argumentResolvers == null) {
  			this.argumentResolvers = null;
  		}
  		else {
  			this.argumentResolvers = new HandlerMethodArgumentResolverComposite();
  			this.argumentResolvers.addResolvers(argumentResolvers);
  		}
  	}
      
      ...중략
  	public void setReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {
  		if (returnValueHandlers == null) {
  			this.returnValueHandlers = null;
  		}
  		else {
  			this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite();
  			this.returnValueHandlers.addHandlers(returnValueHandlers);
  		}
  	}
  
  		
  }
  ```

  - 생성자에서는 기본적으로 messageConverts를 등록한다
  - 기본적으로 argumentResolver 등록, returnValueHandler를 등록하여 처리한다
  - 실제로 메서드를 실행하는 클래스는 ServletInvocableHandlerMethod(변수명 - invocableMethod )에서 처리한다

