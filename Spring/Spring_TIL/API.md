# API 



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