# API 개발 기본

### 회원 등록 API

```java
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequiredArgsConstructor
public class MemberApiController {
 private final MemberService memberService;
 /**
 * 등록 V1: 요청 값으로 Member 엔티티를 직접 받는다.
 * 문제점
 * - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
 * - 엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등등)
 * - 실무에서는 회원 엔티티를 위한 API가 다양하게 만들어지는데, 한 엔티티에 각각의 API를 위한 모든 요청 요구사항을 담기는 어렵다.
 * - 엔티티가 변경되면 API 스펙이 변한다.
 * 결론
 * - API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.
 */
 @PostMapping("/api/v1/members")
 public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member)
 {
  Long id = memberService.join(member);
  return new CreateMemberResponse(id);
 }
 
 /**
 * 등록 V2: 요청 값으로 Member 엔티티 대신에 별도의 DTO를 받는다.
 */
 @PostMapping("/api/v2/members")
 public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
  Member member = new Member();
  member.setName(request.getName());
  Long id = memberService.join(member);
  return new CreateMemberResponse(id);
 }
 
 @Data
 static class CreateMemberRequest {
  private String name;
 }
 
 @Data
 static class CreateMemberResponse {
  private Long id;
  public CreateMemberResponse(Long id) {
    this.id = id;
  }
 }
}
```
엔티티 대신에 DTO를 RequestBody에 매핑해야 한다.

엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다. 엔티티와 API 스펙을 명확하게 분리할 수 있다. 엔티티가 변해도 API 스펙이 변하지 않는다.

실무에서는 엔티티를 API 스펙에 노출하면 안된다.


### 회원 수정 API

```java
/**
 * 수정 API
 */
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id, @RequestBody @Valid UpdateMemberRequest request) {
 memberService.update(id, request.getName());
 Member findMember = memberService.findOne(id);
 return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}

@Data
static class UpdateMemberRequest {
 private String name;
}

@Data
@AllArgsConstructor
static class UpdateMemberResponse {
 private Long id;
 private String name;
}
```
회원 수정도 DTO를 요청 파라미터에 매핑한다.


