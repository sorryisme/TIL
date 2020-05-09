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
-  

