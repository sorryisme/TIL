# JVM





## 자바 성능 튜닝 이야기 

- HotSpot VM의 구조
- JIT 옵티마이저
- JVM의 구동절차
- JVM의 종료절차
- 클래스 로딩의 절차
- 예외 처리의 절차



### HotSpot VM의 구성

- 자바의 성능을 개선하기 위해 JIT 컴파일러 개발
  - 프로그램의 성능에 영향을 주는 지점에 대해 지속적으로 분석
  - 분석된 지점은 부하를 최소화 하고 높은 성능을 내기 위한 최적화 대상
- HotSpot VM은 세 가지 컴포넌트로 구성
  - VM 런타임
  - JIT 컴파일러
  - 메모리 관리자



### JIT Optimizer 

- JIT 컴파일러는 Client 버전과 Server버전이 있음

- 컴파일은 상위레벨의 언어 -> 기계 의존적인 코드로 변환

- JIT는 각각의 메서드를 모두 컴파일 하지 않음
  - 모든 코드는 초기에 인터프리터에 의해서 시작
  
  - 해당 코드가 많이 사용될 경우 컴파일 대상
  
  - 각 메서드에 카운터를 통해 통제
  
  - 메서드에는 두 개의 카운터가 존재
    - 수행 카운터(invocation counter) : 메서드를 시작할 때 증가
    - 백에지 카운터(backed counter) : 높은 바이트 코드 인덱스에서 낮은 인덱스로 컨트롤 흐름이 변경될 때 증가
      - 메서드 루프가 존재하는지 확인할 때 사용
      - 수행 카운터보다 우선 순위가 높음
    
  - 카운터 한계치 도달 시 인터프리터가 컴파일을 요청 
  
  - 수행 카운터 한계치는 CompileThreshold
  
    - CompileThreshold * OnStackReplacePercentage / 100
  
      ```
      JVM에서 옵션으로 지정할 수 있다
      
      XX:CompileTYhreadshold=35000
      XX:OnStackReplacePercentage=80
      
      메서드가 3만 5천 번 호출 시 JIT에서 컴파일
      백에이지 카운터가 35,000 * 80 / 100 =28,000 이 되었을 때 컴파일\
      ```
  
  - 컴파일이 요청되면 큐에 쌓이고 컴파일러 스레드가 큐를 모니터링
  
  - 컴파일러 스레드가 바쁘지 않을 때 큐에서 대상을 꺼내 컴파일 시작
  
  - 인터프리터는 컴파일이 종료를 기다리지 않고 수행카운터를 리셋 후 인터프리터에서 메서드 수행을 계속 진행
  
- OSR : 특별한 컴파일

  - 인터프리터 수행한 코드 중 오랫동안 루프가 지속되는 경우 사용
    - 최적화되지 않은 것은 코드를 수행하는 것을 확인했을 때 컴파일된 코드로 변경



### JRockit의 JIT 컴파일 및 최적화 절차

- JVM은 OS에서 작동 가능하도록 자바 바이트 코드를 받아 아키텍처에서 잘 돌아가는 기계어로 변환,수행

- 최적화 단계

  - JRockit runs JIT compilation
    - 자바 어플리케이션 실행 후 JIT 컴파일 거친 후 실행
    - JIT를 거친 후 컴파일 된 코드를 호출하기 때문에 처리 성능이 빨라진다.
    - 어플리케이션이 시작하는 동안 몇 천개의 새로운 메서드가 수행됨
      - 이로 인해 다른 JVM보다 JRockit JVM이 더 느릴 수 있다.
      - 지속적으로 수행 시 더 빠른 처리가 가능
      - 모든 메서드를 컴파일하고 최적화 작업은 JVM 시작을 느리게 만들기 때문에 모든 메서드를 최적화하지 않음 
  - JRockit monitors threads
    - sampler thread가 주기적으로 어플리케이션 스레드를 점검
    - 어떤 메서드가 많이 사용되었는지 확인 후 최적화 대상을 찾음
  - JRokcit JVM Runs Optimization
    - sampler thread가 식별한 대상을 최적화
    - 백그라운드에서 진행, 수행중인 어플리케이션에서 영향을 주지 않음

  ```java
  class A {
  	B b;
  	public void foo(){
  		y = b.get();
  		z = b.get();
  		sum = y + z;
  	}
  }
  
  class B {
  	int value;
  	final int get(){
  		return value;
  	}
  }
  ```



