
# ORM

Object Relational Mapping, 객체-관계 매핑

객체는 객체대로 설계, 관계형 데이터베이스는 관계형 데이터베이스대로 설계해 ORM 프레임워크가 중간에서 매핑한다. 대중적인 언어에는 대부분 ORM 기술이 존재한다.

객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결)해주는 것을 말한다.

객체 지향 프로그래밍은 클래스를 사용하고, 관계형 데이터베이스는 테이블을 사용한다. 객체 모델과 관계형 모델 간에 불일치가 존재한다. ORM을 통해 객체 간의 관계를 바탕으로 SQL을 자동으로 생성하여 불일치를 해결한다.
```
데이터베이스 데이터 <—매핑—> Object 필드
```

객체를 통해 간접적으로 데이터베이스 데이터를 다룬다.

Persistant API라고도 할 수 있다.

Ex) JPA, Hibernate 등

### ORM의 장점

**객체 지향적인 코드로 인해 더 직관적이고 비즈니스 로직에 더 집중할 수 있게 도와준다.**<br>
ORM을 이용하면 SQL Query가 아닌 직관적인 코드(메서드)로 데이터를 조작할 수 있어 개발자가 객체 모델로 프로그래밍하는 데 집중할 수 있도록 도와준다.
선언문, 할당, 종료 같은 부수적인 코드가 없거나 급격히 줄어든다. SQL의 절차적이고 순차적인 접근이 아닌 객체 지향적인 접근으로 인해 생산성이 증가한다.

**재사용 및 유지보수의 편리성이 증가한다.**<br>
ORM은 독립적으로 작성되어있고, 해당 객체들을 재활용 할 수 있다. 때문에 모델에서 가공된 데이터를 컨트롤러에 의해 뷰와 합쳐지는 형태로 디자인 패턴을 견고하게 다지는데 유리하다.

**DBMS에 대한 종속성이 줄어든다.**<br>
객체 간의 관계를 바탕으로 SQL을 자동으로 생성하기 때문에 RDBMS의 데이터 구조와 Java의 객체지향 모델 사이의 간격을 좁힐 수 있다. 대부분 ORM 솔루션은 DB에 종속적이지 않다.
종속적이지 않다는것은 구현 방법 뿐만아니라 많은 솔루션에서 자료형 타입까지 유효하다.

### ORM의 단점

**완벽한 ORM 으로만 서비스를 구현하기가 어렵다.**<br>
사용하기는 편하지만 설계는 매우 신중하게 해야한다. 프로젝트의 복잡성이 커질경우 난이도 또한 올라갈 수 있다. 잘못 구현된 경우에 속도 저하 및 심각할 경우 일관성이 무너지는 문제점이 생길 수 있다.

**프로시저가 많은 시스템에선 ORM의 객체 지향적인 장점을 활용하기 어렵다.**<br>
이미 프로시저가 많은 시스템에선 다시 객체로 바꿔야하며, 그 과정에서 생산성 저하나 리스크가 많이 발생할 수 있다.

<hr>

# JPA

Java Persistence API

자바 진영의 ORM 기술 표준


### JPA 동작 방식

![image](https://user-images.githubusercontent.com/65898555/179431223-fd995875-7fcf-41ba-bada-2e076d1672da.png)

### JPA는 표준 명세

![image](https://user-images.githubusercontent.com/65898555/179431289-3b80e439-bdca-44d7-bfcc-2879b4f1705a.png)

JPA는 인터페이스의 모음이다. JPA 2.1 표준 명세를 구현한 3가지 구현체 하이버네이트, EclipseLink, DataNucleus가 있다.

### JPA와 CRUD

- 저장: jpa.persist(member)
- 조회: Member member = jpa.find(memberId)
- 수정: member.setName(“변경할 이름”)
- 삭제: jpa.remove(member)



### JPA의 성능 최적화

1. 1차 캐시의 동일성 보장
```java
String memberId = "100";
Member m1 = jpa.find(Member.class, memberId); //SQL
Member m2 = jpa.find(Member.class, memberId); //캐시

println(m1 == m2) //true
// sql 한번만 실행
```
같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상 / DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장

2. 트랜잭션을 지원하는 쓰기 지연
```java
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
트랜잭션을 커밋할 때까지 INSERT SQL을 모음. JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

```java
transaction.begin(); // [트랜잭션] 시작
changeMember(memberA); 
deleteMember(memberB); 
비즈니스_로직_수행(); //비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다. 
//커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
UPDATE, DELETE로 인한 로우(ROW)락 시간 최소화. 트랜잭션 커밋 시 UPDATE, DELETE SQL 실행하고, 바로 커밋

3. 지연로딩

![image](https://user-images.githubusercontent.com/65898555/179431484-42b7e61b-592e-45f0-b49e-f3c005350c9f.png)
- 지연 로딩: 객체가 실제 사용될 때 로딩
- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회


# 데이터베이스 방언

JPA는 특정 데이터베이스에 종속 X 

각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름

방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능

![image](https://user-images.githubusercontent.com/65898555/179431614-8704a334-dc7d-460c-9c19-c6072861b214.png)

hibernate.dialect 속성에 지정. 하이버네이트는 40가지 이상의 데이터베이스 방언 지원.

• H2 : org.hibernate.dialect.H2Dialect 
• Oracle 10g : org.hibernate.dialect.Oracle10gDialect 
• MySQL : org.hibernate.dialect.MySQL5InnoDBDialect 

# JPA 구동 방식

![image](https://user-images.githubusercontent.com/65898555/179431656-b37f762d-1eb5-4a03-90bb-d64b24a14267.png)

엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유.

엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다).

JPA의 모든 데이터 변경은 트랜잭션 안에서 실행.


# JPQL


JPA를 사용하면 엔티티 객체를 중심으로 개발하는데 문제는 검색 쿼리.

검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색하는데 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능

애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공. SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원

JPQL은 엔티티 객체를 대상으로 쿼리. SQL은 데이터베이스 테이블을 대상으로 쿼리. 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리

SQL을 추상화해서 특정 데이터베이스 SQL에 의존X. JPQL을 한마디로 정의하면 객체 지향 SQL 



