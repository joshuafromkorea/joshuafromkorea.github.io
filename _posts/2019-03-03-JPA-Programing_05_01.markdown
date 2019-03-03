---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍- 05 연관관계 매핑 기초"
date: 2019-03-03
categories:
---

근 한달만에 JPA책을 다시 펼쳤다. 회사일로 너무 바빳고, 현재 front-end로직 고도화만 하고 있기때문에, 서버쪽 로직을 공부할 겨를이 없었다. 그사이에 당면한 문제를 빨리빨리 해결하기 위해서 현재 프로젝트에서 사용하는 front-end 기술스택을 겉핥기 식으로 공부했다. 그래프를 그려주는 `Raphael.js`부터 (설 연휴 내내 공부해서 기존 코드를 리팩토링 했다 ㅠㅠ)`Handlebar.js`, `Require.js`, 그리고 대망의 `Angular.js`까지.. 아무리 SI개발자란 필요한 기술을 빨리 배우고, 남들이 만든 소스를 수정해가면서 고객사의 요구사항을 완료시키는 사람이라고 하지만, 체계없이 당장 필요한 부분만 익히고 복붙으로 개발하는 것은 뭔가 뿌듯함이 덜한 것 같다. 그런의미에서 오늘 아주 퀵하게 살펴본 연관관계 매핑 chapter는, 관계형 데이터베이스가 제공하는 간단한 기능을 객체지향 언어에서 동일하게 구현하기 위핸 선배 개발자들의 고민과 노력을 볼 수 있어서 좋았다.

---

## 5 연관관계 매핑의 기초

5장에서는 관계형 DB에서 사용하는 테이블의 외래 키의 개념과 엔티티간의 참조의 개념을 매핑하는 것을 목표로 한다. 먼저 연관관계 매핑을 다루기 전에 알아야 하는 세 가지 개념에 대해서 정리한다.

1. **방향**(Deriection)
   1. 단방향: A와 B 사이에 관계가 있을 때 A → B 이거나 B → A 한가지의 관계만 있는 것
   2. 양방향: A ↔ B 처럼 양쪽을 서로 참조하는 것
2. **다중성**(Multiplicity): 테이블 row간에 다중 관계에 대한 개념 N:1 , 1:N, 1:1, N:M 으로 나뉨 
3. **연관관계의 주인**(Onwer): 객체를 양방향 연관관계로 만들때에는 **연관관계의 주인을 정해야 한다.**

### 5.1 단방향 연관관계

가장 먼저 다룰 것은 N:1 단방향 관계이다. 만약 **회원**과 **팀**이라는 데이터가 있고, 회원은 하나의 팀에만 소속될 수 있다는 1:N 관계가 있다고 할때, 관계형 데이터베이스의 테이블과 자바의 객체는 이를 표현하는 방식이 다르다.

#### 테이블 연관관계

* 회원 테이블의 필드로 **TEAM_ID**라는 외래키를 둘 것이고 이를 통해 팀테이블과 연관관계를 맺는다
* 하나의 외래키를 사용해서 맺어진 두 테이블은 **양방향 관계**이다 외래키를 통해서 회원과 팀을 조인할 수 있고, 반대의 경우도 조인 가능하기 때문이다.

#### 객체 연관관계

* 회원 객체 안에 필드(멤버변수)로 팀 객체를 사용하여 연관관계를 맺는다.
* **단방향 연관관계**이다. 테이블에서의 관계와 달리 회원 객체의 팀 정보는 알 수 있지만, 팀에서 회원을 접근하는 필드는 엇다.

#### 객체 연관관계와 테이블 연관관계 정리

* 참조를 사용하는 객체의 연관관계는 **단방향**이다.
* 외래키를 사용하는 테이블 연관관계는 **양방향**이다.
	* A JOIN B가 가능하면 B JOIN A도 가능하다.
* 테이블과 유사하게, 팀 객체에도 회원 관련 필드를 추가해서 참조할 수는 있지만 이것은 엄밀히 말해서, **서로 다른 단방향관계 2개**를 만들 것이지 양방향 관계가 아니다.

