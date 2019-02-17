---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍- 01 JPA 소개"
date: 2019-01-26
categories:
---
기존에 개발하던 결제시스템에 신규 프로젝트가 띄어지지 않아, 사내 클라우드들을 통합관리하는 시스템의 포탈 고도화 프로젝트에 4개월간 투입된 상태이다. 그동안 독학과 학원에서 학습시에 자바 어플리케이션과 관계형 데이터베이스간의 통신에 사용했던 기술은 JDBC와 MyBatis 였고, 작년 부터 회사에서 개발을 시작한 뒤로 항상 MyBatis만 써왔다. JPA라는 기술은 면접 준비를 통해 하나의 또 다른 어플리케이션과 데이터베이스를 연관시키는 기술이라는 것만 어렴풋이 알았었고, ORM이라는 의미는 제대로 파악하지 못했다.

그러다 이번 프로젝트에서, front-end 개발을 맡게 되었기에 해당 시스템의 front-end 기술 스택인 AngularJS 공부가 시급하다고 생각되었으나, 역시 서버개발자가 적성에 맞는 나는 슬금슬금 서버쪽 소스를 보기 시작하면서 JPA를 이해할 필요성을 느끼게 되었다.

일단 국내에 출시된 서적 중에ORM 및 JPA 관련 책 중 Blind에서 중 가장 평이 좋은 아래의 책을 선택했다. 

