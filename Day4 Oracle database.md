# 4일차 오라클 데이터베이스

* JDBC

  * ex) 회원가입 > DB저장 create table / insert 키보드 입력

  * 자바언어에서 DB를 이용하게 해주는 라이브러리 모음

  * java database connectivity

  * 자바프로그램 - sql / java 문법

  * api > java.sql.*

    ```java
    interface Connection{
        void connect();
    }
     
    class OracleConnection implements Connection{
        public void connect(){ 오라클, db 등 종류에 따라 연결 라이브러리 호출 }
        // 이 수업에서는 오라클용으로 실행
    } // jdbc driver 포함 구현
    ```

    작성순서

    1. JDBC driver (자바클래스) 등록 메모리 로드
    2. DB 연결 - DB 종류마다 서로 다른 라이브러리 호출 > 독립적으로 통일하여 호출
    3. SQL 전송
    4. SQL 결과 검색
    5. DB 연결 해제

    * 필요 시 3~4번 반복

</br>

### java + sql

* 이클립스 + run sql command line

1. DB 준비
2. DB 종류에 따라 jdbc driver 설치
3. 이클립스 내부에서 자바 프로젝트를 만들면 기본 jdk 라이브러리 경로 확인
   * C:\kdigital\oraclexe\app\oracle\product\11.2.0\server\jdbc\lib > ojdbc6.jar 파일 존재
   * C:\Program Files\Java\jre1.8.0_251\lib\ext 로 복사 (external lib)
     * 사용자 환경에 따라 다를 수 있다. 
4. SQL 결과 검색
5. DB 연결 해제

```java
//1,2,5 과정
package jdbc;

import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Connection;

public class ConnectionTest {
// 오라클 자동시작 - ojdbc6.jar 설치, 이클립스 세팅
	public static void main(String[] args) {
		Connection conn = null; // finally문에서 반드시 close()를 해주기 위해 바깥에 지정
		try {
			// jdbc driver 메모리 로드, db종류마다 () 내부 내용이 다름
			Class.forName("oracle.jdbc.driver.OracleDriver");
			//db 연결 
			conn = DriverManager.getConnection
					("jdbc:oracle:thin:@127.0.0.1:1521:xe","hr","hr");
			// 127.0.0.1 내 컴퓨터 라는 의미 ,xe , hr 계정, 암호
			// 4명이 한대의 db 사용 시 > 실제 @ip address 가 필요하다.
			System.out.println("db연결 성공");
		
			System.out.println("db연결 해제 성공");
		} catch(ClassNotFoundException e) { //name 오타가 있을 때, ojdbc 세팅이 안되어있을 때
			System.out.println("드라이버 세팅 확인하세요");
		} catch(SQLException e) { //계정 명 또는 암호 불일치
			System.out.println("DB 연결정보 확인하세요");
		} finally {
			try {
				conn.close(); //tcp 소켓, 파일 close() (반드시 처리해야하는 문장)
			} catch(SQLException e) {} // 예외처리 한번 더 필요
		}
	}

}
```

* DML - insert / update / delete
  * sql에서는 반드시 commit, rollback 필수
  * jdbc에서는 자동 commit
    * Statement st = conn.createStatement();
    * int 변경행 = st.executeUpdate("DML");

```java
Class.forName("oracle.jdbc,driver,OracleDriver");
Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:xe","hr","hr");

Statement st = conn.createStatement();
int 변경행 = st.executeUpdate("insert|update|delete ...");
```

* TCL - commit / rollback
* DDL - create table / alter table / drop table - run sql command line, jdbc 동일 자동 commit
  * jdbc 1 - hr emp create
  * jdbc 2 - hr emp drop
  * jdbc 3 - hr emp alter
    * 이는 실행 권고방법이 아니다. (db내부에서 사용)
    * jdbc에서는 DML만 주로 사용
* DQL (조회)

```java
Statement st = con.createStatement();
ResultSet rs = st.executeQuery("select id, name, salary from emp");
// 변경된 행의 개수가 넘어온다.
```

* rs 구조 (여러개 행, 여러개 열)

| id(정수) | name(문자) | salary(실수) |
| -------- | ---------- | ------------ |
| 100      | 이자바     | 5000.9       |
| 200      | 김사원     | 3000.5       |
| ...      | ...        | ...          |

* while(rs.next()) - true/false 반환, 데이터가 있는 만큼 반복 (인덱스 1번부터 시작)

```java
while(rs.next()){
    rs.getInt(1);  //== rs.getIInt("id") id가 1번 인덱스
    rs.getString(2); // == rs.getString(name)
    rs.getDouble(3); // == rs.getDouble("salary")
}
```

|                       | java          | oracle                                                       |
| --------------------- | ------------- | ------------------------------------------------------------ |
| 정수 : rs.getInt()    | int           | number(n) = int                                              |
| 실수 : rs.getDouble() | double        | number(n,s) = float                                          |
| 문자 : rs.getString() | String        | char, varchar2(n)                                            |
| 날짜 : rs.getDate()   | java.sql.Date | date                                                         |
| rs.getString()        | String        | to_char(sysdate, 'yyyy-mm-dd')<br />=> 날짜가 아닌 문자형 결과 |

```java
// 메서드로 만듬
Class A{}
	test(){
    	driver 메모리로드;
    	con생성;
   	 	st, pt 생성;
  	  	rs 생성 > DB와 연결이 무관한 객체에 복사 (연결을 끊으면 사용할 수 없는 문제 해결);
            (배열, ArrayList.. 복사 >> 서버에서는 이를 서버가 계속 사용할 수 있게 복사 해야함)
    	rs.close();
    	st.close()
        pt.close();
    	con.close();
        return ArrayList
	}
}

class Main{
    A a1 = new A();
    ArrayList <<< a1.test();
} >> 조회 결과 리턴
```

