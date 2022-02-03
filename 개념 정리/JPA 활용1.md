스프링부트 : 스프링의 기능들을 쉽게 사용할 수 있도록 도와줌

스프링부트 + JPA + Querydsl(동적쿼리 처리)을 보통 많이 사용



### 프로젝트 생성

https://start.spring.io/

- 스프링부트 기반으로 스프링 관련 프로젝트를 만들어주는 사이트

1.  Gradle Project
2. 언어 : Java
3. 정식 릴리즈 버전 중에 가장 최신인 2.5.6선택
   - snapshot: 아직 만들고 있는 버전
   - m1:정식 릴리즈 되지 않은 버전

4. group : 기업도메인명
5. artifact : 프로젝트명
6. dependencies : 라이브러리
   - spring web, thymeleaf(jsp 대체), spring data JPA, h2 database(개발, 테스트할 때 용이), lombok(간단한 어노테이션으로 getter, setter등 사용 가능)

7. generate 버튼
8. 알집 풀어서 적당한 폴더에 옮기기
   - C:\Spring Project
9. intelliJ에서 Open > build.gradle 파일 선택해서 Open



### build.gradle

- `id 'org.springframework.book' version '2.5.6'` 에서 버전 숫자만 수정해줘도 다른 라이브러리의 버전들도 모두 바뀜

- `implementation 'org.springframework.boot:spring-boot-devtools'` 추가
  - 원래 resources 폴더에 있는 html을 수정하면 웹서버를 껐다가 켜야 변경내용이 반영됨. 하지만 저걸 추가하면 **수정한 html파일을 intelliJ에 띄워두고, Build>Recompile"파일.html" 클릭하면, 웹브라우저 새로고침만 하면 변경내용 반영!**
- 라이브러리 추가시 반드시 코끼리 아이콘 클릭!



### 프로젝트 실행 방법

1. src\main\java\jpabook.jpashop\JpashopApplication 의 메인메소드 실행
2. Run 결과창에 써있는 포트번호(8080) -> 웹 브라우저에 `http://localhost:8080/` 쳐서 들어가기
3. Setting창(Ctrl + Alt + S) > annotation processors > Enable annotation processing 체크! 해야 lombok 사용 가능
   - Lombok : 클래스 앞에 어노테이션으로 `@Getter`, `@Setter` 적어두면 getter, setter 별도로 정의하지 않고도 바로 사용 가능



### @Controller, @GetMapping

```java
@Controller
public class HelloController {

    // /hello라는 url이 오면 이 컨트롤러가 호출됨
    @GetMapping("hello")
    public String hello(Model model) {
        //model에 키가 "data", 값이 "hello!!!"인 데이터를 실어서
        model.addAttribute("data", "hello!!!");
        //resources\templates\hello.html에 전달함
        //hello.html에 ${data}에 값이 전달돼서 그 값이 화면에 출력됨
        return "hello";
    }
}
```

- resources\templates 폴더에 있는 html들은 서버사이드 렌더링을 거치는 html(컨트롤러에서 Model에 실어 넘겨주는 데이터에 따라 다른 값을 화면에 출력할 수 있음)
- resources\static 폴더에 있는 html들은 정적 html(항상 고정된 화면)
  - index.html : 맨 처음 기본 url을 입력하면 뜨는 화면
    - `http://localhost:8080`



### H2 database

1. h2 콘솔 앱 실행
2. uri에서 `192.168.0.16:8082/` -> `localhost:8082/` 로 바꾸기
3. JDBC URL : db 파일이 생성되는 경로
   - `jdbc:h2:tcp://localhost/~/test`: 사용자/wus2363/test.mv.db 파일로 생성됨
   - 새 db 생성 안될 시 해결 방법 : https://www.inflearn.com/questions/271461
   - db에 접근할 때는 JDBC URL로 접근



### application.yml

application.properties 대체(설정파일)

- 데이터베이스, jpa 관련 설정
- 스프링부트 메뉴얼(reference document)에서 보고 작성
- 들여쓰기 주의!!(스페이스 개수 맞춰야 함)
- 오타나도 빨간줄 안뜨니까 주의



- `org.hibernate.SQL: debug` : log를 통해 sql 출력

- `org.hibernate.type: trace` : log를 통해 쿼리 파라미터 출력

  - sql 로그 아래에 이렇게 뜸

    ```java
    2021-11-20 18:03:20.431 TRACE 17112 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [memberA]
    2021-11-20 18:03:20.432 TRACE 17112 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [BIGINT] - [1]
        
    //첫번째 파라미터: VARCHAR타입 memberA, 두번째 파라미터:BIGINT타입 1
    ```

  - 더 간단하게 보려면 라이브러리 다운

    - https://github.com/gavlyukovskiy/spring-boot-data-source-decorator 에서 P6Spy 적용

    - sql 로그 아래에 이렇게 뜸

      ```java
      insert into member (username, id) values (?, ?)
      insert into member (username, id) values ('memberA', 1);
      ```

    - 이런 라이브러리들은 개발 단계에서만 쓰는걸 추천(운영 단계에서는 생각해보기)

    



### @Transactional

- springframework 어쩌구 선택
- 이 어노테이션이 테스트 코드에 있으면 테스트가 끝난 후 Rollback을 함. 그래서 데이터 넣었어도 롤백되면 사라짐
- 자동롤백을 안하고 싶으면 `@Rollback(false)`



### 연관관계 매핑

<img width="356" alt="dddd" src="https://user-images.githubusercontent.com/43772750/142723000-c22bab8e-dcb8-4d30-9ee3-fb21402b36c4.PNG">

- 다대다 관계는 중간에 매핑 테이블을 둬서 일대다, 다대일로 바꿔야함!! 다대다x(Order, ''OrderItem'', Item)

