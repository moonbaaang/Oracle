

# 오라클 데이터베이스 

>  0325 강의내용

1. 데이터베이스 - 자료(데이터)의 의미있는 모음
2. 영구적 저장
3. 여러 프로그램에서 동시 사용가능
4. 관계형
5. 행(1개 데이터모음) , 열(1개 행을 구성하는 여러가지 요소)로 이루어진 테이블로 구성되어있다.
6. 컴퓨터 부팅 시 자동시작
7. 수동시작 필요 시
   * windows메뉴 - oracle 11g express edition - start database
8. 관계형 데이터베이스 접근 언어 - sql
9. run sql command (SQL PLUS) - sql을 작성 ,편집, 실행
10. 이클립스 - DATA Explorer 설정 사용 가능

| DQL               | SELECT |
| ----------------- | ------ |
| DDL 테이블        |        |
| DML 데이터        |        |
| TCL 트랜잭션 관리 |        |

| 산술                            | + - * /                  |
| ------------------------------- | ------------------------ |
| 비교                            | > >= < <= != =           |
| 논리                            | not and or               |
| 목록                            | in(...  , ....)          |
| 범위                            | between A and B          |
| 유사패턴<br />문자데이터만 가능 | like %<br />like _       |
| null 비교연산자                 | is null<br />is not null |

### 작성 방법

```sql
select 컬럼명 as 별칭, sysdate, upper('aaa'), +-*/ ...
from 테이블명
where 조회조건식
group by
having
order by 컬럼명|컬럼인덱스|별칭 asc(생략가능)|desc
```

* subquery 작성 예시

```sql
SELECT ROWNUM, SALARY
FROM (SELECT SALARY FROM EMPLOYEES ORDER BY SALARY DESC)
WHERE ROWNUM <= 3;
```

* Susan과 같은 부서의 사원의 부서코드, 이름을 조회하라 (Susan은 한명)

```sql
SELECT DEPARTMENT_ID, FIRST_NAME
FROM EMPLOYEES
WHERE DEPARTMENT_ID = 
	(SELECT DEPARTMENT_ID FROM FROM EMPLOYEES WHERE UPPER(FIRST_NAME) = 'SUSAN');
```

* Susan과 같거나 더 많은 급여를 받는 사원의 부서코드, 이름 조회하라

```sql
select department_id, first_name
from employees
where salary >= (select salary from employees where first_name = 'Susan');
```

* William과 같거나 더 많은 급여를 받는 사원의 부서코드, 이름을 조회하라 ( Willian은 두명 )

```sql
select department_id, first_name
from employees
// 모든 willian보다 모두 클 때
where salary >= all(select salary from employees where first_name = 'William');

// 작은 급여를 받는 Willian 보다 클 때
where salary >= any(select salary from employees where first_name = 'William');


WHERE DEPARTMENT_ID IN (SELECT DEPARTMENT_ID FROM EMPLOYEES WHERE FIRST_NAME = 'William')
```

### 그룹함수

| sum      | select sum(salary) from employees<br />숫자 형태의 데이터만 가능하다. |
| -------- | ------------------------------------------------------------ |
| avg      | 숫자 형태                                                    |
| count    | 숫자, 문자, 날짜 형태<br />null값 제외<br />count(*) null값 포함 |
| max      | 최대값 - 숫자, 문자, 날짜                                    |
| min      | 최소값 - 숫자, 문자, 날짜                                    |
| stdev    | 표준편차 - 숫자                                              |
| variance | 분산 - 숫자                                                  |

* 100, 200, 300 데이터를 합치면 값은 하나가 나온다.
* sum()
  * select sum(salary) from employees
  * 모든 sum()함수의 결과는 하나만 나온다.

* employees테이블 급여 총합 조회

```sql
SELECT SUM(SALARY) FROM EMPLOYEES;
```

* employees테이블  급여 평균 조회

```SQL
SELECT AVG(SALARY) FROM EMPLOYEES;
```

* employees테이블  급여 갯수 조회

```SQL
select count(department_id), count(salary) from employees;
```

* employees테이블  급여 최대값, 최소값 조회

```SQL
select max(salary), min(salary) from employees
```

* 사원 이름, 최대 급여 조회