#### 5.1.1 순수한 객체 연관관계

위에서 설명한 객체 연관관계를 JPA를 배제한 순수 자바 코드로 살펴보자.

```java
public lcass Member{
    private String id;
    private String username;
    
    private Team team; //팀 참조를 보관
    
    public void setTeam(Team team){
        this.team = team;
    }
    //기타 getter & setter
}

public class Team{
    
    private String id;
    private String name;
}
```

위와 같은 두 개의 클래스가 존재할 때, 아래의 예제를 실행 시키면 2개의 Member 인스턴스가 하나의 Team에 소속된다.

```java
public static void main(String[] args){
    
    Member member1 = new Member("001", "1번");
    Member member1 = new Member("002", "2번");
    Team team1 = new Team("t1", "team1");
    
    member1.setTeam(team1);
    member2.setTeam(team2);
}
```

이렇게 맺어진 클래스 관계와 인스턴스 관계는 아래와 같다.

```mermaid
graph LR
	subgraph 인스턴스관계
	m1(member1)-->t1(team1)
	m2(member2)-->t1
	end
	subgraph 클래스관계
	Member-->|0..1|Team
	end
```

또한 마지막으로 아래의 코드처럼 객체의 참조를 사용해서 연관관계를 찾는 것을 **객체 그래프 탐색**이라고 한다.

```java
Team findTeam = member1.getTeam()
```



#### 5.1.2 테이블 연관관계

이번엔 테이블 연관관계를 DDL로 만드는 과정이다.

```SQL
--MEMBER 테이블 생성
CREATE TABLE MEMBER(
	MEMBER_ID VARCHAR(255) NOT NULL,
	TEAM_ID VARCHAR(255),
	USERNAME VARCHAR(255),
	PRIMARY KEY(MEMBER_ID)
)
--TEAM 테이블 생성
CREATE TABLE TEAM(
	TEAM_ID VARCHAR(255) NOT NULL,
    NAME VARACHAR(255),
    PRIMARY KEY (TEAM_ID)
)
--테이블 연관관계 설정
ALTER TABLE MEMBER ADD CONSTRAINT FK_MEMBER_TEAM
	FOREIGN KEY (TEAM_ID)
	REFERENCES TEAM
```

테이블 연관관계를 바탕으로, 회원1과 회원2를 팀1에 소속시키는 DML이다.

```sql
INSERT INTO TEAM(TEAM_ID, NAME) VALUES('team1', '팀1');
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERANME)
VALUES('member1', 'team1', '회원1');
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERANME)
VALUES('member2', 'team1', '회원2');
```

이렇게 생성된 관계는 조인이라는 개념으로 연관관계 탐색을 할 수 있다.

```sql
SELECT T.* 
FROM MEMBER M
	JOIN TEAAM T ON M.TEAM_ID = T.ID
WHERE M.MEMBER_ID = 'member1'
```

#### 5.1.3 객체 관계 매핑

JPA를 사용하면, 객체간의 연관관계와 테이블의 연관관계를 매핑할 수 있다.

```java
@Entity
public class Member{
    
    @Id
    @Column(name="MEMBER_ID")
    private String id;
    
    private String username;
    
    //연관관계 매핑
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    
    //연관관계 설정
    public void setTeam(Team team){
        this.team = team;
    }
    //getter & setters
}
```

순수한 객체와 엔티티에서 두 연관관계를 매핑하는 코드에서 주목할 것은 추가된 두개의 어노테이션이다.

* `@ManyToOne` : N:1의 관계라는 정보다. 회원쪽이 다수가 되고, 팀은 하나가 되니, 이렇게 N:1의 관계가 있음을 나타내는 어노테이션을 **필수**로 사용해야 한다..
* `@JoinColumn(name="TEAM_ID")`: 조인 컬럼은 외래 키를 매핑할 때 사용한다. `name`속성에는 매핑할 외래 키 이름을 지정할 수 있다. 이 어노테이션은 **생략 가능**하다.

#### 5.1.4 `@JoinColumn`