- (양방향 연관관계의) 일대다 관계에서 '다'가 되는 엔티티가 FK를 가짐

- 양방향 연관관계 시, FK가 있는 엔티티에 있는 속성을 연관관계의 주인으로 정하기

  - 양방향 연관관계 : Member는 orders를 리스트로 가지고 있음. Order도 member를 가지고 있음. -> 양방향 참조가 일어남

  - Member의 orders에는 값을 셋팅했는데, Order의 member에는 값을 안넣음. 이런 경우 어떤 엔티티의 값이 맞는 거지?

    - Member의 orders, Order의 member

    - 이럴 때 기준이 되는 것이 연관관계의 주인
    - Order 엔티티가 FK를 가지고 있으니, Order 엔티티의 member를 연관관계의 주인으로 잡기
    - 즉, '다'가 되는 엔티티(=FK가 있는 엔티티)에 있는 필드가 연관관계의 주인...?(확인해보기)

  - 연관관계의 주인이 되는 필드에는 `@ManyToOne` `@JoinColumn(name = "member_id")`

  - 다른 필드에는 `@OneToMany(mappedBy = "member")`

- 일대일 관계는 어디에 FK를 두든 상관 없음

  - 하지만 주로 접근(조회)을 많이 하는 엔티티에 FK를 두는게 좋음
  - ex) Order, Delivery : 배송정보를 보는 일 보다 주문정보를 보는 일이 많으니 Order에 FK를 두기
  - 마찬가지로 연관관계의 주인을 정해야 함. FK가 있는 엔티티에 있는 필드가 주인!
  - 연관관계의 주인이 되는 필드에 `@OneToOne` `@JoinColumn(name = "delivery_id")`
  - 다른 필드에는 `@OneToOne(mappedBy = "delivery")`



---

### ㄹㅇㄹㅇ 간단 예제

- hello.html

  ```html
  <!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hello</title>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
  <!-- 순수 html일 경우, "안녕하세요. 손님" 출력. 서버사이드 렌더링을 거치면 "안녕하세요. ${data}" 출력. -->
  <p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
  </body>
  </html>
  ```

  

- Member

  ```java
  /**
   * 앱 실행하면 자동으로 스프링이 Member 테이블 만들어줌
   * aplication.yml에서 ddl-auto: create 이므로 엔티티를 쫙 읽어서 자동으로 테이블 생성
   * Member 클래스는 @Entity가 붙었으므로 엔티티임
   */
  package jpabook.jpashop;
  
  import lombok.Getter;
  import lombok.Setter;
  
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.Id;
  
  @Entity
  @Getter @Setter
  public class Member {
  
      //@GeneratedValue: 자동생성
      @Id @GeneratedValue
      private Long id;
      private String username;
  }
  
  ```

- MemberRepository

  ```java
  package jpabook.jpashop;
  
  import org.springframework.stereotype.Repository;
  
  import javax.persistence.EntityManager;
  import javax.persistence.PersistenceContext;
  
  @Repository
  public class MemberRepository {
  
      //이 어노테이션이 있으면 스프링부트가 EntityManger를 주입해줌
      @PersistenceContext
      private EntityManager em;
  
      public Long save(Member member) {
          em.persist(member);
          //직접 member를 반한하지 않고 id만 반환하는 이유?
          //"커맨드랑 쿼리를 분리해라"
          //저장 후 리턴값은 거의 안만드는게 좋음. 대신 id정도는 반환하면 나중에 조회할 때 좋으니까
          return member.getId();
      }
  
      public Member find(Long id) {
          return em.find(Member.class, id);
      }
  }
  
  ```



- MemberRepositoryTest

  ```java
  package jpabook.jpashop;
  
  import org.assertj.core.api.Assertions;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.test.annotation.Rollback;
  import org.springframework.transaction.annotation.Transactional;
  
  import static org.junit.jupiter.api.Assertions.*;
  
  //스프링 프레임워크를 실제로 띄워서 하는 테스트
  @SpringBootTest
  //@RunWith(SpringRunner.class) 는 JUnit5에서는 안써도 됨
  class MemberRepositoryTest {
  
      //의존관계 필드주입 : 강의가 JUnit4 기준이라, 이때는 테스트코드에 생성자주입이 지원x
      @Autowired MemberRepository memberRepository;
  
    @Test
    //@Transactional은 springframework 어쩌구 선택
    //이 어노테이션이 테스트 코드에 있으면 테스트가 끝난 후 Rollback을 함
    //그래서 데이터 넣었어도 롤백되면 사라짐
    //자동롤백을 안하고 싶으면 @Rollback(false)
    @Transactional
    @Rollback(false)
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setUsername("memberA");
  
        //when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(savedId);
  
        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        //결과 : findMember와 member는 같은 객체(같은 트랜잭션 안에서는 id값이 같으면 같은 엔티티로 인식)
        Assertions.assertThat(findMember).isEqualTo(member);
    }
  
  
  }
  ```



- HelloController

  ```java
  package jpabook.jpashop;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.GetMapping;
  
  @Controller
  public class HelloController {
  
      // /hello라는 url이 get 방식으로 오면 이 컨트롤러가 호출됨
      @GetMapping("hello")
      public String hello(Model model) {
          //model에 키가 "data", 값이 "hello!!!"인 데이터를 실어서
          model.addAttribute("data", "hello!!!");
          //resources\templates\hello.html에 전달함
          //hello.html에 ${data}에 값이 전달돼서 그 값이 화면에 출력됨
          return "hello";
      }
  }
  
  ```

  

---



## 실전 예제

