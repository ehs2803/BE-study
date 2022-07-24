# 문제점

JPA는 다양한 쿼리 방법을 지원. JPQL, JPA Criteria, QueryDSL, 네이티브 SQL, JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용 등 방법이 많다.

JPA를 사용하면 엔티티 객체를 중심으로 개발

<hr>

문제는 검색 쿼리. 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색

모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능. 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

# Criteria

```java
//Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder(); 
CriteriaQuery<Member> query = cb.createQuery(Member.class); 

//루트 클래스 (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class); 

//쿼리 생성 
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), “kim”)); 
List<Member> resultList = em.createQuery(cq).getResultList();
```
문자가 아닌 자바코드로 JPQL을 작성할 수 있음

JPQL 빌더 역할. JPA 공식 기능

단점: 너무 복잡하고 실용성이 없다. 

Criteria 대신에 QueryDSL 사용 권장

# QueryDSL

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member; 
List<Member> list = query.selectFrom(m)
                        .where(m.age.gt(18)) 
                        .orderBy(m.name.desc())
                        .fetch();
```
문자가 아닌 자바코드로 JPQL을 작성할 수 있음

JPQL 빌더 역할. 컴파일 시점에 문법 오류를 찾을 수 있음

동적쿼리 작성 편리함. 단순하고 쉬움

실무 사용 권장

# 네이티브 SQL 소개

```java
String sql = “SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’"; 
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList(); 
```
JPA가 제공하는 SQL을 직접 사용하는 기능

JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

# JDBC, SpringjdbcTemplate 등

JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능

단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요  예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시



# JPQL(Java Persistence Query Language)(객체지향 쿼리 언어)

JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리 하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다. 

JPQL은 SQL을 추상화해서 특정데이터베이스 SQL에 의존하 지 않는다. 

JPQL은 결국 SQL로 변환된다.

SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원


```java
//검색
 String jpql = "select m From Member m where m.name like ‘%hello%'"; 
 List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```
테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리. SQL을 추상화해서 특정 데이터베이스 SQL에 의존X. JPQL을 한마디로 정의하면 객체 지향 SQL


# JPQL 문법


<img width="607" alt="image" src="https://user-images.githubusercontent.com/65898555/180604259-d229bff0-b90d-49ad-a80b-883c13f5d35f.png">

```java
select m from Member as m where m.age > 18 
```
엔티티와 속성은 대소문자 구분O (Member, age) 

JPQL 키워드는 대소문자 구분X (SELECT, FROM, where) 

엔티티 이름 사용, 테이블 이름이 아님(Member) 

별칭은 필수(m) (as는 생략가능)


### 집합과 정렬

```java
select
 COUNT(m), //회원수
 SUM(m.age), //나이 합
 AVG(m.age), //평균 나이
 MAX(m.age), //최대 나이
 MIN(m.age) //최소 나이
from Member m
```
GROUP BY, HAVING, ORDER BY 지원


### TypeQuery, Query

```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
```
TypeQuery: 반환 타입이 명확할 때 사용

```java
Query query = em.createQuery("SELECT m.username, m.age from Member m"); 
```
Query: 반환 타입이 명확하지 않을 때 사용

### 결과 조회 API

query.getResultList(): 결과가 하나 이상일 때, 리스트 반환. 결과가 없으면 빈 리스트 반환

query.getSingleResult(): 결과가 정확히 하나, 단일 객체 반환. 결과가 없으면: javax.persistence.NoResultException. 둘 이상이면: javax.persistence.NonUniqueResultException

### 파라미터 바인딩 - 이름기준, 위치기준

```java
SELECT m FROM Member m where m.username=:username 

query.setParameter("username", usernameParam);
```

```java
SELECT m FROM Member m where m.username=?1 

