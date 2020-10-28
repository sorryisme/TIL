# API 



### 200 OK

```java
new ResponseEntity<Todo>(todo, HttpStatus.OK);
```

- 성공의 의미는 HTTP 메소드에 따라 달라짐
  - GET : 리소스를 불러와 메세지 바디에 전송
  - HEAD : 개체의 헤더가 메세지 바디에 있음
  - PUT 또는 POSt : 수행결과에 대한 리소스가 요청한 메세지에 포함
  - TRACE : 메시지 바디는 서버에서 수신한 요청 메세지를 포함
    

###  204 No Content 코드

```java
ResponseEntity.noContent().build();
```

- 요청은 성공했으나 리턴되는 컨텐츠가 없음
- Delete 요청 시 주로 사용
- 정보에 따른 결과를 얻을 수 없어 권장하지 않는다고 함 [링크](https://mygumi.tistory.com/230)





###  201 Created

```java
URI location = ServletUriComponentsBuilder.fromCurrentRequest().path("/{id}").buildAndExpand(createTodo.getId()).toUri();
ResponseEntity.created(location).build();
```

- 새로운 자원 생성 성공을 뜻함
- request 경로에 추가적으로 리소스를 포함하여 uri를 생성
- response에 uri을 포함하여 전달 : request uri + / {생성된 id값}



### 404 Not found

```java
ResponseEntity.notFound().build();
```

- 서버에서 요청받은 리소스를 찾을 수 없음
- 리소스 자체가 존재하지 않음을 의미