### 회원 조회 API

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {
 private final MemberService memberService;
 /**
 * 조회 V1: 응답 값으로 엔티티를 직접 외부에 노출한다.
 * 문제점
 * - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
 * - 기본적으로 엔티티의 모든 값이 노출된다.
 * - 응답 스펙을 맞추기 위해 로직이 추가된다. (@JsonIgnore, 별도의 뷰 로직 등등)
 * - 실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어지는데, 한 엔티티에 각각의 API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.
 * - 엔티티가 변경되면 API 스펙이 변한다.
 * - 추가로 컬렉션을 직접 반환하면 항후 API 스펙을 변경하기 어렵다.(별도의 Result 클래스 생성으로 해결)
 * 결론
 * - API 응답 스펙에 맞추어 별도의 DTO를 반환한다.
 */
 //조회 V1: 안 좋은 버전, 모든 엔티티가 노출
 @GetMapping("/api/v1/members")
 public List<Member> membersV1() {
  return memberService.findMembers();
 }
 
 /**
 * 조회 V2: 응답 값으로 엔티티가 아닌 별도의 DTO를 반환한다.
 */
 @GetMapping("/api/v2/members")
 public Result membersV2() {
  List<Member> findMembers = memberService.findMembers();
  //엔티티 -> DTO 변환
  List<MemberDto> collect = findMembers.stream()
    .map(m -> new MemberDto(m.getName()))
    .collect(Collectors.toList());
  return new Result(collect);
 }
 
 @Data
 @AllArgsConstructor
 static class Result<T> {
  private T data;
 }
 
 @Data
 @AllArgsConstructor
 static class MemberDto {
  private String name;
 }
}
```
엔티티를 DTO로 변환해서 반환한다.

엔티티가 변해도 API 스펙이 변경되지 않는다.

추가로 Result 클래스로 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.


# 지연 로딩과 조회 성능 최적화

주문 + 배송정보 + 회원을 조회하는 API를 만든다.

```java
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.repository.*;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;
import java.util.List;
import static java.util.stream.Collectors.toList;
/**
 *
 * xToOne(ManyToOne, OneToOne) 관계 최적화
 * Order
 * Order -> Member
 * Order -> Delivery
 *
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
 private final OrderRepository orderRepository;
 /**
 * V1. 엔티티 직접 노출
 * - Hibernate5Module 모듈 등록, LAZY=null 처리
 * - 양방향 관계 문제 발생 -> @JsonIgnore
 */
 @GetMapping("/api/v1/simple-orders")
 public List<Order> ordersV1() {
  List<Order> all = orderRepository.findAllByString(new OrderSearch());
  for (Order order : all) {
    order.getMember().getName(); //Lazy 강제 초기화
    order.getDelivery().getAddress(); //Lazy 강제 초기환
  }
  return all;
 }
}
```
```order->member```와 ```order->address``` 는 지연 로딩이다. 
따라서 실제 엔티티 대신에 프록시 존재하는데, jackson 라이브러리는 기본적으로 이 프록시 객체를 json으로 어떻게 생성해야 하는지 몰라 예외가 발생한다.

```java
@Bean
Hibernate5Module hibernate5Module() {
 return new Hibernate5Module();
}
```
Hibernate5Module을 스프링 빈으로 등록하면 해결된다.

```java
@Bean
Hibernate5Module hibernate5Module() {
 Hibernate5Module hibernate5Module = new Hibernate5Module();
 //강제 지연 로딩 설정
 hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);
 return hibernate5Module;
}
```
다음과 같이 설정하면 강제로 지연 로딩이 가능하지만, 이 옵션을 키면 order -> member , member -> orders 양방향 연관관계를 계속 로딩하게 된다. 따라서 @JsonIgnore 옵션을 한곳에 주어야 한다.

엔티티를 직접 노출할 때는 양방향 연관관계가 걸린 곳은 꼭 한곳을 @JsonIgnore처리 해야 한다. 안그러면 양쪽을 서로 호출하면서 무한 루프가 걸린다. 

정말 간단한 애플리케이션이 아니면 엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 따라서 Hibernate5Module를 사용하기 보다는 DTO로 변환해서 반환하는 것이 더 좋은 방법이다.

지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다. 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 즉시 로딩으로
설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용한다.


```java
/**
 * V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
 * - 단점: 지연로딩으로 쿼리 N번 호출
 */
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<SimpleOrderDto> result = orders.stream()
 .map(o -> new SimpleOrderDto(o))
 .collect(toList());
 return result;
}

@Data
static class SimpleOrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 
 public SimpleOrderDto(Order order) {
  orderId = order.getId();
  name = order.getMember().getName();
  orderDate = order.getOrderDate();
  orderStatus = order.getStatus();
  address = order.getDelivery().getAddress();
  }
}
```
엔티티를 DTO로 변환하는 일반적인 방법이다.

쿼리가 총 1 + N + N번 실행된다. (v1과 쿼리수 결과는 같다.)

- order 조회 1번(order 조회 결과 수가 N이 된다.)
- order -> member 지연 로딩 조회 N 번
- order -> delivery 지연 로딩 조회 N 번

지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.

```java
/**
 * V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용O)
 * - fetch join으로 쿼리 1번 호출
 * 참고: fetch join에 대한 자세한 내용은 JPA 기본편 참고(정말 중요함)
 */
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
 List<Order> orders = orderRepository.findAllWithMemberDelivery();
 List<SimpleOrderDto> result = orders.stream()
 .map(o -> new SimpleOrderDto(o))
 .collect(toList());
 return result;
}
```
```java
public List<Order> findAllWithMemberDelivery() {
 return em.createQuery(
 "select o from Order o" +
 " join fetch o.member m" +
 " join fetch o.delivery d", Order.class)
 .getResultList();
}
```
엔티티를 페치 조인(fetch join)을 사용해서 쿼리 1번에 조회한다. 페치 조인으로 order -> member , order -> delivery는 이미 조회 된 상태 이므로 지연로딩이 아니다.

```java
private final OrderSimpleQueryRepository orderSimpleQueryRepository; //의존관계
주입
/**
 * V4. JPA에서 DTO로 바로 조회
 * - 쿼리 1번 호출
 * - select 절에서 원하는 데이터만 선택해서 조회
 */
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
 return orderSimpleQueryRepository.findOrderDtos();
}
```
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;
import javax.persistence.EntityManager;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
 private final EntityManager em;
 public List<OrderSimpleQueryDto> findOrderDtos() {
  return em.createQuery(
  "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
  " from Order o" +
  " join o.member m" +
  " join o.delivery d", OrderSimpleQueryDto.class).getResultList();
 }
}
```
```java
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;
import java.time.LocalDateTime;

@Data
public class OrderSimpleQueryDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 
 public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
  this.orderId = orderId;
  this.name = name;
  this.orderDate = orderDate;
  this.orderStatus = orderStatus;
  this.address = address;
 }
}
```
JPA에서 DTO로 바로 조회하는 방법이다. OrderSimpleQueryRepository 조회 전용 리포지토리를 만들고, OrderSimpleQueryDto 리포지토리에서 DTO를 직접 조회한다.

