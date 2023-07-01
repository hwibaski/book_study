# 02. 설치와 설정

## 2.1.1 버전과 에디션 선택

MySQL 서버의 버전을 선택할 때는 다른 제약 사항이 없다면 가능한 최신 버전을 설치하는 것이 좋다.

다만 특정한 기능이 필요하다면 그 버전을 선택하면 된다.

기존 버전에서 새로운 메이저 버전으로 업그레이드 하는 경우라면 최소 패치 버전이 15~20번 이상 릴리즈된 버전을 선택하는 것이 안정적인 서비스에 도움을 줄 것이다.

> MySQL 버전이 8.0 이라면 8.0.15 이후의 버전을 선택하는 것을 권장

초기의 MySQL 에디션 간의 차이점은 기술 지원의 차이만 있었다. 하지만 5.5 버전부터는 기능이 달리지면서 소스코드도 달라졌고, 엔터프라이즈 에디션의 소스코드는 더 이상 공개되지 않는다.

하지만 핵심 기능은 거의 차이가 없으며 아래의 부가적인 기능과 서비스들이 엔터프라이즈 버전과 차이가 난다.

- Thread Pool
- Enterpise Audit
- Enterprise TDE (Master Key 관리)
- Enterprice Authentication
- Enterprice Firewall
- Enterprice Monitor
- Enterprice Backup
- MySQL 기술지원

저자의 경험상 Percona에서 출시하는 Percona Server 백업 및 모니터링 도구 또는 Percona Server에서 지원하는 플러그인을 활용하면 커뮤니티 에디션의 부족한 점을 메꿀 수 있다.

## innodb_fast_shutdown

MySQL 서버에서는 실제 트랜잭션이 정상적으로 커밋돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일(리두 로그)에만 기록돼 있을 수 있다

아래와 같이 옵션을 변경하면 MySQL 서버가 종료될 때 모든 커밋된 내용을 데이터 파일에 기록하고 종료하게 할 수도 있다
```text
SET GLOBAL innodb_fast_shutdown=0;
SHUTDOWN; (원격으로 MySQL 서버 종료 시) or 종료명령어 (e.g. systemctl stop mysqld.service) 
```

## 서버 연결 테스트

```text
// 1. 소켓 파일을 이용해 접속
$ mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock

// 2. TCP/IP
$ mysql -uroot -p --host=127.0.0.1 --port=3306

// 3. 소켓 파일을 사용
$ mysql -uroot -p
```
- 원격 호스트에 있는 MySQL 서버에 접속할 때는 반드시 두 번째 방법을 사용해야 한다.
- 호스트를 localhost로 명시하는 것과 127.0.0.1로 명시하는 것이 각각 의미가 다르다.
  - --host=localhost 옵션을 사용하면 소켓 파일을 통해 접속 -> 유닉스의 IPC(프로세스 간 통신)의 일종
  - 127.0.0.1 을 사용하는 경우에는 자기 서버를 가리키는 루프백 IP이기는 하지만 TCP/IP 통신 방식을 사용

## telnet & nc

때로는 MySQL 서버에 직접 접속하지 않고 접속가능 여부만 확인하고 싶을 수도 있다. 이 때는 telnet이나 nc 명령을 사용할 수 있다.

```text
telent 10.2.40.61 3306

nc 10.2.40.61 3306
```

## 2.3 MySQL 서버 업그레이드
1. 인플레이스 업그레이드 방법
   - MySQL 서버의 데이터 파일을 그대로 두고 업그레이드 하는 방법 
   - 장점 : 업그레이드 시간 단축
   - 단점 : 여러 가지 제약 사항이 있다.
2. 논리적 업그레이드
   - mysqldump 도구 등을 통해 데이터를 다른 파일로 덤프한 후 새로 업그레이드된 버전의 서버에 적재하는 방법
   - 장점 : 제약 사항이 거의 없다
   - 단점 : 시간이 많이 소요된다.(데이터의 양에 따라 상이)

### 2.3.1 인플레이스 업그레이드 제약 사항
- 동일 메이저 버전에서 마이너 버전 간 업그레이드는 대부분 데이터 파일의 변겨 없이 가능
  - e.g. 8.0.16 -> 8.0.21
- 메이저 버전 간 업그레이드는 대부분 크고 작은 데이터 파일의 변경 필요하기 때문에 반드시 직전 버전에서만 업그레이드 허용된다.
  - e.g. 5.5 -> 5.6 (가능), 5.5 -> 5.7 (불가), 5.5 -> 8.0 (불가)
  - why? 직전 버전에서 사용하던 데이터 파일과 로그 포맷만 패치를 지원하기 때문에
  - 5.1에서 8.0으로 업그레이드를 원한다면 한 단계씩 메이저 버전을 업그레이드 해나가야 한다.
  - 혹은 논리적 업그레이드가 더 효율적일 수 있다.

