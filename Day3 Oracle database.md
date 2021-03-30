# 0326 오라클 강의내용

SELECT 조회컬럼 연산자 별칭 * DISTINCT 함수 그룹함수

FROM  테이블명(SUBQUERY)

WHERE 컬럼명 연산자 값

GROUP BY 그룹함수_컬럼명

HAVING 그룹함수조건식 연산자값

ORDER BY 정렬기준 컬럼명 인덱스 별칭 ASC|DESC

### 함수

| 그룹     | sum avg stdev varianc --> 숫자<br/>count max min --> 숫자 문자 날짜<br/>count(*) --> null 값 포함 갯수 |
| -------- | ------------------------------------------------------------ |
| 타입변환 | cast (123 as date\|number\|varchar2)<br />to_char\|to_date\|to_number<br />to_char (sysdate, 'yyyy/mm/dd') |
| 문자     | upper lower initcap<br />length<br />instr , substr          |
| 숫자     | round trunc mod                                              |
| 날짜     | sysdate , add_months, months_between                         |
| NULL     | NVL(컬럼명, NULL대체값)                                      |
| 순위     | rownum - subquery<br />row_number() over (<br />partition by 순위를 정할 그룹 or<br />order by 순위를 정할 기준 컬럼<br />)<br />rank<br />dense_rank |

### CLOB, BLOB

* character | binary large object
* 최대 1tb

### join

* 두개의 테이블을 합치는 과정
* INNER JOIN 방법 (조건을 만족하는 것만 보여준다.)
* 예)
  * 사번, 사원명, 부서코드 조회 : employees 테이블로만 가능
  * 사번, 사원명, 부서명 조회 : employees + departments 테이블

```sql
SELECT EMPLOYEE_ID, FIRST_NAME, DEPARTMENT_ID
FROM EMPLOYEES
WHERE DEPARTMENT_ID IN (SELECT DEPARTMENT_ID, DEPARTMENT_NAME >> 불가능)

SELECT DEPARTMENT_ID, DEPARTMENT_NAME
FROM DEPARTMENT

# 방법
SELECT EMPLOYEE_ID, FIRST_NAME, EMPLOYEES.DEPARTMENT_ID, DEPARTMENTS.DEPARTMENT_ID
FROM EMPLOYEES, DEPARTMENTS
WHERE EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID 
// 각각 EMPLOYEES의 DEPARTMENT_ID, DEPARTMENTS의 DEPARTMENT_ID

// EMPLOYEES에 NULL데이터가 존재하는데 DEPARTMENTS에는 NULL값이 없다.
WHERE EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID(+) 
>> NULL 존재 = NULL X >> 부서코드가 NULL인 데이터가 존재하는데 기존처럼(+)를 사용하지 않으면 
출력이 되지 않는다. 이때 (+)를 추가하면 부서코드가 NULL인 사람도 보여준다.
>> 부서가 배정되지 않은 사원도 나타내준다.
>> 이를 OUTER JOIN 이라고 한다.

>> (+)표시를 왼쪽으로 이동하면?
WHERE EMPLOYEES.DEPARTMENT_ID(+) = DEPARTMENTS.DEPARTMENT_ID
>> DEPARTMENTS 테이블에 더 많은 부서가 존재해 사원이 없는 부서도 조회한다.
```

* 테이블에도 별칭을 줄 수 있다.

```sql
SELECT EMPLOYEE_ID, FIRST_NAME, E.DEPARTMENT_ID, D.DEPARTMENT_ID
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID 
```

* LEFT, RIGHT OUTER JOIN > 한쪽만 (+)표시를 붙일 수 있다.

```SQL
SELECT EMPLOYEE_ID, FIRST_NAME, EMPLOYEES.DEPARTMENT_ID, DEPARTMENTS.DEPARTMENT_ID
FROM EMPLOYEES, DEPARTMENTS
WHERE EMPLOYEES.DEPARTMENT_ID(+) = DEPARTMENTS.DEPARTMENT_ID
//
WHERE EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID(+)
```

