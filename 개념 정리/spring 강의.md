https://start.spring.io/

스프링부트 기반으로 스프링 관련 프로젝트를 만들어주는 사이트

요즘에는 Gradle 프로젝트를 많이 씀

언어는 Java

snapshot: 아직 만들고 있는 버전, m1:정식 릴리즈 되지 않은 버전

정식 릴리즈 버전 중에 가장 최신인 2.3.1선택

group:기업도메인명

artifact:프로젝트명

나머지는 그대로 유지

dependencies: 어떤 라이브러리를 땡겨서 쓸지. spring web, thymeleaf 선택

generate버튼 누르기

intellij에서 open/import 선택해서 만든 프로젝트(프로젝트명 폴더)에서 build.gradle 을 선택 - open as project - build클릭

test폴더 - test코드 관련된 소스들 들어감

resources- 자바파일 제외한 나머지

build.gradle - 버전설정, 라이브러리 땡겨오기

sourceCompatibitlity: 자바 버전

repositories - mavenCentral() //mavenCentral이라는 사이트에서 라이브러리 받아라

dependencies - 라이브러리들

.gitignore - git관련

main-java-프로젝트명폴더-메인파일에서 메인메소드 실행시키면 포트번호 확인 가능 : Tomcat started on port(s): 8080 (http)

웹브라우저에 localhost:8080 입력

Ctrl+Alt+S : Setting 창을 열기

build Tools-gradle- build and run using, run test using에서 intelliJ를 선택하면 빌드가 훨씬 빨리 됨(gradle을 통하지 않고 바로 실행됨)

springboot starter web라이브러리만 받으면 얘가 필요한 다른 라이브러리들을 다 가져옴(의존관계에 의해)

왼쪽하단 네모 클릭한 뒤 오른쪽상단에 gradle눌러서 dependencies 누르면 의존관계 확인 가능

소스 라이브러리에서 웹서버를 가지고 있음(임베디드) 그래서 실행만 해도 별도의 설정 없이 웹서버가 뜸 더이상 톰캣서버 직접깔고 그런거 안해도 됨

실무에서는 print보다 로그를 사용해서 테스트해야함(그래야 에러를 모아볼 수도 있고 로그파일을 관리할 수 있음)

스프링부트 라이브러리

- spring-boot-starter-web
  - spring-boot-starter-tomcat : 톰캣(웹서버)
  - spring-webmvc : 스프링 웹 MVC
- spring-boot-starter-thymeleaf : 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통) : 스프링부트 + 스프링코어 + 로깅
  - spring-boot
    - spring-core
  - spring-boot-starter-logging(로그 관련)
    - logback, slf4j

테스트 라이브러리

- spring-boot-starter-test
  - junit: 테스트 프레임 워크
    - 요즘은 junit5를 많이 쓰는 추세
  - spring-test: junit을 스프링과 통합하도록 도와주는 라이 브러리



스프링의 기능은 너무 방대하기 때문에 검색을 하는 능력이 중요하다.

spring.io - projects - spring boot - learn

koda.hellospring - (package로 선택후)controller 생성 - HelloController(java class) 생성



빌드하고 실행하기

1. 프로젝트 끄기(ㅁ버튼 눌러서 실행중이던 서버 종료)

2. cmd창에서 프로젝트 폴더(hello-spring)로 이동

3. `gradlew`

4. `gradlew build`

5. `cd build/libs`

6. `java -jar hello-srping-0.0.1-SNAPSHOT.jar`

   스프링이 시작됨

   이 파일만 배포해서 실행하면 서버에서 스프링이 동작함.



<웹의 동작? 방식>

- 정적컨텐츠 : html 파일을 그대로 웹브라우저에 전달해주는 것

- 템플릿 엔진: jsp, php. 서버에서 html파일을 동적으로 바꿔서 웹브라우저에 전달해주는 것. ex) thymeleaf. mvc=model, view, controller. 아래에서 순서 붙여서 설명한 것들이 대부분 이 mvc 방식

- api 방식: json 데이터 포맷으로 클라이언트에게 데이터를 전달  



static폴더에는 정적 html파일을 넣는듯?



1. 웹브라우저가 `localhost:8080/hello`를 내장 톰캣 서버에게 넘겨줌

2. 톰캣 서버가 스프링 컨테이너에 넘겨줌

3. HelloController에 @GetMapping("hello")가 있으므로 매칭이 됨

4. HelloController에 있는 public String hello 메소드가 실행됨. 

