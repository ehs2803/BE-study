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