## 2.4 서버 설정
1. 일반적으로 MySQL 서버는 단 하나의 설정 파일을 사용
   - 리눅스 포함 유닉스 계열 : my.cnf
   - 윈도우 계열 : my.ini
2. MySQL 서버는 시작될 때만 이 설정 파일을 참조한다.
   3. 이 설정 파일의 경로가 고정돼 있는 것은 아니다. 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 my.cnf 파일을 사용하게 된다. 1,2,4 번은 mysql이 공통적으로 검색하는 경로이며, 3번 파일은 MySQL을 설치하는 방법에 따라 다를 수 있다.
      1. /etc/my.cnf
      2. /etc/mysql/my.cnf
      3. /usr/etc/my.cnf (내 장비에는 /opt/homebrew/etc/my.cnf)
      4. ~/.my.cnf

```text
설정 파일 순회 위치 찾는 명령어

mysqld --verbose --help
mysql --help
```

### 2.4.1 설정 파일의 구성
MySQL 설정 파일은 하나의 파일에 여러 개의 설정 그룹을 당믈 수 있으며, 대체로 실행 프로그램 이름을 그룹명으로 사용한다.

```text
[mysqld]
socket = /usr/local/mysql/tmp/mysql.sock
port = 3306

(...중략)

[mysqldump]
default-character-set = utf8mb4
socket = /usr/local/mysql/tmp/mysql.sock
port = 3305
```

설정 파일의 각 그룹은 같은 파일을 공유하지만 서로 무관하게 적용된다.

### 2.4.2 시스템 변수의 특징

MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화, 접속된 사용자를 제어하기 위해 값을 별도로 저장해둔다.
이렇게 저장된 값을 시스템 변수라고 한다.

```text
글로벌 변수 확인
mysql> SHOW GLOBAL VARIABLES;
```

시스템 변수 값이 어떻게 MySQL 서버와 클라이언트에 영향을 미치는지 판단하려면 각 변수가 글로벌 변수인지 세션 변수인지 구분할 수 있어야 한다.
아래의 시스템 변수를 나타낸 표이다. 실제로는 훨씬 많은 시스템 변수가 있다. 

| Name                        | Cmd-Line | Option File | System Var | Var Scope | Dynamic |
|-----------------------------|----------|-------------|------------|-----------|---------|
| activate_all_roles_on_login | Yes      | Yes         | Yes        | Global    | Yes     |
| admin_address               | Yes      | Yes         | Yes        | Global    | No      |
| admin_port                  | Yes      | Yes         | Yes        | Global    | No      |
| time_zone                   |          |             | Yes        | Both      | Yes     |
| sql_log_bin                 |          |             | Yes        | Session   | Yes     |

- Cmd-Line : MySQL 서버의 커맨드라인으로 설정될 수 있는지 여부
- Option File : MySQL 설정 파일인 my.cnf로 제어할 수 있는지 여부
- System Var : 시스템 변수인지 아닌지 여부
- Var Scope : 시스템 변수의 적용 범위. Global(전역), MySQL 서버와 특정 클라이언트 간의 커넥션(Session), 그리고 두 개 모두 해당(Both)
- Dynamic : 시스템 변수가 동적인지 정적인지 구분

### 2.4.3 글로벌 변수와 세션 변수

MySQL의 시스템 변수는 적용 범위에 따라 글로벌 변수와 세션 변수로 나뉜다.

1. 글로벌 변수
   - 하나의 MySQL 서버 인스턴스에 전체적으로 영향을 미치는 시스템 변수
   - 주로 MySQL 서버 자체에 관련된 설정이 많다.
2. 세션 변수
   - MySQL 클라이언트가 MySQL 서버에 점속할 때 기본으로 부여하는 옵션의 기본값을 제어하는데 사용
   - 각 클라이언트가 별도로 그 값을 변경하지 않는 경우에는 기본값 적용
   - 클라이언트의 필요에 따라 개별 켜넥션 단위로 다른 값으로 변경할 수 있는 것이 세션 변수이다.
   - 세션 변수 가운데 설정 파일(my.cnf)에 명시할 수 설정할 수 있는 변수는 대부분의 범위가 Both라고 명시돼 있다. 그리고 순수하게 Session인 변수는 설정 파일에 초깃값을 명시할 수 없으며, 커넥션이 만들어지는 순간부터 해당 커넥션에서만 유효한 설정 변수를 의미한다.

