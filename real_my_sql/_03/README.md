# 03. 사용자 및 권한

MySQL은 다른 DBMS와 조금 다르게 해당 사용자가 어느 IP에서 접속하고 있는지도 확인한다.

또한 8.0버전부터는 역할(Role)의 개념이 도입됐기 때문에 각 사용자의 권한으로 미리 준비된 권한 세트를 부하여하는 것도 가능하다.

## 3.1 사용자 식별

- 사용자의 계정뿐 아니라 사용자의 접속 지점 (클라이언트가 실행된 호스트명이나 도메인 또는 IP주소)도 계정의 일부가 된다.
- 아이디와 IP 주소를 감싸는 역따옴표(`)는 MySQL에서 식별자를 감싸는 따옴표 역할을 하는데, 이는 종종 홑따옴표(')로 바뀌어서 사용되기도 한다.
- 다음의 사용자 계정은 test_user 라는 아이디를 가진 사용자는 로컬호스트 환경에서만 접속이 가능하고 다른 IP에서는 test_user라는 아이디로 접속할 수 없다.

```text
'test_user'@'127.0.0.1'
```

- 모든 IP에서 접속 가능하게 하려면 IP주소 부분에 '%'를 적어주면된다.

```text
'test_user'@'%'
'test_user'@'192.168.0.10'
```

하지만 위와 같이 두 개의 계정이 있을 때 더 자세하게 명세된 정보를 가지고 계정을 판별한다. 즉, IP 주소가 명시된 계정을 대상으로만 판별하기 때문에 주의해야한다.

## 3.2 사용자 계정 관리

- MySQL 8.0 부터는 `SYSTEM_USER` 권한을 가지고 있느냐에 따라 시스템 계정과 일반 계정으로 구분됨
- 시스템 계정은 데이터베이스 서버 관리자를 위한 계정이며, 일반 계정은 응용 프로그램 또는 개발자를 위한 계정이라고 이해하면 됨.

시스템 계정만이 가지는 권한
- 계정 관리(계정 생성 및 삭제, 그리고 계정의 권한 부여 및 제거)
- 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

MySQL 서버에는 내장된 계정들이 있는데 'root@localhost'를 제외한 3개의 계정은 내부적으로 사용되므로 삭제하지 않도록 주의, 아래의 계정은 애초에 잠겨 있기 때문에 의도적으로 잠금을 풀지 않는 한 악의적인 용도로 사용할 수 없으므로 보안 걱정을 하지 않아도 된다.

- 'mysql.sys'@'localhost' : 8.0부터 기본으로 내장된 sys 스키마의 객체들(뷰, 함수, 프로시저)의 DEFINER로 사용되는 계정
- 'mysql.session'@'localhost' : MySQL 플러그인이 서버로 접근할 때 사용되는 계정 
- 'mysql.infoschema'@'localhost' : information_schema에 정의된 뷰의 DEFINER로 사용되는 계정

###  3.2.2 계정 생성

5.7 버전까지는 GRANT 명령으로 권한의 부여와 동시에 계정 생성이 가능했다. 하지만 8.0버전부터는 계정의 생성은 `CREATE USER`, 권한 부여는 `GRANT`로 분리

계정 생성
- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

```mysql
CREATE USER 'user'@'%'
  IDENTIFIED WITH 'mysql_natvie_password' BY 'password'
  REQUIRE NONE
  PASSWORD EXPIRE INTERVAL 30 DAY
  ACCOUNT UNLOCK 
  PASSWORD HISTORY DEFAULT 
  PASSWORD REUSE INTERVAL DEFAULT 
  PASSWORD REQUIRE CURRENT DEFAULT;
```

#### 3.2.2.1 IDENTIFIED WITH
사용자의 인증 방식과 비밀번호를 설정

비밀번호 인증 방식
1. Native Pluggable Authentication : 5.7까지의 기본 방식
2. Caching SHA-2 Pluggable Authentication : 8.0버전의 기본 인증 방식
3. PAM Pluggable Authentication
4. LDAP Pluggable Authentication

Chacing SHA-2 방식은 SSL 또는 RSA 키페어를 필요로 하기 때문에 기존의 방식과 다르게 접속해야 한다.
그래서 보안 수준은 좀 낮아지겠지만 기존 버전과의 호환성을 고려한다면 Native Authentication 인증 방식으로 계정을 생성해야할 수도 있다.
8.0에서도 Native 방식으로 인증 방식을 설정하려면 다음과 같이 설정을 변경하거나 my.cnf 파일에 추가하면 된다.

```mysql
SET GLOBAL default_authentication_plugin="mysql_native_password"
```

`ALTER USER` 명령을 통해 기생성된 계정의 옵션으 변경할 수 있다.

#### 3.2.2.2 REQUIRE

- 계정이 DB 서버에 접속할 때 SSL/TLS 채널을 사용할지 여부를 설정
- Caching SHA-2 Authentication은 REQUIRE 옵션을 비활성화 해도 암호화된 SSL 채널을 이용한다.

#### 3.2.2.3 PASSWORD EXPIRE

- 비밀번호의 유효 기간을 설정
- 별도로 명시하지 않으면 default_password_lifetime 시스템 변수에 저장된 기간으로 유효 기간 설정
- 개발자나 관리자의 비밀번호 유효기간 설정은 보안상 안전하지만 응용 프로그램 접속용 계정에 유효 기간을 설정하는 것은 위험할 수 있으니 주의!

사용 가능 옵션
- `PASSWORD EXPIRE` : 계정 생성과 동시에 비밀번호의 만료 처리
- `PASSWORD EXPIRE NEVER` : 만료기간 없음
- `PASSWORD EXPIRE DEFAULT` : default_password_lifetime 시스템 변수에 저장된 기간으로 비밀번호의 유효 기간을 설정
- `PASSWORD EXPIRE INTERVAL n DAY` : 비밀번호의 유효기간을 오늘로부터 n일자로 설정

#### 3.2.2.4 PASSWORD HISTORY

- 한 번 사용했떤 비밀번호를 재사용하지 못하게 설정하는 옵션

사용 가능 옵션
- `PASSWORD HISTORY DEFAULT` : password_history 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하며, 저장된 이력에 남아있는 비밀번호를 재사용할 수 없다.
- `PASSWORD HISTORY n` : 비밀번호의 이력을 최근 n개까지만 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.

한 번 사용했던 비밀번호를 사용하지 못하게 하려면 이전에 사용했던 비밀번호를 MySQL 서버가 기어갷야 한다. 이를 위해 MySQL 서버는 password_history 테이블 사용

```mysql
SELECT * FROM mysql.password_history;
```

#### 3.2.2.5 PASSWORD REUSE INTERVAL

한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션

사용 가능 옵션 
- `PASSWORD REUSE INTERVAL DEFAULT` : password_reuse_interval 시스템 변수에 저장된 기간으로 설정
- `PASSWORD REUSE INTERVAL n DAY` : n일자 이후에 비밀번호를 재사용할 수 있게 설정

#### 3.2.2.6 PASSWORD REQUIRE

비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호(변경하기 전 만료된 비밀번호)를 필요로 할지 말지를 결정하는 옵션

사용 가능 옵션
- `PASSWORD REQUIRE CURRENT` : 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- `PASSWORD REQUIRE OPTIONAL` : 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- `PASSWORD REQUIRE DEFAULT` : password_require_current 시스템 변수의 값으로 설정

#### 3.2.2.7 ACCOUNT LOCK / UNLOCK

계정 생성 시 또는 ALTER `USER` 명령을 이용할 때 계정을 사용하지 못하게 잠글지 여부를 결정

사용 가능 옵션
- `ACCOUNT LOCK` : 계정 잠금
- `ACCOUNT UNLOCK` : 잠긴 계정을 다시 사용 가능 상태로 잠금 해제

## 3.3 비밀번호 관리

### 3.3.1 고수준 비밀번호

비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 있다.

validate_password 컴포넌트 이용 (설치 필요)

```mysql
# validate_password 컴포넌트 설치
INSTALL COMPONENT 'file://component_validate_password';

# 설치된 컴포넌트 확인
SELECT * FROM mysql.component;
```

```mysql
SHOW GLOBAL VARIABLES LIKE 'validate_password%';
```
| Variable_name                        | Value  | 비고             |
|--------------------------------------|--------|----------------|
| validate_password.check_user_name    | ON     |                |
| validate_password.dictionary_file    |        |                |
| validate_password.length             | 8      | 비밀번호 길이        |
| validate_password.mixed_case_count   | 2      | 포함되어야할 대문자의 수  |
| validate_password.number_count       | 2      | 포함되어야할 숫자의 수   |
| validate_password.policy             | STRONG | 비밀번호 정책        |
| validate_password.special_char_count | 2      | 포함되어야할 특수문자의 수 |

비밀번호 정책은 크게 다음 3가지중 선택
- LOW : 비밀번호 길이만 건증
- MEDIUM : 비밀번호 길이를 검증하며, 숫자와 대소문자, 그리고 특수문자의 배합을 검증
- STROING : MEDIUM 레벨의 검증을 모두 수행, 금칙어가 포함됐는지 여부까지 검증

### 3.3.2 이중 비밀번호(Dual Password)

다수의 응용 프로그램 서버들이 데이터베이스 서버를 이용하기 때문에 데이터베이스 계정 정보는 다수의 서버에서 공용으로 사용하는 경우가 많다.
이러한 특성상 데이터베이스 서버의 계정 정보는 쉽게 변경하기 어려운데, 그중에서도 데이터베이스 계정 비밀번호는 서비스가 실행중인 상태에서 변경이 불가능했다.

이 문제를 해결하기 위해 MySQL 8.0 버전부터는 계정의 비밀번호로 2개의 값을 동시에 사용할 수 있는 기능을 추가했다.

- Primary Password : 최근 설정된 비밀번호
- Second Password : 이전 비밀번호

이중 비밀번호를 사용하려면 다음과 같이 기본 비밀번호 변경 구문에 RETAIN CURRENT PASSWORD 옵션만 추가하면 된다.

```mysql
# 단순 비밀번호 변경
ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password';

# 이전 비밀번호를 Second Password로 설정하면서 현재 비밀번호 변경
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
```


> 비밀번호 변경 후 세컨더리 비밀번호를 꼭 삭제해야하는 것은 아니지만 계정의 보안을 위해 삭제하는 것이 좋다.

```mysql
ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```


## 3.4 권한

5.7 버전까지 권한은 글로벌 권한과 객체 단위의 권한으로 구분

- 글로벌 권한 : 데이터베이스나 테이블 이외의 객체에 적용되는 권한
  - GRANT 명령에서 특정 객체를 명시하지 말아야 한다.
- 데이터베이스나 테이블을 제어하는 데 필요한 권한
  - GRANT 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야 한다.

> 예외적으로 ALL은 글로벌과 객체 권한 두 가지 용도로 모두 사용 가능, 특정 객체에 ALL 권한이 부여되면 해당 객체에 적용될 수 있는 모든 객체 권한 부여, 글로벌로 ALL이 사용되면 글로벌 수준에서 가능한 모든 권한 부여

사용자에게 권한을 부여할 때는 GRANT 명령을 사용한다.

```mysql
GRANT 부여할 권한들 ON db.table TO 'user'@'host';
```

- MySQL 8.0 부터는 존재하지 않는 사용자에 대해 GRANT 명령이 실행되면 에러가 발생하므로 반드시 사용자를 먼저 생성하고 GRANT 명령으로 권한 부여
- `GRANT OPTION`(권한 이름) 권한은 다른 권한과 달리 명령의 마지막에 WITH GRANT OPTION을 명시해서 부여한다.

#### 글로벌 권한 부여 예시 (ON 절에 항상 *.*을 명시, *.*은 모든 오브젝트를 포함해서 서버 전체를 의미)
```mysql
GRANT SUPER ON *.* TO 'user'@'localhost';
```

#### DB 권한
```mysql
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';
```

권한 범위가 DB인 경우에는 DB 권한에도 ON 절에 db.table 에서 table 이름을 명시할 수 없다.

테이블 권한은 모든 DB에 대해 권한을 부여하는 것도 가능하며 특정 DB의 오브젝트에 대해서만 권한을 부여하는 것도 가능하다. 그리고 특정 DB의 특정 테이블에 대해서만 권한을 부여하는 것도 가능하다

#### 테이블 권한
```mysql
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.departement TO 'user'@'localhost';
```

#### 컬럼 권한

GRANT 문법이 조금 달려져야함.

컬럼에 부여할 수 있는 권한은 DELETE를 제외한 INSERT, UPDATE, SELECT이다.

```mysql
GRANT SELECT, INSERT, UPDATE(컬럼 이름) ON db.table TO 'user'@'localhost';
```

> 테이블이나 컬럼단위의 권한은 잘 사용하지 않는다. 하나의 컬럼에만 권한을 설정해도 나머지 모든 테이블의 모든 컬럼에 대해서도 권한 체크를 하기 때문에 성능이 저하될 수 있음
> 꼭 필요하다면 권한을 허용하고자 하는 컬럼만으로 별도의 뷰를 만들어 사용하는 방법도 있다.

#### 권한 확인
각 계정이나 권한에 부여된 권한이나 역할을 확인하기 위해서는 `SHOW GRANTS` 명령을 사용할 수도 있지만 표현태로 깔깜흐게 보고자 한다면 아래의 테이블들을 조회해보자

| 구분    | 테이블                 | 설명                            |
|-------|---------------------|-------------------------------|
| 정적 권한 | mysql.user          | 계정 정보 & 계정이나 역할에 부여된 글로벌 권한   |
| 정적 권한 | mysql.db            | 계정이나 역할에 DB 단위로 부여된 권한        |
| 정적 권한 | mysql.table_priv    | 계정이나 역할에 테이블 단위로 부여된 권한       |
| 정적 권한 | mysql.colums_priv   | 계정이나 역할에 컬럼 단위로 부여된 권한        |
| 정적 권한 | mysql.procs_priv    | 계정이나 역할에 스토어드 프로그램 단위로 부여된 권한 |
| 동적 권한 | mysql.global_grants | 계정이나 역할에 부여되는 동적 글로벌 권한       |


## 3.5 역할

8.0부터는 권한을 묶어서 역할을 사용할 수 있게 됐다. 실제 MySQL 서버 내부적으로는 역할은 계정과 똑같은 모습을 하고 있다.

```mysql
# 역할 생성 -> 빈 껍데기인 역할만 정의
CREATE ROLE 
  role_emp_read,
  role_emp_write;

# role_emp_read 역할에 SELECT 권한 부여
GRANT SELECT ON employees.* TO role_emp_read;

# role_emp_write 역할에 INSERT, UPDATE, DELETE 권한 부여
GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;

# reader라는 유저에게 role_emp_read 역할 부여
GRANT role_emp_read TO reader@'localhost';

# writer라는 유저에세 role_emp_write 역할 부여
GRANT role_emp_write TO writer@'localhost';
```

> 기본적으로 역할은 그 자체로 사용될 수 없고 계정에 부여해야 한다. 

> 주의! 역할을 부여하고 SHOW GRANTS 명령으로 계정에 가진 역할이 잘 적용되었더라도 바로 적용되지 않는다.

reader 계정이 role_emp_read 역할을 사용할 수 있게 하려면 다음과 같이 SET ROLE 명령을 실행해 해당 역할을 활성해야 한다.

```mysql
SET ROLE 'role_emp_read';

SELECT current_role();
```

계정이 로그아웃 됐다가 다시 로그인하면 역할이 활성화되지 않은 상태로 초기화돼 버린다.

아래와 같이 설정을 바꾸면 로그인과 동시에 부여된 역할이 자동으로 활성화 된다.
```mysql
SET GLOBAL activate_all_roles_on_login=ON;
```

> 역할은 기본적으로 사용자 계정과 같은 테이블에 기록된다. 따라서 역할 자체도 host를 가질 수 있다.
> 하지만 역할에 host를 부여한다고 해서 역할에 부여된 host가 계정에 영향을 주지는 않는다. 
> e.g. role_emp_read@'localhost' 라고 해서 host가 'localhost'인 계정에만 적용되고 그런 것이 아니라는 뜻이다.

### 어차피 user 테이블에 기록되는데 왜 CREATE ROLE과 CREATE USER를 분리해놨을까?

데이터베이스 관리의 직무를 분리할 수 있게 해서 보안을 강화하는 용도로 사용할 수 있게 하기 위해서이다.
CREATE USER 명령을 실행가능하게 하고 CREATE ROLE은 실행하지 못하게 하는 등의 분리가 가능해진다.
생성된 ROLE은 account_locked 컬럼이 Y로 잠겨있기 때문에 로그인 용도로 사용할 수  없다.

| 테이블                 | 설명                |
|---------------------|-------------------|
| mysql.deafult_roles | 계정별 기본 역할         |
| mysql.role_edges    | 역할에 부여된 역할 관계 그래프 |