```sql
select first_name, max(salary) from employees; >> 불가능(생성되는 컬럼의 개수가 다르다.)

select first_name, salary
from employees
where salary = (select max(salary) from employees);
```

### Group by, Having

* 80번 부서에 있는 부서원들만 급여 총합을 조회 ( 부서 배정이 안된 사원은 제외 )
* 그룹함수를 조회할 때는 select절에 다른 컬럼을 기술 불가능하다.
  * 단, group by 뒤 기술한 컬럼은 제외한다.
  * 그룹함수에 있는 내용에 조건식을 줄 때는 having을 사용한다.

```sql
select department_id, sum(salary)
from employees
group by department_id;
>> null 값이 나온다.

select department_id, sum(salary)
from employees
where department_id is not null
group by department_id;
```

* 이때 직종도 함께 조회하고 싶다면?

```sql
select department_id, job_id, sum(salary)
from employees
where department_id is not null
group by department_id, job_id;
```

* 부서 순서대로 조회하고 싶다면?

```sql
order by department_id;
```

* 부서별로 급여 총합을 조회하되, 부서별 급여총합이 10000 미만인 부서의 결과만 조회

```sql
select department_id, sum(salary)
from employees
group by department_id
having sum(salary)<10000;
```

* from > where (일반조건식)> group by > having(그룹함수 조건식) > select (단, 3, 4번이 되어야 가능)
  * 그룹함수는 조건을 지정할 수 없다?
  * having으로 변경해서 뒤에 열거해줌

* 부서별로 급여 총합 조회하되 **사원 급여가 5000 미만은 제외**하고 **부서별 급여총합이 50000 이상인 부서의 결과만 조회** 

```sql
select department_id, sum(salary)
from employees
where salary >= 5000
group by department_id
having sum(salary)>=50000
order by sum(salary);
```

### rollup

* group by 뒤 rollup 추가 > 중간중간 부분합을 보여준다.

```sql
select department_id, job_id, sum(salary)
from employees
where department_id is not null
group by rollup(department_id, job_id);
```

job_id 로 나누고 department_id 별로 다시한번 부분합을 본다.

### cube

* 그룹을 지어줄 때 중간 결과를 반환한다.

```sql
select department_id, job_id, sum(salary)
from employees
where department_id is not null
group by cube(department_id, job_id);
```

</br>

# 7장 Oracle의 데이터 형식

> 오라클 내 데이터 형식 약 31개

| 문자<br />(VARCHAR2)  | CHAR / VARCHAR2 > 영문 - 1바이트 저장/ 한글 - 3바이트로 저장<br /><br />CHAR(5) > 한글 1글자만 들어감<br /><br />CHAR(50) > 'ABC' > [ABC+47바이트 고정]<br /><br />VARCHAR2(50) > 'ABC' > [ABC]<br />NCHAR / NVARCHAR2 / CLOB > 유니코드로 저장 2바이트<br />'데이터' -> 대소문자 구분 |
| :-------------------- | ------------------------------------------------------------ |
| 정수<br />NUMBER(8)   | BINARY_INT / INT / NUMBER(8) / NUMBER(8,0)                   |
| 실수<br />NUMBER(8,2) | BINARY_FLOAT / FLOAT / NUMBER(8,2) > 정수 6, 소수 2자리      |
| 날짜<br />DATE        | 초 표현 > DATE <br />1/1000초 표현 > TIMESTAMP TIMESTAMPXXXXX |
| 대용량/기타           | CLOB - 1TB 문자열 대용량 데이터<br />웹서버(자바)  > 네트워크 > DB<br />BLOB - 1TB 바이너리 대용량 데이터<br />BFILE<br />BIN |

* SQL은 반복, 조건, 변수선언 X

### 데이터 형식과 형변환

![image-20210325205919566](image/image-20210325205919566.png)

#### dual

```sql
Select sysdate
from dual;
// 날짜(RR/MM/DD)
```

* select tname from tab; 명령을 할 시에 dual 테이블은 보이지 않는다.
* dual이라는 테이블은 실제 존재하지 않고 가상 임시 테이블이다.
  * 특정 함수 결과를 조회할 때 사용하고, 결과로 여러 행이 나오지 않는다.

#### cast

* 타입을 변환하여 표시해주는 함수