- 실무에서는 Getter는 다 열고 Setter는 꼭 필요한 경우에만 가능하도록 하는 것 추천!
  - Setter대신 별도의 메소드를 만들어서 수정하도록 하는게 좋다
  - Item 클래스의 addStock, removeStock 처럼 비즈니스 로직 메소드를 만들어서 setter 대신 값을 변경!
- 항상 디비를 켜고 서버를 실행해야 함(그래야 정상실행됨)

- 엔티티분석을 토대로 각 클래스의 필드를 짬(테이블 분석 다이어그램과는 다름)(테이블 분석 다이어그램은 디비짤 때)
- @Entity 가 적힌 클래스들은 서버 실행시 각 클래스는 테이블로, 필드는 속성으로 하는 테이블들이 생성됨(디비 확인해보면 생성되어 있음)

디비 개념에서 슈퍼테이블, 서브테이블 그거인듯

- `@Inheritance(strategy = InheritanceType.JOINED)` : 가장 정규화된 테이블(슈퍼테이블에 공통속성과 구분자, 서브테이블에 각자 속성 넣고, 테이블 조인) ex) Item, Album, Book, Movie

- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` : 한 테이블에 속성 다 때려넣기(한 테이블에 공통속성, 서브테이블 속성 모두 넣기) ex) Item

- 공통필드를 가지는 클래스는 추상클래스로 설계 (Item)
  - 클래스 앞에 `@DiscriminatorColumn(name = "dtype")` 붙이면 dtype이라는 컬럼이 Item 테이블에 구분자(?)로서 추가됨
- 서브테이블 속성을 필드로 가지는 클래스는 추상클래스를 상속받은 클래스로 설계 (Album, Book, Movie)
  - 클래스 앞에 `@DiscriminatorValue("A")` 붙이면, dtype 컬럼의 값으로 A를 가지는 느낌스..?

- `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)` : 서브테이블에 공통속성, 각자 속성 넣기 ex) Album, Book, Movie



- 엔티티가 될 클래스 앞에는 `@Entity`

- 테이블의 PK가 될 필드 앞에는 `@ID` `@GeneratedValue`

- 컬럼 이름을 직접 지정하고 싶을 땐 `@Column(name = "지정할이름")`
  - 이거 없으면 필드 이름 = 컬럼 이름 으로 자동 지정됨
  - 주로 PK가 될 필드에 대해 사용
    - 필드 이름은 id로 하고, 컬럼 이름을 "엔티티이름_id"

- value type이 될 클래스 앞에는 `@Embeddable`

  - 내장타입. '어딘가에 내장이 될 수 있다'

- value type을 필드로 쓸 경우, 필드 선언 앞에 @Embedded

  - ```java
    @Embedded
    private Address address;
    ```

  - 'value type을 포함했다'

- 양방향 연관관계 매핑

  - 양방향 연관관계 : Member는 orders를 리스트로 가지고 있음. Order도 member를 가지고 있음. -> 양방향 참조가 일어남

  #### 1:다

  - 1:다 관계일 때 '다' 쪽의 엔티티가 FK를 가짐
  - '다'쪽의 엔티티에 있는 필드가 연관관계의 주인
  - '다'쪽의 엔티티에 있는 필드 앞에  `@ManyToOne` `@JoinColumn(name = "member_id")`
    - Order테이블이 '다'쪽이므로 @ManyToOne
    - Member테이블의 member_id를 기준으로 조인
    - member_id가 FK가 될 것
  - '1'쪽의 엔티티에 있는 필드 앞에 `@OneToMany(mappedBy = "member")`
    - Member테이블이 '1'쪽이므로 @OneToMany
    - Order테이블의 member에 의해 매핑됨
    - 이 필드는 읽기 전용

  #### 1:1

  - 일대일 관계는 어디에 FK를 두든 상관 없음

  - 하지만 주로 접근(조회)을 많이 하는 엔티티에 FK를 두는게 좋음
    - ex) Order, Delivery : 배송정보를 보는 일 보다 주문정보를 보는 일이 많으니 Order에 FK를 두기
  - FK가 있는 엔티티에 있는 필드가 주인!
  - 연관관계의 주인이 되는 필드에 `@OneToOne` `@JoinColumn(name = "delivery_id")`
  - 다른 필드에는 `@OneToOne(mappedBy = "delivery")`

  #### 다:다

  - 실무에서는 쓰지 말기
    - 중간 테이블에 (PK를 제외한)속성을 넣을 수가 없음!!

  	- 어쩌고
  	- 저쩌고

  #### 결론

  - 연관관계의 주인이 되는 필드에 `@JoinColumn(name="어쩌고_id")`
  - 다른 필드에 `@연관관계(mappedBy="어쩌고")`
  - 양방향 연관관계에 있는 필드들에 `@ManyToOne` `@OneToMany` `@OneToOne` `@ManyToMany`(사용x)



- 상태값(ex) 배송상태 : 배송준비, 배송) 같은 건 enum으로 클래스 만들기

  - ```java
    package jpabook.jpashop.domain;
    
    public enum DeliveryStatus {
        READY, COMP
    }
    ```

- enum타입을 필드로 쓸 때는 선언 앞에 `@Enumerated(EnumType.STRING)` 붙이기

  - 디폴트값은 ORDINAL(숫자)
  - 반드시 STRING으로 바꿔야 함
    - 안그러면 나중에 enum값이 수정됐을 때 다 고쳐야 함 ex) 배송준비, 배송중, 배송

- 모든 연관관계는 반드시 지연로딩(`LAZY`)으로 설정!

  - 즉시로딩(`EAGER`) : 한 엔티티를 로딩할 때 연관된 모든 엔티티들을 다 가져오는 것 -> 문제가 자주 발생
  - 지연로딩(`LAZY`) : 한 엔티티가 로딩되는 시점에 LAZY 로딩 설정이 되어있는 연관된 엔티티는 프록시 객체(가짜객체)로 가져옴. 후에 연관된 엔티티의 객체를 실제로 사용하는 시점(객체의 값을 꺼내거나 수정)에 쿼리가 나가서 객체의 값을 채워줌.
  - 클래스 내에서 필드 선언할 때 필드 앞에`@연관관계(fetch = FetchType.LAZY)`
  - `@ManyToOne`, `@OneToOne` (`@어쩌고ToOne`)은 디폴트가 EAGER이기 때문에 반드시 LAZY로 바꿔줘야 함
  - `fetch = FetchType.LAZY`는 static import(alt + 엔터)해서 깔끔하게 `fetch = LAZY`로 바꿔주면 좋음'

- 컬렉션은 필드에서 바로 초기화하기!

  - ```java
    //필드에서 초기화(O)
    @OneToMany(mappedBy = "member")
        private List<Order> orders = new ArrayList<>();
    
    //필드 선언 후, 생성자에서 초기화(X)
    @OneToMany(mappedBy = "member")  private List<Order> orders;
    public Member() {
        orders = new ArrayList<>();
    }
    ```

  - 초기화 이후 수정하지 말기(수정하면 hibernate가 원하는 메커니즘대로 동작 안할 수 있음)

- `@Table(name = "테이블이름")` `@Column(name = "컬럼이름")` 이렇게 직접 이름 지정하지 않는 경우에 자동 설정되는 테이블(엔티티)이름, 컬럼 이름

  - 클래스(엔티티) 이름 -> 테이블 이름
  - 필드 이름 -> 컬럼 이름

  1. 카멜 케이스(memberPoint) -> 언더스코어(member_point)
  2. . -> _
  3. 대문자 -> 소문자

- 필드 선언 앞에 `@연관관계(cascade = CascadeType.ALL)`

  - 이 엔티티를 persist할 때 이 필드도 같이 persist 해줌

  -  원래는 각각 persist 해줘야 함

  - OrderService 에서 원래대로라면 deliveryRepository.save 통해 em에 delivery를 persist 해주고, orderItemRepository.save 통해 em에 orderItem을 persist 해주고... 각각 persist해준 후 주문 생성이 가능! 하지만 Order 클래스의 orderItems, delivery 필드에 대해 `cascade=CascadeType.ALL` 옵션이 걸려있기 때문에 order를 persist하면 OrderItem, Delivery도 강제로 persist 해줌

  - OrderItem, Delivery를 오직 Order만 참조해서 사용 : 이런 경우에만 cascade 옵션 걸기

    - 여러 엔티티가 참조해서 사용할 때는 별도로 persist하는게 낫다

    

- 연관관계 편의 메소드

  - 모르겠음!!!!!!!!!!다시 보기(엔티티 설계시 주의점 거의 마지막)

- @PersistenceContext

  - ```java
    //@Repository
    //스프링이 EntityManager를 만들어서 주입해줌
    @PersistenceContext
     private EntityManager em;
    ```

  - JPA 쓰면 EntityManager에 대해서 이렇게 가능

    ```java
    //이 코드를
    @Repository
    public class MemberRepository {
        //스프링이 EntityManager를 만들어서 주입해줌
        @PersistenceContext
        private EntityManager em;
        ...
    }
    ```

    ```java
    //이렇게 바꿀 수 있음 -> @Service 클래스 내 코드들과 일관성이 생김
    @Repository
    @RequiredArgsConstructor
    public class MemberRepository {
        private final EntityManager em;
    	...
    }
    ```

    - 원래 EntityManger는 @PersistenceContext로만 주입 가능. 하지만 JPA가 @Autowired로도 주입 가능하도록 해줌. 따라서 저런 코드가 가능!(@Autowired 위치 개념 참고)

- @Transactional

  - JPA의 데이터 변경, 모든 로직은 트랜잭션 안에서 실행되어야 함

  - `@Transactioanl(readOnly = true)` : 읽기(조회)만 할 때 사용 -> 성능이 좀더 향상됨

  - 조회 기능의 메소드가 많은 @Service 클래스에는

    ```java
    @Service
    @Transactional(readOnly = true)
    public class MemberService {
        ...
        //데이터 변경(쓰기)을 하는 메소드에는 @Transactional 붙이기
        @Transactional
        public Long join(Member member) {
            ...
        }
    }
    ```

    - 쓰기 기능의 메소드가 많은 경우 반대로 가능

- @Autowired 위치

  - 생성자 의존관계 주입이 최고

  - 생성자가 하나만 있는 경우 생성자 위에 @Autowired 쓰지 않아도 자동으로 의존관계 주입 됨
  - 필드는 final로 할 것
    - final : 반드시 최초에 초기화 해야 함, 그 이후 수정 불가
  - 클래스 앞에 @RequiredArgsConstructor
    - final인 필드만 가지고 생성자를 자동으로 만들어줌
    - @AllArgsConstructor : 모든 필드를 가지고 생성자를 자동으로 만들어줌
  - 결론 : 클래스 앞에 @RequiredArgsConstructor, 의존관계에 있는 필드는 final -> final이 붙은 필드에 대해 자동으로 생성자 생성, 의존관계 주입

- 테스트 코드 짜기

  ```java
  @SpringBootTest
  @Transactional
  class MemberServiceTest {
  
      @Autowired MemberService memberService;
      @Autowired MemberRepository memberRepository;
  
      @Test
      public void 회원가입() throws Exception {
          //given(이게 주어졌을 때)
          Member member = new Member();
          member.setName("kim");
  
          //when(이걸 실행하면)
          Long savedId = memberService.join(member);
  
          //then(이런 결과가 나와야 함)
          //member가 memberRepository에서 찾아온 member랑 똑같으면 ok
          //@Trnasactioanl -> 같은 트랜잭션 안이기 때문에 가능한 것
          assertEquals(member, memberRepository.findOne(savedId));
      }
  }
  ```

   - `@SpringBootTest` : 스프링부트를 띄운 상태에서 테스트하려면 이게 있어야 함

   - @Transactional 하면 트랜잭션 실행 후 rollback하기 때문에 디비에 쿼리를 날리지 않음. 디비에 넣는 순서가 오기도 전에 롤백 해버려서 디비에 쿼리를 날리지 않음. 따라서 디비로 날리는 쿼리문을  결과창으로 볼 수 없음.

     1. `@Autowired EntityManager em;`을 필드로 선언하고, //then 첫 문장에 `em.flush();`를 적어서 강제로 디비에 쿼리를 날리기
        - -> 강제로 디비에 쿼리 날렸다가, 트랜잭션 실행 후 롤백돼서 쿼리문을 결과창으로 볼 수 있음
     2. 클래스 앞에 `@Rollback(false)` 추가
        - -> 롤백이 안됨 -> 디비에 데이터가 들어가고 그대로 유지

  - 쓸모있는 메소드들

    1. `assertEquals(기대값, 실제값);`

       - 기대값, 실제값이 같으면 ok
       - `Assert.assertEquals("메시지", 기대값, 실제값);`

    2. Assertions.assertThat

    3. `assertThrows(예외.class, () -> 실행하면 예외가 발생해야 하는 코드);`

       - 이 코드를 실행해서 이런 예외가 발생하면 ok

       - ```java
         assertThrows(IllegalStateException.class, () -> memberService.join(member2));
         ```

  - 테스트 돌릴 때만 쓰는 메모리 db 만들기
    - test 폴더에서는 test폴더 내에 있는 것들이 우선권을 가짐
    - test 폴더 안에 resources 패키지 만들고, 그 안에 application.yml 파일 복붙
    - application.yml 파일에서 spring: 부분의 설정 모두 지우기
      - 별도의 설정이 없으면 기본적으로 메모리모드로 돌아가기 때문
  - 좋은 테스트는 db와의 연동 없이 순수하게 메소드를 단위테스트 하는 것

- 예외 클래스도 직접 만들 수 있음!

  - exception 패키지 만들고 그 안에 클래스 생성
  - 기본제공 예외 상속 받아야 함
    - `public class NotEnoughStockException extends RuntimeException`
  - 기본제공 예외의 메소드들 override 해야 함

- 예외 날리는 법

  - `throw new IllegalStateException("예외메시지");`

- 엔티티 객체를 오직 생성 메서드를 통해서만 생성할 수 있도록 하고 싶을 때 다른 클래스에서 OrderItem 생성자 호출 불가하도록 하기

  1. protected 생성자를 정의
     - `protected OrderItem() {}`
  2. 클래스 이름 앞에 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`