5. viewResolver에게 `return "hello";` 를 리턴하고`model(data:hello!!)`(키가 data이고 값은 hello!!인 데이터를 넣은 모델)를 넘겨줌

6. viewResolver가 templates폴더에 hello.html(리턴한 string과 똑같은 이름의 파일)이 있는지 찾아보고 있으면, thymeleaf 템플릿엔진에 처리해달라고 넘김

7. 템플릿 엔진이 렌더링하고 변환한 (${data}의 값이 받아온 값으로 변환됨) html을 웹브라우저에 반환 

   cf) 정적컨텐츠는 변환을 하지 않았음

   

`localhost:8080/hello-static.html` 이렇게 치면 정적 페이지(hello-static.html) 열 수 있음

1. 웹브라우저에 저 주소를 치니까, 내장 톰켓 서버가 요청을 받고 스프링 컨테이너에게 넘김
2. 스프링 컨테이너는 hello-static 컨트롤러가 있는지(hello-static이 매핑되어있는지) 찾아봄
3. 앞에서 못찾았으면, resources/static에서 hello-static.html이 있는지 찾아봄
4. 있으면 웹브라우저에 그대로 hello-static.html을 반환해줌

=> 스프링 컨테이너가 톰캣 서버에게 주소를 받았을 때,

     1. 해당 컨트롤러가 있는지 찾아봄
        2. 없으면, resources/static에서 해당 이름의 html파일(정적)이 있는지 찾아봄
        
        ->따라서 "/"주소를 받았을 때도 1.컨트롤러에서 먼저 찾고 2.없으면 index.html을 찾음(index.html은 welcome page.즉 "/"주소를 받았을 때 찾는 기본 페이지임)
        ->
         @GetMapping("/")
        //localhost 8080으로 들어올 때 호출됨
        //따로 매핑 안해놨으면 원래 index.html파일과 연결됨
        //근데 스프링이 톰캣 서버로부터 주소를 받았을 때
        //1. 컨트롤러 안에서 찾아봄(->@GetMapping("/") 매핑된게 있음) 2.없으면 static파일(정적 html)에서 찾아봄
        //순서로 찾기 때문에 index.html파일 말고 home.html파일을 먼저 찾아서 연결된 것
        public String home() {
            return "home";
        }



mvc = model, view, controller

(예전에는 컨트롤러, 뷰가 분리되어있지 않고 뷰에 다 때려넣었음.)

프로그래밍 시에는 각 부분을 쪼개야 한다(특히 컨트롤러, 뷰를 쪼개야 함)

view: 화면을 그리는데에 집중

controller: 비즈니스 로직, 내부적 처리에 집중

thymeleaf 템플릿의 장점 : html파일을 서버없이 그냥 열어봐도 껍데기를 볼 수 있음(파일 우클릭-copy path-absolute path)후 웹브라우저에 붙여넣기



Ctrl+P : 메소드의 파라미터 정보 볼 수 있음



1. 웹브라우저에 `localhost:8080/hello-mvc`

2. 내장 톰켓 서버가 스프링에 넘겨줌

3. HelloController에 hello-mvc가 매핑 되어있는지 찾아봄.

   있으므로 그 메소드를 실행함! 

    viewResolver에게 `return "hello-template";`를 리턴하고 `model(name:spring)`을 넘겨줌

4. viewResolver가 templates폴더에 hello-template.html(리턴한 string과 똑같이름의 파일)이 있는지 찾아봄.

   있음! thymeleaf 템플릿 엔진에 처리해달라고 넘김

5. 템플릿 엔진이 렌더링하고 '변환'한(${name}의 값이 받아온 값으로 변환됨) html을 웹브라우저에 반환

   cf) 정적컨텐츠는 변환을 하지 않았음



ctrl+shift+엔터 : 코드 완성해줌

`Hello hello = new Hello(`만 치고 단축키 누르면 `);`를 자동으로 완성해줌



getter, setter 단축키 : 변수 클릭 - Alt+Fn+Insert - Getter, Setter클릭

생성자 : 변수 클릭 - Alt+Fn+Insert - constructor클릭



@ResponseBody 사용시(api 방식)

1. 웹브라우저가 localhost:8080/hello-api를 톰캣 서버에 넘겨줌
2. 톰켓 서버가 스프링 컨테이너에 넘겨줌
3. 컨트롤러에 hello-api가 있음. 
4. 근데 @ResponseBody가 붙어있으므로 HttpMessageConverter에 그대로 데이터를 넘김. 데이터가 객체인 경우 jsonConverter가 동작해  json방식으로 데이터를 만들어서 웹브라우저에 보내줌, string인 경우 stringConverter가 동작해서 웹브라우저에 보내줌. 



