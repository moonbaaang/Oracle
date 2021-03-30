

# 5일차 오라클데이터베이스

* jdbc와 연동
  * java.sql 패키지 api
    * java.sql
  * 자바에서 db를 이용하기 위한 기술
  * db종류마다(표준 sql, 독자sql, 데이터타입) 표현 방법과 연결 방법이 다르다.
  * 한번의 소스 작성으로 win/mac/linux 에서 실행
  * jdbc소스를 한번 작성하면 모든 db 사용 가능 = db독립적

1. jdbc driver (자바클래스들) 등록, 메모리 로딩

* Class.forName("jdbc driver") >> ()내부는 종류에 따라 달라진다.

2. db연결

* DriverManager.getConnection("jdbc url", "id", "pw")

```java
Connection con = DriverManager.getConnection;
DriverManager.getConnection("jdbc url", "id", "pw");
//jdbc url >> jdbc:oracle:thin:@localhost:1521:xe
//jdbc:다른db
```

3. sql 전송
4. sql 결과 검색
5. db 연결 해제 (conn.close())

### 3-4 과정 구체화

* Statement st = con.createStatement(); 

  * 매개변수 없다.

  * int cnt = st.executeUpdate("insert|update|delete");

  * ResultSet rs = st.executeQuery("select");

    * rs >> 첫번째 행을 참조하는 것이 아니다.
    * rs.next();를 호출해야한다.

    ```java
    while(rs.next()){
        rs.getXXXX("");
    }
    ```

    * 보안 위협이 있다.

* PreparedStatement pt = con.prepareStatement("sql") 
  * sql변수로 sql을 미리 작성해서 넣어도 된다.
  * PreparedStatement pt = con.prepareStatement("insert|update|delete") 
  * pt.setXXX()
  * int cnt = pt.executeUpdate();
  * PreparedStatement pt = con.prepareStatement() 
  * pt.setXXX()
  * ResultSet rs = pt.executeQuery();

```java
while(rs.next()){
    rs.getXXXX("");
}
```

* finally 문에 rs.close(), st.close(), ps.close(), con.close()
* Statement/preparedStatement
  * sql 전송 > Statement는 sql이 드러난다.
  * prerparedStatement는 sql이 드러나지 않는다. 
    * 보안에 더 좋고 실행속도가 더 빠르다.

```java
interface Statement{
    executeQuery();
    executeUpdate();
}
interfaㅊe PreparedStatement extends Statement{
    setInt / setDouble / setString / setDate 추가
}

extends - 클래스끼리, 인터페이스끼리 상속
implements - 상위 인터페이스를 하위클래스에서 상속
```

### jdbc driver 설치

1. 이클립스에서 자바프로젝트의 jre system library 경로를 확인
2. 오라클 설치 경로 확인 후 오라클설치경로\app\oracle\product\11.2.0\server\jdbc\lib\ojdbc6.jar 파일 복사
3. 탐색기에서 2번 파일을 1번에서 확인한 경로\ext 디렉토리에 붙여넣기
4. 이클립스 재시작

### dml 실행

|                    | JDBC내     | RUN SQL CL          |
| ------------------ | ---------- | ------------------- |
| DDL                | 자동COMMIT | 자동COMMIT          |
| DCL                | 자동COMMIT | 자동COMMIT          |
| DML                | 자동COMMIT | 수동COMMIT/ROLLBACK |
| DQL(조회) = SELECT | x          | x                   |

### 이클립스 내 DATA Explorer

window - show view -  other - data source explorer

* Database Connections - new - db종류 설정 - drivers - new driver definition - 버전 설정, jar List에서 버전과 다른 jar삭제 - add jar - 기존 설정 당시 jar 설정 - Properties - General:Connection URL에 url정보가 나온다.

* Host을 ip에 맞게 설정, 계정과 암호 설정후 finish

* 이후 Data Source Explorer에 db정보가 나타나게 된다.

* file - new - other - sq devel - sql file - 이름 작성 후 finish

* type - Oracle_11 , 설정한 이름, 데이터베이스
  * ex) select * from employees; - execute selected text 이후 결과 나타난다.

* 자동 커밋이 기본 설정이다.

</br>

* select - ResultSet - con.close()되면 더이상 조회가 불가 >> 이를 조회가 되면 조회불가 하게 설정
* 메서드 시작 > con생성 ,,, > con.close() > 메소드 종료

* Statement st = con.createStatement()
* ResultSet rs = st.executeQuery()

* ArrayList\<DTO> list = new ArrayList(); 를 이용하여 close()가 되어도 조회가 가능하게 만든다.

### 테이블 만들기

* 콘솔 입력 > db board 테이블 저장 > 10개씩 조회

* class BoardMain - 게시판메뉴

  1. 글쓰기
  2. 게시물리스트 조회
  3. 종료

  * 번호 입력

* class BoardInsertView - 제목입력, 내용입력, 암호입력 화면 출력

* class BoardDAO (데이터가 있는 곳에 직접적으로 접근하는 객체)

  * insertBoard()

* class BoardDTO - board테이블 1개 레코드의 다수개 컬럼 = 변수

* BoardMain > BoardInsertView > DTO > DAO





