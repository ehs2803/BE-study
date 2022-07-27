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



# 커넥션풀

![image](https://user-images.githubusercontent.com/65898555/180898443-3d78286a-a9b9-424a-95cf-6fd83cb28bf8.png)

데이터베이스 커넥션을 획득할 때는 다음과 같은 복잡한 과정을 거친다.

1. 애플리케이션 로직은 DB 드라이버를 통해 커넥션을 조회한다.
2. DB 드라이버는 DB와 TCP/IP 커넥션을 연결한다. 이 과정에서 3 way handshake 같은 TCP/IP 연결을 위한 네트워크 동작이 발생한다.
3. DB 드라이버는 TCP/IP 커넥션이 연결되면 ID, PW와 기타 부가정보를 DB에 전달한다.
4. DB는 ID, PW를 통해 내부 인증을 완료하고, 내부에 DB 세션을 생성한다.
5. DB는 커넥션 생성이 완료되었다는 응답을 보낸다.
6. DB 드라이버는 커넥션 객체를 생성해서 클라이언트에 반환한다.

DB는 물론이고 애플리케이션 서버에서도 TCP/IP 커넥션을 새로 생성하기 위한 리소스를 매번 사용해야 한다.

데이터베이스마다 커넥션을 생성하는 시간은 다르다. 시스템 상황마다 다르지만 MySQL 계열은 수 ms(밀리초) 정도로 매우 빨리 커넥션을 확보할 수 있다. 반면에 수십 밀리초 이상 걸리는 데이터베이스들도 있다.

![image](https://user-images.githubusercontent.com/65898555/180898575-92b6f73f-2fd6-4a79-bb1d-d6a624be0fba.png)

애플리케이션을 시작하는 시점에 커넥션 풀은 필요한 만큼 커넥션을 미리 확보해서 풀에 보관한다. 보통 얼마나 보관할 지는 서비스의 특징과 서버 스펙에 따라 다르지만 기본값은 보통 10개이다

![image](https://user-images.githubusercontent.com/65898555/180898591-cabebcae-8678-4c14-838e-57d60589319e.png)

커넥션 풀에 들어 있는 커넥션은 TCP/IP로 DB와 커넥션이 연결되어 있는 상태이기 때문에 언제든지 즉시 SQL을 DB에 전달할 수 있다.

![image](https://user-images.githubusercontent.com/65898555/180898666-6eed4849-241e-462e-a403-0c1eeddb64a0.png)

애플리케이션 로직에서 DB 드라이버를 통해서 새로운 커넥션을 획득하는 것이 아니다. 커넥션 풀을 통해 이미 생성되어 있는 커넥션을 객체 참조로 그냥 가져다 쓰기만 하면 된다.
커넥션 풀에 커넥션을 요청하면 커넥션 풀은 자신이 가지고 있는 커넥션 중에 하나를 반환한다.

![image](https://user-images.githubusercontent.com/65898555/180898729-c9b56458-83b4-46c2-bc5b-7f5759744c7a.png)

애플리케이션 로직은 커넥션 풀에서 받은 커넥션을 사용해서 SQL을 데이터베이스에 전달하고 그 결과를 받아서 처리한다.

커넥션을 모두 사용하고 나면 이제는 커넥션을 종료하는 것이 아니라, 다음에 다시 사용할 수 있도록 해당 커넥션을 그대로 커넥션 풀에 반환하면 된다. 여기서 주의할 점은 커넥션을 종료하는 것이 아니라 커넥션이 살아있는 상태로 커넥션 풀에 반환해야 한다는 것이다.


커넥션 풀은 서버당 최대 커넥션 수를 제한할 수 있다. 따라서 DB에 무한정 연결이 생성되는 것을 막아주어서 DB를 보호하는 효과도 있다.
이런 커넥션 풀은 얻는 이점이 매우 크기 때문에 실무에서는 항상 기본으로 사용한다.

커넥션 풀은 개념적으로 단순해서 직접 구현할 수도 있지만, 사용도 편리하고 성능도 뛰어난 오픈소스 커넥션 풀이 많기 때문에 오픈소스를 사용하는 것이 좋다.

대표적인 커넥션 풀 오픈소스는 commons-dbcp2 , tomcat-jdbc pool , HikariCP 등이 있다.

성능과 사용의 편리함 측면에서 최근에는 hikariCP 를 주로 사용한다. 스프링 부트 2.0 부터는 기본 커넥션 풀로 hikariCP 를 제공한다. 성능, 사용의 편리함, 안전성 측면에서 이미 검증이 되었기 때문에 커넥션 풀을 사용할 때는 고민할 것 없이 hikariCP 를 사용하면 된다. 


# DataSource


커넥션을 얻는 방법은 JDBC DriverManager를 직접 사용하거나, 커넥션 풀을 사용하는 등 다양한 방법이 존재한다.

![image](https://user-images.githubusercontent.com/65898555/180898962-86055ffc-f1a5-45d9-aa29-3df3363eb0ae.png)

애플리케이션 로직에서 DriverManager를 사용해서 커넥션을 획득하다가 HikariCP 같은 커넥션 풀을 사용하도록 변경하면 커넥션을 획득하는 애플리케이션 코드도 함께 변경해야 한다. 의존관계가 DriverManager 에서 HikariCP 로 변경되기 때문이다. 


![image](https://user-images.githubusercontent.com/65898555/180899041-d6d68394-68db-4fe1-93a7-87a8a1664145.png)

자바에서는 이런 문제를 해결하기 위해 javax.sql.DataSource라는 인터페이스를 제공한다.

DataSource는 커넥션을 획득하는 방법을 추상화 하는 인터페이스이다. 이 인터페이스의 핵심 기능은 커넥션 조회 하나이다. (다른 일부 기능도 있지만 크게 중요하지 않다.)

---

대부분의 커넥션 풀은 DataSource 인터페이스를 이미 구현해두었다. 따라서 개발자는 DBCP2 커넥션 풀 , HikariCP 커넥션 풀 의 코드를 직접 의존하는 것이 아니라 DataSource 인터페이스에만 의존하도록 애플리케이션 로직을 작성하면 된다.  커넥션 풀 구현 기술을 변경하고 싶으면 해당 구현체로 갈아끼우기만 하면 된다.

DriverManager는 DataSource 인터페이스를 사용하지 않는다. 따라서 DriverManager 는 직접 사용해야 한다. 따라서 DriverManager를 사용하다가 DataSource 기반의 커넥션 풀을 사용하도록
변경하면 관련 코드를 다 고쳐야 한다. 이런 문제를 해결하기 위해 스프링은 DriverManager도 DataSource를 통해서 사용할 수 있도록 DriverManagerDataSource라는 DataSource를 구현한 클래스를 제공한다.

자바는 DataSource를 통해 커넥션을 획득하는 방법을 추상화했다. 이제 애플리케이션 로직은 DataSource 인터페이스에만 의존하면 된다. 덕분에 DriverManagerDataSource를 통해서
DriverManager를 사용하다가 커넥션 풀을 사용하도록 코드를 변경해도 애플리케이션 로직은 변경하지 않아도 된다.



```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import static hello.jdbc.connection.ConnectionConst.*;
@Slf4j
public class ConnectionTest {
  @Test
  void driverManager() throws SQLException {
   Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
   Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
   log.info("connection={}, class={}", con1, con1.getClass());
   log.info("connection={}, class={}", con2, con2.getClass());
  }
  
  @Test
  void dataSourceDriverManager() throws SQLException {
   //DriverManagerDataSource - 항상 새로운 커넥션 획득
   DriverManagerDataSource dataSource = new DriverManagerDataSource(URL,USERNAME, PASSWORD);
   useDataSource(dataSource);
  }
  
  private void useDataSource(DataSource dataSource) throws SQLException {
   Connection con1 = dataSource.getConnection();
   Connection con2 = dataSource.getConnection();
   log.info("connection={}, class={}", con1, con1.getClass());
   log.info("connection={}, class={}", con2, con2.getClass());
  }
}
```
기존 코드와 비슷하지만 DriverManagerDataSource 는 DataSource 를 통해서 커넥션을 획득할 수 있다. DriverManagerDataSource는 스프링이 제공하는 코드이다.

DriverManager는 커넥션을 획득할 때 마다 URL , USERNAME , PASSWORD 같은 파라미터를 계속 전달해야 한다. 반면에 DataSource를 사용하는 방식은 처음 객체를 생성할 때만 필요한 파리미터를 넘겨두고, 커넥션을 획득할 때는 단순히 dataSource.getConnection()만 호출하면 된다. 이렇게 설정과 사용을 분리했다. 


```java
import com.zaxxer.hikari.HikariDataSource;
@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
  //커넥션 풀링: HikariProxyConnection(Proxy) -> JdbcConnection(Target)
  HikariDataSource dataSource = new HikariDataSource();
  dataSource.setJdbcUrl(URL);
  dataSource.setUsername(USERNAME);
  dataSource.setPassword(PASSWORD);
  dataSource.setMaximumPoolSize(10);
  dataSource.setPoolName("MyPool");
  useDataSource(dataSource);
  Thread.sleep(1000); //커넥션 풀에서 커넥션 생성 시간 대기
}
```
HikariDataSource는 DataSource 인터페이스를 구현하고 있다.
커넥션 풀 최대 사이즈를 10으로 지정하고, 풀의 이름을 MyPool이라고 지정했다.
커넥션 풀에서 커넥션을 생성하는 작업은 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동한다. 별도의 쓰레드에서 동작하기 때문에 테스트가 먼저 종료되어 버린다. 예제처럼 Thread.sleep을 통해 대기 시간을 주어야 쓰레드 풀에 커넥션이 생성되는 로그를 확인할 수 있다.

커넥션 풀에 커넥션을 채우는 것은 상대적으로 오래 걸리는 일이다. 애플리케이션을 실행할 때 커넥션 풀을 채울 때 까지 마냥 대기하고 있다면 애플리케이션 실행 시간이 늦어진다. 따라서 이렇게 별도의 쓰레드를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.


```java
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.support.JdbcUtils;
import javax.sql.DataSource;
import java.sql.*;
import java.util.NoSuchElementException;
/**
 * JDBC - DataSource 사용, JdbcUtils 사용
 */
@Slf4j
public class MemberRepositoryV1 {

  private final DataSource dataSource;
  
  public MemberRepositoryV1(DataSource dataSource) {
   this.dataSource = dataSource;
  }
  
  //save()...
  //findById()...
  //update()....
  //delete()....
  
  private void close(Connection con, Statement stmt, ResultSet rs) {
   JdbcUtils.closeResultSet(rs);
   JdbcUtils.closeStatement(stmt);
   JdbcUtils.closeConnection(con);
  }
  
  private Connection getConnection() throws SQLException {
   Connection con = dataSource.getConnection();
   log.info("get connection={}, class={}", con, con.getClass());
   return con;
  }
}
```
```java
import com.zaxxer.hikari.HikariDataSource;
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import java.sql.SQLException;
import java.util.NoSuchElementException;
import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

@Slf4j
class MemberRepositoryV1Test {
  MemberRepositoryV1 repository;
  @BeforeEach
  void beforeEach() throws Exception {
   //기본 DriverManager - 항상 새로운 커넥션 획득
   //DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
   //커넥션 풀링: HikariProxyConnection -> JdbcConnection
   HikariDataSource dataSource = new HikariDataSource();
   dataSource.setJdbcUrl(URL);
   dataSource.setUsername(USERNAME);
   dataSource.setPassword(PASSWORD);
   repository = new MemberRepositoryV1(dataSource);
  }
  
  @Test
  void crud() throws SQLException, InterruptedException {
   log.info("start");
   //save
   Member member = new Member("memberV0", 10000);
   repository.save(member);
   //findById
   Member memberById = repository.findById(member.getMemberId());
   assertThat(memberById).isNotNull();
   //update: money: 10000 -> 20000
   repository.update(member.getMemberId(), 20000);
   Member updatedMember = repository.findById(member.getMemberId());
   assertThat(updatedMember.getMoney()).isEqualTo(20000);
   //delete
   repository.delete(member.getMemberId());
   assertThatThrownBy(() -> repository.findById(member.getMemberId())).isInstanceOf(NoSuchElementException.class);
  }
}
```
DataSource 의존관계 주입 : 외부에서 DataSource 를 주입 받아서 사용한다. 이제 직접 만든 DBConnectionUtil을 사용하지 않아도 된다.

DataSource는 표준 인터페이스 이기때문에 DriverManagerDataSource 에서 HikariDataSource로 변경되어도 해당 코드를 변경하지 않아도 된다.

JdbcUtils 편의 메서드 : 스프링은 JDBC를 편리하게 다룰 수 있는 JdbcUtils 라는 편의 메서드를 제공한다. JdbcUtils을 사용하면 커넥션을 좀 더 편리하게 닫을 수 있다.



---

DriverManagerDataSource HikariDataSource로 변경해도 MemberRepositoryV1의 코드는 전혀 변경하지 않아도 된다. 

MemberRepositoryV1는 DataSource인터페이스에만 의존하기 때문이다. 이것이 DataSource를 사용하는 장점이다.(DI + OCP)


# JDBC 반복 문제 해결 - JdbcTemplate

리포지토리에서 JDBC를 사용하기 때문에 발생하는 반복 문제가 있다.

- 커넥션 조회, 커넥션 동기화
- PreparedStatement 생성 및 파라미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생시 스프링 예외 변환기 실행
- 리소스 종료


리포지토리의 각각의 메서드를 살펴보면 상당히 많은 부분이 반복된다. 이런 반복을 효과적으로 처리하는 방법이 바로 템플릿 콜백 패턴이다.
스프링은 JDBC의 반복 문제를 해결하기 위해 JdbcTemplate이라는 템플릿을 제공한다.

```java
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import javax.sql.DataSource;
/**
 * JdbcTemplate 사용
 */
@Slf4j
public class MemberRepositoryV5 implements MemberRepository {
  private final JdbcTemplate template;
  
  public MemberRepositoryV5(DataSource dataSource) {
   template = new JdbcTemplate(dataSource);
  }
  
  @Override
  public Member save(Member member) {
   String sql = "insert into member(member_id, money) values(?, ?)";
   template.update(sql, member.getMemberId(), member.getMoney());
   return member;
  }
  
  @Override
  public Member findById(String memberId) {
   String sql = "select * from member where member_id = ?";
   return template.queryForObject(sql, memberRowMapper(), memberId);
  }
  
  @Override
  public void update(String memberId, int money) {
   String sql = "update member set money=? where member_id=?";
   template.update(sql, money, memberId);
  }
  
  @Override
  public void delete(String memberId) {
   String sql = "delete from member where member_id=?";
   template.update(sql, memberId);
  }
  
  private RowMapper<Member> memberRowMapper() {
   return (rs, rowNum) -> {
   Member member = new Member();
   member.setMemberId(rs.getString("member_id"));
   member.setMoney(rs.getInt("money"));
   return member;};
  }
}
```
JdbcTemplate은 JDBC로 개발할 때 발생하는 반복을 대부분 해결해준다. 그 뿐만 아니라 트랜잭션을 위한 커넥션 동기화는 물론이고, 예외 발생시 스프링 예외 변환기도 자동으로 실행해준다.