- JRockit 컴파일러는 다음과 같이 최적화 한다

  ```java
  class A{
  	B b;
  	public void foo(){
  		y = b.value;
  		sum = y + y;
  	}
  }
  ```

- 최적화는 다음과 같은 단계를 거친다

  | 최적화단계                       | 코드 변환    | 설명                                                         |
  | -------------------------------- | ------------ | ------------------------------------------------------------ |
  | 시작단계                         | 변경없음     |                                                              |
  | final로 선언된 메서드 인라인처리 | b.value      | b.get이 b.value로 변환되며 이로 인해 <br />메서드 호출로 인한 성능저하 개선 |
  | 불필요한 부하 제거               | z = y        | z와 y값이 동일함므올 z에 y값을 할당한다                      |
  | 복제                             | sum = y + y; | z와 y값이 동일함으로 불필요한 변수인 z를 y로 변경            |
  | 죽은코드 삭제                    |              | y= y 코드가 불필요함으로 삭제한다                            |



### IBM JVM의 JIT 컴파일 및 최적화 절차

- 인라이닝 : 메서드가 단순할 때 그 내용이 호출한 메서드의 코드에 포함시켜버린다
- 지역 최적화 : 작은 단위 코드를J 분석 및 개선작업
- 조건 구문 최적화 : 메서드 내의 조건 구문 최적화 및 효율성을 위해 코드 수행 경로 변경
- 글로벌 최적화 : 메서드 전체 최적화 
- 네이티브 코드 최적화 : 아키텍처에 따라 최적화를 달리함



### JVM 시작절차

- Hell World 클래스 실행 시 다음과 같은 절차를 거친다.
  - java 명령 줄에 있는 옵션 파싱
    - 자바 실행 프로그램에 적절한 JIT 컴파일러 선택하는 등의 작업을 하기위해 사용하고 다른 명령들은 HotSpot VM에 전달된다.
  - 자바 힙 크기 할당 및 JIT 컴파일러 타입 지정
    - 명시적으로 지정되지 않은 경우 시스템이나 프로그램에 맞게 선정
  - CLASSPATH와 LD_LIBRARY_PATH 같은 환경변수를 지정
  - 자바의 Main 클래스가 지정되지 않았으면 JAR 파일의 mainfest 파일에서 Main 클래스를 확인한다
  - JNI 표준 API인 JNI_CreateJavaVM를 사용하여 새로 생성한 non-primordial이라는 스레드에서 HotspotVM을 생성한다.
  - Hotspot VM이 생성되고 초기화 되면 Main 클래스가 로딩된 런처에서는 main 메서드의 속성정보를 읽는다.
  - CallStaticVoidMethod는 네이티브 인터페이스를 불러 HotSpot VM에 있는 main 메서드가 수행된다. 이 때 Main 클래스 뒤에 있는 값들이 전달된다.

#### JNI_CreateJavaVM 단계

- JNI_CreateJavaVM는 동시에 두 개의 스레드에서 호출할 수 없고, 오직 하나의 HotStop VM 인스턴스가 프로세스 내에 생성될 수 있도록 보장
- JNI 버전이 호환성이 있는지 점검하고 GC 로깅을 위한 준비도 오나료
- OS 모듈들이 초기화 , 랜덤 번호 생성기, PID 할당 등
- 커맨드 라인 변수와 속성들이 JNI_CreateJavaVM 변수에 전달되고 나중에 사용하기 위해 파싱한 후 보관
- 표준 자바 시스템 속성이 초기화
- 동기화, 메모리, safepoint 페이지와 같은 모듀들이 초기화
- 라이브러들이 로드
- 시그널 처리기가 초기화 및 설정
- 스레드 라이브러리 초기화
- 출력 스트림 로거가 초기화
- JVM 모니터링을 하기 위한 라이브러리가 설정되어있으면 초기화 및 시작
- 스레드 처리를 위해서 필요한 스레드 상태와 스레드 로컬 저장소가 초기화
- HotSpot VM의 글로벌 데이터들이 초기화, 글로벌 데이터에는 이벤트 로그 메모리 할당자들이 있다.
- HotSpot VM 에서 스레드를 생성할 수 있는 상태가 된다. main 스레드가 생성되고 현재 OS 스레드가 붙지만 스레드 목록에는 추가되지 않는다.
- 자바 레벨의 동기화가 초기화 및 활성화
- 부트 클래스 로더, 코드 캐시, 인터프리터, JIT 컴파일러 JNI, 시스템 dictionary, 글로벌 데이터 구조 집합인 universe 등이 초기화
- 스레드 목록에 자바 main 스레드 추가 universe 상태가 점검한다.
- java.lang 패키지에 있는 String, System, Thread, ThreadGroup, Class와 MEthod, Finalizer 클래스 등이 로딩되고 초기화 된다,.
- HotSpot VM의 시그널 핸들러 스레드가 시작되고 JIT 컴파일러가 초기화 되며, HotSpot 컴파일러 브로커 스레드가 시작된다.
- JNIEnv 시작 되며 HotSpot VM을 시작한 호출자에게 JNI 요청을 처리할 상황이 되었다고 전달