일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회

new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환

SELECT 절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트웍 용량 최적화(생각보다미비)

리포지토리 재사용성 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점이 있다.

---

쿼리 방식 선택 권장 순서<br>
1. 우선 엔티티를 DTO로 변환하는 방법을 선택.
2. 필요하면 페치 조인으로 성능을 최적화. 대부분의 성능 이슈가 해결.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용

# 컬렉션 조회 최적화

컬렉션인 일대다 관계(OneToMany)를 조회하고, 최적화하는 방법이다.

```java
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.repository.*;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;
import java.util.List;
import static java.util.stream.Collectors.*;
/**
 * V1. 엔티티 직접 노출
 * - 엔티티가 변하면 API 스펙이 변한다.
 * - 트랜잭션 안에서 지연 로딩 필요
 * - 양방향 연관관계 문제
 *
 * V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
 * - 트랜잭션 안에서 지연 로딩 필요
 * V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용O)
 * - 페이징 시에는 N 부분을 포기해야함(대신에 batch fetch size? 옵션 주면 N -> 1 쿼리로 변경
가능)
 *
 * V4. JPA에서 DTO로 바로 조회, 컬렉션 N 조회 (1 + N Query)
 * - 페이징 가능
 * V5. JPA에서 DTO로 바로 조회, 컬렉션 1 조회 최적화 버전 (1 + 1 Query)
 * - 페이징 가능
 * V6. JPA에서 DTO로 바로 조회, 플랫 데이터(1Query) (1 Query)
 * - 페이징 불가능...
 *
 */
@RestController
@RequiredArgsConstructor
public class OrderApiController {
  private final OrderRepository orderRepository;
  /**
  * V1. 엔티티 직접 노출
  * - Hibernate5Module 모듈 등록, LAZY=null 처리
  * - 양방향 관계 문제 발생 -> @JsonIgnore
  */
  @GetMapping("/api/v1/orders")
  public List<Order> ordersV1() {
   List<Order> all = orderRepository.findAll();
   for (Order order : all) {
    order.getMember().getName(); //Lazy 강제 초기화
    order.getDelivery().getAddress(); //Lazy 강제 초기환
    List<OrderItem> orderItems = order.getOrderItems();
    orderItems.stream().forEach(o -> o.getItem().getName()); //Lazy 강제초기화
   }
   return all;
  }
}
```
orderItem, item 관계를 직접 초기화하면 Hibernate5Module 설정에 의해 엔티티를 JSON으로 생성한다.
양방향 연관관계면 무한 루프에 걸리지 않게 한곳에 @JsonIgnore 를 추가해야 한다. 엔티티를 직접 노출하므로 좋은 방법은 아니다.