### 2.4.4 정적 변수와 동적 변수

MySQL 서버의 시스템 변수는 MySQL 서버가 기동 중인 상태에서 변경 가능한지에 따라 동적 변수와 정적 변수로 구분된다.

MySQL 서버의 시스템 변수는 두 가지 방법으로 설정을 변경할 수 있다.
1. 디스크에 저장되어 있는 my.cnf를 통한 설정
2. 기동 중인 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우

1번 방법은 설정 파일의 내용을 변경하더라도 서버가 재시작하기 전까지는 그 내용이 반영되지 않는다. 하지만 SET 명령을 이용해 값을 바꿀 수 있다.
만약 변수명을 정확히 모른다면 아래와 같은 방법을 사용할 수 있다.

```text
SHOW GLOBAL VARIABLES LIKE '%max_connections%';

SET GLOBAL max_connections=500;
```

>  SET 명령을 통해 변경되는 시스템 변수는 my.cnf에 반영되는 것이 아니므로 서버 재시작 시 날아가버린다. 재시작 시에도 적용하려면 my.cnf 파일에도 명시하거나 변경을 해야한다.
>  이 작업은 잊어버릴 수도 있고 서버가 재시작할 때까지 기다려야하기 때문에 누락되기 쉽다. MySQL 8.0 버전 부터는 SET PERSIST 명령을 지원해준다.

GLOBAL 키워드를 빼면 자동으로 세션 변수를 조회하고 변경한다.

일반적으로 글로벌 변수는 서버의 기동 중에는 변경할 수 없는 것이 많지만 실시간으로 변경할 수 있는 것도 있다.

시스템 변수의 번위가 'Both'인 경우에는 글로벌 시스템 변수의 값을 변경해도 이미 존재하는 커넥션의 세션 변숫값은 변경되지 않고 그대로 유지된다.

### 2.4.5 SET PERSIST

SET PERSIST를 사용하면 변경된 값을 즉시 적용함과 동시에 별도의 설정 파일(mysqld-auto.cnf)에 변경 내용을 추가로 기록해 둔다.

서버가 재시작 시 my.cnf와 mysqld-auto.cnf를 같이 참조해서 시스템 변수를 적용한다.

세션 변수에는 적용되지 않으며 자동으로 GLOBAL 시스템 변수의 변경으로 인식하고 변경한다.

현재 기동중인 서버에는 변경 내용을 적용하지 않고 다음 재시작을 위해 mysqld-auto.cnf 파일에만 변경내용을 기록해두고자 한다면 PERSIST_ONLY 명령을 사용하면 된다.

정적인 변수의 값을 영구적으로 변경하고자 할 때도 사용할 수 있다.

SET PERSIST나 SET PERSIST_ONLY로 추가된 시스템 변수를 삭제해야할 때도 있다. 물론 mysqld-auto.cnf 파일을 직접 수정할 수도 있지만 오타 입력시 서버 시작 시 문제가 될 수 있으므로 아래의 명령어를 이용하자
```text
특정 시스템 변수 삭제
RESET PERSIST max_connections;
RESET PERSIST max_connections IF EXISTS max_connections;

mysqld-auto.cnf 파일의 모든 시스템 변수를 삭제
RESET PERSIST;
```


## 기타
- 설치 위치
```text
책에 나온 내용
/usr/local/mysql

Homebrew 설치 시
/opt/homebrew/var/mysql
```

- 초기 설치 시 root 계정의 비밀번호를 설정할 수도 있고, 안 할 수도 있다.
- 서버 시작 명령어
```text
Homebrew 이용해서 데몬으로 실행하기
brew services start mysql

책에서 나온 내용
/usr/local/mysql/support-files/mysql.server start
```

- 서버 종료 명령어
```text
데몬 형태로 실행되고 있는 MySQL 종료하기
brew services stop mysql

책에서 나온 내용
/usr/local/mysql/support-files/mysql.server stop
```

- 서버 접속 명령어
```text
mysql -u root
```

- 서버 재시작 명령어
```text
brew services restart mysql
```

- 설정 파일 위치
```text
Homebrew 설치 시
opt/homebrew/etc/my.cnf

책에서 나용 내용
/usr/local/mysql/my.cnf
```

- 데이터베이스 목록확인
```text
SHOW DATABASES;
```



## 개인적으로 검색한 내용

- [맥 OS에서 설치방법](https://velog.io/@inyong_pang/MySQL-%EC%84%A4%EC%B9%98%EC%99%80-%EC%B4%88%EA%B8%B0-%EC%84%A4%EC%A0%95)
- [Homebrew에서 MySql](https://bsscl.tistory.com/5)