```sql
select sysdae from dual; > 연도 월 일 표현

select cast(sysdate as timestamp) from dual;
select cast(12345.67 as number(4)) from dual; > 네자리의 정수로 표현하기에는 큰 값이다.
select cast(12345.678 as number(10,2)) from dual; > 자동 반올림
```

* to_char / to_number / to_date
  * to_number < > to_char
  * to_date < > to_char
  * 숫자 < > 날짜 불가능
  * 숫자 < > 문자 < > 날짜
* to_number

```sql
select 100+200 from dual;
select '100'+'200' from dual; >> '0-9구성' 자동 숫자 변환

select '123,456'+'200' from dual; >> ','라는 다른 기호가 있기 때문에 자동 변환 불가능
to_number('123,456','999,999') >> 999999 6자리이고 ','로 구분한 숫자이다라는 것을 표현

select to_number('123,456','999,999')+'200' from dual;

to_number(숫자 변환 데이터, 데이터 구성 형식) 
select '$100'+'$200' from dual; >> 자동 숫자 변환이 되지 않는다.
select to_number('$100','$999') + to_number('$200','$999') from dual;
```

* to_char

```sql
select 123456 from dual; >> 123456
select to_char(123456,'$999,999') from dual; >> $123,456

select to_char(123456.789,'$999,999.99') from dual; >>> $123,456.79 (반올림)

```



| $            | $ 기호                                                       |
| ------------ | ------------------------------------------------------------ |
| ,            | , 기호                                                       |
| L            | Locale currency - 현재 지역의 통화표시 \\ (원)               |
| 9            | 1자리 숫자                                                   |
| 0            | 1자리 숫자                                                   |
| YY<br />YYYY | YY - 2000년대<br />YYYY - RR (50-99) 1900년대 / (00-49) 2000년대 |
| MM           | 월                                                           |
| DD           | 일                                                           |
| HH<br />HH24 | 시                                                           |
| MI           | 분                                                           |
| SS           | 초                                                           |
| DAY          | 요일                                                         |

* 현재 오라클 설정 > 설치 시 자동으로 여러 환경을 지정한다.
  * SELECT * FROM DICT; (dictionary) 지정된 환경을 볼 수 있다.

```sql
select table_name from dict
where table_name like '%NLS%'; > NLS포함 테이블

DESC NLS_SESSION_PARAMETERS;
select * from NLS_SESSION_PARAMETERS
where PARAMETER = 'NLS_DATE_FORMAT';

select SYSDATE FROM DUAL; >> 21/03/25 (기본값)

2021/03/25 13:39:11 > 이러한 형태로 변경하고싶다면?
select TO_CHAR(SYSDATE, 'YYYY/MM/DD HH24:MI:SS') from dual;

select to_char(SYSDATE, 'YYYY/MM/DD DAY HH24:MI:SS') from dual;

select to_char(SYSDATE, 'YYYY/MM/DD DAY HH24:MI:SS') from dual;


2021년도 03월 25일 01시 47분 10초 > 이러한 형태로 변경하고 싶다면?
일반 문자가 나올 때는 분리자 (ex, /)로 인식하지 못한다.
"년도" "월" "일" ... 로 분리해야한다.

select to_char(SYSDATE, 'YYYY"년도" MM"월" DD"일" HH24"시 "MI"분 "SS"초"') from dual;

03시 05분 같이 불필요한 0을 없애고 싶다면?
select to_char(SYSDATE, 'FMYYYY"년도" MM"월" DD"일" HH24"시 "MI"분 "SS"초"') from dual;
FM (format masking) 으로 없앤다.
```

* FM - Format masking > 불필요한 내용을 제거하고 출력해준다.
* **TO_DATE**
* '21/03/25' >> 문자로 취급하게 된다. >> 자동으로 날짜로 되게 하고싶다면?

```sql
select sysdate+1 from dual; > 내일 날짜가 나온다.
select to_date('21/03/25', 'yy/mm/dd')+1 from dual; > 21/03/26

select sysdate + 365 from dual; > 단순히 1년 뒤의 날짜를 조회한다.

년, 월, 일 정보만 출력하고싶다면?
select to_char(sysdate, 'yyyy') from dual;
select to_char(sysdate, 'mm') from dual;
select to_char(sysdate, 'dd') from dual;

5년 , 5개월, 5일 후를 구하고싶다면 to_char() 뒤에 +5

05년도 입사자 조회 ( 기존 방법 )
select hire_date from empolyees
where hire_date like '05/%';

select hire_date from employees
where to_char(hire_date, 'yyyy')= '2005';

3월 입사자 조회 ( 기존 방법 )
select hire_date from employees
where hire_date like '___03/%';

select hire_date from employees
where to_char(hire_date, 'mm') = '03';
```