## JVM 종료될 때 절차

- HotSpot VM의 종료는 DestoryJavaVM 메서드의 종료절차를 따른다.
  - HotSpot VM이 작동중인 산황에서는 단 하나의 데몬이 아닌 스레드가 수행될 때까지 대기한다
  - java.lang 패키지에 있는 shudown 클래스의 shutdown 메서드가 수행 
    - 자바 레벨의 shutdown hook이 수행되고 finalization-on-exit이라는 값이 true일 경우 자바 객체 finalizer를 수행한다.
  - HotSpot VM 레벨의 shutdown hook을 수행함으로써 Hotspot VM의 종료를 준비
    - 이 작업은 JVM_OnExit 메서드를 통해 지정 그리고 HotSpot VM의 profiler, stat sampler, watcher, garbage collector 스레드를 종료시킨다.
    - 위 작업이 종료되면 JVNTI를 비활성화 시키며 Signal 스레드를 종료시킨다.
  - HotSpot의 JavaThread::exit() 메서드를 호출하여 JNI 처리 블록을 해제한다. 그리고 guard pages, 스레드 목록에 있는 스레드들을 삭제한다. HotSpot VM에서 자바코드를 실행하지 못한다.
  - HotSpot 스레드를 종료, 이작업을 수행하면 HotSpot VM에 남아 safepoint로 옮기고 JIT 컴파일러 스레드를 중지시킨다
  - JNI, HotSpot VM, JVMTI barrier 있는 추적 기능을 종료시킨다.
  - 네이티브 스레드에서 수행하고 있는 스레들을 위해서 HotSpot의 vm eited 값을 설정
  - 현재 스레드를 삭제한다
  - 입출력 스트림을 삭제하고 perfMemory 리소스 연결을 해제 한다.jvm 
  - JVM 종료를 호출한 호출자로 복귀한다.



### 클래스 로딩 절차

- 주어진 클래스의 이름으로 클래스 패스에 있는 바이너리로 된 자바 클래스를 찾는다
- 자바 클래스를 정의한다
- 해당 클래스를 나타내는 java lang 패키지의 Class 클래스의 객체를 생성한다.
- 링크 작업이 수행된다. 이 단계에서 static 필드를 생성 및 초기화 하고 메서드 테이블을 할당한다
- 클래스의 초기화가 진행되며 클래스 static 블로가 static 필드가 먼저초기화 
- loading -> linking -> initializing 순으로 진행된다고 기억하자

```
클래스 로딩 중 발생되는 에러

NoClassDefFoundError : 클래스 파일을 찾지 못한경우
ClassFormatError : 클래스 포맷이 잘못된 경우
UnsupportedClassVersionError : 상위 버전 JDK에서 컴파일 한 클래스를 하위 버전에서 실행하려는 경우
ClassCircularityError : 부모 클래스를 로딩하는 데 문제가 있는 경우
IncompatibleClassChangeError : 부모가 클래스인데 implements 하는 경우나 부모가 인터페이스인데 extends 하는 경우
VerifyError : 클래스 파일의 semantic 상수풀, 타입 등의 문제가 있을 경우
```

- 클래스 로더가 클래스를 찾고 로딩할 때 다른 클래스 로더에 클래스를 로딩해달라는 경우가 있다.
  - 이를 class loader delegation 
  - 클래스 로더는 계층적으로 구성
  - 기본 클래스 로더는 시스템 클래스 로더
    - main 메서드가 있는 클래스와 클래스 패스에 있는 클래스들이 이 클래스 로더에 속함
  - 그 하위에 어플리케이션 클래스 로더는 SE의 기본 라이브러리에 있는 것이 될 수 있고 개발자가 임의로 만든 것일 수 있다.