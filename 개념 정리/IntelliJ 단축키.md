## IntelliJ 단축키

#### 메소드의 반환타입, 값을 자동으로 =로 받아줌

ctrl + alt + V

- `ac.getBean(NetworkClient.class);` 치고 ctrl + alt + V

  => `NetworkClient client = ac.getBean(NetworkClient.class);` 자동 완성



#### 코드 완성해줌

ctrl + shift + 엔터

 - `Hello hello = new Hello(`만 치고 단축키 누르면 `);`를 자동으로 완성해줌



#### 필요한 클래스 import

클래스(타입)에 마우스 두고 alt+엔터

- 필요한 import 보여줌(선택 가능)



#### 필요한 클래스 생성

클래스(타입)에 마우스 두고 alt+엔터 - create class



#### return 값 간단하게 합치기

ctrl + alt + N

- ```java
  /*
  List<Member> result = em.createQuery("select m from Member m", Member.class).getResultList();
         return result;
  */
  return em.createQuery("select m from Member m", Member.class).getResultList();
  
  
  ```





#### Assertions.assertThat -> assertThat  (static import)

Assertions에 alt+엔터 - static import 선택 - Assertions.assertThat => assertThat로 간단하게 작성 가능

- 주의 : Assertions.assertThat 사용할 땐 assertj.core 어쩌구 선택



#### 배열의 데이터 하나하나에 접근하는 for문 만들기

iter + 엔터



#### `System.out.println("필드명 = " + 필드명 );` 만들기

soutv + 엔터 - 필드 선택

- 필드는 이 문장이 들어가있는 메소드의 매개변수인듯?



#### getter, setter, 생성자 만들기

- getter, setter
  - 변수 클릭 - Alt + Fn + Insert - Getter, Setter클릭

- 생성자
  - 변수 클릭 - Alt + Fn + Insert - Constructor클릭



#### 인터페이스의 메소드 코드 불러오기(implements해서 클래스 작성 시)

추상클래스에 우클릭 - Generate > Override method

추상클래스에 마우스 두고 - Alt + Fn + Insert - Override method

인터페이스는 그냥 alt+엔터만으로 되나?



#### 메소드의 파라미터 정보 볼 수 있음

호출할 메소드 이름 클릭 - Ctrl + P



#### 변수 이름만 한번에 바꾸고 싶을 때

변수이름 드래그 - Shift+F6 - 이름 바꾸기 - 엔터



#### 주석 처리

- 줄 단위 : Ctrl + /
- 블럭 단위 : Ctrl + Shift : /



#### 프로젝트 내에서 검색어 찾기(메소드, 필드, 어노테이션 등 찾기)

 Ctrl + Shift + F



#### 드래그한 코드를 메소드로 추출(extract method)

코드 드래그 - Ctrl + Alt + M - 메소드 이름 정하기 - 엔터



#### 파라미터를 꺼내기

파라미터 클릭 - Ctrl + Alt + P

- ```java
  //이 코드를
  private Item createBook() {
     Item book = new Book();
     book.setName("JPA");
     book.setPrice(10000);
     book.setStockQuantity(10);
     em.persist(book);
     return book;
  }
  ```

- ```java
  //이렇게 바꿔줌
  private Item createBook(String name, int price, int stockQuantity) {
     Item book = new Book();
     book.setName(name);
     book.setPrice(price);
     book.setStockQuantity(stockQuantity);
     em.persist(book);
     return book;
  }
  ```



#### 편하게 테스트코드 생성하기

테스트하고자 하는 클래스 이름 누름(단위테스트는 클래스 별로 짜는듯) - Ctrl + Shift + T - JUnit5선택, 테스트하려는 메소드들 선택 - 자동으로 테스트코드 틀이 짜짐

- 테스트 코드의 위치는 테스트하려는 파일의 층과 같은 깊이



#### 단축키로 편하게 테스트 메소드 만들기

Settings > Lite Templates > custom 패키지에 단축키 추가 > Define 대신 Java 선택

- https://blog.naver.com/nateen7248/222184184776 참고
- `$END$` : 커서가 실행되는 위치
- `$NAME$` : 이름 설정할 수 있음
- 명령어 + tab 키 로 사용

현재 tdd + tab키 누르면 테스트 메소드 생성하도록 해둠



#### 이전에 실행했던 걸 다시 실행

shift+F10



#### 메인메소드 만들기

psvm + 엔터 => `public static void main(String[] args) {  }`



####  모든 클래스를 검색

ctrl + N

- 내가 작성한 클래스, 스프링에서 제공하는 기본 클래스도 가능



#### 클래스 정의 시 toString 작성

Alt + Fn + Insert -> toString() 선택

- Member, Order같은 객체의 필드를 쉽게 출력하려면 클래스에 toString함수를 작성
- 객체를 출력하면 toString이 리턴하는 결과가 출력됨
  - 즉 `System.out.println("order = " + order);` 하면 toString이 호출되어 toString의 리턴값이 출력됨.



#### Setting 창을 열기

Ctrl + Alt + S 