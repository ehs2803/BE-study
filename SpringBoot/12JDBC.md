# JDBC, SQL Mapper, ORM

JDBC는 1997년에 출시될 정도로 오래된 기술이고, 사용하는 방법도 복잡하다. 

그래서 최근에는 JDBC를 직접 사용하기 보다는 JDBC를 편리하게 사용하는 다양한 기술이 존재한다. 대표적으로 SQL Mapper와 ORM 기술로 나눌 수 있다.

![image](https://user-images.githubusercontent.com/65898555/180720562-fee97ac8-cb7d-43bc-9efe-eb3c051b2167.png)

SQL Mapper

장점: JDBC를 편리하게 사용하도록 도와준다. SQL 응답 결과를 객체로 편리하게 변환해준다. JDBC의 반복 코드를 제거해준다.

단점: 개발자가 SQL을 직접 작성해야한다.

대표 기술: 스프링 JdbcTemplate, MyBatis


![image](https://user-images.githubusercontent.com/65898555/180720709-5d2e454d-80b2-4a60-b801-05dcaf3b2a75.png)

ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술이다. 이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어
실행해준다. 추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다.

대표 기술: JPA, 하이버네이트, 이클립스링크

JPA는 자바 진영의 ORM 표준 인터페이스이고, 이것을 구현한 것으로 하이버네이트와 이클립스 링크 등의 구현 기술이 있다.

---

SQL Mapper vs ORM 기술

SQL Mapper와 ORM 기술 둘다 각각 장단점이 있다.

쉽게 설명하자면 SQL Mapper는 SQL만 직접 작성하면 나머지 번거로운 일은 SQL Mapper가 대신 해결해준다. SQL Mapper는 SQL만 작성할 줄 알면 금방 배워서 사용할 수 있다.

ORM기술은 SQL 자체를 작성하지 않아도 되어서 개발 생산성이 매우 높아진다. 편리한 반면에 쉬운 기술은 아니므로 실무에서 사용하려면 깊이있게 학습해야 한다.


---

이런 기술들도 내부에서는 모두 JDBC를 사용한다. 따라서 JDBC를 직접 사용하지는 않더라도, JDBC가 어떻게 동작하는지 기본 원리를 알아두어야 한다. 그래야 해당 기술들을 더 깊이있게 이해할 수 있고, 
무엇보다 문제가 발생했을 때 근본적인 문제를 찾아서 해결할 수 있다.



# JDBC 연결

```java
public abstract class ConnectionConst {
 public static final String URL = "jdbc:h2:tcp://localhost/~/test";
 public static final String USERNAME = "sa";
 public static final String PASSWORD = "";
}
```
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import static hello.jdbc.connection.ConnectionConst.*;
@Slf4j
public class DBConnectionUtil {
  public static Connection getConnection() {
    try {
      Connection connection = DriverManager.getConnection(URL, USERNAME,PASSWORD);
      log.info("get connection={}, class={}", connection,
      connection.getClass());
      return connection;
    } catch (SQLException e) {
      throw new IllegalStateException(e);
    }
  }
}
```
데이터베이스에 연결하려면 JDBC가 제공하는 DriverManager.getConnection(..) 를 사용하면 된다. 

이렇게 하면 라이브러리에 있는 데이터베이스 드라이버를 찾아서 해당 드라이버가 제공하는 커넥션을 반환해준다


### JDBC DriverManager 연결 이해

![image](https://user-images.githubusercontent.com/65898555/180721434-ceb68daa-5615-49ed-9900-efb3c42b9c4b.png)

JDBC는 java.sql.Connection 표준 커넥션 인터페이스를 정의한다.

H2 데이터베이스 드라이버는 JDBC Connection 인터페이스를 구현한 org.h2.jdbc.JdbcConnection 구현체를 제공한다.

![image](https://user-images.githubusercontent.com/65898555/180721570-bb80f302-ba49-4450-85a8-29372d79220a.png)

JDBC가 제공하는 DriverManager 는 라이브러리에 등록된 DB 드라이버들을 관리하고, 커넥션을 획득하는 기능을 제공한다.

---

1. 애플리케이션 로직에서 커넥션이 필요하면 DriverManager.getConnection() 을 호출한다.

2. DriverManager는 라이브러리에 등록된 드라이버 목록을 자동으로 인식한다. 이 드라이버들에게 순서대로 다음 정보를 넘겨서 커넥션을 획득할 수 있는지 확인한다. 

3. 본인이 처리할 수 있으면 데이터베이스에 연결해서 커넥션을 획득하고 이 커넥션을 클라이언트에 반환한다. 

4. 처리할 수 없다면 URL이 본인이 처리할 수 없다는 결과를 반환하게 되고, 다음 드라이버에게 순서가 넘어간다.

5. 이렇게 찾은 커넥션 구현체가 클라이언트에 반환된다.


```java
import hello.jdbc.connection.DBConnectionUtil;
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import java.sql.*;
/**
 * JDBC - DriverManager 사용
 */
@Slf4j
public class MemberRepositoryV0 {

  public Member save(Member member) throws SQLException {
    String sql = "insert into member(member_id, money) values(?, ?)";
    Connection con = null;
    PreparedStatement pstmt = null;
    try {
      con = getConnection();
      pstmt = con.prepareStatement(sql);
      pstmt.setString(1, member.getMemberId());
      pstmt.setInt(2, member.getMoney());
      pstmt.executeUpdate();
      return member;
    } catch (SQLException e) {
      log.error("db error", e);
      throw e;
    } finally {
      close(con, pstmt, null);
    }
  }
  
  private void close(Connection con, Statement stmt, ResultSet rs) {
    if (rs != null) {
      try {
        rs.close();
      } catch (SQLException e) {
        log.info("error", e);
      }
   }
   if (stmt != null) {
      try {
      stmt.close();
    } catch (SQLException e) {
      log.info("error", e);
    }
  }
  if (con != null) {
    try {
    con.close();
    } catch (SQLException e) {
      log.info("error", e);
    }
  }
 }
 
 private Connection getConnection() {
  return DBConnectionUtil.getConnection();
 }
}
```

con.prepareStatement(sql) : 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터들을 준비한다.

pstmt.executeUpdate() : Statement 를 통해 준비된 SQL을 커넥션을 통해 실제 데이터베이스에 전달한다. 참고로 executeUpdate() 은 int 를 반환하는데 영향받은 DB row 수를 반환한다. 여기서는
하나의 row를 등록했으므로 1을 반환한다.

쿼리를 실행하고 나면 리소스를 정리해야 한다. 여기서는 Connection , PreparedStatement를 사용했다. 리소스를 정리할 때는 항상 역순으로 해야한다. Connection을 먼저 획득하고 Connection을
통해 PreparedStatement를 만들었기 때문에 리소스를 반환할 때는 PreparedStatement를 먼저 종료하고, 그 다음에 Connection 을 종료하면 된다. 참고로 여기서 사용하지 않은 ResultSet 은 결과를
조회할 때 사용한다.


```java
public Member findById(String memberId) throws SQLException {
  String sql = "select * from member where member_id = ?";
  Connection con = null;
  PreparedStatement pstmt = null;
  ResultSet rs = null;
  try {
    con = getConnection();
    pstmt = con.prepareStatement(sql);
    pstmt.setString(1, memberId);
    rs = pstmt.executeQuery();
    if (rs.next()) {
      Member member = new Member();
      member.setMemberId(rs.getString("member_id"));
      member.setMoney(rs.getInt("money"));
      return member;
    } else {
      throw new NoSuchElementException("member not found memberId=" +memberId);
    }
  } catch (SQLException e) {
    log.error("db error", e);
      throw e;
    } finally {
      close(con, pstmt, rs);
  }
}
```
rs = pstmt.executeQuery() 데이터를 변경할 때는 executeUpdate()를 사용하지만, 데이터를 조회할 때는 executeQuery()를 사용한다. executeQuery()는 결과를 ResultSet에 담아서 반환한다.


![image](https://user-images.githubusercontent.com/65898555/180723315-9d25a89a-00d5-41fc-89cf-4dcc04627c1f.png)



ResultSet 내부에 있는 커서( cursor )를 이동해서 다음 데이터를 조회할 수 있다.

rs.next() : 이것을 호출하면 커서가 다음으로 이동한다. 참고로 최초의 커서는 데이터를 가리키고 있지않기 때문에 rs.next()를 최초 한번은 호출해야 데이터를 조회할 수 있다.

rs.next()의 결과가 true 면 커서의 이동 결과 데이터가 있다는 뜻이다.

rs.next()의 결과가 false 면 더이상 커서가 가리키는 데이터가 없다는 뜻이다.

rs.getString("member_id") : 현재 커서가 가리키고 있는 위치의 member_id 데이터를 String 타입으로 반환한다.

rs.getInt("money") : 현재 커서가 가리키고 있는 위치의 money 데이터를 int 타입으로 반환한다.

조회 결과가 항상 1건이므로 while 대신에 if를 사용한다.


```java
public void update(String memberId, int money) throws SQLException {
  String sql = "update member set money=? where member_id=?";
  Connection con = null;
  PreparedStatement pstmt = null;
  try {
   con = getConnection();
   pstmt = con.prepareStatement(sql);
   pstmt.setInt(1, money);
   pstmt.setString(2, memberId);
   int resultSize = pstmt.executeUpdate();
   log.info("resultSize={}", resultSize);
  } catch (SQLException e) {
   log.error("db error", e);
   throw e;
  } finally {
   close(con, pstmt, null);
  }
}

public void delete(String memberId) throws SQLException {
  String sql = "delete from member where member_id=?";
  Connection con = null;
  PreparedStatement pstmt = null;
  try {
   con = getConnection();
   pstmt = con.prepareStatement(sql);
   pstmt.setString(1, memberId);
   pstmt.executeUpdate();
  } catch (SQLException e) {
   log.error("db error", e);
   throw e;
  } finally {
   close(con, pstmt, null);
  }
}
```
수정과 삭제는 등록과 비슷하다. 등록, 수정, 삭제처럼 데이터를 변경하는 쿼리는 executeUpdate()를 사용하면 된다.

executeUpdate()는 쿼리를 실행하고 영향받은 row수를 반환한다.







