#  테스트하는 다양한 방법

- [강좌 : 더 자바, 애플리케이션을 테스트하는 다양한 방법](https://www.inflearn.com/course/the-java-application-test)



## JUnit5

![image1](./img/image1.jpg)

- Jupiter : Test Engine API 구현체 Junit5를   제공
- Vintage : Junit 4와 3를 지원하는 TestEngine 구현체
- Test 클래스 생성 단축키 : 클래스에서 ctrl + shift + T
- JUnit5부터는 public이 아니여도 상관없음
- 실행 단축키 
  - 윈도우 : ctrl + shift + F10
  - Mac O/S : ctrl + shift + R



## 기본 애노테이션

- @Test
- @BeforeAll / @AfterAll
  - 테스트 실행 전/ 실행 후 단 한 번만 작동한다
  - 반드시 static void으로 선언해야한다
- @BeforeEach / @AfterEach
  - 모든 테스트 실행할 때 각각 테스트 이전과 이후에 실행한다
- @Disabled
  - 테스트에서 실행하기 싫을 때 애노테이션으로 표기



## JUnit5 테스트 이름 표시하기

- @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
  - 언더스코어를 제거한 공백으로 바꾼 메소드 이름 표시
- @DisplayName
  - 테스트 이름을 따로 표기하고 싶을 때 사용

## Assertion

- org.junit.jupiter.api.Assertions

| 메소드명                               | 내용                                  |
| -------------------------------------- | ------------------------------------- |
| assertEquals(Expected, actual)         | 실제값과 기대값이 같은지 확인         |
| assertNotNull(actual)                  | 값이 Null 아닌지 확인                 |
| assertTrue(boolean)                    | 다음 조건이 참인지 확인               |
| assertAll(executables...)              | 모든 확인 구문 확인                   |
| assertThrows(expectedType, executable) | 예외 발생 확인                        |
| assertTimeout(duration, executable)    | 특정 시간 안에 실행이 완료되는지 확인 |

- assertAll 
  - 여러 개의 assert구문을 람다식으로 묶어버리면 개별적으로 실행하지 않아도 된다
- assertThrows(NoSuchElementException.class, 람다); 형태로 작성



## 조건에 따라 테스트 실행

- 특정한 OS, 특정한 자바버전, 환경변수에 따라 실행 여부를 결정할 수 있음
  - assumeTrue() : 조건에 맞으면 아래 테스트 코드 실행
  - assumingThat() : 조건에 맞으면 안에 작성된 코드가 실행
- @Enabled와 @Disabled
  - OnOs : 운영체제 입력
  - OnJre : 특정 자바 버전
  - IfSystemProperty :
  - IfEnvironmentVariable : 환경변수 매칭 값
  - If



## 태깅과 필터링

- @Tag

  - 테스트 실행할 때 지정한 테스트만 실행되도록 처리할 수 있
  - 메이븐에서 프로파일로 테스트 태깅을 지정해줄 수 있음

- 커스텀 태그

  - 애노테이션을 직접 만들어서 사용가능

    ```java
    @Target(ElementType.Method)
    @Retention(RetentionPolicy.RUNTIME)
    @Test
    @Tag("test")
    public @interface FastTest {
    
    }
    ```



## 테스트 반복하기

### @RepeatedTest

- 반복 횟수와 반복 테스트 이름을 설정할 수 있다.
  - {displayName}
  - {currentRepetition}
  - {totalRepetitions}
- RepetitionInfo 타입의 인자를 받을 수 있다.
  - 메소드 인자에 변수를 선언하여 현재 실행 카운트 / 토탈 카운트 정보를 확인할 수 있음



### @ParameterizedTest

- @ValueSource
  - 파라미터들을 정의할 수 있음
- 테스트에 여러 매개변수를 대입해가며 반복 실행한다.
  - {displayName}
  - {index}
  - {arguments}
  - {0}, {1}....
- 인자값들의 소스
  - @ValueSource
  - @NullSource, @EmptySource, @NullAndEmptySource
  - @EnumSource
  - @MethodSource
  - @CvsSource, @CvsFileSource
  - @ArgurmentSource
- 인자값 타입변환
  - 암묵적인 타입변환
  - 명시적인 타입변환  : 한개의 아규먼트 
    - SimpleArgumentconverter 상속받은 구현체 제공
    - @ConvertWith
    - ex:) @ConvertWith(StudyConverter.class) Study study
  - ArgumentsAccessor argumentsAccessor
    - argument 순서대로 가져올 수 있음
  - ArguementsAggregater 인터페이스 :  두 개 이상 아규먼트
    - @AggregateWith



## Mockito

