## JPA 활용 2

- 요즘은 리액트, 뷰 등을 사용하기 때문에 백엔드에서 html(템플릿)을 return하는게 아니라 프론트가 담당함. 영한님은 템플릿엔진을 사용해서 렌더링하는 컨트롤러(화면을 반환, JPA활용1에서 하던 방식), api 스타일의 컨트롤러(api 스펙을 반환?) 패키지를 분리해두심.
  - 전자는 controller 폴더 안에, 후자는 api 폴더 안에 넣었음

- `@RestController` : @Controller + @ResponseBody
  - apiController 앞에는 @Controller 대신 이거 붙임
  - `@ResponseBody` : 데이터(body)를 바로 Json(/XML)로 보냄
  
- api스펙과 엔티티는 명확히 분리
  - controller의 메소드의 파라미터, 반환형으로 절대 엔티티는 쓰지 말기! 무조건 DTO를 사용
  - 이유는 spring project 구조 2 문서 참고
    - api스펙 문서를 까보지 않아도 DTO를 보면 어떤 값이 넘어오는지(api 스펙) 알 수 있다
    - 엔티티를 외부에 노출x
  
- body에서 값이 null이면 안될 땐, DTO의 필드에 `@NotEmpty` 붙이기
  - 엔티티에 붙이는 것 보단 DTO에 붙이는게 좋음
  - 왜냐하면 api 스펙별로 not null이어야 하는 필드가 다를 수 있기 때문

- postman으로 api 테스트 가능
  - 서버를 켠 후, postman을 통해 url에 요청

- `@RequestBody` : json으로 온 body를 이 어노테이션이 붙은 파라미터에 자동으로 넣어줌

- 커맨드와 쿼리를 분리하기

  - ```java
    @Transactional
    public void update(Long id, String name) {
        Member member = memberRepository.findOne(id);
        member.setName(name);
        //트랜잭션이 끝나면 변경감지(dirty check) 됨
        //메소드가 member를 반환하도록 하면, member를 update하는 동시에 조회하는 꼴이 되어버림
       //즉, 커맨드와 쿼리가 같이 있다
       //영한님은 커맨드와 쿼리를 분리하는걸 추천하심
    }
    ```
    
  - ```java
    memberService.update(id, request.getName());
    //커맨드와 쿼리를 분리하기 위해(update와 조회를 분리) 조회하는 코드를 따로 작성
    Member findMember = memberService.findOne(id);
    ```

- `@AllArgsConstructor`  : 이 어노테이션이 붙은 클래스는, 생성자가 모든 필드를 포함한다

- application.yml에서 `jpa.hibernate.ddl-auto: create` : 기존테이블 삭제 후 다시 생성(DROP+CREATE)
  - create는 반드시 로컬환경에서만 사용하기
  
- build.gradle에 모듈 적을때 버전정보 적을 필요 없음 : 스프링부트가 최적화된 버전을 알아서 등록하기 때문

- repository파일에 controller 파일을 import하면 절대 안됨(의존관계 생기면 절대 안됨)

- postman 돌렸을 때 에러메세지 중에 'no properties' 어쩌구 적혀있는 경우, 대부분 프로퍼티는 getter, setter를 말함. 즉 클래스에 lombok으로 getter, setter를 붙여주면 됨(getter만 붙여도 ㄱㅊ)

- 루트 json 배열 안됨?

- 쿼리 메소드(**Spring data JPA**가 제공) 

  - 조회 기능을 별도의 쿼리 없이 메소드명 만으로 수행할 수 있다
  
    - 알아서 메소드명을 분석해서 jpql을 생성해줌. 따라서 메소드명은 규칙에 따라야 함
  
  - 순수 JPA의 경우
  
    ```java
    public List<Member> findById(String id) {
        return em.createQuery("select m from Member m where m.member_id = :member_id", Member.class)
                .setParameter("member_id", id)
                .getResultList();
    }
    ```
  
    이렇게 해야 하는걸,
  
    Spring Data JPA를 사용하는 경우
  
    ```java
    boolean existsByMemberId(String memberId);
    ```
  
    이 한 줄로 같은 기능을 수행할 수 있다.
  
  - JpaRepository를 extends한 인터페이스에서만 사용 가능
  
    ```java
    @Repository //생략 가능
    public interface MemberRepository extends JpaRepository<Member, Integer>{}
    ```
  
    - `<매핑할 엔티티 클래스명, 엔티티의 ID타입>`
    - ID타입은 Wrapper클래스로 적어야 함 ex) int형 -> INTEGER
    - 일반적으로 인터페이스는 구현할 클래스를 작성하기 위해 사용. 하지만 위 인터페이스를 상속하게 되면 직접 클래스를 작성하지 않아도 사용할 수 있음. 이는 스프링 부트가 내부적으로 구현 객체를 자동으로 생성해주기 때문.
    - JPA를 이용한 데이터베이스 연동을 위해 사용했던 EntityManagerFactory, EntityManager, EntityTransaction 역시 스프링 데이터 JPA에서 자동으로 객체를 생성하여 활용하기 때문에 필요 없음.