* 문제 ) LONDON 도시에 근무하는 사원명, 부서명, 도시명 조회
  * 사원명 = employees 테이블
  * 부서명 = departments 테이블
  * 도시명 = locations 테이블

```SQL
SELECT FIRST_NAME 사원명, DEPARTMENT_NAME 부서명, CITY 도시명
FROM EMPLOYEES E, DEPARTMENTS D, LOCATIONS L
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
AND D.LOCATION_ID = L.LOCATION_ID
AND UPPER(CITY)='LONDON'
```

* 조인을 해야할 테이블이 3개면, 조건은 최소 2개
* 문제 ) IT관련 부서의 사원명, 부서명, 도시명 조회
  * (IT관련부서는 부서명에 IT포함)

```SQL
SELECT FIRST_NAME 사원명, DEPARTMENT_NAME 부서명, CITY 도시명
FROM EMPLOYEES E, DEPARTMENTS D, LOCATIONS L
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
AND D.LOCATION_ID = L.LOCATION_ID
AND UPPER(DEPARTMENT_NAME) LIKE '%IT%'
```

* 문제 ) 사원명 부서명 도시명 국가명 조회

```SQL
select first_name 사원명 , depart ment_name 부서명 city 도시명
from employees e, departments d, locations l
where e.department_id = d.department_id
and d.location_id = l.location_id
and UPPER(city)='LONDON'
```

* 문제 ) 내 사번, 내 이름, 상사 사번, 상사 이름 조회

```SQL
SELECT 사번, 이름, 상사사번
FROM 내정보, 이름, 상사사번
WHERE 내정보.상사사번 - 상사정보.사번;

SELECT ME.EMPLOYEE_ID 내사번, ME.FIRST_NAME 내이름, ME.MANAGER_ID 상사사번, MAN.EMPLOYEE_ID 상사사번2, MAN.FIRST_NAME 상사이름
FROM EMPLOYEES ME, EMPLOYEES MAN
WHERE ME.MANAGER_ID=MAN.EMPLOYEE_ID(+)
# 상사가 NULL > 존재 = NULL 존재 X
```

* SELF INNER JOIN/ OUTER JOIN

* 내 상사보다 급여를 많이 받는 사원의 이름 조회

```sql
SELECT ME.FIRST_NAME 내이름, ME.SALARY 내급여, ME.MANAGER_ID 상사사번, MAN.SALARY 상사급여
FROM EMPLOYEES ME, EMPLOYEES MAN
WHERE ME.MANAGER_ID=MAN.EMPLOYEE_ID
AND ME.SALARY>MAN.SALARY;
```

* JOIN > INNER, OUTER, CROSS, SELF JOIN
* CROSS JOIN 
* 표준JOIN - ANSI 조인
  * select - from A inner/(left-right)outer on B

```SQL
INNER JOIN 방법
select employee_id, first_name, e.department_id, department_name
from employees e inner join departments d
on e.department_id = d.department_id;


OUTER JOIN 방법
select employee_id, first_name, e.department_id, department_name
from employees e left outer join departments d
on e.department_id = d.department_id;

select employee_id, first_name, e.department_id, department_name
from employees e right outer join departments d
on e.department_id = d.department_id;


SELF JOIN 방법 (같은 테이블끼리 조인)
SELECT me.first_name 내이름, me.salary 내급여, me.manager_id 상사사번, 
man.salary 상사급여
FROM EMPLOYEES me inner join EMPLOYEES man
on me.manager_id=man.employee_id

```

* 오라클JOIN - A,B TABLE

```SQL
INNER JOIN 방법
select employee_id, first_name, e.department_id, department_name
from employees e, departments d
where e.department_id = d.department_id;


OUTER JOIN 방법
select employee_id, first_name, e.department_id, department_name
from employees e, departments d
where e.department_id = d.department_id(+);

select employee_id, first_name, e.department_id, department_name
from employees e, departments d
where e.department_id(+) = d.department_id;


SELF JOIN 방법
SELECT me.first_name 내이름, me.salary 내급여, me.manager_id 상사사번, 
man.salary 상사급여
FROM EMPLOYEES me, EMPLOYEES man
WHERE me.manager_id=man.employee_id
```