- Mock: 진짜 객체와 비슷하게 동작하지만 프로그래머가 그 객체의 행동을 관리하는 객체
- 이미 구현된 클래스에 대해 mock할 필요는 없다. 하지만 외부 서비스같이 분리되어있는 건 mocking하는 걸 추천
- 스프링 부트 2.2+ 프로젝트 생성 시 spring-boot-starter-test에서 자동으로  mockito를 추가해줌
- 다음 세 가지만 알면 테스트를 쉽게 작성할 수 있음
  - Mock을 만드는 방법
    - Mock이 어떻게 동작하는지 관리하는 방법
  - Mock의 행동을 검증하는 방법

###  Mock 객체 만들기

- Mockito.mock() : 메소드로 만드는 법
  - MemberService memberService = mock(MemberService.class)
- Mock 애노테이션으로 만들기
  - Junit 5 extension으로 MockitoExtension을 사용
  - 필드
  - 메소드 매개변수

```java
@ExtendWith(MockitoExtension.calss)
class StudyServiceTest {
	
	@Mock 
	MemberService memberService;
	
	@Mock
	StudyRepository studyRepository;
	
}
```

- Mock 애노테이션을 사용하기 위해 @ExtendWith를 사용

> Mockito로 객체생성

```java
StudyRepository sutdyRepository = Mockito.mock(StudyRepository.class);
```

- mock 메서드를 통해 객체를 생성할 수 있음(인터페이스)
  - 코드를 줄이기 위해 @Mock을 사용

```java
@ExtendWith(MockitoExtension.class)
class StudyServiceTest{
	
	@Test
	void createStudyService(@Mock MemberService memberService, @Mock StudyService studyService){
		
		StudyService studyService = new StudyService(memberService, studyRepository);
		assertNotNull(studyService);
	}
	

}
```

- MeberService와 StudyService는 인터페이스이기 때문에 구현이 불가하다
  - Mock 객체를 통해 임의의 객체를 생성해서 전달한다



### Mock 객체 Stubbing

- Stubbing이란 Mock 객체의 행동을 조작하는 것

- 모든 Mock 객체의 행동
  - Null을 리턴한다(Optional => Optional.empty 리턴)
  - Primitive 타입은 기본 Primitive 값
  - 콜렉션은 비어있는 콜렉션
  - Void 메서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다
- Mock 객체를 조작해서
  - 특정한 매개변수를 받은 경우 특정한 값을 리턴하거나 예외를 던지도록 만들 수 있다.
  - Void 메소드 특정 매개변수를 받거나 호출된 경우 예외를 발생시킬 수 있다.
  - 메소드가 동일한 매개변수로 여러번 호출될 때 각자 다르게 호출 되도록 조작할 수도 있다.



#### Stubbing 사용 

> When(Stubbing)

```java
when(memberService.findById(1L)).thenReturn(member);
```

- memberService가 호출될 때 다음과 같이 값을 리턴해라
- 파라미터를 범용적으로 사용하고 싶으면 Matcher 사용
  - any()



#### 예외 처리

> when 예외

```
when(memberService.findById(1L)).thenThrow(new RuntimeException());
```

- 예외를 강제로 발생시킴

> doThrow 

```
doThrow(new IllegalArgunmentException()).when(memberService).validate(1L);
```





#### 같은 메소드를 여러번 호출

```
when(memberService.findById(any()))
					.thenReturn(Optional.of(member))
					.thenThrow(new RuntimeException())
					.thenReturn(Optional.empty());
```

- 첫 번째 객체 리턴
- 두 번째 예외 리턴
- 세 번째 비어있는 객체 리턴



### Mock 객체 확인

- Mock 객체가 어떻게 사용 되었는지 확인할 수 있다.
  - 몇 번 호출되었는지, 최소 한 번은 호출되었는지, 전혀 호출되지 않았는지
  - 어떤 순서대로 호출했는지
  - 특정 시간 이내에 호출 됐는지
  - 특정 시점 이후에 아무 일도 벌어지지 않았는지



> 몇 번 호출 되었는지

```java
verify(memberService, times(1)).notify(any());
```

- notify메서드를 호출 안했을 경우 에러 호출



> 전혀 호출되지 않아야하는 경우

```
verify(memberService, never()).validate(any());
```





### Mockito BDD 

- 어플리케이션이 어떻게 행동해야하는지 공통된 이해 구성하는 방법

- 행동에 대한 스펙 

  - Title
  - Narrative
    - As a : 역할
    -  I want : 원하는 것
    -  so that : 그 결과
  - Acceptance criteria
    - Given : 상황이 주어졌을 때 
    - When : 어떤 행위를 하면
    - Then : 그 결과 

  

  > when  -> given

  ```java
  when(memberService.findById(1L)).thenReturn(Optional.of(member));
  given(memberService.findById(1L)).willReturn(Optional.of(member));
  ```

  

  > verify -> then

  ```
  verify(memberService, times(1)).notify(study);
  then(memberService).should(times(1)).notify(study);
  ```

  

  

  