---

### 엔티티를 직접 노출

#### 회원조회

```java
@GetMapping("/api/v1/members")
public List<Member> membersV1() {
    return memberService.findMembers();
}
```

- member 엔티티의 모든 필드를 반환하기 때문에 엔티티의 모든 정보가 노출이 됨
- member엔티티의 필드 중 하나인 orders는 지금 조회할 필요가 없는데도 조회가 됨
- 노출을 원하지 않는 엔티티의 필드에 `@JsonIgnore`를 붙이면 그 필드만 조회x
  - 다만 저걸 붙이면 다른 API에서 member 엔티티를 조회할 때 저 어노테이션이 붙은 필드는 조회 불가!

<br>

#### 회원 등록

```java
@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}

@Data
static class CreateMemberResponse {
    private Long id;
    public CreateMemberResponse(Long id) {
        this.id = id;
    }
}
```

- `@RequestBody` : json으로 온 데이터를 member에 매핑에서 넣어줌
  - Post 방식으로 이 url에, body를 채워서 요청을 하면, body의 각 키, 값이 member에 매핑됨
  - Postman을 통해 body를 채우고 테스트 가능
- 그 member를 통해 회원가입(join)하고, 생성된 id를 반환
- 그 id를 파라미터로 넣은 CreateMemberRespones를 반환(CreateMemberResponse의 키,값이 반환됨)

<br>

<br>

### API스펙과 엔티티를 분리 - DTO 사용

#### 회원 조회

```java
@GetMapping("/api/v2/members")
public Result memberV2() {
    List<Member> findMembers = memberService.findMembers();
        
    //List<Member>를 List<MemberDto>로 바꿔치기
    List<MemberDto> collect = findMembers.stream()
            //findMembers의 각 Member들의 name을 생성자의 파라미터로 넣은 MemberDto 객체를 만들고, 각 Member들을 MemberDto로 바꿔치기
            .map(m -> new MemberDto(m.getName()))
            //리스트로 만들기
            .collect(Collectors.toList());
    //Result로 한 번 감싸서 반환하는 이유:
    //List<MemberDto>를 바로 반환하면, json 배열 타입으로 반환되기 때문에 유연성이 떨어짐(count필드 같은거 추가 불가)
    return new Result(collect.size(), collect);
}

@Data
@AllArgsConstructor
static class Result<T> {
    private int count;
    private T data;
}

@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

- DTO를 Result로 한 번 감싸서 반환
  - 이유 : `List<MemberDto>`를 바로 반환하면, json 배열 타입으로 반환되기 때문에 유연성이 떨어짐 = count필드 같은거 추가 불가

<br>

#### 회원 등록

```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}

//DTO for 회원등록 API(파라미터용)
@Data
static class CreateMemberRequest {
    @NotEmpty
    private String name;
}

//DTO for 회원등록 API(반환용)
@Data
static class CreateMemberResponse {
    private Long id;
    public CreateMemberResponse(Long id) {
        this.id = id;
    }
}
```

- 이렇게 DTO를 inner class로 선언한 이유
  - 이 DTO는 이 inner class를 포함하는 클래스 내에서만 사용된다는 의미를 부여할 수 있음
  - 다른 클래스에서도 DTO를 사용하는 경우, 별도의 클래스로 외부에 정의해야 함(별도의 패키지)

<br>

#### 회원 수정

```java
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(
        @PathVariable("id") Long id,
        @RequestBody @Valid UpdateMemberRequest request) {

    memberService.update(id, request.getName());
    //커맨드와 쿼리를 분리하기 위해(update와 조회를 분리) 조회하는 코드를 따로 작성
    Member findMember = memberService.findOne(id);
    //UpdateMemberResponse는 @AllArgsConstructor니까 생성자가 모든 필드를 포함함
    return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}

//DTO for 회원수정 API(파라미터용)
@Data
static class UpdateMemberRequest {
    private String name;
}

