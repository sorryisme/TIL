# DispatcherServlet

참고 링크 : https://jeong-pro.tistory.com/225

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





작성 진행중 