```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<OrderDto> result = orders.stream()
 .map(o -> new OrderDto(o))
 .collect(toList());
 return result;
}
```
```java
@Data
static class OrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemDto> orderItems;
 
 public OrderDto(Order order) {
  orderId = order.getId();
  name = order.getMember().getName();
  orderDate = order.getOrderDate();
  orderStatus = order.getStatus();
  address = order.getDelivery().getAddress();
  orderItems = order.getOrderItems().stream()
  .map(orderItem -> new OrderItemDto(orderItem))
  .collect(toList());
 }
}

@Data
static class OrderItemDto {
 private String itemName;//상품 명
 private int orderPrice; //주문 가격
 private int count; //주문 수량
 
 public OrderItemDto(OrderItem orderItem) {
  itemName = orderItem.getItem().getName();
  orderPrice = orderItem.getOrderPrice();
  count = orderItem.getCount();
 }
}
```
지연 로딩으로 너무 많은 SQL이 실행된다.

SQL 실행 수는...

- order 1번
- member , address N번(order 조회 수 만큼)
- orderItem N번(order 조회 수 만큼)
- item N번(orderItem 조회 수 만큼)

지연 로딩은 영속성 컨텍스트에 있으면 영속성 컨텍스트에 있는 엔티티를 사용하고 없으면 SQL을 실행한다. 따라서 같은 영속성 컨텍스트에서 이미 로딩한 회원 엔티티를 추가로 조회하면 SQL을 실행하지 않는다.

```java
@GetMapping("/api/v3/orders")
public List<OrderDto> ordersV3() {
 List<Order> orders = orderRepository.findAllWithItem();
 List<OrderDto> result = orders.stream()
 .map(o -> new OrderDto(o))
 .collect(toList());
 return result;
}
```
```java
public List<Order> findAllWithItem() {
 return em.createQuery(
 "select distinct o from Order o" +
 " join fetch o.member m" +
 " join fetch o.delivery d" +
 " join fetch o.orderItems oi" +
 " join fetch oi.item i", Order.class)
 .getResultList();
}
```
페치 조인으로 SQL이 1번만 실행된다.

distinct를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order  엔티티의 조회 수도 증가하게 된다. JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다. 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는 것을 막아준다.

다만 이러한 방법은 페이징이 불가능하다는 단점이 존재한다.

```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
 return em.createQuery(
 "select o from Order o" +
 " join fetch o.member m" +
 " join fetch o.delivery d", Order.class)
 .setFirstResult(offset)
 .setMaxResults(limit)
 .getResultList();
}
```
```java
/**
 * V3.1 엔티티를 조회해서 DTO로 변환 페이징 고려
 * - ToOne 관계만 우선 모두 페치 조인으로 최적화
 * - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
 */
@GetMapping("/api/v3.1/orders")
public List<OrderDto> ordersV3_page(@RequestParam(value = "offset",defaultValue = "0") int offset,
 @RequestParam(value = "limit", defaultValue = "100") int limit) {
 List<Order> orders = orderRepository.findAllWithMemberDelivery(offset,limit);
 List<OrderDto> result = orders.stream()
 .map(o -> new OrderDto(o))
 .collect(toList());
 return result;
}
```
```
// 최적화 옵션
spring:
 jpa:
   properties:
     hibernate:
       default_batch_fetch_size: 1000
```
컬렉션을 페치 조인하면 페이징이 불가능하다.

컬렉션을 페치 조인하면 일대다 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.

Order를 기준으로 페이징 하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 되어버린다.

이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다.  최악의 경우 장애로 이어질 수 있다.

---

먼저 ToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
컬렉션은 지연 로딩으로 조회한다.
지연 로딩 성능 최적화를 위해 ```hibernate.default_batch_fetch_size``` , ```@BatchSize``` 를 적용한다. 

- hibernate.default_batch_fetch_size: 글로벌 설정
- @BatchSize: 개별 최적화
- 
이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.

---

쿼리 호출 수가 1 + N 1 + 1 로 최적화 된다.

조인보다 DB 데이터 전송량이 최적화 된다. (Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)

페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.

컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.

ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 한다.