//DTO for 회원수정 API(반환용)
@Data
@AllArgsConstructor
static class UpdateMemberResponse {
    private Long id;
    private String name;
}

```

- 커맨드와 쿼리를 분리

<br>

---

### 간단한 주문조회 V1 : 엔티티를 직접 노출

- OrderSimpleApiController.java

  ```java
  @GetMapping("/api/v1/simple-orders")
  public List<Order> ordersV1() {
      //이대로 하면, Order의 필드중에 Member가 있어서 Member를 찾아가면, Member의 필드중에 Order가 있어서 무한루프에 빠짐
      //이렇게 양방향 연관관계가 있으면, 둘중 하나에 @JsonIgnore 붙여야 함
      List<Order> all = orderRepository.findAllByString(new OrderSearch());
  
      //필요한 정보만 불러올 수 있음(필요한 정보만 LAZY 강제 초기화 통해 값을 불러옴)
      for (Order order : all) {
          //getMember까지는 프록시객체를 가져옴. getName이 실행돼야 쿼리를 날려서 실제 객체를 불러옴 :LAZY 강제 초기화
          order.getMember().getName();
          //getAddress통해 LAZY 강제 초기화
          order.getDelivery().getAddress();
      }
      return all;
  }
  ```

  - order.getMember().getName()은 name 속성만을 가져오기 위함이 아니라, 실제 Member 객체를 가져오기 위함!
  - getName()을 호출하는 순간 지연로딩이 강제 초기화되어 Member객체를 가져옴
  - 따라서 Member의 name 속성 외에 다른 속성들도 조회가 가능
  - 즉 굳이 getName()이 아니라 다른 getXXX()를 사용해도 Member객체를 조회해오는 사실은 변함이 없으니 똑같은 결과를 볼 수 있음

- Member.java

  ```java
  @JsonIgnore
  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>();
  ```

  - `@JsonIgnore` : 두 엔티티가 양방향 연관관계(서로가 서로를 참조)에 있을 때, 둘 중 하나의 필드에 이걸 붙여줘야 무한루프에 빠지지 않음

- Order.java

  ```java
  @ManyToOne(fetch = LAZY)
  @JoinColumn(name = "member_id")
  private Member member;
  ```

  - 지연로딩(LAZY)로 인해 ByteBuddyInterceptor 관련 에러 뜰 수 있음

  - Member객체를 가져와야 하는데 지연로딩이기 때문에 Order가 로딩되는 시점에서 Member는 프록시객체로 가져오기 때문.

  - 이걸 막기 위해 Jackson DataType Hibernate5 모듈을 받으면 됨

    - 검색으로 소스 받아와서 build.gradle에 등록

    - 메인클래스에 빈으로 등록

      ```java
      @Bean
      Hibernate5Module hibernate5Module() {
          return new Hibernate5Module();
      }
      ```

  - 지연로딩(`LAZY`) : 한 엔티티가 로딩되는 시점에 LAZY 로딩 설정이 되어있는 연관된 엔티티는 프록시 객체(가짜객체)로 가져옴. 후에 연관된 엔티티의 객체를 실제로 사용하는 시점(객체의 값을 꺼내거나 수정)에 쿼리가 나가서 객체의 값을 채워줌.

- 엔티티를 직접 노출하는 것의 문제점

  - api 스펙상 Order엔티티와 연관된 Member, Delivery만 불러오면 되는데, 필요없는 orderItems 같은 것들도 불러옴 : api에서 필요로 하는 정보보다 더 많은 정보를 불러옴

  - 성능 낭비

<br>

---

### 간단한 주문조회 V2 : 엔티티를 DTO로 변환

- OrderSimpleApiController.java

  ```java
  @GetMapping("/api/v2/simple-orders")
  public List<SimpleOrderDto> ordersV2() {
      List<Order> orders = orderRepository.findAllByString(new OrderSearch());
      List<SimpleOrderDto> result = orders.stream()
              //orders의 각 Order들을 생성자의 파라미터로 넣은 SimpleOrder 객체를 만들고, 각 Order들을 SimpleOrderDto 객체로 바꿔치기
              .map(o -> new SimpleOrderDto(o))
              //리스트로 만들기
              .collect(Collectors.toList());
  
      //원래는 Result로 한 번 감싸서 반환해야 함
      return result;
  }
  
  @Data
  static class SimpleOrderDto {
      private Long orderId;
      private String name;
      private LocalDateTime orderDate;
      private OrderStatus orderStatus;
      private Address address;
  
      //DTO가 파라미터로 엔티티를 받는 건 문제없음
      public SimpleOrderDto(Order order) {
          orderId = order.getId();
          //LAZY 초기화 일어남 : getName 실행될 때 쿼리가 날라가서 db의 실제 값을 불러옴
          name = order.getMember().getName();
          orderDate = order.getOrderDate();
          orderStatus = order.getStatus();
          //LAZY 초기화 일어남 : getAddress 실행될 때
          address = order.getDelivery().getAddress();
      }
  }
  ```

  - 원래 반환형은 따로 클래스(Result) 만들어서 한번 감싸야 함 

    ```java
    @Data
    @AllArgsConstructor
    static class Result<T> {
        private int count;
        private T data;
    }
    ```

    이렇게 Result형을 정의하고

    ```java
    return new Result(result.size(), result);
    ```

    api 메소드에서 이렇게 Result형으로 한 번 감싸서 반환

- V1, V2의 공통적인 문제점

  : 지연로딩(LAZY)으로 인해 DB쿼리가 너무 많이 호출됨

  - V2가 돌아갈때 Order, Member, Delivery 3개의 테이블을 조회해야 하는 상황
  - 우선 Order테이블이 조회
    - `List<Order> orders = orderRepository.findAllByString(new OrderSearch());`
  - -> 결과값이 2개 나왔으므로(DB에 2개의 주문 데이터를 넣어놓음) 각 Order마다 `SimpleOrderDto 생성할 때 Member조회, Delivery조회`가 2번 일어남
  - -> 최악의 경우 총 5개의 쿼리가 나감. Order테이블의 레코드수가 많아지면 쿼리 수가 너무너무 많아짐!!
    - **N+1 문제** :  =1+N. 1번의 쿼리로 인해 나온 결과의 수(N)만큼 쿼리가 더 실행됨
    - 이 경우 Member조회 2번, Delivery조회 2번이므로 1+N+N
    - 만약 같은 Member의 주문이라면, 1+1+2번 쿼리 호출
  - 해결 : **패치 조인 최적화**(V3)

<br>

---

### 간단한 주문조회 V3 : 엔티티를 DTO로 변환 - 패치 조인 최적화

- OrderSimpleApiController.java

  ```java
  @GetMapping("/api/v3/simple-orders")
  public List<SimpleOrderDto> ordersV3() {
      List<Order> orders = orderRepository.findALlWithMemberDelivery();
      List<SimpleOrderDto> result = orders.stream()
              .map(o -> new SimpleOrderDto(o))
              .collect(Collectors.toList());
  
      return result;
  }
  ```

  - V2와 마찬가지로 원래 반환형은 따로 클래스(Result) 만들어서 한번 감싸야 함
  
- OrderRepository.java

  ```java
  public List<Order> findAllWithMemberDelivery() {
      return em.createQuery(
              "select o from Order o" +
                      " join fetch o.member m" +
                      " join fetch o.delivery d", Order.class
      ).getResultList();
      //반환형은 Order 클래스
  }
  ```

  - **패치 조인**
  - Order를 조회할 때 Member, Delivery를 조인해서 쿼리 한번에 select절로 다 조회해오는 것
  - Member, Delivery 모두 지연로딩 설정되어있지만, 이 경우에는 진짜 값으로 채워서 진짜 객체로 받아오므로 지연로딩이 일어나지 않음
  - fetch 명령어는 JPA에만 있는 문법
    - 개중요!! JPA기본편 보고 깊이있게 공부할 것
  - 성능문제의 대부분은 N+1문제에서 발생

- 실행되는 쿼리문

  ```sql
   select
          order0_.order_id as order_id1_6_0_,
          member1_.member_id as member_i1_4_1_,
          delivery2_.delivery_id as delivery1_2_2_,
          order0_.delivery_id as delivery4_6_0_,
          order0_.member_id as member_i5_6_0_,
          order0_.order_date as order_da2_6_0_,
          order0_.status as status3_6_0_,
          member1_.city as city2_4_1_,
          member1_.street as street3_4_1_,
          member1_.zipcode as zipcode4_4_1_,
          member1_.name as name5_4_1_,
          delivery2_.city as city2_2_2_,
          delivery2_.street as street3_2_2_,
          delivery2_.zipcode as zipcode4_2_2_,
          delivery2_.status as status5_2_2_ 
      from
          orders order0_ 
      inner join
          member member1_ 
              on order0_.member_id=member1_.member_id 
      inner join
          delivery delivery2_ 
              on order0_.delivery_id=delivery2_.delivery_id
  ```

  - V2에서는 5개의 쿼리가 실행됐는데, V3에서는 이렇게 1번의 쿼리로 다 처리함

<br>

---

### 간단한 주문조회 V4 : JPA에서 DTO로 바로 조회

V3에서처럼 엔티티를 DTO로 변환하는 과정이 없음

<br>

- OrderSimpleApiController.java

  ```java
  @GetMapping("/api/v4/simple-orders")
  public List<OrderSimpleQueryDto> ordersV4() {
      return orderSimpleQueryRepository.findOrderDtos();
  }
  ```

- repository > order.simplequery 패키지

  - OrderSimpleQueryDto.java

    ```java
    @Data
    public class OrderSimpleQueryDto {
         private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;
    
        //jpa에서 엔티티->DTO변환 없이 DTO로 바로 조회하기 위한 DTO
        //파라미터로 엔티티를 받으면 안됨. 하나하나 받아야 함
        public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
            this.orderId = orderId;
            this.name = name;
            this.orderDate = orderDate;
            this.orderStatus = orderStatus;
            this.address = address;
        }
    }
    ```

    - 별도의 public class로 따로 판 이유 : OrderSimpleQueryRepository에서 이 Dto를 사용해야 하는데 이걸 OrderSimpleApiController에 두면 repository에서 controller를 import하기 때문에 안됨
      
      -  repository가 controller에 의존하면 안됨!!!
      
    - OrderSimpleQueryRepository.java
    
      ```java
      @Repository
      @RequiredArgsConstructor
      public class OrderSimpleQueryRepository {
      
          private final EntityManager em;
      
          public List<OrderSimpleQueryDto> findOrderDtos() {
              //jpa는 value object(엔티티, embeddable:내장타입.Address같은 것)만 반환할 수 있음
              //DTO를 반환하려면 new jpabook.jpashop.repository.OrderSimpleQueryDto를 추가
              //OrderSimpleQueryDto의 생성자 파라미터로 엔티티를 받으면 안됨. 값 하나하나 넣어줘야 함
              // +) d.address : Address는 value type이기 때문에 ㄱㅊ
              return em.createQuery(
                              "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                                      " from Order o" +
                                      " join o.member m" +
                                      " join o.delivery d",OrderSimpleQueryDto.class)
                      .getResultList();
          }
      
      }
      
      ```
    
      - `new` 명령어를 사용하여 jpql의 결과를 dto로 즉시 변환하는 repository
      - 별도의 repository로 따로 판 이유 : 기존 repository는 엔티티를 조회하는데만 쓰도록 하기 위해, 별도로 패키지를 파서 api나 화면과 관련있는, dto를 조회하는 repository를 팜 ->유지보수성 up
        - repository의 본래 역할 : 엔티티 조회

- 실행되는 쿼리문

  ``` sql
  select
          order0_.order_id as col_0_0_,
          member1_.name as col_1_0_,
          order0_.order_date as col_2_0_,
          order0_.status as col_3_0_,
          delivery2_.city as col_4_0_,
          delivery2_.street as col_4_1_,
          delivery2_.zipcode as col_4_2_ 
      from
          orders order0_ 
      inner join
          member member1_ 
              on order0_.member_id=member1_.member_id 
      inner join
          delivery delivery2_ 
              on order0_.delivery_id=delivery2_.delivery_id
  ```

  - V3과의 차이점 : 원하는 필드만 select함
    - V2는 select절에서 좀 더 많은 필드를 조회해옴
  - 그렇다고 V4가 V3보다 좋은 건 아니다! 우열을 가릴 수 없음
    - V3 : Order를 가지고 올 때 패치 조인으로 내가 원하는 것만 select해온 것. 
      1. 따라서 Order를 조회한다는 것 자체를 건드리지 않은 상태로, 내부의 원하는 것만 가져옴
      2. 재사용성이 좋음
      3. 엔티티를 조회했기 때문에 데이터를 변경할 수 있음.
    - V4 : 일반적인 SQL을 짜듯이 JPQL을 짠 것. 
      1. 따라서 원하는 값을 선택해서 조회
      2. 특정 상황에서는 최적화되어있지만, repository 재사용성이 안좋음
      3.  repository는 엔티티를 조회할 때 사용하는 건데, dto를 조회하므로 api스펙에 맞춰 repository 코드가 짜여져있음
      4. DTO를 조회했기 때문에 데이터 변경이 불가(DTO를 변경해봐야 실제 데이터에는 영향x)
      5. 코드상 조금 더 지저분
      6. select절에서 원하는 데이터를 직접 선택하므로 db->애플리케이션 네트웍 용량 최적화(생각보다 미비)

  - v3, v4 어떤게 나을까?

    - 조인할 때, 혹은 where조건문에서 인덱스를 사용하지 않았을 때 성능을 크게 잡아먹음

    - select절에서의 필드 개수 차이로는 대부분 성능차이 크게 안남
    - 데이터 사이즈가 너무 크거나(select필드가 몇십개 될때) 실시간으로 api가 많이 호출되어야 할 때 제외

<br>

---

### 쿼리 방식 선택 권장 순서

1. 우선 엔티티를 DTO로 변환하는 방법을 선택 (V2)
2. 필요하면 패치 조인으로 성능을 최적화(V3)
   - 대부분의 성능 이슈(95% 이상)가 해결된다
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용(V4)
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용

---

### 주문조회 V1 : 엔티티 직접 노출

- 간단한 주문조회와의 차이

  - 간단한 주문조회 : ManyToOne, OneToOne 조인 -> 패치 조인으로 최적화
  - 주문조회 : 컬렉션 조회! OneToMany -> 1줄의 row도 여러줄로 뻥튀기가 됨. 따라서 성능 최적화 필요
    - ex) Order의 OrderItems에 대한 정보들을 반환( Order과 OrderItems는 OneToMany 관계)

- OrderApiController.java

  ```java
  @GetMapping("/api/v1/orders")
  public List<Order> ordersV1() {
      List<Order> all = orderRepository.findAllByString(new OrderSearch());
      for (Order order : all) {
          order.getMember().getName(); //Member를 lazy 강제 초기화
          order.getDelivery().getAddress(); //Delivery를 lazy 강제 초기화
  
          //OrderItem의 Item을 lazy 강제 초기화
          List<OrderItem> orderItems = order.getOrderItems();
          //foreach문을 람다식으로 표현한 것
          orderItems.stream().forEach(o -> o.getItem().getName());
      }
      return all;
  }
  ```

- 엔티티를 직접 노출하므로 이렇게 하면 안됨

---

### 주문조회 V2 : 엔티티를 DTO로 변환

1. 

- OrderApiController.java

  ```java
  @GetMapping("/api/v2/orders")
  public List<OrderDto> ordersV2() {
      List<Order> orders = orderRepository.findAllByString(new OrderSearch());
      List<OrderDto> collect = orders.stream()
              .map(o -> new OrderDto(o))
              .collect(Collectors.toList());
  
      return collect;
  }
  ```
  
  실행 결과
  
  ```sql
  {
          "orderId": 4,
          "name": "userA",
          "orderDate": "2022-01-28T23:51:57.557769",
          "orderStatus": "ORDER",
          "address": {
              "city": "서울",
              "street": "1",
              "zipcode": "1111"
          },
          "orderItems": null
      }, ...
  ```
  
  - orderItems가 null로 나오는 이유 : orderItems는 lazy 로딩으로 설정되어있으므로, Order가 로딩되는 시점에서 orderItems는  프록시객체로 가져옴. 따라서 db에 쿼리가 안나갔기 때문에 값을 가져오지 않음.
  
  - 이를 해결하기 위해 lazy 강제 초기화를 진행
  
    ```java
    @Getter
    static class OrderDto {
    
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItem> orderItems;
    
        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            //getName을 통해 lazy 강제 초기화(쿼리를 날려 실제 Item 객체를 조회)
            order.getOrderItems().stream().forEach(o -> o.getItem().getName());
            orderItems = order.getOrderItems();
        }
    }
    ```
    
  - 여전히 남아있는 문제점 : 반환결과를 보면 orderItem 엔티티의 속성들이 그대로 노출됨. 저 코드는 그저 엔티티를 그대로 DTO로 감싸서 반환한 것. **엔티티를 그대로 노출시키면 안된다는 뜻은 단순히 DTO로 감싸라는게 아니라  엔티티에 대한 의존을 완전히 끊어야 한다는 것!**
  
    - cf) value object는 그대로 노출해도 됨 *ex)Address*
    - OrderDto의 orderItems필드의 타입이 `List<OrderItem>`이면 OrderItem 엔티티의 필드가 노출됨(DTO로 한번 감쌌음에도 불구하고)
    - 즉 OrderDto라는 DTO로 감싸졌을 뿐 OrderItem 엔티티의 값은 그대로 노출됨
      -> DTO 안에 1대다 관계의 엔티티가 있으면(`List<엔티티>`), 다 부분도 DTO로 해줘야 함

2. 엔티티에 대한 의존을 완전히 끊기

- OrderApiController.java

  ```java
  @Getter
  static class OrderDto {
  
      private Long orderId;
      private String name;
      private LocalDateTime orderDate;
      private OrderStatus orderStatus;
      private Address address;
      private List<OrderItemDto> orderItems; //Dto 안에 있는 1대다 관계 필드도 Dto여야 함
  
      public OrderDto(Order order) {
          orderId = order.getId();
          name = order.getMember().getName();
          orderDate = order.getOrderDate();
          orderStatus = order.getStatus();
          address = order.getDelivery().getAddress();
          order.getOrderItems().stream().forEach(o -> o.getItem().getName()); //lazy 강제 초기화
          orderItems = order.getOrderItems().stream()
                  .map(orderItem -> new OrderItemDto(orderItem))
                  .collect(Collectors.toList());
      }
  }
  
  
  @Getter
  static class OrderItemDto {
  
      private String itemName; //상품명
      private int orderPrice; //주문 가격
      private int count; //주문 수량
  
      public OrderItemDto(OrderItem orderItem) {
          itemName = orderItem.getItem().getName();
          orderPrice = orderItem.getOrderPrice();
          count = orderItem.getCount();
      }   
  }
  ```

  - OrderItem 대신 OrderItemDto를 사용
  - OrderItem 엔티티와의 의존관계 완전히 끊어냄

<br>

- 문제점

  1. 지연로딩이 많아서 쿼리문 개수가 개많음.. 성능 개후짐

  2. Order 데이터 각각에 대해 OrderItem이 조인되기 때문에 Order 2개, 각 Order당 OrderItem 2개씩일때 총 4개의 row로 뻥튀기 됨 -> Order 데이터가 중복이 2줄씩 생김(Order데이터가 2배가 됨)

     - <img width="641" alt="db" src="https://user-images.githubusercontent.com/43772750/151587787-d6e23a42-6b82-4da1-9bc6-cee0d9c43bfc.PNG">

     - ```sql
       {
               "orderId": 4,
               "name": "userA",
               "orderDate": "2022-01-29T01:05:59.543788",
               "orderStatus": "ORDER",
               "address": {
                   "city": "서울",
                   "street": "1",
                   "zipcode": "1111"
               },
               "orderItems": [
                   {
                       "itemName": "JPA1 BOOK",
                       "orderPrice": 10000,
                       "count": 1
                   },
                   {
                       "itemName": "JPA2 BOOK",
                       "orderPrice": 20000,
                       "count": 2
                   }
               ]
           },
           {
               "orderId": 4,
               "name": "userA",
               "orderDate": "2022-01-29T01:05:59.543788",
               "orderStatus": "ORDER",
               "address": {
                   "city": "서울",
                   "street": "1",
                   "zipcode": "1111"
               },
               "orderItems": [
                   {
                       "itemName": "JPA1 BOOK",
                       "orderPrice": 10000,
                       "count": 1
                   },
                   {
                       "itemName": "JPA2 BOOK",
                       "orderPrice": 20000,
                       "count": 2
                   }
               ]
           },...
       ```

<br>

---

### 주문조회 V3 : 엔티티를 DTO로 변환 - 페치 조인 최적화

v2에서의 Order 데이터가 중복되는 문제 해결

- OrderApiController.java - v2와 같음

  ```java
  @GetMapping("/api/v3/orders")
  public List<OrderDto> ordersV3() {
      List<Order> orders = orderRepository.findAllWithItem();
      List<OrderDto> result = orders.stream()
              .map(o -> new OrderDto(o))
              .collect(Collectors.toList());
  
      return result;
  
  }
  ```

- OrderRepository.java

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

  - select문에 `distinct` 추가
    - OrderApiController에서 Order, OrderItem 조인되면서 Order 데이터가 중복으로 OrderItem 개수만큼 생기는 것 방지
    - 일반 sql문에서의 distinct와는 다름
    - 일반 sql문에서의 distinct : 완전히 row의 값이 같아야 중복 제거
    - jpa에서의 distinct : Order를 가져올 때 Order의 데이터의 id값이 같으면 중복을 제거해줌(jpa에서 자체적으로 처리)

- 결과

  ```sql
  {
          "orderId": 4,
          "name": "userA",
          "orderDate": "2022-01-29T01:11:45.419379",
          "orderStatus": "ORDER",
          "address": {
              "city": "서울",
              "street": "1",
              "zipcode": "1111"
          },
          "orderItems": [
              {
                  "itemName": "JPA1 BOOK",
                  "orderPrice": 10000,
                  "count": 1
              },
              {
                  "itemName": "JPA2 BOOK",
                  "orderPrice": 20000,
                  "count": 2
              }
          ]
      },
  ```

  - 하나의 Order데이터에 여러개의 OrderItem 데이터가 들어감! -> Order 데이터 중복x

- 문제점

  - 페이징 불가능

    - 페이징

      ```java
      //.getResultList(); 앞에
      .setFirstResult(1) //페이징 쿼리 : 1번째 데이터부터(시작은 0)
      .setMaxResults(100) //100개 가져오기
      ```

    - 1. 데이터를 모두 db에서 가져온 후에 메모리에 넣고 메모리에서 페이징 처리 -> 메모리 초과할 가능성 생김
      2. 1대다 관계로 조인했을 때, 1을 기준으로 페이징하고 싶은데, 다를 기준으로 페이징이 되어버림
    
    - **1대다 페치 조인(컬렉션 페치 조인)(OneToMany)에서는 페이징 하지 말기**
    
    - **컬렉션 페치 조인은 1개만 사용할 수 있음 : 1대 다 의 다 이면 안됨 - 겹쳐서 조인 안된다는 것일듯**
    
      - XXToOne 관계는 페치 조인을 여러 번 걸어도 됨(데이터 row수 뻥튀기가 되지 않음)

---

### 주문조회 V3.1 : 엔티티를 DTO로 변환 - 페이징과 한계 돌파

영한님 강추!!! (페이징을 쓰는 경우 사실상 유일한 최적화 방법)

> 어렵다..나중에 다시 들어보기

<br>

- OrderApiController.java

  ```java
  @GetMapping("/api/v3.1/orders")
  public List<OrderDto> ordersV3_page(
          // /api/v3.1/orders?offset=1&limit=100 이런식으로 url 입력
          @RequestParam(value = "offset", defaultValue = "0") int offset,
          @RequestParam(value = "limit", defaultValue = "100") int limit) {
      
      //Order와 XXToOne 관계에 있는 Member, Delivery를 페치 조인으로 하나의 select문으로 가져옴
      List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
  
      List<OrderDto> result = orders.stream()
              .map(o -> new OrderDto(o))
              .collect(Collectors.toList());
  
      return result;
  
  }
  ```

<br>

- OrderRepository.java

  ```java
  public List<Order> findAllWithMemberDelivery(int offset, int limit) {
      return em.createQuery(
              "select o from Order o" +
                      " join fetch o.member m" +
                      " join fetch o.delivery d", Order.class
      )       .setFirstResult(offset) //offset번째 데이터부터(시작은 0번째)
              .setMaxResults(limit) //limit개 가져옴
              .getResultList();
  }
  ```

<br>

#### 조인 + 페이징해야 하는 경우 이렇게 하기

1. **ToOne 관계**는 모두 페치조인 한다 

   - ToOne 관계는 조인해도 row수를 증가시키지 않으므로 페이징 쿼리에 영향x

2. **컬렉션(OneToMany)**은 지연 로딩으로 조회한다 - v2의 2번방법인듯

3. 지연 로딩 성능 최적화를 위해 다음을 추가

   **global하게 적용하는 경우**
   
   이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회
   
   ```java
	//application.yml
	hibernate:
	        default_batch_fetch_size: 100
	```

   - 값 : IN 쿼리의 개수. (한번에 가져올 수 있는 데이터 개수..?)
	  - 최대값은 1000개
     - 100~1000사이를 선택하는 걸 권장!
     - 1000으로 설정하는게 성능상 가장 좋지만(쿼리가 가장 적게 나가니까), DB에 순간 부하가 증가할 수 있기 때문에 DB가 순간 부하를 견딜 수 있는 정도까지로 설정해야 한다
   
	**디테일하게(하나하나) 적용하는 경우**
	
   ```java
   //Order.java
   //디테일하게 + OneToMany
	@BatchSize(size = 100)
	@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
	private List<OrderItem> orderItems = new ArrayList<>();
	```
	```java
	//OrderItem.java
	//OrderItem과 Item은 다대1 관계니까
	@BatchSize(size=100)
	public class OrderItem { ... }
	```

>  영한님은 global하게 적용하는 방법을 추천!

<br>

#### 장점

1. 쿼리 호출 수 : `1+N` -> `1+1`로 최적화
2. 조인보다 DB 데이터 전송량이 최적화됨
3. 페치 조인에 비해 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소
4. 페이징 가능(가장 중요)

---

