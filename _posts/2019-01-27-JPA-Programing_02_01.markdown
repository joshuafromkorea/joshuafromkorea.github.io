---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍- 02 JPA 시작"
date: 2019-01-27
categories:
---
1장에서 ORM 및 JPA에 대한 개념적, 그리고 기술적 소개를 하고 이번 장에서부터 본격적으로 실습을 하게 된다. 앞에서 살펴본 JPA의 특징과 장점에 대해서 아주 간단한 예제로 실습을 해보는데, 책 전반적으로, 저자가 반복하는 부분이 많은 편이다. 그래서 더 이해하기도 쉽고, 말하고자 하는 바에 동감할 수 있었다.

간단하게 살펴본 예제에서부터 JPA가 얼마나 편리하고 개발자가 객체중심으로 생각할 수 있게 만들어주는지 알 수 있었고, 아마 2장을 공부하고나서 회사로 돌아가서 이제 JPA가 아닌 시스템에서는 일하고 싶지 않다고 함께 일하는 개발자분에게 이야기했던 기억이난다.

---
## 2 JPA 시작

본 장은 JPA를 사용하는 애플리케이션을 만들기 위해서 개발환경을 세팅하는 부분을 먼저 다룬다. 여타 프로그래밍 기술이나 언어를 습득할 때 기본적으로는 IDE등 개발을 쾌적하게 해주는 툴을 설정하기 보다는 기본 문법등을 먼저 배우게 되는데, JPA의 경우에는 IDE와 데이터베이스에 대한 세팅이 없이는 개발하기에는 어려움이 있기 때문이다. 책에서는 이클립스를 통해서 설정을 하는데, 책의 내용을 참고하면서 IntelliJ IDEA에서 설정해보도록 하겠다.

### 2.1 이클립스(IntelliJ IDEA) 설치와 프로젝트 불러오기