- 원래는 코드에서 데이터 변경이 일어나면, 직접 db에 쿼리 날려서 데이터 변경 해야함. 하지만 JPA는 데이터 변경이 일어나는 걸 감지해서 자동으로 db에 쿼리를 날려줌 : 변경감지

  - ```java
    Book book = em.find(Book.class, 1L);
    book.setName("newName");
    //이후 트랜잭션이 끝나 커밋이 되면 엔티티 변경을 감지해서 자동으로 db에 쿼리를 날려 db에 반영을 함 : dirty checking 
    ```

- 준영속 엔티티 : 이미 한번 db에 갔다와서 식별자가 있는 엔티티

  - ItemController의 updateItem메소드에서 book 객체. book 자체는 새로 생성한 객체지만, id가 셋팅되어있다는 건 이미 db를 한번 갔다왔다는 뜻(book은 form을 통해 값을 주입받지만 이 form도 db에 있는 book객체의 값을 가져온 것!)(persist되어야 - db에 저장되어야 - 식별자를 가짐)

    - ```java
      Book book = new Book();
      book.setId(form.getId()); //book은 식별자를 이미 가지므로 준영속 상태의 객체!
      ```

  - 준영속 엔티티는 JPA가 관리하지 않기 때문에 변경감지를 하지 않아서 자동으로 db 업데이트가 일어나지 않음.

  - 준영속 엔티티 수정 방법 -> 1번 사용하기!

    1. 변경감지 기능 사용

       ```java
       //service/ItemService 에 엔티티 변경 메소드를 만들기
       @Transactional
       public void updateItem(Long itemId, Book param) {
           Item findItem = itemRepository.findOne(itemId);
           findItem.setPrice(param.getPrice());
           findItem.setName(param.getName());
           ...
       }
       ```

       - findItem은 repository를 통해 db에서 가져온 엔티티이므로 JPA가 관리하는 영속 상태 엔티티
       - 따라서 findItem의 값이 변경되면 JPA가 변경감지를 해 자동으로 db에 반영해줌
       - 내가 원하는 필드만 변경할 수 있다
       - 파라미터가 너무 많아진다면 DTO 클래스를 생성하기

    2. merge(병합) 사용

       ```java
       public void save(Item item) {
           if (item.getId() == null) {
               em.persist(item);
           } else {
               //id값이 있는 item이면(=준영속상태)
               em.merge(item);
           }
       }
       ```

       - 객체의 모든 필드를 교체하므로, 값이 없으면 null로 업데이트 될 위험이 있다 -> 실무에서 앵간하면 안쓰고 1번의 방법 사용하기