| 속성                   | 기능                                                    | 기본값                                          |
| ---------------------- | ------------------------------------------------------- | ----------------------------------------------- |
| `name`                 | 매핑할 외래 키 이름                                     | 필드명 + "_" + 참조하는 테이블의 기본 키 컬럼명 |
| `referencedColumnName` | 외래 키가 참조하는 대상 테이블의 컬럼명                 | 참조하는 테이블의 기본키 컬럼명                 |
| `foreignKey(DDL)`      | 수동을 제약조건을 직접 지정한다. 테이블 생성시만 사용됨 |                                                 |
| 그 외                  | `@Column`의 속성과 같다.                                |                                                 |

#### 5.1.5 `@manyToOne`

| 속성 |기능| 기본값 |
| ---- | ---- | ---- |
| `optional` | `false`로 설정하면 연관된 엔티티가 항상 있어야 한다. | `false` |
| `fetch` | 글로벌 fetch 전략을 설정한다 (8장 참고) | `@ManyToOne=FetchType.EAGER`<br/>`@OneToMany=FetchType.LAZY` |
| `cascade` | 영속성 전이 기능을 사용한다 (8장 참고) |      |
| `targetEntity` | 연관된 엔티티 타입정보를 사용한다. 이 기능은 거의 사용하지 않는다. |      |

만약 `targetEntity` 속성을 사용할 경우 아래와 같이 써주면 된다.

```java
@OneToMany(targetEntity=Member.class)
private List members;
```

다만 제네릭을 사용하면 해당 속성을 사용하지 않아도 된다.

### 5.2 연관관계 사용

#### 5.2.1 저장

연관관계가 매핑된 엔티티를 저장할 때 주의할 점은, JPA에서 엔티티를 저장할 때에는 연관된 모든 엔티티가 **영속 상태**여야 한다는 점이다.

```java
public void teamSave(){
    
    //팀1 저장
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1);
    em.persist(member1);
    
    Member member2 = new Member("member2", "회원1");
    member2.setTeam(team1);
    em.persist(member2)
}
```

#### 5.2.2 조회

조회는 2가지의 경우가 있다.

* 객체 그래프 탐색(객체의 연관관계 사용한 조회)
* 객체지향 쿼리(JPQL) 사용

##### 객체 그래프 탐색

```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam(); //객체 그래프 탐색
```

##### 객체지향 쿼리 사용

객체는 단방향 연관관계만을 지원하기 때문에 만약에 팀1에 소속된 회원만 조회하려면, 객체 그래프 탐색을 할 수 없다. 앞서 살펴봤듯이 테이블 연관관계는 양방향이기 때문에 SQL을 사용하면 JOIN을 사용해 쉽게 가능하다. JPQL도 조인을 지원하기때문에 이런 경우엔 JPQL을 사용한다.

```java
private static void queryLogicJoin(EntityManager em){
    String jpql = "select m from Member m join m.team t where " +
        "t.name=:teamName";
    
    List<Member> resultList = em.createQuery(jpql, Member.class)
        .setParameter("teamName", "팀1")
        .getResultList();
}
```

SQL과 유사하지만, JPQL은 객체(entity)를 대상으로 하고, 파라미터 바인딩을 받는 문법이 추가로 있다는 점이이있다. 자세한 내용은 10장에서 다룬다.

#### 5.2.3 수정

앞서서 다뤘듯이 JPA에서는 수정을 위한 em.update()와 같은 메소드가 없다. 영속화된 엔티티의 값을 변경하면, 트랜잭션 커밋시에 플러시가 일어남면서, 변경 감지 기능을 통해 데이터베이스에 반영된다.

```java
private void updateRelation(EntityManger em){
    
    Team team2 = new Team("team2","팀2");
    em.persist(team2);
    
    Member member = em.find(Member.class, "member1");
    member.setTeam(team2); //커밋시 변경감지로 반영
}
```

#### 5.2.4 연관관계 제거

간단하게 연관관계 필드에 `null`을 넣어주면 된다.

```java
member.setTeam(null);
```

