# 테스트하는 다양한 방법



## JUnit5

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
  - 반드시 static으로 선언해야한다
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