* join
* 테이블1.행 + 테이블2.행 >> 1개행으로 열 결합

### UNION

* 행 합침

| 사원 - 100행                   | 회원 - 200행            |
| ------------------------------ | ----------------------- |
| 사번(문자) 이름(문자)          | 아이디(문자) 이름(문자) |
| 회원 - 아이디(문자) 이름(문자) |                         |

* 합치려는 컬럼의 개수와 타입이 일치해야한다.
* 같은 구조의 두 테이블을 합칠 때 유용하다.

```sql
SELECT 사번, 이름 FROM 사원; > 100행
SELECT 아이디, 이름 FROM 회원; > 200행

SELECT 사번, 이름 FROM 사원
UNION
SELECT 아이디, 이름 FROM 회원;

```

* 예) 재난지원금 50번 부서나 급여 5000이하 급여를 받는 사원에게 지급

```SQL
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 50 OR SALARY <= 5000;


SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 50 
UNION
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE SALARY <= 5000;
>> 위와 동일한 결과를 반환하나 문장이 오히려 어렵다
>> 중복값을 제외하지 않고싶다면? (중복자 2번)

SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 50 
UNION ALL
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE SALARY <= 5000;


SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 50 
MINUS
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE SALARY <= 5000;
// 첫번째 조건을 만족하는 것 중에 두번째 조건을 만족하는 조회 결과를 빼라.
```

### 집합연산자

* 합집합 - UNION/UNION ALL
* 교집합 - INTERSECT
* 차집합 - MINUS

* UNION > 2조건을 만족하거나 하나의 조건만 만족하는 사원들 모두 조회(1회)
* UNION ALL > 2조건을 만족하는 사원들 모두 조회(2회)
* MINUS > 조건 1을 만족하지만 조건2를 만족하지 못하는 사원 조회

* INTERSECT > 2조건을 만족하는 사원'만' 조회(1회)

```SQL
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 50 
INTERSECT
SELECT FIRST_NAME, DEPARTMENT_ID, SALARY
FROM EMPLOYEES
WHERE SALARY <= 5000;
```

</BR>

# 8장

* DDL : 테이블 정의 / 변경 / 삭제 -> 데이터 저장
* 테이블 정의 

```SQL
CREATE TABLE 테이블이름(
컬럼명1 타입(길이) 제약조건,
컬럼명2 타입(길이) 제약조건,
...
컬럼명N 타입(길이) 제약조건
);
ALTER TABLE (ADD 컬럼명10 타입(길이) 제약조건);
ALTER TABLE (MODIFY 컬럼명1 ??(??) ???);
ALTER TABLE (DROP COLUMN 컬럼명1);
DROP TABLE 테이블이름; >> 복구 불가능
```

* DML : 데이터 저장 / 수정 / 삭제
* TCL : 트랜잭션 처리

</BR>

* DB테이블 소유주는 SCHEMA = 사용자 = 계정 (DDL의 일종)

```sql
conn system/암호

create user 계정 identified by 암호; (계정 jdbc, 암호 jdbc)

grant resource, connect to jdbc;
(REVOKE RESOURCE, CONNECT FROM JDBC; > 데이터베이스 더이상 사용 불가능)

conn jdbc/jdbc
```

* 테이블/컬럼 - 숫자시작 불가능, 오라클키워드 불가능, 길이제한, _
* 오라클 주석 --
* emp테이블을 만드려 한다.

| id 정수 5자리                        | 사번 |
| ------------------------------------ | ---- |
| name 문자열 20자리                   | 이름 |
| title 문자열 20자리                  | 직급 |
| dept_id 정수 5자리                   | 부서 |
| salary 실수 정수 10자리 소수점 2자리 | 급여 |