#### 5.2.5 연관되 엔티티 삭제

만약에 팀1을 삭제하고 싶다면, 해당 엔티티와 연관관계를 맺고있던 관계들을 모두 제거하고, 삭제해야 한다. 그렇지 않으면 외래 키 제약조건으로 인해, 데이터베이스에서 오류가 발생한다.

### 5.3 양방향 연관관계

지금까지는 `Member` 의 필드에 `@ManyToOne` 어노테이션을 사용해서 단방향  다대일 연관관계를 설정하였다. 반대로 `Team`의 필드에 `Member`정보를 담으면, 앞서서 언급한 2개의 단방향 관계로 양방향 연관관계를 만들 수 있다.

단, 팀에서 회원은 일대다의 관계를 갖기 떄문에, 컬렉션을 사용해야 한다.

#### 5.3.1 양방향 연관관계 매핑

`Member`엔티티는 앞서서 살펴본 것에서 바꿀 것이 없다.

```java
@Entity
public class Team{
    @Id
    @Column(name="TEAM_ID")
    private String id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    
    //getter, setters
    
}
```

`@OneToMany`에 사용된 속성인 `mapped` 속성은 양방향으로 관계를 맺을 때 사용하는데, 반대쪽 매핑의 필드 값을 알려주는 것이다. 자세한 내용은 뒤에서 **연관관계의 주인**의 개념과 같이 설명한다. 이제 팀에서도, JPQL을 사용하지 않고 객체 그래프 탐색을 할 수 있다.

```java
Team team  = em.find(Team.class, "team1");
List<Member> members = team.getMembers();
```

이제 `members`에는 팀1에 소속된 모든 회원정보가 들어갈 것이다.

### 5.4 연관관계의 주인

`@ManyToOne`을 뒤집어놓은 `@OneToMany`는 N:1과 1:N의 관계라고 이해하면 편하다. 하지만 `mappedBy`속성이 필요한 이유는 무엇일까?

앞서서 여러번 강조했듯이, 객체는 양방향관계를 가질 수 없다. 서로 다른 2개의 단방향 관계를 어플리케이션 로직으로 묶어서 양방향처럼 보일 수 있게 하는 것이다. 테이블이 **한 개의 외래키**로 해낼 수 있는 것을, 2개의 단방향 매핑을 사용하면, 연관관계를 관리하는 포인트를 두배로 늘리는 꼴이 된다. 또한 실제 데이터가 영속화된 테이블에서는 한개의 외래키를 사용하지만, 객체의 참조는 둘이 되는 차이가 발생한다. 이를 해결하기 위해서 JPA에서는 두 엔티티의 연관관계 중 하나를 **정의해서** 테이블이 하나의 외래키로 관리되는 것과 동일하게 관리하는 데 이를 **연관관계의 주인**이라고 한다.

#### 5.4.1 양방향 매핑의 규칙: 연관관계의 주인

엔티티간의 양방향 연관관계가 생기면, 두 연관관계 중 하나를 연관관계의 주인으로 반드시 정한다. 주인과 주인이 아닌 엔티티의 차이는, **외래키 를 관리**할 수 있다는 점과, 주인이 아닌 쪽은 **읽기만 가능** 하다는 것이다.  이러한 연관관계 주인 설정은 다음을 따른다.

* 주인은 `mappedBy`속성을 사용하지 않는다.
* 주인이 아니면, `mappedBy`속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야 한다.

> 연관관계의 주인을 정한다는 것은 사실 외래 키 관리자를 선택하는 것이다.

#### 5.4.2 연관관계의 주인은 외래키가 있는 곳

데이터베이스 테이블의 N:1관계에서는 항상, N쪽이 외래키를 가지게 된다. 따라서 `@ManyToOne`은 항상 연관관계의 주인이 되므로, `mappedBy`를 속성으로 가지지 않는다. 따라서 앞서 살펴본 예제에서, 연관관계의 주인이 아닌 팀 객체는 연관관계를 설정할 수 없고, 읽기만 가능하다.

### 5.5 양방향 연관관계 저장