query.setParameter(1, usernameParam);
```
이름기준 바인딩 사용 권장


### 프로젝션

SELECT 절에 조회할 대상을 지정하는 것

프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입) 

DISTINCT로 중복 제거 가능

- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
- SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션


### 프로젝션 - 여러 값 조회

```java
SELECT m.username, m.age FROM Member m 
```
1. Query 타입으로 조회
2. Object[] 타입으로 조회
3. new 명령어로 조회<br>
단순 값을 DTO로 바로 조회
```SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m ``` 패키지 명을 포함한 전체 클래스 명 입력. 순서와 타입이 일치하는 생성자 필요


### 페이징 API

JPA는 페이징을 다음 두 API로 추상화

- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작) 
- setMaxResults(int maxResult) : 조회할 데이터 수

```java
//페이징 쿼리
 String jpql = "select m from Member m order by m.name desc";
 List<Member> resultList = em.createQuery(jpql, Member.class)
                          .setFirstResult(10)
                          .setMaxResults(20)
                          .getResultList();
 ```

### 조인

내부 조인 ```SELECT m FROM Member m [INNER] JOIN m.team t```

외부 조인 ```SELECT m FROM Member m LEFT [OUTER] JOIN m.team t ```

세타 조인 ```select count(m) from Member m, Team t where m.username = t.name```


### 조인 - ON 절

ON절을 활용한 조인(JPA 2.1부터 지원) 

1. 조인 대상 필터링

예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

```
JPQL:
SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A' 

SQL:
SELECT m.*, t.* FROM 
Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A' 
```

3. 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)

예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

```
JPQL:
SELECT m, t FROM
Member m LEFT JOIN Team t on m.username = t.name