- em.persist 해야 엔티티가 db에 저장됨. 그 전까지는 객체가 생성되어도 db에 저장x. 

  - @Id 필드를 @GeneratedValue로 해두면 엔티티가 db에 저장될 때 id값이 자동생성됨. 즉 persist하기 전까지는 id값이 없음.

- `@NotEmpty(message = "메시지")` : 이 어노테이션이 붙은 필드는 반드시 값이 있어야 함. 안그러면 오류남

  - build.gradle에서 `implementation 'org.springframework.boot:spring-boot-starter-validation'` 추가 후 사용 가능

  - `@NotNull`과의 차이 : NotNull은 Null만 허용하지 않음. NotEmpty는 Null, "" 둘 다 허용하지 않음.

    - `@NotBlank` : Null, "", " " 모두 허용하지 않음

  - 컨트롤러에서 @NotEmpty 가 붙은 필드가 있는 객체를 파라미러로 받아오려면 `@Valid` 를 붙여야 함

    - ```java
      public String create(@Valid MemberForm form) { ... }
      ```

- 컨트롤러에서 addAttribute를 통해 '엔티티'를 Model에 담아서 넘기는 건 괜찮지만, API에서는 절대 엔티티를 그대로 외부로 넘기면 안됨

  - 엔티티에 필드를 하나 추가할 경우, API의 스펙이 변해버리기 때문
  - 컨트롤러에서도 DTO/ 화면에 맞는 데이터트랜스 오브젝트로 변환해서 넘기는게 좋음