| 타입변환함수   | CAST<br />TO_DATE, TO_CHAR, TO_NUMBER                        |
| -------------- | ------------------------------------------------------------ |
| 그룹함수       | SUM, AVG, MIN ,MAX, COUNT, STDEV, VARIANCE                   |
| 문자데이터함수 | UPPER, LOWER, INITCAP, LENGTH, LENGTHB<br />SUBSTR, INSTR<br />ITRIM, RTRIM<br />concat |
| 숫자데이터함수 | MOD - 나머지 MOD(10, 3)<br />ROUND - 반올림 ex) ROUND(10/3, 0) (소수점이하 0번째 자리 = 나타내지않음)<br />ROUND(10,3,3) > 3.333<br />ROUND(333.8, -1) > 330 <br />TRUNC > 버림 TRUNC(3.6324, 0) >> 3 |
| 날짜데이터함수 | SYSDATE<br />SYSTIMESTAMP - 1/1000초<br />ADD_MONTHS<br />MONTH_BETWEEN |
| 순위함수       | ROW_NUMBER()<br />  ex) Row_number() over(order by salary desc)<br />RANK()<br />DENSE_RANK()<br />순위함수() <br />OVER ( PARTITION BY 순위를 만들어야할 컬럼명<br />                                ORDER BY 컬럼명 ASC\|DESC) |
| NULL처리함수   | NVL(SALARY, 0) (NULL값이면 0으로 변경한다는 뜻이다.)         |

* 예시

```SQL
SELETE LENGTHB('가') FROM DUAL;
>> 3 (메모리를 저장하는 바이트 수)
SELECT LENGTHB('A') FROM DUAL;
>> 1

SELECT LENGTH('A') FROM DUAL;
>> 1

SELECT CONCAT('A', '가') FROM DUAL; (CONCATENATE)

SELECT INSTR('이것이 자바다', '자바') FROM DUAL;
> 5번째 인덱스부터 '자바' 존재

SELECT FIRST_NAME
FROM EMPLOYEES
WHERE FIRST_NAME like '%er%'; 

SELECT FIRST_NAME
FROM EMPLOYEES
WHERE INSTR(FIRST_NAME, 'er') > 0;

# 3월 입사자 조회
SELECT HIRE_DATE FROM EMPLOYEES
WHERE INSTR(HIRE_DATE,  '03') = 4; >> 21/03/12 에서 03이 들어가는 인덱스가 4번째이다.

# 이름을 대문자, 소문자로 변경해서 출력 
SELECT FIRST_NAME, UPPER(FIRST_NAME), LOWER(FIRST_NAME), INITCAP(FIRST_NAME) 
FROM EMPLOYEES WHERE DEPARTMENT_ID = 50;


# 이름이 대문자인지 소문자인지 알 수 없을 때 (양쪽을 동일하게 만들어주고 비교한다.)
SELECT FIRST_NAME 
FROM EMPLOYEES
WHERE LOWER(FIRST_NAME) = 'jennifer';
WHERE UPPER(FIRST_NAME) = 'JENNIFER';
WHERE INITCAP(FIRST_NAME) = INITCAP('JENNIFER');

REPLACE / TRANSLATE ('이것이 자바다', '자바', '오라클')
>> 문자 일부를 변경해준다.
SUBSTR('이것이 자바다', 1, 2) >> '이것' (1번째 인덱스에서 2개만 가져온다.)
INSTR('이것이 자바다', '이것') >> 1 ('이것'의 인덱스인 1을 반환한다.)
```

* INITCAP >> 첫문자만 대문자로 만든다.

</br>

숫자나 날짜 > 자동형변환 > 문자열

```sql
select hire_date from employees
where substr(hire_date, 4, 2) ='03';
```

* 의미없는 내용을 없애는 방법

