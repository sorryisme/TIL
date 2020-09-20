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

    

