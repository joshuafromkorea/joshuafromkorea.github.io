---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍- 06  다양한 연관관계 매핑"
date: 2019-08-16
categories:
---
오랜만에 JPA책을 펼쳐 들었다. 팀내 사원들이랑 공부를 시작했는데, 슬슬 공부하는 진도가 내가 한부분까지 따라오고 있어서 다시 공부하게 되었다. 조금은 복잡하고 이해가 안되는 부분이 있지만, 일단 어떤 걸 활용하는게 좋은지정도를 깨달았으니 무리는 없다.

---
## 6 다양한 연관관계 매핑

다양한 연관관계를 살펴보기전에 먼저 숙지해야 하는 사항이 세 가지 있다.

1. 연관관계의 반대 방향을 항상 고려한다 ( 다대일 관계의 반대는 일대다 관계)
2. 데이터베이스 테이블에서 외래키는 항상 **다(N)**에 있다.
3. 따라서 객체 양방향 관계에서도 연관관계 주인은 항상 **다(N)**쪽이다.

### 6.1 다대일

#### 6.1.1 다대일 단방향

<div class="mermaid">
graph LR
M["<u>MEMBER</u></br>MEMBER_ID(PK)</br><b>TEAM_ID(FK)</b></br>USERNAME"]
T["<u>TEAM</u></br>TEAM_ID(PK)</br>NAME"]
M---|n..1|T
</div>

* 위와 같은 테이블 연관관계를 다음의 다대일 관계로 표현할 수 있다.

##### 회원 엔티티

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    
    //Getter, Setter...
}
```

##### 팀 엔티티

```java
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    //Getter, Setter...
}
```

* 위의 다대일 관계는 회원엔티티가 `@JoinColumn(name="TEAM_ID")`를 사용해서 `Member.team` 필드로 외래키를 매핑해 관리한다.

#### 6.1.2 다대일 양방향[N:1, 1:N]

* 엔티티의 단방향 관계만으로는 데이터베이스의 외래키로 이루어진 관계를 완전히 표현할 수 없다.
* 따라서 다대일 단방향과 매칭되는 반대방향인 **일대다 단방향관계를 추가한다.**
  * 또한 무한루프에 빠지지 않도록 헬퍼 메소드를 만들어 주어야 한다.

##### 회원 엔티티

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    
    //무한루프 방지 setter
    public void setTeam(Team team){
        this.team = team;
        if(!team.getMembers().contains(this)){
            team.getMembers().add(this);
        }
    }
    
    //Getter, Setter..
}
```

##### 팀 엔티티

```java
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy ="team")
    private List<Member> members = new ArrayList<Member>();
    
    public void addMember(Member member){
        this.members.add(member);
        if(member.getTeam()!=this){
            member.setTeam(this);
        }
    }
    
    //Getter, Setter...
}
```

* 양방향 연관관계에서 외래키가 있는 쪽이 연관관계의 주인이다.
  * 위의 예제의 경우엔 `Member.team`이 연관관계의 주인이다.
* 양방향 관계에서, 연관관계 편의메소드를 양쪽에 만들경우 무한루프를 방지해야 한다.

### 6.2 일대다

#### 6.2.1 일대다 단방향 [1:N]

* 일대다 단방향 매핑은 관리의 부담이나 성능의 문제등 단점이 많은 방법이다.
* 따라서 **일대일 단방향 보다 다대일 양방향을 사용하자**

#### 6.2.2 일대다 양방향 [1:N, N:1]

> 일대다 양방향 매핑은 존재하지 않는다, 대신 다대일 양방향 매핑을 사용해야 한다.

### 6.3 일대일[1:1]

* 일대일 관계는 양쪽이 서로 하나의 관계만을 가지는 경우로 아래와 같은 특징이 있다.
  * 일대일 관계는 그 반대도 일대일 관계이다
  * 일대일 관계는 주 테이블이나 대상테이블 중에 어느곳이나 외래키를 가지고 있다.
* 일대일 관계를 외래키를 어느테이블에 가지는가로 두 가지로 나눌 수 있다.
  * 주 테이블에 외래키
    * 주테이블이 외래키를 가지고 있으므로 주테이블만 확인해도 대상테이블과 연관관계를 찾을 수 있다.
  * 대상 테이블에 외래키
    * 전통적인 데이터베이스 개발 방식으로 일대일에서 일대다로 변경할때 유리하다

#### 6.3.1 주 테이블에 외래키