```sql
select ltrim('      aaa      ') from dual;
select rtrim('      aaa      ') from dual;

select ltrim('###aaa###', '#') from dual;
select rtrim('###aaa###', '#') from dual;
```

* 숫자와 관련된 함수

```SQL
SELECT 3456.789, TRUNC(3456.789, 2), TRUNC(3456.789, 1), TRUNC(3456.789, 0), TRUNC(3456.789, -1), TRUNC(3456.789 ,-2)
FROM DUAL;

ROUND / TRUNC 에서 두번째 매개변수가 0일 경우 정수로 표현한다. 이때 0은 생략가능하다.
```

* EMPLOYEES 테이블에서 홀수 사번만 조회

```SQL
SELECT EMPLOYEE_ID
FROM EMPLOYEES
WHERE MOD(EMPLOYEE_ID, 2) = 1;
```

* EMPLOYEES 테이블에서 입사년도별 급여 평균을 조회하되 평균은 정수로 출력, 소수점 이하는 버림

```SQL
SELECT SUBSTR(HIRE_DATE, 1, 2) AS 입사연도, TRUNC(AVG(SALARY)) AS 평균급여
FROM EMPLOYEES
GROUP BY SUBSTR(HIRE_DATE, 1, 2)
ORDER BY SUBSTR(HIRE_DATE, 1, 2);

SELECT SUBSTR(HIRE_DATE, 4, 2) AS 입사월, TRUNC(AVG(SALARY)) AS 평균급여
FROM EMPLOYEES
GROUP BY SUBSTR(HIRE_DATE, 4, 2)
ORDER BY 입사월;
```

```SQL
SELECT SYSDATE FROM DUAL;
SELECT SYSTIMESTAMP FROM DUAL;
SELECT TO_CHAR(SYSDATE, 'YYYY')+1 FROM DUAL;
SELECT TO_CHAR(SYSDATE, 'MM')+1 FROM DUAL;
>> 는 이와 동일
SELECT ADD_MONTHS(SYSDATE, 1) FROM DUAL;

>> 입사한지 얼마나 됐는지 확인하기
SELECT SYSDATE - HIRE_DATE
FROM EMPLOYEES;

>> 입사한지 경과된 년 수 조회
SELECT ROUND((SYSDATE - HIRE_DATE)/365, 0) (0생략가능)
FROM EMPLOYEES;

>> 입사한지 몇 주 된지 조회
SELECT ROUND((SYSDATE - HIRE_DATE)/7, 0) (0생략가능)
FROM EMPLOYEES;

>> 몇 개월이 됐는지 조회 (MONTHS_BETWEEN)
SELECT MONTHS_BETWEEN(SYSDATE, HIRE_DATE) FROM EMPLOYEES;
```

* MONTHS_BETWEEN 경과된 개월 수 를 확인한다.

#### 순위함수

* 급여가 많은 사원부터 순위를 매긴다.

```SQL
SELECT FIRST_NAME, SALARY
, ROW_NUMBER() OVER(ORDER BY SALARY DESC) AS 급여순위
FROM EMPLOYEES

SELECT FIRST_NAME, SALARY
, RANK() OVER(ORDER BY SALARY DESC) AS 급여순위
FROM EMPLOYEES;
# 중복 순위는 동일하게, EX) 1, 2, 2, 4

SELECT FIRST_NAME, SALARY
, DENSE_RANK() OVER(ORDER BY SALARY DESC) AS 급여순위
FROM EMPLOYEES;
# 동일한 순위는 동일하게, 중간에 빠진 숫자가 없도록 순서대로

SELECT FIRST_NAME, SALARY, DEPARTMENT_ID
, ROW_NUMBER() OVER(PARTITION BY DEPARTMENT_ID ORDER BY SALARY DESC)
FROM EMPLOYEES;
```

* COMMISION_PCT 컬럼

  * 커미션 표현 컬럼의 표현방식을 변경

  * 커미션을 못받는 사원 - NULL > 공백
  * 커미션 받는 사원 - .4 > 0.4 