```sql
CREATE TABLE EMP(
ID NUMBER(5,0),
NAME VARCHAR2(20),
TITLE VARCHAR2(20),
DEPT_ID NUMBER(5),
SALARY NUMBER(12,2)
);

# ALTER TABLE
#EMP 테이블에 입사일 저장 컬럼 추가 (ADD)
ALTER TABLE EMP ADD INDATE DATE;

#EMP 테이블에 TITLE컬럼 길이를 20 > 10자리로 변경 (MODIFY)
ALTER TABLE EMP MODIFY TITLE VARCHAR2(10);
단, 데이터가 20자리를 넘어가있다면 실행되지않음

#EMP 테이블에서 입사일 컬럼 삭제 (DROP)
ALTER TABLE EMP DROP COLUMN INDATE; > 다시 복구할 수 없다.


# INSERT INTO
#EMP에 데이터 저장 - 수정 - 삭제
INSERT INTO EMP VALUES(100,'이사원','사원','10',99000.5);
INSERT INTO EMP VALUES(200,'김대리', NULL, NULL, NULL);
INSERT INTO EMP(ID, NAME) VALUES(300,'박과장'); // 언급되지 않은 컬럼에는 자동으로 NULL 입력
INSERT INTO EMP VALUES(400,'최부장','부장','20',99000.5);
INSERT INTO EMP VALUES(500,'박대리','대리','20',99000.5);
COMMIT;

# emp 테이블에서 급여를 못받는 사원의 (salary=null;) 급여를 수정하는 방법
update emp set salary=1000 where salary is null;


# UPDATE ~ SET
# 이름이 박대리인 사원의 부서를 이사원의 부서로 이동
update emp 
set DEPT_ID=(SELECT dept_id FROM EMP WHERE NAME='이사원' AND ROWNUM=1)
WHERE NAME='박대리';
이후 COMMIT


# DELETE
# EMP테이블에서 ID가 100인 사원을 삭제
DELETE EMP WHERE DEPT_ID = 10;




INSERT INTO EMP VALUES(600, '최사장', '임원', NULL, 100000);
ROLLBACK;

# TITLE이 없는 컬럼에 TITLE추가
UPDATE EMP SET TITLE='대리' WHERE ID=200;
```

* 작동 원리
  * insert 명령 후 메모리에 임시 저장한다. 이후 db에 영구적으로 저장하거나 취소하는 sql을 실행해야한다.
    * 영구적으로 저장 > COMMIT > DB에 반영 > 다른 SESSION에 결과 반영
    * 취소 > ROLLBACK > 메모리 삭제 > 다른 SESSION에서 결과가 반영되지 않음

* TCL -  TRANSACTION CONTROL LANG

* DDL - 자동 COMMIT (CREATE, ALTER, DROP)

| DDL 자동 COMMIT<br />DROP TABLE EMP;                         | CREATE<br />ALTER<br />DROP    |
| ------------------------------------------------------------ | ------------------------------ |
| DML - COMMIT/ROLLBACK 결정<br />TRANSACTION 처리언어<br />1) COMMIT 하지 않은 상태이면 다른 세선이 처리 결과 미반영<br />2) COMMIT하면 다른 세션이 처리결과 반영<br /> | INSERT<br />UPDATE<br />DELETE |

* 오라클 RUN SQL - 오라클 연결 - UPDATE 실행 (WHERE ID = 100)
* 자바프로그램 - 오라클 연결 - UPDATE실행 (WHERE ID =100)
  * 오라클에서 UPDATE후 COMMIT하지 않았다면 자바프로그램에서 대기중인 상태로 다른 작동을 하지 않는다.
  * 반드시 COMMIT을 해야한다.

```JAVA
insert into 테이블명[(컬럼명, ...)] values (값, '문자', ..);
insert into 테이블명[(컬럼명, ...)] values (값, '문자', ..);
commit;


update 테이블명 set 변경컬럼명 = 변경값 where 변경조건식;
update 테이블명 set 변경컬럼명 = 변경값; >> 테이블의 모든 행 변경
    
delete 테이블명 where 삭제조건식; >> where절이 없을 경우 모든 데이터 삭제
    >> rollback으로 복구 가능
drop table 테이블명; >> 테이블의 모든 데이터와 테이블 구조까지 삭제
    >> rollback으로 복구 불가능
    
select ~ where
update ~ where
delete ~ where
```