```java
private final OrderQueryRepository orderQueryRepository;
@GetMapping("/api/v4/orders")
public List<OrderQueryDto> ordersV4() {
 return orderQueryRepository.findOrderQueryDtos();
}
```
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;
import javax.persistence.EntityManager;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {
 private final EntityManager em;
 /**
 * 컬렉션은 별도로 조회
 * Query: 루트 1번, 컬렉션 N 번
 * 단건 조회에서 많이 사용하는 방식
 */
 public List<OrderQueryDto> findOrderQueryDtos() {
  //루트 조회(toOne 코드를 모두 한번에 조회)
  List<OrderQueryDto> result = findOrders();
  //루프를 돌면서 컬렉션 추가(추가 쿼리 실행)
  result.forEach(o -> {
  List<OrderItemQueryDto> orderItems =findOrderItems(o.getOrderId());
  o.setOrderItems(orderItems);
  });
  return result;
 }
 
 /**
 * 1:N 관계(컬렉션)를 제외한 나머지를 한번에 조회
 */
 private List<OrderQueryDto> findOrders() {
  return em.createQuery(
   "select new 
  jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
  " from Order o" +
  " join o.member m" +
  " join o.delivery d", OrderQueryDto.class).getResultList();
 }
 
 /**
 * 1:N 관계인 orderItems 조회
 */
 private List<OrderItemQueryDto> findOrderItems(Long orderId) {
  return em.createQuery(
  "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
  " from OrderItem oi" +
  " join oi.item i" +
  " where oi.order.id = : orderId", OrderItemQueryDto.class).setParameter("orderId", orderId).getResultList();
 }
}
```
```java
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;
import lombok.EqualsAndHashCode;
import java.time.LocalDateTime;
import java.util.List;
@Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemQueryDto> orderItems;
 
 public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,OrderStatus orderStatus, Address address) {
  this.orderId = orderId;
  this.name = name;
  this.orderDate = orderDate;
  this.orderStatus = orderStatus;
  this.address = address;
 }
}
``` 
```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;
@Data
public class OrderItemQueryDto {
 @JsonIgnore
 private Long orderId; //주문번호
 private String itemName;//상품 명
 private int orderPrice; //주문 가격
 private int count; //주문 수량
 
 public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, intcount) {
  this.orderId = orderId;
  this.itemName = itemName;
  this.orderPrice = orderPrice;
  this.count = count;
 }
}
```
JPA에서 DTO 직접 조회하는 코드이다.

Query: 루트 1번, 컬렉션 N 번 실행

ToOne(N:1, 1:1) 관계들을 먼저 조회하고, ToMany(1:N) 관계는 각각 별도로 처리한다.

이런 방식을 선택한 이유는 다음과 같다.
- ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.
- ToMany(1:N) 관계는 조인하면 row 수가 증가한다.

row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화 하기 쉬우므로 한번에 조회하고, ToMany 관계는 최적화 하기 어려우므로 findOrderItems() 같은 별도의 메서드로 조회한다.


```java
@GetMapping("/api/v5/orders")
public List<OrderQueryDto> ordersV5() {
 return orderQueryRepository.findAllByDto_optimization();
}
```
```java
/**
 * 최적화
 * Query: 루트 1번, 컬렉션 1번
 * 데이터를 한꺼번에 처리할 때 많이 사용하는 방식
 *
 */
public List<OrderQueryDto> findAllByDto_optimization() {
 //루트 조회(toOne 코드를 모두 한번에 조회)
 List<OrderQueryDto> result = findOrders();
 //orderItem 컬렉션을 MAP 한방에 조회
 Map<Long, List<OrderItemQueryDto>> orderItemMap =findOrderItemMap(toOrderIds(result));
 //루프를 돌면서 컬렉션 추가(추가 쿼리 실행X)
 result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
 return result;
}

private List<Long> toOrderIds(List<OrderQueryDto> result) {
 return result.stream()
 .map(o -> o.getOrderId())
 .collect(Collectors.toList());
}

