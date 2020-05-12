# GC 모니터링

자바 성능 튜닝이야기 18장



- GC 모니터링 하기 위한 여러가지 방법 있음

  - jstat 명령사용
  - Verbosegc 옵션 사용
  - Visual CG (oracle jdk)

- 자바 인스턴스 확인을 위한 jps

  - 해당 머신에서 운영 중인 JVM 목록을 보여준다

  - JDK bin 디렉토리에 있음

  - jps 는 다음과 같은 옵션으로 사용

    ```
    jps [-q] [-mlvV] [-Joption] [<hostid>]
    ```

    - -q: 클래스나 JAR 파읾 명, 인수 등을 생략하고 내용을 나타낸다.(프로세스 id만 나타낸다)
- -m : main 메서드에 지정한 인수들을 나타낸다
  
    - -l : 어플리케이션 main 클랫래스나 어플리케이션 JAR 파일 전체 경로 이름을 나타낸다
- -v : JVM에 전달된 자바 옵션 목록을 나타낸다.
  
    - -V : JVM의 플래그 파일을 통해 전달된 인수를 나타낸다
- -option : 자바 옵션을 이 옵션 뒤에 지정할 수 있다.

### GC상황을 알려주는 jstat

```
jstat -<option> [-t] [-h<lines] <vmid> [<interval> [<count]]
```

- -t : 수행 시간을 표시한다
- -h : 각 열의 설명을 지정된 라인 주기로 표시한다
- interval : 로그를 남기는 시간 차이(밀리초 단위)를 의미
- count : 로그를 남기는 횟수
- 옵션의 종류
  - class : 클래스 로더에 대한 통계
  - compiler 핫스팟 JIT 컴파일러에 대한 통계
  - gc : GC 힙 영역에 대한 통계
  - gccapacity : 각 영역의 허용치와 연관된 영역에 대한 통계
  - gccause : GC 요약정보와 마지막 GC와 현재 GC에 대한 통계
  - gcnew : 각 영역에 대한 통계
  - gcnewcapacity : Young 영역과 관련된 영역에 대한 통계
  - gcold : Old와 Perm에 영역에 대한 통계
  - gcoldcapacity : Old영역에 대한 통계
  - gcpermcapcity : Perm 영역에 대한 통계
  - gcutil : GC 요약 정보
  - Print compilation : 핫스팟 컴파일 메서드에 대한 통계
- jstat 로그로 분석에 한계가 있음
  - 로그를 남기는 주기에 GC가 한 번 발생할 수도 있고 10번 발생할 수도 있다.
  - verbosegc 옵션을 사용 권장
- 가장 많이 사용하는 옵션
  - -gcutil
    - YGC : Young의 GC횟수, 
    - YGCT : GC 수행된 누적 시간
    - FGC : Old, Perm GC 횟수
    - FGCT : Old, Perm GC 수행 누적 시간
    - GCT : 수행 시간의 합(YGCT +FGCT)
  - -gccapacity
    - NGC : Young 크기
    - OGC : Old 영역 크기
    - PGC : Perm 크기 영역
    - S0C, S1C, EC, OC ,PC : survivor,  survivor1, Eden, Old, Perm 영역에 할당된 크기
    - MN, MX, C : Min, Max, Committed를 의미
    - YGC, FGC : Minor GC 횟수, Full GC 횟수
- CMS GC를 사용하면 수행시간이 다르다는 점
- 가장 좋은 것은 verbosegc 사용

### 원격으로 JVM 상황 모니터링 - jstatd

- 위 내용들은 로컬 시스템에서만 모니터링 가능

- jstatd를 동작 중지하면 어플리케이션이 작동되더라도 모니터링 불가

- 명령어

  ```
  jstatd [-nr] [-p port] [-n rminame]
  ```

  - nr : RMI registry가 존재하지 않을 경우 새로운 RMI 레지스트리를 jstatd 프로세스 내에서 시작하지 않는 것을 정의하기 위한 옵션
  - p : RMI 레지스트리를 식별하기 위한 포트번호
  - n : RMI 객체 이름을 지정한다 기본 이름은 JStatRemoteHost

- jstatd를 실행하면 에러 발생 - 보안문제 

  - lib/security/java.policy에 허가 명령어 추가

    ```
    grant codebase "file:${java.home}"/../lib/tools.jar" { 
    permission java.security.AllPermissionl;
    };
    ```