* INSERT, UPDATE, DELETE, CREATE 에 모두 SUBQUERY 사용 가능
* UPDATE 
  * UPDATE 테이블명 SET 컬럼이름=(SUBQUERY) WHERE 컬럼이름 연산자 (SUBQUERY)
* DELETE
  * DELETE 테이블명 WHERE 컬럼이름 연산자 (SUBQUERY)
* INSERT시 SUBQUERY
  * INSERT INTO EMP VALUES(....) > 1행 삽입 (기본 형식)
  * INSERT INTO EMP (SELECT * FROM EMPLOYEES); > 테이블의 데이터 복사
    * EMP에 들어가있는 컬럼 갯수, 컬럼 타입 / EMPLOYEES 에 들어가있는 컬럼갯수, 컬럼타입 확인해야한다.
    * 따라서 이러한 식으로 이용한다.

```sql
INSERT INTO EMP(id, name, title, dept_id) 
	>> (id, name, title, dept_id) 모든 컬럼을 사용할 시 생략 가능
SELECT employee_id, first_name, job_id, department_id, salary 
FROM hr.EMPLOYEES;

	>> 단, 권한이 없기에 employees테이블이 있는 위치에 권한을 부여한다.
grant select on employees to jdbc;
	>> jdbc에서 employees 테이블 조회 권한을 받는다.
# jdbc 소유주에서 조회
select * from hr.employees;
```

* CREATE
  * CREATE TABLE EMP(컬럼1, 타입(길이), ...)0
  * EMPLOYEES 테이블 처럼 11개 컬럼, 이름, 타입, 자리, 데이터까지 그대로 복사할 것이면?

```SQL
CREATE TABLE EMP_COPY
AS
SELECT * FROM HR.EMPLOYEES
SELECT FIRST_NAME, SALARY FROM HR.EMPLOYEES;
```

* 자동으로 증가하는 시퀀스 = 객체
* CREATE TABLE
* CREATE USER
* CREATE SQUENCE

```SQL
INSERT INTO EMP VALUES(???,'이사원','사원','10',99000.5);
INSERT INTO EMP VALUES(???,'김대리', NULL, NULL, NULL);

숫자데이터값을 자동으로 증가시킨다
```

1. 시퀀스를 생성한다

```SQL
CREATE SEQUENCE 시퀀스이름 >> 10부터 시작, 1씩 증가, MAX값 100까지 증가
CREATE SEQUENCE 시퀀스이름 START WITH 10 INCREMENT BY 5 MAXVALUE 100
```

2. 시퀀스를 활용한다.

```SQL
시퀀스명.CURRVAL >> 10, 10, 10
시퀀스명.NEXTVAL >> 1, 2, 3씩 증가 (설정값에 따라 다름)

값을 확인할 때는 DUAL 사용
select 시퀀스명.CURRVAL/NEXTVAL FROM DUAL;
```

3. 수정이나 삭제한다.

```SQL
ALTER SEQUENCE 시퀀스이름 START WITH 10;
ALTER SEQUENCE 시퀀스이름 INCREMENT BY 5;

DROP SEQUENCE 시퀀스이름;
```

#### 예시

```sql
CREATE SEQUENCE EMP_SEQ;

SELECT EMP_SEQ.NEXTVAL FROM DUAL;
	>> 다음 값 출력 (최초 1)

SELECT EMP_SEQ.CURRVAL FROM DUAL;
	>> 현재 값
	
	
# 데이터 추가
INSERT INTO EMP VALUES(700, '이자바', '사원', 30, 45000,55);
	>> 사번은 중복되면 안된다.
INSERT INTO EMP VALUES(EMP_SEQ.NEXTVAL, '이자바', '사원', 30, 45000,55);
```



## 제약조건

> Constraint

* 제약조건 = 현실세계 모델링 = 테이블 데이터 모순이 없어야한다. (무결성)
* 제약조건 타입

| 중복 X                        | unique      |
| ----------------------------- | ----------- |
| Null값 허용 X                 | not null    |
| 중복 X + Null값 허용 X        | primary key |
| 다른 테이블 포함 값 사용 가능 | foreign key |
| 사용자 조건                   | check       |

* 예시

