## Spring Project 구조

### main/java/controller

client의 url 요청에 대한 응답 메소드를 작성

---

### main/java/domain

#### entity

- 식별자(@Id)를 가짐
- 엔티티의 필드 = db의 컬럼
- 식별자를 제외하고 필드값이 바뀔 수 있음
  - 식별자만 같다면 동일한 엔티티

> 현식쟝 플젝은 jpa가 아닌 MyBatis를 사용했기 때문에 dto, vo로 나눈 것!
>
> MyBatis에서 VO가 JPA의 Entity라고 할 수 있다. 

#### value

- 식별자를 가지지 않음
- 데이터(값) 그 자체
-  read-only
  - Setter구현 x
  - 각 필드의 값은 생성자로만 받기
  - 값을 가져올 땐 @Getter
  - 새로운 값을 넣고 싶다면 객체를 새로 생성

---

### main/java/dto

- data transfer object
- entity와 dto의 차이
  - entity
    - DB Layer와 데이터 교환을 위한 클래스
  - dto
    - View Layer와 데이터 교환을 위한 클래스
    - db에서 꺼낸 엔티티를 view layer에서 사용할 목적으로 필요한 필드만 모은 클래스
    - $$$$$$$$어디서 쓰는지, entity로 변환하기 등은 강의 듣고 작성!$$$$$$$$
    - api의 요청 스펙에 맞춰서 DTO클래스를 만들어야 함
    - api를 만들 때에는 파라미터로 무조건 DTO를 받아야함(엔티티 x), 엔티티를 외부에 노출해서도 안됨

- entity와 dto는 분리해줘야 한다!
  - View와 통신하는 Dto 클래스는 UI에 따라 자주 변경이 된다. 반면, 테이블에 매핑되는 Entity는 그에 비해 변경도 적고, 변경됐을 때 영향범위도 매우 크다. (Repository 클래스의 Entity Manager의 flush가 호출될 때 DB에 값이 반영되고, 이는 다른 로직들에도 영향 미친다) 
  - DB로부터 조회된 모든 Entity를 View로 넘기게 되면, 원하지 않는 정보(ex) password 필드)까지 전달하게 되어 정보 노출에 대한 문제가 생길 수 있고, 이를 막기 위한 비즈니스 로직과는 상관없는 방어 로직들이 생겨서 Entity가 지저분해진다.
  - UI 계층에서 Entity 클래스의 메서드를 호출하거나 상태를 변경시킬 위험이 존재한다.
  - Entity의 필드명은 바뀌는 경우가 많다. Controller에서 Entity를 사용하면, Entity와 API스펙이 1:1 매핑되어있기 때문에 Entity가 바뀌는 경우 api스펙 자체가 바뀌어야 한다.

---

### main/java/exception

- 다른 코드에서 터지는 예외를 작성(다른 코드에서는 이 예외가 터지면 어떻게 처리하겠다고 작성!)
- 현식쟝은 @ResponseStatus(HttpStatus.NOT_FOUND 혹은 UNAUTHORIZED...) 이런 어노테이션을 붙였는데 이게 뭐지?

### main/java/repository

- db에 데이터를 CRUD(sql을 사용하여 db에 접근) 하는 역할

- repository = dao

  - 개념적으로는 조금 다르지만, dao 또는 repository 둘 중 하나만 패키지로 두는 듯

  - #### dao

    - data access object

    - db에 접근하려면 sql을 사용하기 때문에, 이를 위해 connection을 생성하고 직접 쿼리문을 작성하여 connection을 닫아야 함. 이는 번거롭기 때문에 사용할 db로직을 객체 하나에 메서드로 구현, 이를 호출하여 사용하도록 만든 것

      -> 오직 1개의 connection으로 다수의 요청을 모두 수행할 수 있다!

    - repository와의 차이 : respoitory는 집합처리, dao는 개별적 처리(?)

      -> 하나의 repository에서 다수의 dao를 호출하는 방식으로 repository를 구현

- jpql을 사용하는 방법과 그냥 sql문을 사용하는 방법이 다른 듯

- 



### main/java/service