- 클래스 이름 앞에 `@Slf4j ` : logger를 사용할 수 있음

  - `log.info("메시지");`

- `@PathVariable`

  - http://127.0.0.1/index/1 형식(index=1을 저런식으로 표현)의 URL을 처리할 때 사용

  - 클라이언트에서 URL에 파라미터를 전달할 때, 그 파라미터를 메소드의 매개변수로 쓰는 경우에 앞에 붙임

  - ```java
    @GetMapping("items/{itemId}/edit")
        public String updateItemForm(@PathVariable("itemId") Long itemId, Model model) {}
    ```

- `@ModelAttribute`

  - 어노테이션이 붙는 클래스에 Getter, Setter가 존재해야 함

  - `@ModelAttribute("이름")` : 설정 안하면 스프링이 자동으로 Command 객체의 이름을 클래스이름을 소문자로 바꾼 이름으로 설정함. 설정하면 이대로 지정!

  - 1. 파라미터로 넘겨준 타입의 객체를 자동 생성
    2. 생성된 객체에, http로 넘어온 값들을 자동으로 채워넣음
    3. 이 어노테이션이 붙은 객체가 자동으로 Model에 추가되고 return문의 html로 전달됨

  - ```java
    //ItemController
    @PostMapping("items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form) {
            Book book = new Book();
            book.setId(form.getId()); //자동으로 form객체가 채워지므로 이렇게 사용 가능
        	...
    }
    
    //updateItemForm.html
    <form th:object="${form}" method="post">
    //에서 입력받은 값들을 form 객체에 채워넣음
    ```

    ```java
    //OrderController
    @GetMapping("/orders")
        public String orderList(@ModelAttribute("orderSearch")OrderSearch orderSearch, Model model) {
            List<Order> orders = orderService.findOrders(orderSearch);
            model.addAttribute("orders", orders);
    
            return "order/orderList";
        }
    
    //orderList.html
    <form th:object="${orderSearch}" class="form-inline">
    //에서 입력받은 값이 orderSearch 객체에 담김
    ```

    - `<form th:object="${} ..."` 연결해줌???

- `@RequestParam`





- EntityManager 관련 메소드
  - Repository에서 `private final EntityManager em;` 선언 후 사용
  - `em.persist(저장할객체)`
    - 저장
  - `em.merge(객체)`
    - 준영속 엔티티를 수정할 때 사용
    - 영속 상태의 객체를 반환(괄호안의 객체는 여전히 준영속 상태)
    - 객체의 모든 필드를 교체하므로, 값이 없으면 null로 업데이트 될 위험이 있다 -> 실무에서 앵간하면 안씀
  - `em.find(찾을객체의타입.class, PK)`
  - `em.createQuery("jpql문", 반환할객체의타입.class)`





### 패키지 구성

#### domain

: Entity 를 넣는 패키지

: 회원, 주문, 쿠폰 등 db에 저장되고 관리되는 비즈니스 도메인 객체를 넣는 패키지

: enum타입도 정의할 수 있음

```java
@Entity
//굳이 orders로 한 이유 : Order 그대로 하면 sql문법 중에 ORDER BY의 예약어가 ORDER라서, ddl-auto:create 옵션을 적용할 경우 에러가 발생하기 때문
@Table(name = "orders") 
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status; //주문상태 [ORDER,CANCEL]

    //==연관관계 편의 메서드==//
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }

    //==생성 메서드==//
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    //==비즈니스 로직==//
    /**
     * 주문 취소
     */
    public void cancel() {
        if(delivery.getStatus() == DeliveryStatus.COMP) {
            throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }
        this.setStatus(OrderStatus.CANCEL);
        for (OrderItem orderItem : orderItems) {
            orderItem.cancel();
        }
    }

    //==조회 로직==//
    /**
     * 전체 주문 가격 조회
     */
    public int getTotalPrice() {
        int totalPrice = 0;
        for (OrderItem orderItem : orderItems) {
            totalPrice += orderItem.getTotalPrice();
        }
        return totalPrice;
    }
}

```

- 클래스 이름 앞에 `@Entity`
- 클래스 이름 앞에 `@Getter` -> lombok을 통해 getter 자동 생성
- 수정 가능하도록 할 필드는 별도의 메소드(비즈니스 로직)를 만들기
  - `@Setter` 보다는 딱 필요한 필드에 관해서만 수정할 수 있도록
- id로 쓸 필드(PK가 될 필드) 앞에 `@Id`, `@GeneratedValue`
  - `@Column(name = "엔티티명_id")` 로 컬럼명을 지정해주는게 좋음
