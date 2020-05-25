# MySQL 설치 후 설정



## mysql 서버에 접속하기

로컬 MySQL 서버에 접속
> mysql -u root -p
> Enter password: 암호입력



원격 MySQL 서버에 접속
> mysql -h 서버주소 -u root -p
> Enter password: 암호입력



## mysql root 암호 변경
> alter user 'root'@'localhost' identified by '1111';



## MySQL 사용자 추가
> CREATE USER '사용자아이디'@'서버주소' IDENTIFIED BY '암호';



로컬에서만 접속할 수 있는 사용자를 만들기:
> CREATE USER 'study'@'localhost' IDENTIFIED BY '1111';
> => 이 경우 stidu 사용자는 오직 로컬(서버를 실행하는 컴퓨터)에서만 접속 가능한다.
> => 다른 컴퓨터에서 실행하는 MySQL 서버에 접속할 수 없다는 것을 의미한다.



원격에서만 접속할 수 있는 사용자를 만들기: 
> CREATE USER 'study'@'%' IDENTIFIED BY '1111';
> => 이 경우 study 사용자는 원력에서만 접속 가능



## MySQL 사용자 목록 조회
> select user from 데이터베이스명.테이블명;
> select user from mysql.user;



## MySQL 데이터베이스 생성
> CREATE DATABASE 데이터베이스명
> DEFAULT CHARACTER SET utf8
> DEFAULT COLLATE utf8_general_ci;



```sql
CREATE DATABASE studydb
DEFAULT CHARACTER SET utf8
DEFAULT COLLATE utf8_general_ci;
```



## MySQL 사용자에게 데이터베이스 사용 권한 부여
> GRANT ALL ON 데이터베이스명.* TO '사용자아이디'@'서버주소';
> GRANT ALL ON studydb.* TO 'study'@'localhost';



## 데이터베이스 목록 조회
> show databases;



## 사용자 교체
> quit    (프로그램 종료 후)
> mysql -u study -p   (다시 실행)



## 기본으로 사용할 데이터베이스 지정하기
> use 데이터베이스명
> use studydb;



## 데이터베이스의 전체 테이블 목록 조회
> show tables;



## 테이블 생성

>  create table ex_students (
>     uid varchar(20) not null,
>     pwd varchar(20) not null,
>     name varchar(20) not null,
>     tel varchar(20),
>     email varchar(50) not null,
>     work char(1) not null,
>     byear int,
>     schl varchar(30)
>   );



## 테이블 제약조건

[테이블의 데이터를 구분하는 식별자(Primary Key; PK)로 사용할 컬럼 지정]
> alter table 테이블명
> add primary key (컬럼명, 컬럼명, ...);



> alter table ex_contacts
> add primary key (email);



> alter table ex_students
> add primary key (uid);

=> PK는 아니지만 중복되어서는 안되는 컬럼인 경우 다음과 같이 유니크 컬럼으로 지정한다.
  alter table ex_students
  add unique index (email);



[테이블 삭제]

> drop table 테이블명;
> drop table ex_contacts;



[테이블의 상세 정보 출력]
> desc 테이블명;
> describe 테이블명;
> desc ex_contacts;
