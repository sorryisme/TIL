# 리플레션

- [더 자바 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation)
  

## 스프링 DI는 어떻게 동작할까?



### 리플렉션 API 1부

- class 정보를 가져올 수 있음

  ```java
  //1.
  Class<Book> bookClass = Book.class;
  //2.
  Book book = new Book();
  Class<? extends Book> bookClass2 = book.getClass();
  //3.
  Class<?> bookClass3 = Class.forName("com.reflection.api.Book");
  ```

- 필드를 출력

  ```java
  Arrays.stream(bookClass3.getFields()).forEach(a->System.out.println(a));
  > 결과 : public String 필드만 출력
  ```

  - public 필드만 출력 , 전부 가져오고 싶다면 DeclaredFields 사용

- 접근자를 무시하고 모든 필드 값을 출력

  ```java
  Arrays.stream(bookClass2.getDeclaredFields()).forEach(a->{
    try {
      a.setAccessible(true);
      System.out.println(a.get(book));
    } catch (IllegalArgumentException e) {
    	e.printStackTrace();
    } catch (IllegalAccessException e) {
    	e.printStackTrace();
    }
  });
  ```



- 그 외 메소드, 슈퍼클래스, 인터페이스 등 모두 출력가능 



## 애노테이션과 리플렉션

- 중요 애노테이션
  - @Retention : 해당 애노테이션을 언제까지 유지
    - 소스, 클래스 , 런타임
  - @Inherit : 해당 애노테이션을 하위 클래스까지 전달
  - @Target : 어디에 사용할수 있는가?
- 리플렉션
  - getAnnotations() : 상속 받은 (@Inherit) 애노테이션까지 조회
  - getDeclaredAnootations() : 자기 자신에만 붙어있는 애노테이션 조회



- 애노테이션 개요

  - 인터페이스 앞에 @를 붙여 애노테이션 클래스 생성가능

  - 생성된 애노테이션은 어떤 클래스에서도 사용가능

  - 애노테이션은 주석과 동일함

    - 클래스에서는 남지만 바이트코드를 로딩할 때는 남아있지 않음

      - @RententionPolicy을 Runtime으로 지정해야한다(default는 class)

        ```java
        Arrays.stream(bookClass2.getAnnotations()).forEach(System.out::println);
        ```

  - 애노테이션은 값을 가질 수 있음

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyAnnotation {
    
    	String name() default "sorry";
    	
    	int number() default 1;
    
    }
    
    @MyAnnotation(name ="test", number = 1)
    public class Book {
    }
    ```

    - 만약 변수명이 value면 명시적으로 표시하지 않아도 된다.

  - 상위 클래스에 있는 애노테이션을 상속받게 하고 싶다면 애노테이션에 @Inherited를 붙여준다

  - 애노테이션에 지정된 값을 가져오는 방법

    ```java
    if ( a instanceof MyAnnotation ){
    	MyAnnotation myAnnotation = (MyAnnotation) a
    	System.out.println(myAnnotation.value());
    	System.out.println(myAnnotation.number());
    } 
    ```

    

### 클래스 정보 수정

- 클래스 인스턴스 만들기

  - Class.newInstance()는 deprecated
  - 생성자를 통해 생성

- 생성자로 인스턴스 만들기

  - Constructor.newInstance(params)

- 필드 값 접근하기 설정하기

  - 특정 인스턴스가 가지고 있는 값을 가져오는 것이기 때문에 인스턴스가 필요하다
  - Field.get(object)
  - Field.set(obejct, value)
  - Static 필드를 가져올 때는 object가 없어도 되기에 null을 넘기면 된다

- 메소드 실행하기

  - Object Method.invoke(object, params)

- 개요 

  - 생성자를 통한 인스턴스 생성

    ```java
    Class<?> bookClass = Class.forName("com.reflection.api.Book");
    Constructor <?> constructor = bookClass.getConstructor(null);
    Book book = (Book)constructor.newInstance();
    ```

  -  static 변수일 때는 따로 객체를 주지 않아도 가져오거나 값을 넣을 수 있다.

    ```java
    Field a = Book.class.getDeclaredField("A");
    System.out.println(a.get(null));
    a.set(null, "BBBBB");
    System.out.println(a.get(null));
    ```

  - 특별한 인스턴스에 해당하는 메소드인 경우, 객체와 파라미터를 넘겨준다

    ```java
    Method c = Book.class.getDeclaredMethod("c");
    c.invoke(book);
    ```

  - 메소드 실행 후 값을 리턴해주는 메소드의 경우 형변환하여 저장이 가능하다

    ```java
    Method c = Book.class.getDeclaredMethod("sum", int.class, int.class);
    int invoke = (int)c.invoke(book, 1,2);
    System.out.println(invoke);
    ```

    

### DI 프레임워크 만들기

- Inject라는 애노테이션을 만들어 필드에 주입하는 컨테이너 서비스 만들기

  ```java
  public class ContainerService {
  
  	public static <T> T getObject(Class<T> classType) {
  		T instance = createInstance(classType);
  		Arrays.stream(classType.getDeclaredFields()).forEach(f -> {
  			Inject annotation = f.getAnnotation(Inject.class);
  			if( annotation != null) {
  				Object fieldInstance = createInstance(f.getType());
  				f.setAccessible(true);
  				try {
  					f.set(instance, fieldInstance);
  				} catch (IllegalArgumentException e) {
  					throw new RuntimeException(e);
  				} catch (IllegalAccessException e) {
  					throw new RuntimeException(e);
  				}
  			}
  		});
  		
  		return instance;
  	}
  	
  	private static <T> T createInstance(Class<T> classType) {
  		try {
  			return classType.getConstructor(null).newInstance();
  		} catch (Exception e) {
  			e.printStackTrace();
  			return null;
  		}
  	}
  }
  ```



### 리플렉션 주의사항

- 지나친 사용은 성능 이슈를 야기할 수 있음
- 컴파일 타임에 확인하지 않고 런타임시에만 발생하는 문제를 만들 가능성이 있다.
- 접근 지시자를 무시할 수 있다.



### 사용 예시

- 스프링 : 의존성 주입
- 스프링 : MVC 뷰에서 넘어온 데이터를 객체에 바인딩
- 하이버네이트 : @Entity에 Setter가 없으면 리플렉션 사용