- null 불가능한 필드 앞에 `@Column(nullable = false)`
- 양방향 연관관계를 가지는 필드 앞에 `@연관관계` 
  - `@ManyToOne`, `@OneToMany`, `@OneToOne`
  - 양방향 연관관계 : 두 클래스에서 서로의 타입을 참조해서 사용중
  - @ㅇㅇToOne은 반드시 `(fetch = FetchType.LAZY)`
- 연관관계 편의 메서드 : 양방향 연관관계에 있는 필드를 수정할 때 서로 엮여있는 필드들을 모두 수정해줘야 함. 따라서 그런 필드를 수정할 때 메소드로 편하게 수정하기 위해 정의 (내 추측)
- 생성 메서드 : 이 클래스의 객체를 생성할 때, 이 클래스의 필드로 다른 타입의 객체가 선언되어 있을 때(다른 타입의 객체가 엮여있을 때, 즉 양방향 연결관계), 별도의 메서드를 둬서 편하게 객체 생성하기 위해 정의
  - 주문 생성은 delivery, orderItem 등 여러개랑 엮여있음(필드로 선언되어 있음). Order 객체를 생성할 때 이 필드들의 값을 모두 설정해줘야 하므로 메소드를 통해 한번에 편하게 모두 설정 가능
  - static으로 정의 -> 다른 클래스에서 `클래스이름.생성메서드()`로 호출 가능
  - 다른 클래스에서 생성 메서드로만 객체 생성 가능하도록 하고 싶을 때 : 클래스 이름 앞에 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- 비즈니스 로직 : 엔티티 안의 필드와 관련있는 비즈니스 로직은 엔티티 클래스 안에 작성
  - 도메인 모델 패턴 : 엔티티 안에 비즈니스 로직을 작성하는 스타일
  - 트랜잭션 스크립트 패턴 : 서비스 계층에서 비즈니스 로직을 작성하는 스타일
  - 뭐가 더 낫다 얘기할 수 x, 한 프로젝트 안에 두 패턴 동시에 사용 가능
- 조회 로직 : 복잡한 조회는 별도의 메서드를 정의



##### Enum

```java
package jpabook.jpashop.domain;

public enum DeliveryStatus {
    READY, COMP
}
```



#### repository

: db에 접근하고, 도메인 객체를 db에  저장하고 관리

```java
@Repository
@RequiredArgsConstructor
public class MemberRepository {

   private final EntityManager em;

   public void save(Member member) {
       em.persist(member);
   }

   public Member findOne(Long id) {
       return em.find(Member.class, id);
   }

   public List<Member> findAll() {
       //createQuery : (JPQL, 반환타입)
       //JPQL은 쿼리의 대상이 테이블이 아니라 엔티티(Member)
       return em.createQuery("select m from Member m", Member.class)
               .getResultList();
   }

   public List<Member> findByName(String name) {
       return em.createQuery("select m from Member m where m.name = :name", Member.class)
               .setParameter("name", name)
               .getResultList();
   }
}


```

- 클래스 이름 앞에 `@Repository`
- 클래스 이름 앞에 `@RequiredArgsConstructor` 
  - EntityManager에게 의존관계 자동 주입
- 클래스 안에 db에 관한 기능에 대한 메소드를 정의
  - 메소드 내용물은 EntityManager의 메소드를 사용해서 작성
    - `em.persist(객체);`
    - `em.find(찾을객체타입.class, id);`



#### service

: 도메인 객체를 가지고 핵심 비즈니스 로직이 동작하도록 구현한 객체

```java
@Service
@Transactional
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    /**
     * 회원 가입
     */
    public Long join(Member member) {
        validateDuplicateMember(member); //중복회원 검증
        memberRepository.save(member);
        return member.getId();
    }

    //중복회원 검증
    private void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    /**
     * 회원 전체 조회
     */
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    /**
     * 회원 한명 조회
     */
    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }
}

```

- 클래스 이름 앞에 `@Service`
- 클래스 이름 앞에 `@Transactional`
  - JPA의 데이터 변경, 로직은 트랜잭션 안에서 실행되어야 함
  - `@Transactional(readOnly = true)` : 데이터 변경 없이 조회만 하는 기능이 많을 경우에 클래스 이름 앞에 붙이기 -> 성능 up
    - 데이터 변경이 있는 메소드가 있을 경우 메소드 정의 앞에 `@Transactional` 별도로 붙이기
- 클래스 이름 앞에 `@RequiredArgsConstructor`
- 다루는 엔티티에 대한 repository를 final 필드로 선언
  - 엔티티 객체의 값을 꺼내오거나 수정하려면 repository가 필요
- 클래스 안에 각 기능에 대한 메소드를 정의



#### controller

: 웹 MVC의 컨트롤러 역할

```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    /**
     * home 화면에서 '회원가입' 버튼을 누르면 회원가입 창을 불러옴(get)
     */
    // /members/new 라는 url로 get요청이 오면 이 컨트롤러가 호출됨
    @GetMapping("/members/new")
    public String createForm(Model model) {
        
        //model에 키가 "memberForm", 값이 MemberForm객체 인 데이터를 실어서
        model.addAttribute("memberForm", new MemberForm());
        //resources/templates/members/createMemberForm.html에 전달하고 html이 열림
        return "members/createMemberForm";
    }

    /**
     * 회원가입 창(/members/new)에서 submit 버튼을 누르면 회원가입이 됨(회원 등록 : post)
     */
    // /members/new 라는 url로 post 요청이 오면 이 컨트롤러가 호출됨
    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result) {

        if (result.hasErrors()) {
            return "members/createMemberForm";
        }
        
        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());
        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);

        //첫 번째 페이지로 넘어감
        return "redirect:/";
    }


}
```