* 회원이 주테이블이고, 사물함이 대상 테이블이라고 할때 아래의 예제를 보자

##### 단방향

```java
@Entity
public class Member{
    
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    ...
}
```

* 다대일 단방향과 어노테이션 하나만 다르고 매우 유사하다.

##### 양방향

```java
@Entity
public class Member{
    
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Member member;
    ...
}
```

* 양방향 관계에서는 `mappedBy` 속성이 존재하는 곳이 주인이 아니므로, `Member` 엔티티가 주인이 된다.

#### 6.3.2 대상 테이블에 외래키

##### 단방향

> 일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계는 JPA에서 지원하지 않는다.

##### 양방향

```java
@Entity
public class Member{
    
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(mappedBy="member")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;
    ...
}
```

* 대상 테이블인 `Locker`클래스가 외래키를 관리하도록 했다.
* 만약에 대상테이블이 N이 되는 관계로 변경된다면 쉽게 다대일 양방향 관계로 바꿀 수 있다.

### 6.4 다대다 [N:N]

* 관계형 데이터베이스에서 다대다 관계는 중간에 매핑 테이블을 두는 방식으로 표현한다.

<div class="mermaid">
graph LR
	M["<b><u>Member</u></b></br>MEMBER_ID(PK)</br>USERNAME"]
	P["<b><u>Product</u></b></br>PRODUCT_ID(PK)</br>NAME"]
	M---|n..n|P
</div>

* 위의 관계는 아래와 같이 `Member_Product`테이블을 중간에 두어서 표현해야 한다.

<div class="mermaid">
graph LR
	M["<b><u>Member</u></b></br>MEMBER_ID(PK)</br>USERNAME"]
	P["<b><u>Product</u></b></br>PRODUCT_ID(PK)</br>NAME"]
	MP["<b><u>Member_Product</u></b></br>MEMBER_ID(PK, FK)</br>PRODUCT_ID(PK, FK)"]
	M---|1..n|MP
	MP---|n..1|P
</div>

* 하지만 이런 제약사항이 있는 RDB와 달리 JPA 엔티티는 `@ManyToMany`를 사용해서 엔티티간 다대다 관계를 편리하게 매핑할 수 있다.

#### 6.4.1 다대다: 단방향

##### 회원 엔티티

```java
@Entity
public class Member{
    
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
              joinColumns = @JoinColumn(name="MEMBER_ID"),
              inverseJoinColumns = @JoinColumn(name="PRODUCT_ID"))
    private List<Product> products = new ArrayList<Product>
    ...
}
```

##### 상품 엔티티

```java
@Entity
public class Product{
    
    @Id @Column(name="PRODUCT_ID")
    private String id;
    
    private String name;
    ...
}
```

* `@JoinTable`이라는 새로운 어노테이션을 통해 연결 테이블을 바로 매핑했다. 사용한 속성을 알아보자
  * `@JoinTable.name`: 연결테이블을 지정한다
  * `@JoinTable.joinColumns`: 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다
  * `@JoinTable.inverseJoinColumns`: 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.
* 실제로 JPA에서 해당 조인 테이블을 사용하는 것을 살펴보면 일반적인 그래프탐색으로 해결됨을 알 수 있다.

##### 저장

```java
public void save(){
    Product productA = new Product();
    productA.setId("productA");
    productA.setName("상품A");
    em.persist(productA);
    
    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    member1.getProducts().add(productA);//연관관계 설정
    em.persist(member1);
}
```

##### 탐색

```java
public void find(){
    
    Member member = em.find(Member.class, "member1");
    List<Product> products = member.getProducts(); //객체 그래프 탐색
    for(Product product :products){
        System.out.println(product);
    }
}
```

#### 6.4.2 다대다: 양방향

* 역방향의 경우에도 다대다 이므로 동일하게 `@ManyToMany`를 사용한다 다만 양쪽 중 한곳에 `mappedBy`를 사용해서 연관관계 주인이 아님을 명시한다.

```java
@Entity
public class Product{
    
    @Id @Column(name="PRODUCT_ID")
    private String id;
    
    private String name;
    
    @ManyToMany(mappedBy = "products")//역방향 추가
    private List<Member> members;
    ...
}
```

* 앞서 살펴본 양방향 관계들 처럼 연관과계 편의 메소드를 추가해서 관리하는 것이 편리하다.

```java
public void addProduct(Product product){
    products.add(product);
    product.getMembers().add(this);
}
```