저자가 제공하는 예제 프로젝트의 [github link](https://github.com/holyeye/jpabook)에 방문해보면 친절하신 분이 IDEA를 통한 설정에 대한 부분을 제공해주셔서 해당 내용을 참고하면서 진행한다.

#### 이클립스 설치  

책에서는 LUNA 버젼을 권하고 있고, Eclipsee ID for Java EE Devlopeers 패키지를 사용해서 JPA를 위한 도구들을 한꺼번에 받기를 권한다. 나의 경우 기존에 설치된 IDEA를 사용한다.

#### 예제 프로젝트 불러오기

##### forking
저자의   [github link](https://github.com/holyeye/jpabook) 에서 내 repository로 **fork**한다.  보통 주말에는 집의 데스크탑 환경에서 공부하고, 평일에는 카페등에서 랩탑을 사용하기 때문이다.

##### cloning

git의 clone 기능을 사용해서 github에 있는 소스를 내 컴퓨터의 로컬환경에 clone 한다. (git bash 사용)

```bash
$ git clone https://github.com/kiwonseo/jpabook.git jpabook/
Cloning into 'jpabook'...
remote: Enumerating objects: 281, done.
remote: Total 281 (delta 0), reused 0 (delta 0), pack-reused 281
Receiving objects: 100% (281/281), 381.04 KiB | 536.00 KiB/s, done.
Resolving deltas: 100% (64/64), done.
$ ls -l
total 2
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch02-jpa-start1/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch04-jpa-start2/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch04-model1/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch05-model2/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch06-model3/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch07-model4/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch08-model5/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch09-model6/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch11-jpa-shop/
drwxr-xr-x 1 Joshua 197121   0 1월  26 22:30 ch12-springdata-shop/
-rw-r--r-- 1 Joshua 197121 239 1월  26 22:30 clean.sh
-rw-r--r-- 1 Joshua 197121 638 1월  26 22:30 README.md

```

##### Import Project

clone한 프로젝트의 root 디렉토리를 IDEA 시작화면에서 Import한다. **Import project from external model** 을 선택하고,  Maven을 지정한 후 **Next**를 클릭한다. 

![import settings](./jpa_pic/img001.JPG)

다음화면에서는 위와 같이 상세 설정을 해준 후 **Next**를 클릭한다. 이후 **Select Maven projects to import** 화면에서 몯든 프로젝트 (ch1~ch12)를 선택한 후 **Next**를 클릭한다. 프로젝트를 위한 SDK를 선택하는 화면인데 책에서는 1.6버전 이상을 사용할 것을 권한다. 나는 JDK 1.8을 사용했다. SDK 버전을 선택하고 **Next**를 클릭한다. 마지막으로 프로젝트명과 로케이션을 확인 한 후에 **Finish**를 클릭한다. 이후 IDEA가 자동으로 Maven 설정 파일을 바탕으로 라이브러리를 받는 과정을 기다리면 된다.

### 2.2 H2 데이터베이스 설치

H2 데이터베이스 웹사이트에 [접속](http://h2database.com)에서 다운 받을 수 있다. 압축으로 되어있는 파일을 해제하고, 해당 경로에서 `bin/h2.sh`를 실행 하면 서버모드로 실행 된다. 

> 현재 개발 공부를 집의 데스크탑과 외부에서는 랩탑으로 진행하고 있어서, 이참에 어플리케이션과 함께 인 메모리로 구동되도록 설정해서 하는 방법을 해보려고 했으나, 아직 지식이 부족해서 실패했다. 일단 궁여지책으로 Dropbox를 사용해서 편법으로나마 같은 환경을 구축하였다.

실행하면 웹브라우저가 열리는데 설정을 embedded에서 server로 변경한 후 연결하면 된다. 연결후 SQL입력창에 아래와 같이 입력해서 `MEMBER`테이블을 생성한다. 아래의 쿼리는 clone한 프로젝트의 경로에도 존재한다.

```sql
CREATE TABLE MEMBER (
    ID VARCHAR(255) NOT NULL, --아이디(기본 키)
    NAME VARCHAR(255),        --이름
    AGE INTEGER NOT NULL,     --나이
    PRIMARY KEY (ID)
)
```

책에서 나온 것처럼, 웹 콘솔을 사용해서 데이터베이스에 접근 하여도 되지만, IntelliJ의 경우 Database에 접근 할 수 있는 기능을 가지고 있기 때문에 해당기능을 사용하여, 테이블을 생성했다.

### 2.3 라이브러리와 프로젝트 구조

JPA 구현체로 하이버네이트를 사용하기 위한 핵심 라이브러리는 다음과 같다.

* hibernate-core
* hibernate-entitymanager : JPA구현체로 동작하도록 표준을 구현한 라이브러리
* hibernate-jpa-2.1-api: JPA 2.1의 표준 API를 모아놓은 라이브러리

위의 라이브러리들과 관련된 메이븐 설정을 담은 pom.xml은 아래와 같다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>jpabook</groupId>
	<artifactId>ch02-jpa-start1</artifactId>
	<version>1.0-SNAPSHOT</version>

	<properties>

		<!-- 기본 설정 -->
		<java.version>1.6</java.version>
		<!-- 프로젝트 코드 인코딩 설정 -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

		<!-- JPA, 하이버네이트 버전 -->
		<hibernate.version>4.3.10.Final</hibernate.version>
		<!-- 데이터베이스 버전 -->
		<h2db.version>1.4.197</h2db.version>

	</properties>


	<dependencies>
		<!-- JPA, 하이버네이트 -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<!-- H2 데이터베이스 -->
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>${h2db.version}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>

```

실제로 위에서 3개의 JPA관련 라이브러리가 필요하다고 하였으나 `hibernate-entitymanager`만 내려받으면 core와 api 라이브러리는 한꺼번에 받게 된다. 

> 책에 나온 pom.xml 설정에는 H2 최신 버전과 드라이버 버전이 다르기 때문에 최신 버전 DB를 설치했다면 에러가 발생한다. DB버전과 해당 설정을 일치 시켜야 한다.

### 2.4 객체 매핑 시작

앞서 DB를 설치하면서 만든 회원 테이블에 매핑되는, 회원 클래스를 만들고, JPA제공하는 매핑 어노테이션을 소스코드에 추가해 아래와 같이 클래스를 완성한다.

```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

JPA는 어노테이션 정보를 활용해서 객체가 어떤 테이블과 관계가 있는지를 파악한다. 

#### 사용한 어노테이션 정리

##### `@Entity`

클래스 위에 선언되어 해당 클래스가 관계형데이터베이스의 테이블과 매핑 된다고 알려준다. 일반적으로 해당 어노테이션이 붙은 클래스를 엔티티 클래스라고 한다.

##### `@Table`

해당 어노테이션을 사용하여 `name`이라는 속성에 정확히 사용할 테이블 명을 매핑해 줄 시 있다. 이 어노테이션의 생략을 하게 되면 **엔티티 이름**을 테이블과 매핑한다. 엔티티이름과 클래스 이름의 차이는 4.1에서 설명한다.

##### `@Id`

엔티티 클래스의 **필드**를 테이블의 **기본 키**와 매핑한다. 이렇게 `@Id`를 부여한 필드를 **식별자 필드**라고 한다.

##### `@Column`

필드를 컬럼에 매핑한다. `@Table`에 부여한 `name`속성과 같이 동일 속성을 사용해서 테이블의 실제 컬럼명과 매핑해 줄 수 있다.

##### 어노테이션이 없는 필드

`age`필드의 경우, 어노테이션을 생략했는데, 엔티티 클래스의 필드는 기본적으로 동일한 이름을 가지는 테이블의 컬럼과 매핑된다. (대소문자 무시)

### 2.5 persistence.xml 설정

JPA에 대한 설정은 프로젝트 경로내의 psersistence.xml 파일을 통해서 관리한다. 이 파일이 클래스패스의 META-INF/persistence.xml 에 존재하면 자동으로 인식되어 사용된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <persistence-unit name="jpabook">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" 
                      value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" 
                      value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>
```

#### persistence.xml 설정 분석

##### `<persistence-unit name="jpabook">`

**영속성 유닛**은 JPA가 관리하는 단위 개념으로 이름을 필요로 한다. 보통 연결할 데이터베이스 당 하나의 영속성 유닛을 등록한다.

##### JPA 표준 속성
```xml
<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
<property name="javax.persistence.jdbc.user" value="sa"/>
<property name="javax.persistence.jdbc.password" value=""/>
<property name="javax.persistence.jdbc.url" value=""/>
```

특정 구현 속성에 종속되지 않는 JPA속성들로, 사용할 드라이버, 데이터베이스 접속 정보등을 담고 있다. H2 DB드라이버 사용을, MySQL 등으로 변경하면 이용할 수 있다.

##### 하이버네이트 속성

```xml
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
```

하이버네이트에 종속되는 속성으로, DB를 사용하기 때문에 h2 Dialect를 사용하도록 한 속성이다. 위의 드라이버 속성을 다른 DB의 것으로 바꾸면 하이버네이트 속성에서 사용하는 방언(Dialect)에 대한 속성도 변경해주어야 한다.

##### 추가적인 하이버네이트 전용속성

```xml
<property name="hibernate.show_sql" value="true" /> <!--SQL을 출력한다-->
<property name="hibernate.format_sql" value="true" /> <!--출력한 SQL을 Format한다 -->
<property name="hibernate.use_sql_comments" value="true" /> <!--주석도 포함한다 -->
<!--JPA 표준에 맞춘 키 생성 전략을 사용한다 4.6절-->
<property name="hibernate.id.new_generator_mappings" value="true" />
```

#### 2.5.1 데이터베이스 방언

**Dialect**(방언)클래스는 각 데이터베이스마다 제공하는 SQL문법과 함수가 가지는 차이를 보완하여서, JPA 구현체가 특정 데이터베이스에 종속적이지 않도록 하는 기능을 담은 클래스이다. 하이버네이트는 H2, Oracle, MySQL등 주요 관계형데이터베이스를 위한 방언을 제공한다.

### 2.6 애플리케이션 개발

#### 2.6.1 엔티티 매니저 생성

엔티티 매니저의 생성 과정은 다음과 같다.

<div class="mermaid">
graph LR
	P[Persistence]
	M[persistence.xml]
	F[EntityManagerFactory]
	E[EntityManager]
	P-->|1.설정 정보 조회|M
	P-->|2.생성|F
	F-->|3.생성|E
</div>

##### 엔티티 매니저 팩토리 생성

JPA를 사용하려면 먼저 persistence.xml의 설정정보를 바탕으로 엔티티 매니저 팩토리를 생성해야 한다. `Persistence`클래스의 메소드를 사용해서 다음과 같이 생성한다.

```java
//엔티티 매니저 팩토리 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
```

`"jpabook"`이라고 이름을 부여한 영속성 유닛을 찾아서 해당 정보를 바탕으로 JPA동작 객체를 만들고 구현체에 따라서는 하위 설정정보를 사용해서 데이터베이스 커넥션 풀도 생성한다. 

> 따라서 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.

##### 엔티티 매니저

엔티티 매니저 팩토리 객체를 통해서 생성한 엔티티 매니저는 **데이터베이스의 CRUD**를 수행 할 수 있게 하는 클래스이다.

```java
EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
```

엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으며, 개발자의 코드 입장에서는 데이터베이스를 추상화 한 객체로 생각할 수 있다. **스레드간에 공유하거나 재사용하지 않는 것이 필수이다.**

##### 종료 처리

엔티티 매니저와 엔티티 매니저 팩토리는 반드시 종료 시켜야 한다. 전자의 경우에는 사용이 끝났을 경우이고, 후자는 애플리케이션 종료시 종료시킨다.

```java
em.close(); //엔티티 매니저 종료
emf.close(); //엔티티 매니저 팩토리 종료
```

#### 2.6.2 트랜잭션 관리

JPA를 사용할 때 데이터에 변경을 트랜잭션 밖에서 진행하면 예외가 발생한다. 트랜잭션은 엔티티 매니저가 관리하며, `try-catch`문을 통해 트랜젹의 커밋과 롤백을 관리한다.

```java
try {
    tx.begin(); //트랜잭션 시작
    logic(em);  //비즈니스 로직
    tx.commit();//트랜잭션 커밋
} catch (Exception e) {
    e.printStackTrace();
    tx.rollback(); //트랜잭션 롤백
```

#### 2.6.3 비즈니스 로직

예제에서 살펴보는 비즈니스 로직은 단순하다. 회원 엔티티를 **하나 생성**하고나서, 데이터베이스에 저장, 수정, 삭제 조회한다.

##### 등록

등록에는 `EntityManager.persist()` 메소드를 사용한다. 엔티티 객체를 매개변수로 전달하면, JPA는 해당 엔티티의 매핑 정보를 분석해서 SQL을 아래와 같이 생성하여 데이터베이스에 전달한다.

```sql
/* insert jpabook.start.Member*/ 
insert 
    into
MEMBER (age, NAME, ID) 
    values
('id1', '지한', 2)
```

##### 수정

```java
//수정
member.setAge(20);
```

수정 코드는 코드 자체로는 엔티티 매니저를 사용하고 있지 않다. 마치 지금까지 해왔던 데이터베이스 연동 코드들처럼 UPDATE작업을 위해 추상화된 메소드를 호출해야 할 것 같지만 필요가 없다. JPA는 엔티티 객체가 변경 되었다는 것을 추적해서 값이 변경 되면 적절한 SQL을 생성해서 실행한다.

```sql
/* update jpabook.start.Member */ 
update
    MEMBER 
set
    age=2,
    NAME='지한' 
where
    ID='id1'
```

나이만 변경했지만, 전체 필드를 변경하는 SQL을 사용하였다.

##### 삭제

엔티티를 삭제하려면, `EntityManager.remove()` 메서드를 호출하면서 매개변수로 삭제하려는 엔티티를 전달한다. 동일하게 삭제를 위한 SQL이 실행된다.

```sql
/* delete jpabook.start.Member */ 
delete 
    from
        MEMBER 
    where
        ID='id1'
```

##### 한 건 조회

```java
//한 건 조회
Member findMember = em.find(Member.class, id);
```

엔티티 매니저의 `find()`메서드를 호출하면서, 조회할 엔티티 타입과, `@Id` 어노테이션을 통해 해당 엔티티에 정의한 식별자 값을 전달하면, 엔티티 하나를 조회한다. 식별자는 테이블의 Primary Key 매핑이 되기 때문에 유니크한 단위인 Row가 반환되고 이게 엔티티로 매핑되어 다시 반환되는 것이다.

#### 2.6.4 JPQL

`EntityManager.createQuery()` 메소드를 호출하면서, `String`타입과, 엔티티 타입을 전달하면, 쿼리를 생성할 수 있고, 해당 쿼리 객체가 제공하는 메소드를 통해 결과를 받아올 수 있다. 이런 기능은 SQL을 전혀 사용하지 않는 JPA 사용 구조에서, 애플리케이션이 필요로 하는 **데이터**만을 찾기 위해 필요하다.  JPA가 제공하는 검색 기능은 반드시 엔티티 객체를 조회하는 것이기 때문이다. 

이때 사용하는 쿼리 언어는 SQL이 아닌 JPQL이라고 부르는데, 그 문법이 SQL과 유사하지만 다음의 차이 점이 존재한다.

* JPQL은 **엔티티 객체**를 대상으로 하는 쿼리이다.
* SQL은 **데이터베이스 테이블**을 대상으로 쿼리한다.

```java
List<Member> members = em.createQuery("select m from Member m", Member.class)
    .getResultList();
```

위의 예제의 JPQL을 보면 SQL문과 다른 점을 알 수 있는데, 철저하게 대소문자를 구별하고 있는 점이다. 그 이유는 앞서 언급한 것처럼 JPQL은 자바로 구성된 엔티티에 대한 조회이기 때문이다. 자바는 문법상 대소문자를 지원하므로 `Member`라는 이름을 가진 엔티티 객체를 조회하는 JPQL은 이와 매핑되는 `MEMBER` 테이블에 대해서는 알지 못한다.

**결국,** JPQL도 동일하게 JPA에 의해서 SQL로 변환되어 JDBC API에 전달되는 과정을 동일하게 거친다.