- 클래스 이름 앞에 `@Controller`

- @GetMapping, @PostMapping, @RequestMapping

- 패키지 안에, html에서 form태그로 받을 정보를 묶은 객체를 넣기도 함.
  
- MemberForm : name, city, street, zipcode를 필드로 함
  
- Model객체.addAttribute("키값", 데이터) : model에 데이터를 담음

  - 모델을 return문의 html로 넘겨줌
    - resources/templates 패키지에 해당 이름의 html이 있을 경우 거기로, 없을경우 resources/static패키지의 해당 html에 넘겨줌
  - html에서 ${키값} 으로 받을 수 있음

- service 를 필드로 두는 경우가 많음(service의 메소드를 이용)

- PostMapping에서 데이터 등록(post) 후 특정 페이지로 이동할 때

  `return "redirect:[url]"`

  - `return "redirect:/items"` `return "redirect:/";`
  
- 간단한 조회는 controller에서 직접 repository를 참조하기도 한다(영한님 왈)

  - 원래는 service를 통해서 해야 함
  - 전체적인 사용 흐름 : repository -> service -> controller

하나의 정보 묶음(어떤 한 개체에 대한 정보들)을 한번에 화면에 보여줘야 할 때, 그 정보들을 객체에 담아서 통째로 Model에 실어 전달하기도 함.

```java
//BookForm.java
//ItemController에서 updateItemForm 메소드 참고
package jpabook.jpashop.controller;

import lombok.Getter;
import lombok.Setter;

@Getter @Setter
public class BookForm {

    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;
}

```



---



#### 레이아웃

Bootstrap 사용해서 간단하게 레이아웃 만들기(나는 백엔드니까 프론트는 대충!)

##### 직접 파일 다운하는 방법

1. Bootstrap 사이트에서 Compiled CSS and JS 다운로드

2. css, js 폴더 복사
3. 프로젝트/resources/static에 붙여넣기
4. Build>Build Project 후 서버 재시작
5. html의 레이아웃이 예쁘게 바뀌어있음

##### 다운 없이 CDN을 이용해 적용하는 방법

resources/templates/fragments/header.html의 `<head></head>`안에 복붙

```html
<!-- Bootstrap CDN -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
```



#### Controller와 templates폴더의 html파일 연결

- MemberController.java

  ```java
  @Controller
  @RequiredArgsConstructor
  public class MemberController {
  
      private final MemberService memberService;
  
      /**
       * home 화면에서 '회원가입' 버튼을 누르면 회원가입 창을 불러옴(get)
       */
      // /members/new 라는 url로 get요청이 오면 이 컨트롤러가 호출됨
      @GetMapping("/members/new")
      public String createForm(Model model) {
          
          //model에 키가 "memberForm", 값이 MemberForm객체 인 데이터를 실어서
          model.addAttribute("memberForm", new MemberForm());
          //resources/templates/members/createMemberForm.html에 전달하고 html이 열림
          return "members/createMemberForm";
      }
  
      /**
       * 회원가입 창(/members/new)에서 submit 버튼을 누르면 회원가입이 됨(회원 등록 : post)
       */
      // /members/new 라는 url로 post 요청이 오면 이 컨트롤러가 호출됨
      @PostMapping("/members/new")
      public String create(@Valid MemberForm form, BindingResult result) {
  
          if (result.hasErrors()) {
              return "members/createMemberForm";
          }
          
          Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());
          Member member = new Member();
          member.setName(form.getName());
          member.setAddress(address);
  
          memberService.join(member);
  
          //첫 번째 페이지로 넘어감
          return "redirect:/";
      }
  
  
  }
  ```

  - `return "redirect:/items";` : /items 로 넘어감

- createMemberForm.html 의 form 태그 부분

  ```html
  <form role="form" action="/members/new" th:object="${memberForm}"
          method="post">
      <div class="form-group">
        <label th:for="name">이름</label>
        <input type="text" th:field="*{name}" class="form-control"
               placeholder="이름을 입력하세요"
               th:class="${#fields.hasErrors('name')}? 'form-control
  fieldError' : 'form-control'">
        <p th:if="${#fields.hasErrors('name')}"
           th:errors="*{name}">Incorrect date</p>
      </div>
      <div class="form-group">
        <label th:for="city">도시</label>
        <input type="text" th:field="*{city}" class="form-control"
               placeholder="도시를 입력하세요">
      </div>
      <div class="form-group">
        <label th:for="street">거리</label>
        <input type="text" th:field="*{street}" class="form-control"
               placeholder="거리를 입력하세요">
      </div>
      <div class="form-group">
        <label th:for="zipcode">우편번호</label>
        <input type="text" th:field="*{zipcode}" class="form-control"
               placeholder="우편번호를 입력하세요">
      </div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
  ```

  - `<form role="form" action="/members/new" th:object="${memberForm}" method="post">`
    
    - th: 에 걸려있는 건 thymeleaf 문법
    - `th:object="${memberForm}" `: 이 form태그 안에서는, controller에서 넘어온, 키가 "memberForm"인 데이터(여기선 MemberForm 객체)에 접근할 수 있음
    - `th:field="*{name}"` :  input 태그에서 id, name속성을 보통 똑같게 지정해주곤 하는데, field 문법이 자동으로 id, name 속성의 값을 "name"으로 지정해줌. 그리고 form태그에서 지정한 객체의 name 필드에 접근.
    
    - 앞에 @ModelAttribute 설명이랑 같이 보면... `object="${키값}"`도 되지만 컨트롤러 메소드의 파라미터에 @ModelAttribute 가 붙은 경우, `object="${@ModelAttribute가 붙은 객체의 이름}"` 으로도 연결되는 것 같기도 함...ㅠㅠ