* 양방향 관계가 만들어지면, 반대 방향에서도 객체 그래프 탐색을 할 수 있다.

```java
public void findInverse(){
    
    Product product = em.find(Product.class, "productA");
    List<Member> members = product.getMembers();
    for(Member member : members){
        System.out.println(member);
    }
}
```

#### 6.4.3 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

* 앞서 살펴본 `@ManyToMany`를 통한 연결관계는 실무에서 사용하기에 한계가 있다.
  * 연결 테이블이 단순히 연결 역할 뿐만 아니라 다른 데이터를 포함하기 때문이다.

##### 추가 필드를 가진 연결테이블

<div class="mermaid">
graph LR
	M["<b><u>Member</u></b></br>MEMBER_ID(PK)</br>USERNAME"]
	P["<b><u>Product</u></b></br>PRODUCT_ID(PK)</br>NAME"]
	MP["<b><u>Member_Product</u></b></br>MEMBER_ID(PK, FK)</br>PRODUCT_ID(PK, FK)</br>ORDERAMOUNT</br>ORDERDATE"]
	M---|1..n|MP
	MP---|n..1|P
</div>

* 위와 같이 연결테이블에 컬럼이 추가되면 더이상 `@ManyToMany`를 사용할 수 없다.
* 따라서 관계형 데이터베이스에서 하는 방식대로, 별도의 엔티티를 만들어서 풀어나가야 한다.

##### 회원 엔티티

```java
@Entity
public class Member{
    
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToMany(mappedBy="member")
    private List<Product> products = new ArrayList<Product>
    ...
}
```

##### 상품 엔티티

```java
@Entity
public class Product{
    
    @Id @Column(name="PRODUCT_ID")
    private String id;
    
    private String name;
    ...
}
```

* 상품에서 회원으로 그래프 탐색을 할 수 없는 상품 엔티티 코드이다.

##### 회원상품 엔티티

```java
@Entity
@IdClass(MemberProductId.class) //회원 상품 식별자 클래스
public class MemberProduct{
    
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member; //MemberProductId.member와 연결
    
    @Id
    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product; //MemberProductId.product와 연결
    
    private int orderAmount;
    ...
        
}
```

##### 회원상품 식별자 클래스

```java
public class MemberProductId implements Serializable{
    private String member; //MemberProduct.member와 연결
    private String product; //MemberProduct.product와 연결
    
    @Override
    public boolean equals(Object o){..}
    
    @Override
    public int hashCode(){..}
}
```

* `MemberProduct` 엔티티에 사용한 `@IdClass` 어노테이션은 **복합 기본 키**를 매핑한다.
* **복합 기본 키**
  * 한 개 이상의 기본키로 이루어진 복합 기본키로 `@IdClass`를 사용해서 식별자 클래스를 지정해야 한다.
  * 복합키는 반드시 별도의 식별자 클래스로 만들어야 한다.
  * `Serializable`을 구현하고, `equals`와 `hashcode`메서드를 구현해야 한다.
  * 기본생성자가 있어야 한다.
  * `public`이어야 한다.

#### 6.4.4 다대다 : 새로운 기본 키 사용

* 복잡한 복합기본키를 사용하지 않고, "데이터베이스에서 자동으로 생성해주는 대리 키"를 사용해보자

<div class="mermaid">
graph LR
	M["<b><u>Member</u></b></br>MEMBER_ID(PK)</br>USERNAME"]
	P["<b><u>Product</u></b></br>PRODUCT_ID(PK)</br>NAME"]
	O["<b><u>Orders</u></b></br>ORDER_ID(PK)</br>MEMBER_ID(FK)</br>PRODUCT_ID(FK)</br>ORDERAMOUNT</br>ORDERDATE"]
	M---|1..n|O
	O---|n..1|P
</div>

* 두개의 외래키를 복합키로 사용하지 않고, `Orders`테이블을 위해 별도의 기본키(`ORDER_ID`)를 만들었다.

##### 주문 엔티티

```java
@Entity
public class Orders{
    
    @Id @GeneratedValue
    @Column(name="ORDER_ID")
    private Long id;
    
    @ManyToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;
    
    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product;
    
    private int orderAmount;
}
```

* 기존의 회원상품엔티티와 식별자 코드를 과감히 버리고 위의 `Orders` 엔티티를 사용하면, 기존의 회원과 상품 엔티티를 그대로 사용할 수 있다.
* 객체지향적 코드 입장에서는 위와 같이 비식별 관계로 만드는 것이 더 개발에 용이하다.