웹 어플리케이션 계층 구조

1. 컨트롤러 : 앞에서 한 것 같은 웹 MVC의 컨트롤러 역할
2. 서비스 : 도메인 객체를 가지고 핵심 비즈니스 로직이 동작하도록 구현한 객체 ex) 중복 가입이 안된다.
3. 도메인 : 회원, 주문, 쿠폰 등 db에 저장되고 관리되는 비즈니스 도메인 객체
4. 리포지토리 :  db에 접근하고, 도메인 객체를 db에  저장하고 관리



타입?에 마우스 두고 alt+엔터 : 필요한 import 보여줌(선택 가능)

인터페이스에 마우스 두고 alt+엔터 : 그 인터페이스의 메소드 코드 불러올 수 있음



테스트 코드 작성(매우 중요!!)

개발한 기능을 테스트할 때, 다른 여러 방법들은 반복실행하기 어렵고 여러 테스트를 한번에 실행하기가 어렵다. 따라서 자바는 JUnit이라는 프레임워크로 테스트코드를 만들어서 테스트를 실행함.

test>java>hello.hellospring>repository>MemoryMemberRepositoryTest에 테스트 코드 작성

특정 테스트 메소드만 돌려보려면 메소드 옆에 재생버튼 누르면 됨

특정 테스트 클래스만 돌려보려면 클래스 옆에 재생버튼 누르면 됨

전체 테스트 코드를 다 돌려보려면 test>java>hello.hellospring를 run하면 됨

(어플리케이션파일 실행해서 서버 킨 후에, 테스트 코드 실행해야 함)

**주의점**

테스트 코드 여러개를 한번에 돌릴 때 각 테스트 코드의 실행 순서는 제멋대로임. 그래서 이전 테스트 메소드에서 데이터를 넣었다면 그 다음 테스트 메소드에도 영향을 받아 이상하게 돌아갈 수 있음.

따라서 각 테스트 메소드를 실행할 때마다 데이터를 clear 해줘야 함.

```
@AfterEach
public void afterEach() {
  repository.clearStore(); //clearStore는 레포지토리 파일에 정의
}
```





어떤 코드들에서 변수 이름만 한번에 바꾸고 싶을 때

변수이름 드래그 - Shift+F6 - 이름 바꾸기 - 엔터

 

드래그한 코드를 메소드로 추출(extract method)

코드 드래그 - Ctrl+Alt+M - 메소드 이름 정하기 - 엔터



Service 패키지의 메소드들은 비즈니스(업무)에 가까운 용어로 이름지어야 좋음. 반면 repository는 좀더 개발스럽게 이름 지음.



편하게 테스트코드 생성하기

테스트하고자 하는 클래스(클래스 별로 테스트클래스(파일)짜는듯) 이름 누름 - Ctrl+Shift+T - JUnit5선택, 테스트하려는 메소드들 선택 - 자동으로 테스트코드 틀이 짜짐(테스트 코드파일의 위치는 테스트하려는 파일의 층과 같은 층에)

테스트코드 메소드 이름은 한글로 써도 됨(직관적)

테스트 코드 짤 땐 given(이런 상황이 주어져서-테스트 데이터), when(이걸 실행했을 때-테스트할 메소드 실행), then(이런 결과가 나온다-잘 실행됐는지 결과 확인) 주석 활용



Assertions에 alt+엔터 -> static import 선택 -> Assertions.assertThat => assertThat로 간단하게 작성 가능



ctrl + alt+ V : ?????뭔가 코드 짜잔 완성해주는데 뭐지.

주로 커맨드 = ctrl, 옵션 = alt



shift+F10 : 이전에 실행했던 걸 다시 실행



컨트롤러(HelloController, MemberController...)는 스프링 컨데이너가 관리함. 스프링 컨테이너가 뜰때(?)(작동할때?) 컨트롤러 객체를 생성함(즉 컨트롤러의 생성자가 호출됨)



**컴포넌트 스캔과 자동의존관계 설정 방식**(스프링이 자동으로 등록함)

서비스, 레포지토리, 컨트롤러 파일의 클래스 앞에 @Service, @Repository, @Controller 라고 적어줘야 스프링 컨테이너가 각 파일들을 인식할 수 있음(안그러면 스프링이 아닌 그냥 자바 파일일 뿐)

컨트롤러, 서비스를 연결시킬땐 컨트롤러파일의 생성자 앞에 @Autowired 붙이고 서비스를 인자로 받아옴