![](http://www.acornpub.co.kr/image/book/hu/hu/1436250312NtvfUYe9.jpg)

그리고 4장까지 공부한 지금은 일단은 급하게 기존 소스들을 분석하는데에 문제가 없고 제법 활용까지 할 수 있게 되었다.

---
# 자바 ORM 표준 JPA 프로그래밍

## 1 JPA 소개

### 1.1 SQL을 직접 다룰 때 발생하는 문제점

관계형 데이터베이스(RDB)는 대중성과 신뢰성으로 인해 엔터프라이즈 시스템 개발에서 가장 많이 사용하는 데이터 저장소 중 하나이다. 또한 자바 어플리케이션은 일반적으로 이러한 RDB를 사용하며, SQL 전달을 위해 JDBC API를 사용한다. 이 때문에 대부분의 자바개발자들은 SQL을 능숙하게 다루는 것이 중요 덕목이었다.

#### 1.1.1 반복, 반복 그리고 반복

예를 들어 `MEMBER`라는 회원정보를 관리하는 RDB 테이블이 있다고 가정할때 이를 관리하는 기능을 가진 어플리케이션은 아마도 아래와 같을 것이다.

##### 먼저 `Member` 객체를 만든다

```java
public class Member {

    private String memberId;
    private String name;

    public String getMemberId() {
        return memberId;
    }

    public void setMemberId(String memberId) {
        this.memberId = memberId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

##### `Member` 객체를 데이터베이스에 관리할 목적으로 `DAO`객체를 만든다

```java
public class MemberDAO {

    public Member find(String memberId){...}
}
```

`find()`메소드의 로직 완성을 위해서 개발자는 일반적으로 아래의 순서로 코딩할 것이다.

1. 회원 조회용 SQL을 작성한다.
```sql
SELECT MEMBER_ID, NAME FROM MEMBER WHERE MEMBER_ID = ?
```
2. JDBC API를 사용해서 SQL을 실행한다.
```java
ResultSet rs = stmt.executeQuery(sql);
```
3. 조회 결과를 Member 객체로 매핑 한다.
```java
String member_id = rs.getString("MEMBER_ID");
String name = rs.getString("NAME");

Member member = new Member();
member.setMemberId(member_id);
member.setName(name);
```

만약 회원 등록을 위한 `save()`를 DAO에 추가한다면 그 개발 순서도 이와 비슷하다

1. 회원 등록용 SQL을 작성한다
```sql
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES(?,?)
```
2. 회원 객체가 가진 값을 불러와 SQL에 전달한다.
```java
pstmt.setString(1, member.getMember();
pstmt.setString(2, member.getName());
```
3. JDBC API를 이용해서 SQL을 실행한다.
```java
pstmt.executeUpdate(sql);
```

회원 삭제 수정 기능도 이와같이 SQL작성과 JDBC API를 호출하는 과정을 비슷하게 반복해야만 한다. 데이터베이스가 아닌 회원정보를 자바의 컬렉션에 추가한다면 아래와 같이 한줄로 객체를 저장할 수 있다.

```java
list.add(member);	
```

RDB는 이러한 자바의 객체 구조와 다른 데이터구조를 가지고 있기에, 이를 중간에서 매개하는 JDBC API와 SQL을 통해서 변환 작업이 필요하다. 문제는 이러한 CRUD 작업에 필요한 코드가 너무나도 많이 반복된다는 것이다.

#### 1.1.2 SQL에 의존적인 개발

위에서 만든 회원관리에 새로운 요구사항이 생긴다면 어떻게 될까. 예를들어 회원의 연락처가 추가되어야 한다는 요구사항말이다. 일단 `MEMBER`테이블에 `TEL`이라는 컬럼이 추가되고 이를 위한 자바 코드 수정이 필요하다

1. `Member` 객체에 연락처 필드가 추가되어야 한다.
2. INSERT문에 연락처 값을 전달하기 위한 수정이 필요하다.
3. 위의 INSERT문에 담을 연락처 필드 값을 가져오는 `get`메소드도 추가되어야 한다.
4. 회원 조회를 위한 SELECT 문도 변경되어야 한다.
5. SELECT문의 결과를 자바 객체에 답는 `set`메소드도 추가되어야 한다.
6. 수정을 위한 UPDATE문 등, 만약에 조건에 따른 다른 쿼리문들이 있다면 모두 위의 작업이 필요하다.

회원 객체가 RDB가 아닌 앞서 언급한 자바의 컬렉션에 보관 되었다면 몇줄의 코드만으로 관리가 되었을 것이다.

```java
list.add(member); //등록
Member member = list.get(xxx); //조회
member.setTel("xxx");
```

##### 연관된 객체

이번엔 다른 요구사항이 추가된다, 모든 회원은 반드시 한 팀에 소속되어야 한다는 요구사항이다. 이를 위해 팀 관련 기능을 전담하는 개발자가 개발한 코드를 받아보니, `Member`객체에 팀을 관리하는 필드가 추가 되어 있다.

```java
public class Member {

    private String memberId;
    private String name;
    private Team team;
    ...
}
```

팀 정보를 출력하기 위해서 `Member`가 제공하는 `getTeam()`을 사용해서 팀이름을 출력하려고 해보니, `null`값이 출력되는데 데이터베이스에는 모든 회원정보에 팀정보가 들어가 있다. 결국 클라이언트 코드와 RDB간의 모든 레이어의 코드를 검토하고 나서야 다른 개발자가 `find()`메소드는 건드리지 않고, `findWithTeam()` 메소드를 추가해놓은 것을 발견했다. 회원정보를 팀정보와 조회하기 위해서는 아래와 같이 SQL JOIN이 필요했기 때문이다.

```sql
SELECT M.MEMBER_ID, M.NAME, M.TEL, T.TEAM_ID, T.TEAM_NAME
FROM MEMBER M
JOIN TEAM T
	ON M.TEAM_ID = TEAM.ID
```

결국 해당 메서드를 변경하여 문제를 해결하게 된다.

##### 문제점 정리

* Member 객체와 연관된 Team 객체의 사용은 DAO가 가진 SQL에 종속되어있다.
* 비즈니스 요구사항을 모델링한 객체인 엔티티만 신뢰할 수 없다
* DAO를 통해 JDBC API와 SQL을 분리해 놓았다 할지라도, 결국 엔티티와 강한 의존관계가 형성된다.
* **즉 SQL에 의존적인 개발을 피하기가 어렵다**

#### 1.1.3 JPA와 문제 해결

JPA를 사용하면 개발자가 객체를 데이터베이스에 저장하고 관리할 때 스스로 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하고, JPA가 SQL을 적절히 생성해서 데이터베이스에 전달한다. 이러한 CRUD API를 앞서 다뤄본 `Member`객체를 예제로 해서 살펴보자.

##### 저장 기능

```java
jpa.persist(member);
```

`persist()`는 객체를 데이터베이스에 저장한다. JPA는 객체와 매핑정보를 보고 INSERT문을 생성해서 데이터베이스에 전달한다.

##### 조회 기능

```java
Member member = jpa.find(Member.class, memberId);
```

`find()`는 객체 하나를 데이터베이스에 조회하여 그 결과로 `Member` 객체를 생성해서 반환한다.

##### 수정 기능

```java
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경") //수정
```

JPA는 별도의 수정 메소드를 제공하지 않고, 객체를 조회한 후 값을 변경만 하면 트랜잭션 커밋 할 때 데이터베이스 적절한 UPDATE SQL이 전달된다.

##### 연관된 객체 조회

```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam(); //연관된 객체 조회
```

JPA를 사용하면, 이미 연관된 객체를 조회하는 시점에 적절한 SELECT SQL이 실행된다. 따라서 `TEAM`테이블에 매핑된 정보를 위해서 SQL수정을 할필요 없이, `member`가 제공하는 메소드만 사용해서 연관된 `TEAM`정보를 받아볼 수 있다.

### 1.2 패러다임의 불일치

자바와 같은 객체지향 프로그래밍언어는 추상화, 캡슐화, 정보은닉, 상속, 다형성 등을 통해 엔터프라이즈 시스템의 복잡성을 제어할 수 있게 해준다. 또한 비즈니스 요구사항을 정의한 도메인 모델을 객체로 모델리하여 객체지향 언어가 가진 장점을 활용할 수 있게도 해준다. 문제는 이러한 객체의 영구보관에 **다른 패러다임을 가진** 관계형 데이터베이스를 사용한다는 것이다. 

서로 다른 목적을 가진 자바 애플리케이션과 관계형 데이터베이스 사이의 패러다임 불일치를 해결하는 데에 지금까지 많은 시간과 코드를 소비해왔다. 대표적인 패러다임의 불일치를 살펴보자.

#### 1.2.1 상속

아래와 같은 상속구조가 있다고 해보자.

```java
abstract class Item {
    Long id;
    String name;
    int price;
}

class Album extends Item{
    String artist;
}

class Movie extends Item{
    String director;
    String actor;
}

class Book extends Item{
    String author;
    String isbn;
}
```

자 이러한 상속관계를 관계형 데이터베이스에서 표현하기 위해서는 `Item` 추상 클래스를 테이블화 한 `ITEM`테이블에 어떤 자식 테이블과 관계가 있는 지를 정의하는 컬럼을 정의해야 한다. 또한 이러한 데이터베이스 모델링을 한 경우에는, 구현 객체를 저장하게 될 때에는 다음과 같이 두 테이블에 INSERT문을 만들어야 한다.

```sql
INSERT INTO ITEM ...
INSERT INTO ALBUM ...
```

조회의 경우에도 `ITEM` 테이블과 `ALBUM`테이블을 조인해서 조회한다음에 그 결과로 `Album` 객체를 생성해야 한다. 데이터베이스가 아닌 자바 컬렉션에 보관한다면 타입에 대한 고민없이 할 수 있을 것이다.

```java
list.add(album);
list.add(movie);

Album album = list.get(albumId);
```

##### JPA와 상속

객체지향 프로그램의 상속과 관계형데이터베이스가 가진 패러다임의 불일치를 JPA에선 컬렉션에 객체를 저장하듯이 저장하고, 조회하면 된다.

```java
jpa.persist(album); // 저장
String albumId = "id100";
Album album = jpa.find(Album.class, albumId); //조회
```

위의 코드가 실행되면 JPA는 다음과 같이 SQL을 실행하여, 실제 개발자가 작성해야 하는 무의미하고 반복적인 쿼리문의 양을 줄여준다.

```sql
--저장
INSERT INTO ITEM...
INSERT INTO ALBUM...

--조회
SELECT I.*, A.*
	FROM ITEM I
	JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID;
```

#### 1.2.2 연관관계

개발 코드에서는 필드안에 객체를 생성하는 참조를 사용해서 객체간의 연관간계를 가지게한다. 테이블의 경우 외래 키라는 것을 사용해서 테이블간의 연관간계를 가지게하고, 조인을 사용해서 이를 조회한다.

##### 필드에 객체 참조를 보관해, 관계 설정

```java
public class Member {

    private Team team;

    public Team getTeam() {
        return team;
    }
}
class Team{
    ...
}
```
##### 관계에 대한 접근
```java
member.getTeam();
```

##### 외래 키 컬럼이 설정된 테이블간 조인을 통한 관계 조회

```sql
SELECT M.*, T.*
	FROM MEMBER M
	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

다른점이 있다면, 개발 코드상에서는 참조가 있는 방향, 즉 참조하려는 객체를 가지고 있는 모델에서만 조회가 가능하지만, 테이블에서는 외래 키 하나로 양쪽 정보에 대한 조회가 가능하다.

##### 객체를 테이블에 맞추어 모델링

테이블의 외래키를 통한 관계설정을 객체에 맞추기 위해서 모델링을 하면 다음과 같다.

```java
public class Member {

    private String memberId;
    private String name;
    Long teamId; //TEAM_ID FK 컬럼 사용

}

class Team{
    
    Long teamId; //TEAM_ID PK 컬럼 사용
    String name;
}
```

이렇게 데이터가 저장될 테이블에 맞추어 객체를 모델링하면 객체 자체의 저장과 조회에는 분명히 편리할 수 있다. 하지만, `Member` 클래스의 `teamId` 필드가 가진 문제가 있다. 객체의 경우에는 참조를 통해서 연관객체를 찾을 수 있는데, 이 경우에는 코드에서 객체간의 관계를 조회할 수 없다는 것이다. 결국 참조를 통해서 객체를 조회하지 못한다면 객체지향의 특징을 잃어버리게 되는 것이다.

##### 객체지향 모델링

```java
public class Member {

    private String memberId;
    private String name;
    private Team team; // 참조로 연관관계를 설정한다.

    public Team getTeam() {
        return team;
    }
}

class Team{
    
    Long teamId;
    String name;
}
```

위와 같은 객체지향 모델링은, 데이터베이스의 패러다임인 외래키를 보관하는 것이 아니라, `Team` 객체 자체의 참조를 보관해, 연관된 팀을 조회할 수 있다. 하지만 이 경우에는 앞서 살펴본 것과 반대로 객체를 테이블에 저장하고 조회하기 위해 개발자의 변환 역할이 필요하다

###### 저장

`team`필드가 참조하는 객체의 `teamId`를 찾아서 이를 외래키로 변환해야 한다.

```java
member.getMemberId();
member.getName();
member.getTeam().getTeamId(); //MEMBER.TEAM_ID FK 로 저장
```

###### 조회

일단 조회를 위해서는 외래키를 사용한 조인 SQL을 통해 얻은 결과를 통해, 이를 개발자가 직접 연관관계를 설정해주어야 한다.

```java
//SQL 실행 후 

//데이터베이스에서 조회한 회원 정보를 setter로 입력
Member member = new Member();
...

//데이터베이스에서 조회한 팀 정보를 setter로 입력
Team team = new Team();
...

//마지막으로 관계 설정 후 리턴
member.setTeam(team);
return member;
```

결국 객체지향 프로그래밍을 따르는 코드도 그역할을 수행할 수 있도록 객체간의 참조로 관계를 설정하고, 테이블은 이를 외래키를 사용한 관계로 한다는 것은 이러한 개발의 오버헤드를 필수로 요구한다.

##### JPA와 연관관계

JPA는 연관관계와 관련된 패러다임의 불일치와 오버헤드를 마치 객체를 컬렉션에 저장하는 것처럼 처리해준다. 개발자는 코드를 통해 객체간의 관계를 설정하고 저장하면, JPA가 참조를 외래키로 바꾸어 INSERT SQL을 데이터베이스에 전달한다. 조회를 할 때도 마찬가지이다.

```java
// 연관관계 설정 후 저장
member.setTeam(team);
jpa.persist(member);

//데이터 조회 후 연관관계 객체 조회
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```

지금까지 설명한 패러다임의 문제들은 개발자의 시간과 노력을 소모하면 해결가능한 패러다임이었다. 이후에 나오는 패러다임의 불일치는 실제 극복하기 어려운 것들이다.

#### 1.2.3 객체 그래프 탐색

아래 코드와 같이 지금까지 객체간의 관계를 조회할 때 사용했던 것을 **객체 그래프 탐색**이라 한다.

```java
Team team = member.getTeam();
```

이를 확장하여 아래와 같이 연관관계들이 설계되어 있다고 가정해보자

{% mermaid %}
graph LR
	Member --- Team
	Member --- Order
	Order --- OrderItem
	Order --- Delivery
	OrderItem --- Item
	Item --- Category
{% endmermaid %}

이런 객체 연관관계가 존재한다면 다음과 같은 객체 그래프 탐색이 이루어질 수 있어야 한다.

```java
member.getOrder().getOrderItem().getItem().getCategory();
```

하지만 데이터베이스에 객체들을 저장하고 외래키로 관계 설정을 해놓은 경우에는 만약 위의 그래프 탐색과 같은 결과를 얻기 위해선, 모든 관계가 설정된 테이블에 대한 조인 SQL이 이뤄져야 한다. 즉,

> SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.

이러한 제약은 객체지향 프로그래밍을 추구하는 개발자에겐 큰 벽이된다. 수행된 쿼리에 따라 탐색의 제한을 가지는 객체를 가지고, 다양한 비즈니스 로직에 따라 탐색의 depth가 달라지는 개발을 수행할 수 없기 때문이다.

```java
public class MemberService {
    
    MemberDAO memberDAO = new MemberDAO();
    
    public void process(){
        
        Member member = memberDAO.find(memberId);
        
        member.getTeam();
        member.getOrder().getDelievery();
    }
}
```

위와 같은 `MemberService`에서 `memberDAO`를 통해 조회해서 생성한 객체는 현재 코드만 가지고는 이 객체와 연관된 `Team`, `Order`, `Delivery`에 대한 탐색이 정상적으로 수행될지, 아니면  `null`을 리턴할지는 전혀 예측할 수가 없다. 결국 객체지향 원칙에 따라 역할을 분리하고, 데이터베이스의 접근을 추상화하고 은닉한 결과인 DAO의 코드를 직접 확인해야만 한다. 엔티티가 SQL에 논리적으로 종속되었기 때문에 발생하는 문제다.

그렇다고 member와 연관된 모든 객체 그래프를 데이터베이스에서 조회할 수 없기 때문에, 다양한 비즈니스 로직의 필요에 따라 섬세하게 설계된 SQL들을 만들고, 이를 사용하는 여러벌의 조회 메서드를 DAO에 포함시켜야 한다.

##### JPA와 객체 그래프 탐색

> JPA를 사용하면 객체 그래프를 마음껏 탐색할 수 있다.

앞서서 계속 객체지향을 따르는 엔티티 모델링과 관계형 데이터베이스간의 매핑의 불일치를 JPA가 해결해주는 방법으로 객체를 사용하는 시점에 JPA가 적절한 SQL생성 및 실행이었다. 이러한 JPA 특성은 실제 객체를 사용할 때까지 데이터베이스 조회를 미루는 **지연 로딩** 기능을 통해 객체를 신뢰하고 마음 껏 조회할 수 있게 해준다.

```java
//처음 조회 시점에 SELECT MEMBER SQL
Member meber = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); //Order 객체가 사용되는 시점에 SELECT ORDER SQL
```

만약 `Member` 객체에 대한 조회가 반드시 `Order`객체와 함께 이루어져야 한다면, SQL의 조인을 사용하는 것이 좋다. JPA는 이런 설정을 제공하기 때문에, 아래와 같은 SQL도 JPA설정을 통해 실행 시킬 수 있다.

```sql
SELECT M.*, O.*
	FROM MEMBER M
	JOIN ORDER O ON M.MEMBER_ID = O.MEMBER_ID
```

#### 1.2.4 비교

데이터베이스는 유니크한 기본 키의 값으로 각 row를 구별짓는다. 반면 동일한 타입을 구현한 객체의 비교는 두 가지인데, **동일성(identity)** 와 **동등성(equality)**로 나뉘어 진다.

* **동일성**: 객체 인스턴스의 주소 값 비교이다, 즉 자바에서 비교연산자(`==`)결과가 `true`이면 동일하다.
* **동등성**: 객체가 제공하는 `eqauls()`메소드를 통해서 객체 내부의 값이 같은지를 비교한다.

따라서 테이블 row를 구분하는 방법과 객체를 구분하는 방법은 차이점이 있다. 예를 들어 동일한 조회를 두 번 실행한 경우, SQL결과로 새로운 객체를 두 번 생성(`new`)하기 때문에, 이 두 객체는 **동일하지 않다.** 만약 객체를 컬렉션에 보관 했다면, 항상 컬렉션에서 `get()`을 사용해서 동일한 객체를 가지고 오기 때문에 **동일성**비교가 성공했을 것이다.

데이터베이스에서 동일한 row에 대한 조회 결과로 항상 동일한 객체, 즉, **Singleton** 객체를 반환하도록 하는 것은 코드로 구현하기 어려우며, 트랜잭션과 동시성을 고려하면 더욱 어려워 진다.

#### JPA와 비교

> JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; // true
```

#### 1.2.5 정리

JPA에 대한 소개를 위해서 객체 모델과 관계형 데이터베이스 모델의 지향점이 다르고, 이를 JPA가 해결해 줄 수 있다는 것을 살펴보았다. 가장 큰 문제는 객체지향 프로그래밍적인 객체 모델링을 하면 할수록 이러한 패러다임의 불일치가 커지고 개발 오버헤드가 커진다는 점이다. 결국 객체 모델링은 힘을 잃고 점점 데이터 중심의 모델로 변해간다.

실제 기업에서 중요하게 생각하는 것은 어떻게 영속적으로 데이터를 보관하냐이지, 얼마나 객체지향적으로 시스템이 설계되었느냐가 아니기 때문이다. 내가 처음 개발을 시작하게 된 시스템의 경우에도 모든 데이터에 대한 관리를 위해 `Box`라는 `HashMap`을 상속한 Key, Value 중심의 객체를 사용하였고, 이를 위해 항상 복잡한 SQL작성이 요구 되었다. 특히 사업부서의 요구나 비즈니스의 확장을 통해, 데이터간의 연관관계가 변경되거나 추가되면, 어김 없이 비즈니스 로직의 구현보다 더 많은 SQL 수정과 작성이 필요했고 이것이 유지보수의 주 업무였다.

JPA를 통해 이런 패러다임 불일치를 어떻게 해결하는지 자세하게 살펴보고, 정교한 객체 모델링을 유지할 수 있는 방법을 알아보자.

### 1.3 JPA란 무엇인가?

> JPA<sub>Java Persistence Api</sub>는 자바 진영의 ORM 기술 표준이다.

기존에 자바 어플리케이션과 관계형 데이터베이스 사이에서 JDBC API가 SQL전달 역할을 수행하였다면, JPA는 JDBC와 어플리케이션 사이에서 동작한다.

**ORM**은 Object-Relational Mapping의 약자로, 이름 그대로 객체와 관계형 데이터베이스를 매핑한다는 것이다. 앞서서 살펴본 정교한 객체 모델링으로 발생하는 관계형 데이터베이스와의 패러다임의 불일치를, ORM 프레임워크가 해결해 주는 것이다. 마치 컬렉션에 객체를 저장하듯이 ORM 프레임워크가 제공하는 메소드를 사용하면, 이를 JPA와 같은 ORM 프레임워크가 아래의 단계로 처리해준다.

**객체를 저장하는 경우**

1. `DAO`객체에서 Entity 객체를 저장하는 JPA 메소드를 호출한다. (`persist()`)
2. JPA는 다음과 같은 작업을 수행한다
   * Entity 분석
   * INSERT SQL 생성
   * 패러다임 불일치 해결
   * JDBC API 사용
3. JPA가 사용한 JDBC API를 통해 데이터베이스에 INSERT SQL 전달

**객체를 조회하는 경우**

1. `DAO`객체에서 Entity 객체를 조회하는  JPA 메소드를 호출한다. (`find()`)
2. JPA는 다음과 같은 작업을 수행한다
   * SELECT SQL 생성
   * JDBC API 사용
   * JDBC API 사용
3. JPA가 사용한 JDBC API를 통해 데이터베이스에 SQL에 전달하여 반환 받음
4. JPA는 JDBC API를 통해 반환 받은 Result Set을 매핑한다.
5. 매핑된 결과인 Entity 객체가 반환된다.

따라서 ORM 프레임워크를 사용하면, 자바 코드에서는 객체지향의 패러다임에 어울리게 객체 모델링을 할 수 있고, 동일한 내용의 정보를 관계형 데이터베이스에서는 해당 패러다임에 맞는 모델링 할 수 있다. 개발자는 이 둘을 어떻게 매핑하면 될지를 ORM 프레임워크에게 알려주면, 이후에는 관계형 데이터베이스 패러다임은 고려하지 않고 객체지향 프로그래밍을 추구하는 개발을 할 수 있다.

그리고 이러한 ORM프레임워크중 자바진영에서는 하이버네이트가 가장 많이 사용된다.

#### 1.3.1 JPA 소개

> JPA는 자바 ORM 기술에 대한 API 표준 명세다.

이런 표준 API 명세를 실제 구현 한것이 앞서 언급한 하이버네이트이고, 이외에도 EclipseLink, DatakNucleus가 있다. JPA라는 추상화된 표준은 언제나 처럼 특정 구현 기술에 대한 의존도를 감소시키고, 다른 기술로의 전환에 오버헤드를 감소시켜 준다. 이 책에서는 JPA 2.1 버전을 사용한다.

#### 1.3.2 왜 JPA를 사용해야 하는가?

##### 생산성

JPA를 사용하면 반복적인 SQL 작성과 JDBC API에 대한 관리를 개발자가 하지 않아도 된다. 또한 JPA는 DDL문을 자동으로 생성해주는 기능도 있다. 이런 기능들을 사용하면 엔터프라이즈 시스템 구성에서 데이터 중심 설계에서 객체 중심의 설계로 역전할 수도 있다.

##### 유지보수

SQL에 의존적인 개발은, 엔티티의 필드 추가가 곧 JDBC API 코드에 대한 모든 변경을 의미한다. 이는 MyBatis와 같은 **객체의 필드**와 데이터베이스의 컬럼을 매핑해주는 데에서도 일어나는 문제이다. 하지만 JPA는 자바 코드를 바라보고 이를 대신 해석해서 처리해주기 때문에, 필드 변경으로 발생하는 영향도가 상당히 감소한다.

##### 패러다임의 불일치 해결

이번 장에서 게속 살펴본 패러다임의 불일치들, 상속, 연관관계, 객체 그래프 탐색, 비교하기와 같은 문제들을 해결해준다.

##### 성능

기존 JDBC API와 어플리케이션 구조 사이에 새로운 계층이 추가된다는 것은 역설적으로 성능 최적화의 기회를 제공한다. 앞서 객체의 동일성 비교에서 살펴봤듯이, JPA를 통한 동일 데이터베이스 row에 대한 조회는 두 번째부터 이미 조회한 회원 객체를 재사용하기 때문에 성능 최적화에 기여할 수 있다.

이러한 성능 개선을 JDBC API만을 사용해서 수행한다면 이를 처리하기 위한 개발자의 시간과 노력이 필요할 것이고, 실제 그것을 구현한다고 해도 앞서서 언급한 유지보수의 문제의 포인트가 늘어나게 되는 것이다.

##### 데이터 접근 추상화와 벤더 독립성

관계형 데이터베이스를 사용할 때 같은 기능도 벤더에 따라 다르게 제공하는 경우가 많다. 기업의 웹 어플리케이션에서 필수로 사용해야하는 페이징 처리만 하더라도 데이터베이스마다 그 방법을 다르게 제공하고 있어서 이것을 각각 숙지해야 한다. DBA도 서로 다른 벤더의 데이터베이스에 대한 전문화를 갖추기 어려운데, 개발자가 이를 습득하기란 쉽지 않고, 실제로 이렇게 파편화된 사용법을 개발자가 숙지한다고 할지라도, 시스템의 규모가 성장하면 성장할수록 한번 로우레벨 기술에 종속된 프로그램은 변경이 어려워진다.

JPA는 앞서서 기술 표준이라고 살펴보았다. JPA기술 표준을 따르는 어떠한 ORM 프레임워크도 자바로 만들어진 어플리케이션과 연동할 수 있는 것처럼, JPA가 데이터 접근 계층을 추상화해서 애플리케이션에 제공하기 때문에 애플리케이션은 특정 데이타베이스에 종속되지 않게 되는 것이다.

```mermaid
graph LR
	J[JPA]
	D[Dialect]
	MD[MySQLDialect]
	OD[OracleDialect]
	HD[H2Dialect]
	MB(MySQL DB)
	OB(Oracle DB)
	HB(H2 DB)
	J-- 사용 -->D
	D-.-> MD
	D-.-> OD
	D-.-> HD
	MD---|MySQL SQL 생성|MB
	OD---|Oracle SQL 생성|OB
	HD---|H2 SQL 생성|HB

	
```
이를 통해 얻을 수 있는 장점은 너무나도 크다. 예를 들어 개발환경에서는 각자의 개발자가 H2 데이터베이스를 사용하여 개발할 수 있고, 이를 Staging 환경에서는 MySQL을 통해서 관리하고 검증할 수 있다. 최종적으로 상용 서비는 Oracle 데이터베이스를 사용한다고 할지라도, 개발 코드에서는 각 환경에 따라서 JPA가 어떤 데이터베이스를 사용할지 알려주기만 하면 된다.

### 1.4 정리

SQL을 자바 어플리케이션에서 직접 다룰 떄의 어려움을 패러다임의 차이라는 관점에서 살펴보았고, 이를 JPA가 해결할 수 있음을 알아보았다. 그리고 JPA가 어떤 것인지 구체적으로 살펴보기전에 JPA장점과 역할을 살펴보았다.