```SQL
SELECT FIRST_NAME, COMMISSION_PCT FROM EMPLOYEES
WHERE COMMISSION_PCT IS NULL;

# 전체데이터를 표현, 커미션을 못받는 사람은 0으로 표시
SELECT FIRST_NAME, NVL(COMMISSION_PCT, 0) FROM EMPLOYEES;

SELECT FIRST_NAME, NVL(TO_CHAR(COMMISSION_PCT), '보너스없음') FROM EMPLOYEES;
>> 원래는 '보너스없음' 자리에 커미션과 같은 타입인 숫자값을 넣어야 하기에 TO_CHAR()로 형변환을 해준다.
NVL(컬럼명, NULL대체값) > 이때 컬럼명과 대체값의 타입은 같아야한다.

SELECT FIRST_NAME, NVL(TO_CHAR(COMMISSION_PCT), '보너스없음') AS "보 너 스"FROM EMPLOYEES;
>> 이때 별칭의 이름을 '보 너 스' 로 하고싶다면? "보 너 스" 로 이중따옴표를 사용하면된다.
```

</br>

## 문제풀이

1. 이름이 'adam' 인 직원의 급여와 입사일을 조회하시오. 

```sql
SELECT FIRST_NAME, SALARY, HIRE_DATE
FROM EMPLOYEES
WHERE FIRST_NAME = initcap('adam');
```


2. 나라 명이 'united states of america' 인 나라의 국가 코드를 조회하시오.

```sql
SELECT COUNTRY_ID FROM COUNTRIES
WHERE lower(COUNTRY_NAME) = 'united states of america';
```


3. 'Adam의 입사일은 05/11/2 이고, 급여는 7,000\ 입니다.' 의 형식으로 직원
정보를 조회하시오.

직원정보

Adam의 입사일은 05/11/2 이고, 급여는 7,000\ 입니다. 
......

```SQL
SELECT CONCAT(first_name,"의 입사일은 ", TO_CHAR(hire_date," 이고, 급여는"), TO_CHAR(salary, "입니다.")
FROM EMPLOYEES;
              
SELECT FIRST_NAME||'의 입사일은 '||TO_CHAR(HIRE_DATE, 'YY/FMMM/DD')||'이고, 급여는 '||LTRIM(TO_CHAR(SALARY, '999,999L'))||' 입니다'
AS 직원정보
FROM EMPLOYEES;
```

4. 이름이 5글자 이하인 직원들의 이름, 급여, 입사일을 조회하시오.

```SQL
SELECT FIRST_NAME, SALARY, HIRE_DATE
FROM EMPLOYEES
WHERE LENGTH(FIRST_NAME)<=5;
```

5. 06년도에 입사한 직원의 이름, 입사일을 조회하시오.

```SQL
SELECT FIRST_NAME, HIRE_DATE
FROM EMPLOYEES
WHERE TO_CHAR(HIRE_DATE, 'RR') = '06';
```

6. 10년 이상 장기 근속한 직원들의 이름, 입사일, 급여, 근무년차를 조회하시오.

```SQL
SELECT FIRST_NAME, HIRE_DATE, SALARY, ROUND((SYSDATE-HIRE_DATE)/365, 0) AS 근무년차
FROM EMPLOYEES
WHERE ROUND((SYSDATE-HIRE_DATE)/365, 1) >= 10;
```

7. employees 테이블에서 
   직종이(job_id) 'st_clerk'인 사원 중에서 급여가 3000 이상인 사원의
   first_name, job_id, salary 를 조회하시오. 단 이름은 모두 대문자로 출력하시오.

```SQL
SELECT UPPER(FIRST_NAME) AS 이름, JOB_ID, SALARY
FROM EMPLOYEES
WHERE LOWER(JOB_ID) = 'st_clerk' AND SALARY>=3000;
```

8. 급여합계가 20000 이상인 직종(job_id)의 job_id, 급여합계를 조회하시오.
   단, 급여합계는 10자리로 출력하되 공백은 '0'으로 표시하시오. 

```SQL
SELECT JOB_ID, to_char(SUM(SALARY), '0,000,000,000') as 급여합계
FROM EMPLOYEES
GROUP BY JOB_ID
HAVING SUM(SALARY)>= 20000;
```

10. 직원의 이름, 급여, 직원의 관리자 사번을 조회하시오. 단, 관리자가 없는 직원은
    '<관리자 없음>'이 출력되도록 합니다.

```sql
SELECT FIRST_NAME, SALARY, NVL(TO_CHAR(MANAGER_ID), '<관리자없음>') AS MANAGER
FROM EMPLOYEES;
```