```sql
c_dept 테이블
dept_id 10  20
dept_name 인재개발부  교육부
city 제주도  서울

c_emp 테이블
emp_id
emp_name
title
salary
dept_id

 ## 컬럼명 타입 constraint 제약조건명 제약조건타입
 
CREATE TABLE c_dept(
dept_id number(5) constraint c_dept_id_pk primary key,
dept_name varchar2(20) constraint c_dept_name_uk unique,
city varchar(20) constraint c_dept_city_nn not null
);

CREATE TABLE c_emp(
emp_id number(5) constraint c_emp_emp_id_pk primary key,
emp_name varchar2(20) constraint c_emp_emp_name_nn not null,
title varchar2(10) constraint c_emp_title_ck check (title in ('사원','대리','과장','부장','임원','사장')),
salary number(12, 2) constraint c_emp_salary_ck check (salary>=1000),
dept_id number(5) constraint c_emp_dept_id_fk references c_dept(dept_id)
);

>> 제약조건 정의는 위와같이 DDL에서 진행 > CREATE, ALTER
>> 제약조건에 효력이 발생하는 때 > insert, update, delete (dml)


### 테이블 생성 후 오류 시 오류메시지가 출력된다
insert into c_dept values(10, '인재개발부', '제주');
insert into c_dept values(10, '교육부', '서울'); >> 중복 불가능, 생성되지않음
insert into c_dept values(20, '교육부', null); >> null값 불가능
insert into c_dept values(30, '전산개발부', '대전');

작업이 완료됐을 때 COMMIT 하는 것 잊지않기

INSERT INTO C_EMP VALUES(100, '김사원', '사원', 1000, 10);
INSERT INTO C_EMP VALUES(200, '박대리', '대리', 1999 ,10);
INSERT INTO C_EMP VALUES(300, '안대리', '대리', 2200 ,20);
INSERT INTO C_EMP VALUES(400, '박과장', '과장', 3000 ,30);
INSERT INTO C_EMP VALUES(500, '이부자', '부장', 5000 ,10);

UPDATE C_EMP
SET SALARY = SALARY-100
WHERE EMP_ID = 100;

DELETE C_DEPT WHERE DEPT_ID = 10; >>> FOREIGN KEY 참조하기에 삭제할 수 없음

1 >> 부서를 이동한다.
UPDATE C_EMP
SET DEPT_ID=20
WHERE DEPT_ID = 10;

2>>
DELETE C_DEPT WHERE DEPT_ID = 10; 

# 제약조건 해제 후 테이블 삭제
DROP TABLE C_DEPT CASCADE CONSTRAINTS; >> 제약조건을 없애준다.
DROP TABLE C_DEPT; >>> 실행오류 (C_EMP 테이블이 참조중 (자식테이블 C_EMP))

# 컬럼 이름 변경
ALTER TABLE C_EMP RENAME 이전컬럼명 TO 새로운컬럼명;
RENAME 이전테이블명 TO 새로운테이블명;
```

* JAVA - ORACLE 연동 과정
* 아이디입력 - XXXX > DB C_EMP저장(SQL) > DB전송 > 오라클 DTO, DAO

```sql
select * from user_constraints;
# 오라클 딕셔너리
```

* 제약조건을 보여준다.

```sql
select constraint_name, constraint_type, search_condition, table_name 
from user_constraints
where LOWER(table_name)='c_emp'


CONSTRAINT_NAME                                              CO SEARCH_CONDITION
------------------------------------------------------------ -- --------------------------------------------------------------------------------
TABLE_NAME
------------------------------------------------------------
C_EMP_DEPT_ID_FK                                             R
C_EMP

C_EMP_SALARY_CK                                              C  salary>=1000
C_EMP

C_EMP_TITLE_CK                                               C  title in ('사원','대리','과장','부장','임원','사장')
C_EMP

C_EMP_EMP_NAME_NN                                            C  "EMP_NAME" IS NOT NULL
C_EMP

C_EMP_EMP_ID_PK                                              P
C_EMP
```

* R - foreign key
* P - primary key
* C - check type (부가 설명 추가)