private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
 List<OrderItemQueryDto> orderItems
 = em.createQuery("select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice,oi.count)" +
 " from OrderItem oi" +
 " join oi.item i" +
 " where oi.order.id in :orderIds", OrderItemQueryDto.class).setParameter("orderIds", orderIds).getResultList();
  return orderItems.stream().collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
}
```
JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

Query: 루트 1번, 컬렉션 1번

ToOne 관계들을 먼저 조회하고, 여기서 얻은 식별자 orderId로 ToMany 관계인 OrderItem 을 한꺼번에 조회

MAP을 사용해서 매칭 성능 향상(O(1))


```java
@GetMapping("/api/v6/orders")
public List<OrderQueryDto> ordersV6() {
 List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();
 return flats.stream()
 .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(),
o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
 mapping(o -> new OrderItemQueryDto(o.getOrderId(),
o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
 )).entrySet().stream()
 .map(e -> new OrderQueryDto(e.getKey().getOrderId(),
e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),
e.getKey().getAddress(), e.getValue()))
 .collect(toList());
}
```
```java
public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,
OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
 this.orderId = orderId;
 this.name = name;
 this.orderDate = orderDate;
 this.orderStatus = orderStatus;
 this.address = address;
 this.orderItems = orderItems;
}
```
OrderQueryDto에 생성자 추가

```java
public List<OrderFlatDto> findAllByDto_flat() {
 return em.createQuery(
 "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
 " from Order o" +
 " join o.member m" +
 " join o.delivery d" +
 " join o.orderItems oi" +
 " join oi.item i", OrderFlatDto.class)
 .getResultList();
}
```
```java
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;
import java.time.LocalDateTime;
@Data
public class OrderFlatDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private Address address;
 private OrderStatus orderStatus;
 private String itemName;//상품 명
 private int orderPrice; //주문 가격
 private int count; //주문 수량
 
 public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate,OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
  this.orderId = orderId;
  this.name = name;
  this.orderDate = orderDate;
  this.orderStatus = orderStatus;
  this.address = address;
  this.itemName = itemName;
  this.orderPrice = orderPrice;
  this.count = count;
 }
}
```
JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

Query: 1번

쿼리는 한번이지만 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가되므로 상황에 따라 V5 보다 더 느릴 수 도 있다.

애플리케이션에서 추가 작업이 크다. 페이징 불가능함.




---

1.엔티티 조회 방식으로 우선 접근

1-1. 페치조인으로 쿼리 수를 최적화

1-2. 컬렉션 최적화
- 페이징 필요 hibernate.default_batch_fetch_size , @BatchSize 로 최적화
- 페이징 필요X 페치 조인 사용

<br>

2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용

3. DTO 조회 방식으로 해결이 안되면 NativeSQL or 스프링 JdbcTemplate


# OSIV와 성능 최적화


- Open Session In View: 하이버네이트
- Open EntityManager In View: JPA

관례상 OSIV라 한다.

![image](https://user-images.githubusercontent.com/65898555/182502982-1dcd2700-33c2-4cd9-851a-9e00accfa530.png)

```spring.jpa.open-in-view : true``` 기본값

이 기본값을 뿌리면서 애플리케이션 시작 시점에 warn 로그를 남기는 것은 이유가 있다.

OSIV 전략은 트랜잭션 시작처럼 최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때 까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지한다. 그래서 지금까지 View Template이나 API 컨트롤러에서 지연 로딩이 가능했던 것이다.

지연 로딩은 영속성 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 데이터베이스 커넥션을 유지한다. 이것 자체가 큰 장점이다.
그런데 이 전략은 너무 오랜시간동안 데이터베이스 커넥션 리소스를 사용하기 때문에, 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다. 이것은 결국 장애로 이어진다.
예를 들어서 컨트롤러에서 외부 API를 호출하면 외부 API 대기 시간 만큼 커넥션 리소스를 반환하지 못하고, 유지해야 한다.

![image](https://user-images.githubusercontent.com/65898555/182503064-493509b6-55d8-4905-8014-9c560e539546.png)

```spring.jpa.open-in-view: false``` OSIV 종료

OSIV를 끄면 트랜잭션을 종료할 때 영속성 컨텍스트를 닫고, 데이터베이스 커넥션도 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다.
OSIV를 끄면 모든 지연로딩을 트랜잭션 안에서 처리해야 한다. 따라서 지금까지 작성한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어야 하는 단점이 있다. 그리고 view template에서 지연로딩이 동작하지 않는다. 결론적으로 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다

---

실무에서 OSIV를 끈 상태로 복잡성을 관리하는 좋은 방법이 있다. 바로 Command와 Query를 분리하는 것이다.

