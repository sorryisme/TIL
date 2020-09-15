# JVM 

- 자바를 조작하는 다양한 방법 수업 내용 정리



## 1. JVM, JDK, JRE의 차이

- 바이트코드(.class 파일)을. OS에 특화된 코드로 변환( 인터프리터와 JIT 컴파일러)하여 실행

  - ex : javap -c HelloJava로 실행했을 때 변환되는 코드

    ```
    Compiled from "HelloJava.java"
    public class HelloJava {
      public HelloJava();
        Code:
           0: aload_0
           1: invokespecial #1                  // Method java/lang/Object."<init>":()V
           4: return
    
      public static void main(java.lang.String[]);
        Code:
           0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
           3: ldc           #3                  // String hello world
           5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
           8: return
    }
    ```

    - javap 는 역어셈블하는 명령어 , 클래스 내부 상수/함수 목록들을 확인하기 위함

- JRE

  - 자바 애플리케이션을 실행할 수 있도록 구성된 배포판(개발 관련 도구는 미포함)
  - 참고 : java compile에 필요한 javac는 포함하지 않는다
  - java 11부터 jre을 따로 제공하지 않음 , jdk만 제공 
  - java 9에서는 jlink를 통해 jre를 구성할 수 있음

- JDK

  - JRE + 개발할 때 필요한 툴
    

- 이미지 
  
  - 추후 추가



## 2. JVM 구조



- 이미지 추가 예정



### 클래스 로더 시스템

- .class에서 바이트 코드를 읽고 메모리 저장
- 로딩 : 클래스 읽어오는 과정
- 링크 : 레퍼런스를 연결하는 과정
- 초기화 : static 값들 초기화 및 변수 할당



### 메모리

- 메소드 영역에는 클래스 수준 정보를 저장, 공유
  - 클래스 이름, 부모 클래스 이름, 메소드, 변수
- 힙 영역 : 객체 저장, 공유 자원
- 나머지 영역은 쓰레드 영역
  - 스택 
    - 쓰레드 마다 런타임 스택을 만들고 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다
    - 스택 프레임 => 메소드 콜과 동일하다고 봄
  - PC
    - 쓰레드 마다 쓰레드 내 실행할 스택 프레임을 가르키는 포인터 생성
  - 네이티브 메소드 스택
    - 네이티브 메소드 라이브러리나 인터페이스를 통한 호출할 때 쓰레드 마다 생성
    - 참고 : 코드가 네이티브 되어있고 자바가 아닌 C나 C++로 구현되어있는 것을 **네이티브 메소드**라 부름
- 실행 엔진
  - 인터프리터 : 바이트코드를 한줄 씩 실행
    - 바이트 => 네이티브로 번역하지만 매번 같은 코드를 네이티브 코드로 바꾸지 않는다
    - JIT 컴파일러 : 인터프리터가 반복되는 코드를 JIT로 보내고 이를 한 번에 네이티브 코드로 변경한다



### 클래스 로더

- 이미지 추가



- 로딩, 링크, 초기화 순으로 진행
  
- 로딩 (부트스트랩 -> Extension -> application)
  - 클래스 로더가 .class 을 읽어 메소드 영역에 저장
  - 저장하는 데이터
    - FQCN : 패키지 경로까지 포함한 클래스명
    - 클래스 | 인터페이스 | 이늄
    - 메소드와 변수
  - 로딩이 끝나면 클래스의 타입의 Class 생성하여 Heap에 저장
  - 클래스 로더 종류
    - 부트 스트랩 클래스 로더 
      - JAVA_HOME\lib에 있는 코어 자바 API를 제공, 최상위 우선순위
    - 플랫폼 클래스 로더
      - JAVA_HOME\lib\ext 폴떠 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽음
    - 애플리케이션 클래스 로더
      - 어플리케이션 -classpath 변수 값에 해당하는 클래스를 읽음



- 링크
  - verify :  .class파일 형식이 유효한지 체크
  - Preparation : 클래스 변수(static 변수)와 기본 값에 필요한 메모리 준비
  - Resolve : 심볼릭 메모리 레퍼런스를 메소드에 있는 실제 레퍼런스(Heap)와 교체(Optional)
- 초기화 
  - static 변수를 할당 , static 블럭 또한 실행