이 @Service, @Repository, @Controller는 @Component키워드를 포함함. 스프링이 올라올때 @Component가 붙어있는 애들은 스프링 객체를 하나씩(=싱글톤으로 등록. 유일하게 하나만 등록. 따라서 같은 스프링빈이면 모두 같은 인스턴스임. *설정으로 싱글톤이 아니게 할 수 있지만 거의 안함) 생성해 스프링 컨테이너에 등록함.(스프링 빈으로 등록) @Autowired는 이 객체들을 서로 연관시켜 서로를 쓸 수 있도록 해줌(이게 자동의존관계설정)(스프링 빈으로 등록된 객체들만 Autowired가 동작함)

정형화된 컨트롤러, 서비스 리포지토리 같은 코드는 이 방식을 사용.



**자바코드로 직접 스프링 빈 등록하는 방식**

@Controller는 그대로 두고 SpringConfig파일을 등록해 직접 스프링 빈을 등록함

정형화 되지 않거나 상황에 따라 **구현 클래스를 변경해야 하면** 이 방식을 사용. 구현클래스를 변경해야 하는 경우는, 비즈니스 요구사항이 바뀔 때(현재 수업 예시상황에서 데이터저장소는 뭐쓸지 아직 정하지 않은 상태. 따라서 만들어둔 MemoryMemberRepository를 나중에 수정해서 실제 데이터 저장소에 연결하는 리포지토리로 바꿔야 함. 근데 이 방식을 사용하면 기존 코드 수정 없이 SpringConfig 파일 만 바꾸면 쉽게 바꿀 수 있음)

```java
//SpringConfig.java 파일
@Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
//나중에 데이터저장소를 바꿔서 리포지토리를 바꿔야 하면,
    //return new 새로운Repository();
    //로 바꾸기만 하면 됨.
```



데이터 조회(페이지 조회) : GetMapping

데이터 등록(post 방식 통해-input 태그) : PostMapping

메모리 안에 두는 데이터는 서버를 내려버리면 사라짐. 따라서 데이터를 데이터베이스에 저장해야 함!

- h2 database : 교육용으로 쓰기 좋은 db(로컬에서 자주 사용)

- 순수 jdbc : db와 애플리케이션 서버를 연결할 때 필요한 기술(예전 개발자들 방식)
- 스프링 jdbcTemplate : 스프링이 제공하는 jdbc 템플릿. 애플리케이션에서 db로 sql을 편리하게 날릴 수 있음
- jpa : sql문을 직접 짜는게 아니라, jpa 기술이 sql문을 다 만들어줌. 쿼리문 없이 객체를 바로 db에 저장할 수 있음
- 스프링 데이터 jpa : jpa를 편리하게 쓸 수 있도록 한 번 감싼 기술



h2 데이터베이스

JDBC URL : db파일이 생성되는 경로

- ~/test : 사용자/wus2363/test.mv.db

- 생성한 후에는 `jdbc:h2:tcp://localhost/~/test` 로 접속

- sql문 실행 : ctrl + 엔터

- ```sql
  drop table if exists member CASCADE;
  create table member
  (
   id bigint generated by default as identity,
   name varchar(255),
   primary key (id)
  );
  ```

  - `bigint` : long 타입
  - `generated by default as identity` : 값을 세팅하지 않고 insert하면 db가 값을 자동으로 채워줌

- hello-spring(프로젝트 폴더)/sql 폴더 생성 후 그 안에 sql파일 만들어서 sql문 관리하면 좋음(걍 썼던 쿼리문들 적어두는 듯)



build.gradle에서 코끼리 누르면 자동으로 필요한 라이브러리 import해줌(?)

build.gradle 수정하면 코끼리 눌러줘야 함

spring의 장점 : 객체지향 설계의 좋은 점은 다형성을 활용하는 것(인터페이스를 두고 구현체를 바꿔끼기 할 수 있음. 스프링 컨테이너가 이걸 편리하게 지원해줌) 별다른 코드 수정 없이 애플리케이션을 조립하는 코드만 손대면 된다(SpringConfig 파일)!

개방폐쇄원칙(OCP) : 확장(기능 추가..)에는 열려있고 수정(변경)에는 닫혀있다. 

-  기존 코드를 전혀 손대지 않고, 조립코드 변경(설정)만으로 구현 클래스를 변경할 수 있다.

데이터를 db에 저장하면 서버를 껐다 켜도 데이터가 유지된다.



```java
//resources/application.properties 파일에 추가
//스프링과 h2 db를 연결할 때 필요
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```