계속해서 연관관계의 주인만이 연관관계를 변경할 수 있고, 주인이 아니면 읽기만 가능하다고 강조해왔다. 이점은 양방향 연관관계를 저장하는 아래의 코드를 보면 명확해진다. 

```java
public void teamSave(){
    
    //팀1 저장
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1);
    em.persist(member1);
        
    Member member2 = new Member("member2", "회원2");
    member1.setTeam(team1);
    em.persist(member2);
}
```

**5.2.1**에서 사용한 단방향 연관관계에서 저장하는 코드와 완전히 동일하다. 분명히 `@OneToMany` 어노테이션을 사용해서 새로운 단방향 연관관계가 생겼지만, 해당 연관관계는 연관관계의 주인이 아니기 때문에 별도의 설정이 필요 없고, 자동으로 데이터베이스에 외래 키 값이 저장되는 것이다. 예를들어,

```java
team1.getMembers().add(member1);
```

과 같은 코드가 존재해도, 이는 데이터베이스에 저장할 때 무시된다. 연관관계의 주인이 아닌 필드는 **READ-ONLY**이기 때문이다.

### 5.6 양방향 연관관계의 주의점

양방향 연관관계의 주인이 아닌 곳에만 값을 입력하고, 주인인 곳에 값을 입력하지 않는 실수는 언제든지 일어날법한 실수이다. 실제로 이경우에는 테이블의 외래키가 제대로 입력되지 않는 결과를 초래한다. 따라서 이런일이 발생하면 연관관계의 주인설정과 해당 주인에 제대로 값을 입력했는지를 살펴보는 것이 필요하다.

#### 5.6.1 순수한 객체까지 고려한 양방향 연관관계

JPA를 배제하고 순수한 객체 관점에서 생각한다면, 양쪽 방향에 모두 값을 입력해주는 것이 안전하다고 할 수 있다. 예를들어 JPA를 사용하지 않은 엔티티에 대한 테스트코드가 아래와 같이 작성된다고 해보자.

```java
public void testPureObjectTwoWay(){
    
    Team team1 = new Team("team1", "팀1");
    Member member1 = new Member("member1", "회원1");
    Member member2 = new Member("member2", "회원2");
    
    member1.setTeam(team1);
    member2.setTeam(team1);
    
    List<Member> members = team1.getMembers();
    System.out.println(members.size());  // 0
}
```

JPA를 배제하고, 객체만 사용했을때 `team1.members`에 적절한 설정을 해주지 않았으므로 `members.size`는 0이 된다. 아래 처럼 양쪽 모두의 엔티티에 관계를 설정해주는 방법을 사용하면,

```java
public void testPureObjectTwoWay(){
    
    Team team1 = new Team("team1", "팀1");
    Member member1 = new Member("member1", "회원1");
       
    Member member2 = new Member("member2", "회원2");
    
    member1.setTeam(team1);
    team1.getMembers().add(member1);
    member2.setTeam(team1);
	team1.getMembers().add(member2);
    
    List<Member> members = team1.getMembers();
    System.out.println(members.size());  //2
}
```

기대하는 2가 출력 된다. **결론적으로** ORM이라는 것은 데이터베이스 뿐만 아니라 객체라는 특성도 동일하게 고려되어야 하므로 객체의 양방향 연관관계는 항상 양쪽 모두 관계를 맺어주는 것이 좋다. 연관관계의 주인 여부는 오로지 데이터베이스로 저장하는 부분과 관련된 부분이기 때문이다.

#### 5.6.2 연관관계 편의 메소드

양방향 연관관계는 양쪽 다 신경을 써야 한다. 둘 중 하나만 호출하거나 해서 양방향이 깨지는 것을 예방하기 위해서는, 차라리 주인쪽의 관계 설정 메소드를 수정해서, 항상 순수한 객체안에서 양방향 관계를 가질 수 있게 리팩토링 하는 것도 하나의 방법이다.

```java
public class Member{
    private Team team;
    
    public void setTeam(Team team){
        this.team = team;
        team.getMembers().add(this);
    }
}
```

