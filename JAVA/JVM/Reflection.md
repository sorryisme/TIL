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

