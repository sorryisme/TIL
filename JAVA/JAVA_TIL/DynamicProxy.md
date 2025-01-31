# Dynamic Proxy - Spring Data JPA

- 자바를 조작하는 다양한 방법 강의 내용



## 스프링 데이터 JPA 는 어떻게 동작할까

- bookRepositry는 누가 만들어주는가?
  - 프록시가 생성
  - Spring Aop를 사용하여 RepositoryFactorySupport가 생성



### 프록시 패턴

- 프록시와 리얼 서브젝트가 공유하는 인터페이스가 있고 클라이언트는 인터페이스 타입으로 픔록시를 사용
- 클라이언트는 프록시를 거쳐 서브젝트를 사용
  - 리얼 서브젝트에 대한 접근관리, 부가기능, 리턴값 변경은 프록시가 처리
- 리얼 서브젝트는 자신이 해야할 일만 하면서 프록시를 통해 부가적인 기능(접귽 제한, 로깅, 트랜잭션 등)을 제공할 때 패턴 사용

> 클라이언트

```java
public class BookServiceTest {
//	BookService bookService = new DefaultBookService();
	Bookservice bookService = new BookserviceProxy(new DefaultBookService()); //프록시에게 객체를 넘겨줌
	@Test
	public void di(){
		bookService.rent(book);
	}
}
```

> 리얼 서브젝트 객체

```java
public class DefaultBookService implements BookService{
	@Override
	public void rent(Book book){
		System.out.println("book")
	}
}
```

> 프록시 객체

```java

public class BookProxy implements BookService{
	//프록시 생성시 리얼 서브젝트를 소유
	BookService bookService;
  
  public BookProxy(Bookservice bookService) {
    this.bookService = bookService;
  }
	
  @Override
	public void rent(Book book){
		 System.out.println("before"); // 추가 기능
     bookService.rent(book); // 실제 리얼 프로젝트 메소드 호출
     System.out.println("after");  // 추가 기능2
	}
	
}
```

- 프록시 패턴은 번거롭다 ( 부가적인 기능을 추가하고 싶다면 객체를 생성, 위임코드 중복 발생, 메소드 추가 때 마다 프록시에 추가)
  - 동적으로 런타임에 생성하는 방법 => 자바 Reflection에서 처리 , 다이나믹 프록시



### 다이나믹 프록시

- 런타임에 특정 인터페이스들을 구현하는 클래스 또는 인스턴스를 만드는 기능
- 프록시 인터페이스 프록시 만들기
  - Object Proxy.newProxyInstance(ClassLoader, interfaces, InvocationHandler)
  - 유연한 구조가 아님 -> 스프링 AOP 

```java
BookService bookService = Proxy.newProxyInstance(BookService.class.getClassLoader(), new Class[]{BookService.class}, new InvocationHandler(){ 
			BookService bookservice = new DefaultBookService();
  		@Override
  		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        	// 부가 기능 추가
         	return method.invoke(bookService, args);
      }
	
});
```

- 실제 동작할 부가적인 메서드 구현해야함( 프록시에서 실행할 부가적인 기능)
- 코드가 점점 커지는 문제가 발생할 수 있어 AOP를 통해 유연하게 처리 가능
- 다이나믹 프록시는 **인터페이스가 아닌 클래스 기반 프록시 기반 클래스**를 생성 불가(자바 다이나믹 프록시의 제약사항)



#### 클래스 프록시가 필요하다면

- CGlib

  - 스프링, 하이버네이트가 사용하는 라이브러리

    ```java
    MethodInterceptor handler = new MethodInterceptor() {
    			BookService bookService = new BookService();
    		
    			@Override
    			public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
    				
    				return method.invoke(bookService, args);
    			}
    		};
    		BookService bookService = (BookService) Enhancer.create(BookService.class,handler);
    ```

- Bytebuddy

  ```java
  Class<? extends BookService> proxyClass = new ByteBuddy().subclass(BookService.class)
  																										.make().load(BookService.class.getClassLoader()).getLoaded();
   
  proxyClass.getConsjtructor(null).newInstance();
  ```

  

- 상속을 허용하지 않는 경우 사용불가
  - final 클래스
  - 생성자가 private default 생성자



#### 다이나믹 프록시 사용처

- 스프링 데이터 JPA
- 스프링 AOP
- Mockito
- 하이버네이트 lazy Initialization