- 실행 명령어 예시

  ```
  jstatd -J-Djava.security.policy=all.policy -p 2020
  ```



### verbosegc 옵션으로 gc 로그 남기기

- 자바 수행시 -verbosegc 옵션을 추가하면 된다.
- [GC 8128K -> 848K(130112K), 0.0090257 secs]
  -  Young 영역에 마이너 GC가 발생 8,128kbyte에서 848kbyte로 축소
  -  전체 할당된 크기는 130,112kbyte
  -  GC수행 시간은 0.0090257초
- 언제 GC가 발생했는지 알 수 없음
  - -XX:+PrintGCTimeStampls 옵션을 적용
- 좀 더 상세한 정보들을 확인하고 싶다면
  - -XX:+PrintHeapAtGC
- 간결하고 보기 좋은 옵션
  - -XX:+PrintGCDetails



### 메모리릭 

- 실제로 1%도 안되는 현상
- Young과 Old 영역의 비율은 1:2 ~ 1:9
- 메모리릭을 확인하려면 Full GC 이후 메모리 사용량을 확인(80% 이상인지)



### GC 튜닝 꼭 해야할까?

- 운영 중인 자바 시스템에 기본적으로 다음과 같이 적용되어있어야한다
  - -Xms, -Xmx
  - -server 옵션
  - 타임아웃 관련 로그
    - DB 작업과 관련된 타임아웃
    - 다른 서버와의 통신시 타임아웃
- 다음과 같은 문제가 있다면 GC 튜닝을 하는 것이 좋다
  - JVM 메모리 크기도 지정하지 않았으며
  - Timeout이 지속적으로 발생한다면
- 가장 중요한 건 객체 생성을 줄이는 작업을 진행해야한다
  - StringBuilder, StringBuffer 사용
  - 로그를 최대한 적게 쌓도록 하기
- GC 튜닝의 목적
  - Old 영역으로 넘어가는 객체를 최소화
  - Full GC 실행시간을 줄이는 것



### Old 영역으로 넘어가는 객체 수 최소화

- Old영역의 GC는 New 영역에 비해 시간이 오래 소요된다.
- New 영역의 크기를 조절하면 가능하다

### Full GC 시간 줄이기

- Full GC는 Young GC에 비해 길다
- Full GC실행이 오래 소요되면 타임아웃 발생 가능성이 높다
- Old 영역의 크기를 적절하게 잘 적용해야한다



### 성능에 영향을 주는 옵션들

| 구분             | 옵션              | 설명                             |
| ---------------- | ----------------- | -------------------------------- |
| 힙의 영역의 크기 | -Xms              | JVM 시작히 힙 영역의 크기        |
|                  | -Xmx              | 최대 힙 영역 크기                |
| New 영역의 크기  | -XX:NewRatio      | New 영역과 Old 영역의 비율       |
|                  | -XX:NewSize       | New 영역의 크기                  |
|                  | -XX:SurvivorRatio | Eden 영역과 Survivor 영역의 비율 |

- -Xms, -Xmx,  NewRaio 옵션에 따라 GC 성능차이가 발생될 가능성이 높음



### GC 방식에 따른 옵션

| 구분                   | 옵션                                                         | 비고 |
| ---------------------- | ------------------------------------------------------------ | ---- |
| Serial GC              | -XX:+UseSerialGC                                             |      |
| Parallel GC            | -XX:+UseParallelGC<br />-XX:ParallelGCThreads=value          |      |
| Parallel Compacting GC | -XX:+UIseParalllelOldGC                                      |      |
| CMS GC                 | -XX:+UseConcMarkSweepGC<br />-XX:+UseParNewGC<br />-XX:+CMSParallelRemarkEnabled<br />-XX:CMSInitiatingOccupancyFranction=value<br />-XX:+UseCMSInitiatingOccupancyOnly |      |
| G1                     | -XX:+UnlockExperimentalVMoptions<br />-XX:+UseG1GC           |      |



### GC 튜닝절차

- GC 상황 모니터링
- 모니터링 결과 분석 후 튜닝여부 결정
  - GC 수행 시간이 1~3초 또는 10초가 넘어가면 GC 튜닝진행
  - 1~2GB로 지정하고 OutOfMemoryError 발생 시 힙 덤프로 원인 확인
  - 힙 덤프는 jmap 명령으로 생성가능하다 운영중에는 사용금지(자바 프로세스가 멈춤)
- GC 방식/메모리 크기 지정
- 결과분석
- 결과 만족 시 반영