위와 같은 코드라면, 양방햔관계의 주인에서 호출하는 `setTeam()` 메소드 하나만으로 객체의 양방향 연관관계를 양쪽다 맺어줄 수 있다. 이렇게 한번에 양방향 연관관계를 설정하는 것을 **편의 메소드**라고 한다.

#### 5.6.3 연관관계 편의 메소드 작성 시 주의사항

위의 리팩토링된 메소드는 아직도 한가지 버그를 더 품고 있다, 연관관계 삭제가 일어날 때에도 양방향 연관관계를 고려해야 하는점이 빠진 것이다.

```java
member1.setTeam(teamA);
member1.setTeam(teamB);
Member findMember = teamA.getMembers(); //teamA에서 여전히 member1이 조회된다!
```

엔티티의 양방향 연관관계는 **2개의 단방향**연관관계 이기 때문에 변경이나 삭제시에도 이부분을 고려해 처리해줘야 한다. 즉 아래와 같은 형태가 생길 수 있는 것이다.

```mermaid
graph LR
m1(member1)-->tB
tA(teamA)-->|삭제되지 않은 관계|m1
tB(teamB)-->m1
```

따라서 `setTeam()` 메소드가 호출 될때에는 기존의 팀관계를 찾아서 이를 정리하고 재설정하는 코드가 필요하다.

```java
public void setTeam(Team team){
    if(this.team != null){
        this.team.getMembers().remove(this); //관계가 있으면, 기존관계는 제거
    }
    this.team = team;
    team.getMembers().add(this);
}
```

결국 테이블에서 외래키 하나로 간단하게 해결한 양방향 연관관계를, 객체지향 엔티티에서 얼마나 많은 고민과 견고한 로직으로 대체하는지를 보여준다. **물론** 데이터베이스에서 외래키 관리를 관계의 주인에서만 이뤄지기 때문에, teamA를 테이블에서 조회해서 한다고 할지라도 기존의 관계를 맺고있던 회원1이 반환되지는 않을 것이다. 문제는 영속성 컨텍스트가 아직 사랑있는 상태에서는 **teamA의 `getMembers()`** 메소드 호출에서 회원1을 반환할 것이고 이것은 객체상태와 데이터베이스 상태를 일치화하려는 ORM의 노력에 위배되므로, 관계를 제거한ㄴ 것이 안전하다.

### 5.7 정리

5장의 내용을 정리하면 다음과 같다.

* 단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 완료된다
* 단방향을 양방향으로 만들면 반대방향 객체 그래프 탐색기능이 추가된다.
* 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야 한다.

외래키 하나를 사용하는 데이터베이스의 양방향 매핑을, 로직으로 구현한 엔티티의 **양방향 매핑(실은 2개의 단방향 매핑)**의 장점은 노력에 비해 초라하게 보일 수 있다. 바로 반대 방향으로 객체 그래프 탐색이 가능하다는 점 뿐이기 때문이다.

따라서, 비즈니스 로직에 따라서, 단방향 매핑만을 우선적으로 사용하고, 만약 반대방향 그래프 탐색 기능이 필요할 때에는 양방향을 사용하도록 개선하는 것도 방법이다. 

**마지막으로** 연관관계의 주인을 정하는 기준을 다시한번 명확히 한다. **주인**이라는 이름이 가지는 의미떄문에 비즈니스 로직상 더 중요하다고 판단하는 엔티티의 필드를 **연관관계의 주인**으로 만들지 말아야 한다. 연관관계의 주인은 외래키의 주인일 뿐이지 비즈니스의 주인이 아니다. N:1의 관계에선, 외래키가 존재하는 **N**쪽이 반드시 연관관계의 주인으로 삼아야 한다. 

**추가로** 양방향 매핑은 로직상으로 **무한루프**를 발생시킬 여지를 가지고 있다. `toString()`메소드는 존재하는 필드의 내용을 모두 문자열로 바꾸려고 할 것이고, 양방향 관계에서는 서로 물리고 물린 관계로 인식해서 이를 무한루프에 돌입하게 할 수 있다.