SQL:
SELECT m.*, t.* FROM 
Member m LEFT JOIN Team t ON m.username = t.name
```


### 서브 쿼리

```
select m from Member m
where m.age > (select avg(m2.age) from Member m2) 
```
나이가 평균보다 많은 회원

```
select m from Member m
where (select count(o) from Order o where m = o.member) > 0 
```
한 건이라도 주문한 고객

<hr>

[NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참<br>
- {ALL | ANY | SOME} (subquery) 
- ALL 모두 만족하면 참
- ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참

[NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참


<hr>

```
select m from Member m
where exists (select t from m.team t where t.name = ‘팀A') 
```
팀A 소속인 회원

```
select o from Order o 
where o.orderAmount > ALL (select p.stockAmount from Product p) 
```
전체 상품 각각의 재고보다 주문량이 많은 주문들

```
select m from Member m 
where m.team = ANY (select t from Team t)
```
어떤 팀이든 팀에 소속된 회원

<hr>

JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능

SELECT 절도 가능(하이버네이트에서 지원) 

FROM 절의 서브 쿼리는 현재 JPQL에서 불가능

조인으로 풀 수 있으면 풀어서 해결


### JPQL 타입 표현

- 문자: ‘HELLO’, ‘She’’s’ 
- 숫자: 10L(Long), 10D(Double), 10F(Float) 
- Boolean: TRUE, FALSE 
- ENUM: jpabook.MemberType.Admin (패키지명 포함) 
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)

### 조건식 - CASE 식

<img width="575" alt="image" src="https://user-images.githubusercontent.com/65898555/180604851-3d3dcf88-fcb6-4f07-b6f3-731e46858617.png">

<img width="846" alt="image" src="https://user-images.githubusercontent.com/65898555/180604869-3349f95a-5b53-4e28-bdae-0de8e71f7841.png">

COALESCE: 하나씩 조회해서 null이 아니면 반환

NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

### JPQL 기본 함수

- CONCAT 
- SUBSTRING 
- TRIM 
- LOWER, UPPER 
- LENGTH 
- LOCATE 
- ABS, SQRT, MOD 
- SIZE, INDEX(JPA 용도)

### 사용자 정의 함수 호출

```java
select function('group_concat', i.name) from Item i
```
하이버네이트는 사용전 방언에 추가해야 한다. 

사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다

### JPQL 기타

SQL과 문법이 같은 식

EXISTS, IN 

AND, OR, NOT 

=, >, >=, <, <=, <> 

BETWEEN, LIKE, IS NULL


# JPQL - 경로 표현식

.(점)을 찍어 객체 그래프를 탐색하는 것

```java
select m.username -> 상태 필드
 from Member m 
 join m.team t -> 단일 값 연관 필드
 join m.orders o -> 컬렉션 값 연관 필드
where t.name = '팀A'
```

상태 필드(state field): 단순히 값을 저장하기 위한 필드 (ex: m.username) 

연관 필드(association field): 연관관계를 위한 필드<br>
- 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티(ex: m.team) 
- 컬렉션 값 연관 필드: @OneToMany, @ManyToMany, 대상이 컬렉션(ex: m.orders)

---

상태 필드(state field): 경로 탐색의 끝, 탐색X 

단일 값 연관 경로: 묵시적 내부 조인(inner join) 발생, 탐색O 

컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 탐색X. FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능

---

```
JPQL: select m.username, m.age from Member m 

SQL: select m.username, m.age from Member m
```
상태필드 경로탐색

```
JPQL: select o.member from Order o 

SQL:
select m.* 
 from Orders o 
 inner join Member m on o.member_id = m.id
```
단일 값 연관 경로 탐색


```
select m from Member m join m.team t

select m.team from Member m
```
명시적 조인: join 키워드 직접 사용

묵시적 조인: 경로 표현식에 의해 묵시적으로 SQL 조인 발생(내부 조인만 가능) 

묵시적 조인은 항상 내부 조인. 컬렉션은 경로 탐색의 끝, 명시적 조인을 통해 별칭을 얻어야함

경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM (JOIN) 절에 영향을 줌

가급적 묵시적 조인 대신에 명시적 조인 사용 권장


# JPQL - 페치 조인(fetch join)

SQL 조인 종류X. JPQL에서 성능 최적화를 위해 제공하는 기능. 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능

```
// join fetch 명령어 사용
[ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로
```

```
[JPQL]
select m from Member m join fetch m.team 

[SQL]
SELECT M.*, T.* FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```
회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에) - fetch join 사용


```java
String jpql = "select m from Member m join fetch m.team"; 
List<Member> members = em.createQuery(jpql, Member.class).getResultList(); 

for (Member member : members) {
 //페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
 System.out.println("username = " + member.getUsername() + ", " + 
 "teamName = " + member.getTeam().name()); 
}
```

```
[JPQL]
select t
from Team t join fetch t.members
where t.name = ‘팀A' 

• [SQL]
SELECT T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A' 
```
```java
String jpql = "select t from Team t join fetch t.members where t.name = '팀A'" 
List<Team> teams = em.createQuery(jpql, Team.class).getResultList(); 

for(Team team : teams) { 
 System.out.println("teamname = " + team.getName() + ", team = " + team); 
 for (Member member : team.getMembers()) { 
  //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩 발생 안함
  System.out.println(“-> username = " + member.getUsername()+ ", member = " + member); 
 } 
}
```
![image](https://user-images.githubusercontent.com/65898555/180627107-93b71acc-bceb-478d-bd61-bc3cbc8ffe98.png)

컬렉션 페치 조인 - 일대다 관계, 컬렉션 페치 조인


```
select distinct t
from Team t join fetch t.members
where t.name = ‘팀A’ 
```
SQL의 DISTINCT는 중복된 결과를 제거하는 명령

JPQL의 DISTINCT 2가지 기능 제공<br>
1. SQL에 DISTINCT를 추가
2. 애플리케이션에서 엔티티 중복 제거


![image](https://user-images.githubusercontent.com/65898555/180627293-01004a4a-b6b6-4623-b5e1-f788cf84c979.png)

distinct를 애플리케이션에서 엔티티 중복 제거하는 예시이다.



---

페치 조인 대상에는 별칭을 줄 수 없다. 하이버네이트는 가능, 가급적 사용X 

둘 이상의 컬렉션은 페치 조인 할 수 없다. 

컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다. 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능. 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)

---

연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화

엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함 ex) @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략

실무에서 글로벌 로딩 전략은 모두 지연 로딩. 최적화가 필요한 곳은 페치 조인 적용


# JPQL - 다형성 쿼리

![image](https://user-images.githubusercontent.com/65898555/180627339-85f79f44-400e-4c52-b5fa-b54b2802c617.png)

```
[JPQL]
select i from Item i
where type(i) IN (Book, Movie) 

[SQL]
select i from i
where i.DTYPE in (‘B’, ‘M’)
```
조회 대상을 특정 자식으로 한정 예) Item 중에 Book, Movie를 조회해라

```
[JPQL]
select i from Item i
where treat(i as Book).auther = ‘kim’ 

[SQL]
select i.* from Item i
where i.DTYPE = ‘B’ and i.auther = ‘kim’
```
TREAT(JPA 2.1)가 있다. 자바의 타입 캐스팅과 유사

상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용. FROM, WHERE, SELECT(하이버네이트 지원) 사용


# JPQL - 엔티티 직접 사용

```
[JPQL]
select count(m.id) from Member m //엔티티의 아이디를 사용
select count(m) from Member m //엔티티를 직접 사용

[SQL](JPQL 둘다 같은 다음 SQL 실행)
select count(m.id) as cnt from Member m
```
```java
String jpql = “select m from Member m where m = :member”; 
List resultList = em.createQuery(jpql) 
 .setParameter("member", member) 
 .getResultList(); 
```
엔티티를 전달하는 예지이다.

```java
String jpql = “select m from Member m where m.id = :memberId”; 
List resultList = em.createQuery(jpql) 
 .setParameter("memberId", memberId) 
 .getResultList(); 
```
식별자를 직접 전달하는 예시이다.

```
select m.* from Member m where m.id=?
```
두개모두 실행된 sql는 위와 같다.

```java
Team team = em.find(Team.class, 1L); 

String qlString = “select m from Member m where m.team = :team”; 
List resultList = em.createQuery(qlString) 
 .setParameter("team", team) 
 .getResultList(); 
 
String qlString = “select m from Member m where m.team.id = :teamId”; 
List resultList = em.createQuery(qlString) 
 .setParameter("teamId", teamId) 
 .getResultList();
```
```
select m.* from Member m where m.team_id=? // 실행된 sql
```
외래 키 값을 사용하는 경우이다.



# JPQL - Named 쿼리

정적 쿼리이며 미리 정의해서 이름을 부여해두고 사용하는 JPQL 

어노테이션, XML에 정의하고, 애플리케이션 로딩 시점에 초기화 후 재사용하고 로딩 시점에 쿼리를 검증.

```java
@Entity
@NamedQuery(
 name = "Member.findByUsername",
 query="select m from Member m where m.username = :username")
public class Member {
 ...
}

List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                          .setParameter("username", "회원1")
                          .getResultList();
```
어노테이션에 정의해 사용하는 코드이다.

```xml
[META-INF/persistence.xml]
<persistence-unit name="jpabook" >
 <mapping-file>META-INF/ormMember.xml</mapping-file>

[META-INF/ormMember.xml]
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
 <named-query name="Member.findByUsername">
 <query><![CDATA[
 select m
 from Member m
 where m.username = :username
 ]]></query>
 </named-query>
 <named-query name="Member.count">
 <query>select count(m) from Member m</query>
 </named-query>
</entity-mappings>
```
xml에 정의하는 것이다. XML이 항상 우선권을 가진다. 애플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다.



# JPQL - 벌크 연산

재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면? 

JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL 실행

1. 재고가 10개 미만인 상품을 리스트로 조회한다. 
2. 상품 엔티티의 가격을 10% 증가한다. 
3. 트랜잭션 커밋 시점에 변경감지가 동작한다. 

변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

```java
String qlString = "update Product p " +
                  "set p.price = p.price * 1.1 " + 
                  "where p.stockAmount < :stockAmount"; 
                  
int resultCount = em.createQuery(qlString) 
                    .setParameter("stockAmount", 10) 
                    .executeUpdate(); 
```
쿼리 한 번으로 여러 테이블 로우 변경(엔티티).

executeUpdate()의 결과는 영향받은 엔티티 수 반환

UPDATE, DELETE 지원

INSERT(insert into .. select, 하이버네이트 지원)


---

벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리

그래서 벌크 연산을 먼저 실행하고, 벌크 연산 수행 후 영속성 컨텍스트를 초기화하면 된다.