```
@Transactional //테스트 코드에 넣으면, 테스트가 끝나면 rollback해줘서 디비 데이터를 원상복구시킴
//원상복구 안시키면 테스트 데이터가 들어가있어서 문제가 될 수 있음
```

디비연결해서 하는 테스트(=통합테스트)(스프링 컨테이너 사용)(MemberServiceIntegrationTest.java)보다 순수 자바코드테스트(=단위테스트)(스프링 컨테이너x)(MemberServiceTest.java)가 실행속도도 빠르고, 더 애용해야 함. 



Jdbc템플릿

순수 jdbc와 똑같이 build.gradel파일에

```java
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
	runtimeOnly 'com.h2database:h2'
```



jpa

jdbc 지우고

```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

```java
//application.properties파일
spring.jpa.show-sql=true //jpa가 날리는 sql을 볼 수 있음
spring.jpa.hibernate.ddl-auto=none //jpa는 객체를 보고 테이블도 자동으로 만들어줌. 근데 예제에서는 이미 테이블 만들어놨기 때문에 테이블 자동생성기능을 꺼줌
```

jpa사용시 터미널창에서 `Hibernate: 실행된sql쿼리` 볼 수 있음



스프링 데이터 jpa

리포지토리에 구현 클래스 없이 인터페이스 만으로 개발 가능! 기본 CRUD기능도 모두 제공. 코드를 확 줄일 수 있음.

현업에서 스프링관련 기술 쓸 때는 스프링데이터jpa, jpa는 필수!!!대신 스프링데이터 jpa는 jpa를 도와주는 것임을 잊지 말기

스프링데이터 jpa가 SpringDataJpaMemberRepository(JpaRepository를 상속받은 인터페이스)를 보고 자동으로 구현체를 만들어서 스프링 빈으로 등록. JpaRepository가 finaAll, findById, save(CRUD 등) 이런 기본 메소드를 제공함. 근데 findByName이런건 너무 공통이 아니라 제공하지 않음. 따라서  SpringDataJpaMemberRepository에 그런 메소드는 선언해줘야 함. 근데 규칙이 있음!

```java
//"select m from Member m where m.name = ?" 로 번역이 됨
Optional<Member> findByName(String name);
Optional<Member> findByNameAndId(String name, Long id);
```



AOP가 필요한 상황

악덕상사가 모든 메소드의 실행시간을 측정하라고 한다면? ㅎㅎ

이런 시간측정 로직은 핵심 관심사항은 아니고, 공통관심사항은 맞다. 그리고 시간측정로직과 핵심비즈니스로직이 섞여서 유지보수가 어렵다(메소드 안에 일일이 시간측정코드를 넣어야 함), 시간측정로직을 별도의 공통로직으로 만들기 어렵다, 나중에 시간측정로직을 변경하려면( ex) ms단위가 아니라 초단위로 측정하라는 지시가 떨어지면 ) 모든 코드를 일일이 수정해야 한다.

이런 문제를 해결하는 기술 : AOP(Aspect Oriented Programming - 관점지향프로그래밍) : 공통관심사항과 핵심관심사항을 분리하는 것

시간측정로직을 따로 두고 원하는 곳에 공통관심사항을 적용!

AOP적용전 의존관계

- helloController가 memberService를 호출함

AOP적용후 의존관계

- helloController가 프록시 memberService(가짜)를 호출함. 이 가짜 스프링빈이 실행되다가 joinPoint.proceed()가 끝나면 프록시가 진짜 실제 memberService를 호출함.

AOP는 프록시를 사용하는? 기술인듯 - 프록시 방식의 AOP



메인메소드 만드는 단축키 : psvm + 엔터 -> `public static void main(String[] args) {  }`

빌드해서 코드 배포할때 test코드는 빠짐

Assertions.assertThat 사용할 땐 assertj.core 어쩌구 선택

iter + 엔터 : 배열의 데이터 하나하나에 접근하는 for문 자동으로 만듦

soutv + 엔터 : `System.out.println("필드명 = " + 필드명 );` 자동으로 만듦. 필드는 이 문장이 들어가있는 메소드의 매개변수인듯?

ctrl + alt + V : 메소드의 반환타입, 값을 자동으로 =로 받아줌

- ```java
  Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
  //ac.getBeansOfType(MemberRepository.class);를 쓰고 단축키 누르면
  //자동으로 Map<String, MemberRepository> beansOfType 가 생김
  //반환된 변수의 이름 지정 가능
  ```

ctrl + N : 모든 클래스를 검색할 수 있음(스프링에서 제공하는 클래스도 가능)