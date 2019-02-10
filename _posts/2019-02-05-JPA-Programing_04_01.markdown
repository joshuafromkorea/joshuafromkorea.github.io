---
layout: post
title: "자바 ORM 표준 JPA 프로그래밍- 04 엔티티 매핑"
date: 2019-02-05
categories:
---
4장에서는 연관관계 매핑을 제외한 아주 기본적인 방법을 통해 기존에 우리가 JDBC나 MyBatis등을 통해서 해오던 기본적인 CRUD 작업을 해본다. JPA에서 매핑이란 자바라는 객체지향 언어가 가진 자료의 구조인 엔티티(클래스)를 어떻게 관계형 데이터베이스의 테이블로 매핑시키는가이다.

아주 기본적인 테이블 형태와 엔티티간의 매핑을 다루는 만큼, 흔히 SQL 에서 JOIN으로 조회되는 테이블간의 릴레이션에 대한 매핑은 다루지 않는다. 또한 기본적인 매핑으로 만든 엔티티를 통해서 개발환경에서 이를 바탕으로 DDL을 실행하고 테이블 스키마를 자동생성하는 부분도 다룬다.

---

## 4 엔티티 매핑

JPA의 엔티티-테이블간 매핑은  XML과 어노테이션 중 하나를 사용해서 할 수 있지만, 어노테이션이 좀더 간편하고 직관적이다. 테이블과 엔티티 매핑은 크게 아래의 4가지가 있다.

* **객체와 테이블 매핑**: `@Entity`,`@Table`
* **기본 키 매핑**: `@Id`
* **필드와 컬럼 매핑**: `@Column`
* **연관관계 매핑**: `@ManyToOne`, `@JoinColumn`

이번 장에서는 연관관계 매핑을 제외한 매핑을 알아본다

### `@Entity`

JPA를 사용해서 테이블과 매핑할 클래스는 반드시 `@Entity` 어노테이션을 붙여야한다. 해당어노테이션이 가지는 속성은 다음과 같다.

| 속성   | 기능                                                         | 기본값                                    |
| ------ | ------------------------------------------------------------ | ----------------------------------------- |
| `name` | JPA에서 사용할 엔티티 이름이다. 다른 패키지에 이름이 같은 클래스가 있을 때 충돌을 피하려고 사용한다. | 일반적으로 클래스 이름을 그대로 사용한다. |

클래스를 엔티티로 사용(`@Entity` 적용)하기 위한 조건은 다음과 같다.

* 기본 생성자를 반드시 포함(`public` 혹은 `protected`)
  * 기본생성자 이외에 다른 생성자가 있을시, 자바가 기본생성자를 만들지 않으므로 반드시 생성
* `final`클래스나, `enum`,`interface`, 및 Inner 클래스에는 사용 불가
* 저장할 필드에 `final`을 사용불가

### `@Table`

엔티티와 매핑할 테이블을 지정하는데 사용하는 `@Table`어노테이션은 생략하면 엔티티 이름을 사용한다. 속성은 다음과 같다.

| 속성                | 기능                                                         | 기본값      |
| ------------------- | ------------------------------------------------------------ | ----------- |
| `name`              | 매핑할 테이블 이름                                           | 엔티티 이름 |
| `catalog`           | 카탈로그 기능이 있는 DB사용시 매핑                           |             |
| `schema`            | 스키마 기능이 있는 DB사용시 매핑                             |             |
| `uniqueConstraints` | DDL생성시 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. |             |

### 4.3 다양한 매핑 사용

이외의 다양한 형태의 필드 매핑을 지원하는 어노테이션이 아래와 같이 있다.

* **`@Enumerated`**: 자바의 `Enum`을 타입으로 하는 필드와 매핑한다.
* **`@Temporal`**: 자바의 날짜 타입과 매핑하기 위해 사용한다.
* **`@Lob`**: 필드 길이에 제한이 없는 필드를 DB의 `VARCAHR`대신 `CLOB`,`BLOB`등에 매핑하기 위해 사용한다.

### 4.4 데이터베이스 스키마 자동 생성

JPA는 데이터베이스 스키마를 자동생성 하는 기능을 지원한다. 어노테이션으로 만든 매핑정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 사용한다. 이를 위해 아래의 `persistence.xml`설정이 필요하다.

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

해당 기능을 사용하면, 기존의 테이블을 삭제(`drop`)하고 새로 생성하게 된다. 스키마 자동생성 기구를 통해서 개발환경에서 테이블 생성을 하게 되면, JPA가 어떻게 엔티티와 테이블을 매핑하는지 알 수 있다. 테이블 생성을 위한 설정의 속성정보는 다음과 같다.

| 옵션          | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| `create`      | 기존 테이블 삭제 후 생성 (`DROP` then `CREATE`)              |
| `create-drop` | 어플리케이션 종료 시 생성한 DDL제거 (`DROP` then `CREATE`, and `DROP` when finished) |
| `update`      | 테이블과 엔티티를 비교해서 변경사항만 수정                   |
| `validate`    | DB와 매핑정보를 비교해서 차이점을 경고만 남긴다. DDL수정하지 않는다 |
| `none`        | 유효하지 않은 옵션 값을 주면 설정이 무시된다.                |

일반적으로 자바와 RDB는 관례상 네이밍법이 차이가 있다. 자바의 경우에는 카멜표기법을 사용하나 데이터베이스에서는 언더스코어를 주로 사용한다. 스키마 자동생성에서 해당 부분을 제어할 수 있는 옵션도 제공한다. `@Column`어노테이션의 이름속성을 사용하여 해당 전략을 적용할 수도 있지만, 아래와 같은 xml설정으로 간단히 모든 엔티티에 적용할 수도 있다.

```xml
<property name="hibernate.ejb.naming_strategy" 
          value="org.hibernate.cfg.ImprovedNamingStrategy" />  
```

### 4.5 DDL 생성 기능

어노테이션 속성을 사용해서 DDL 생성시 수행되는 제약조건을 만들수도 있다.

```java
@Column(name = "NAME", nullable = false, length = 10) //추가 //**
private String username;
```

위의 예제에선 `nullable=false` 속성과, `length=10` 속성을 주었다. 이제 어플리케이션을 실행해서 실제로 입력되는 DDL을 모니터링하면 해당 속성들이 데이터베이스의 어떤 제약조건과 매핑되는 지 알 수 있다.

```sql
create table MEMBER (
    ID varchar(255) not null, 
    NAME varchar(10) not null, --nullable = false , length = 10
    ...
    primary key (ID)
)
```

또한 `@Table`어노테이션에 속성을 주어, 테이블에 유니크 제약 조건을 줄 수도 있다.

```java
@Entity
@Table(name="MEMBER", uniqueConstraints = {@UniqueConstraint(
        name = "NAME_AGE_UNIQUE",
        columnNames = {"NAME", "AGE"} )})
public class Member {
    ...
}
```

생성된 DDL은 아래와 같다.

```sql
ALTER TABLE MEMBER 
    add constraint NAME_AGE_UNIQUE  unique (NAME, age)
```

이렇게 엔티티와 필드에 준 JPA속성 값들은 DDL 자동생성에만 영향을 줄 뿐, **어플리케이션이 JPA를 통해서 DB에 접근하는 실행 로직에는 영향을 주지 않는다.** 다만 기능을 통해서 개발자가 엔티티만 보고도 손십게 다양한 제약조건을 파악할 수 있는 장점이 있다.

### 4.6 기본 키 매핑

지금까지는 `@Id`어노테이션을 사용해 기본 키를 어플리케이션에서 할당하였다. 데이터베이스가 생서해주는 기본 키를 사용하여야 하는 경우에는, 각각의 DB마다 기본 키 생성 방식이 다르므로, JPA만의 해결전략이 필요하다.

* **직접 할당**: 기본 키를 애플리케이션에서 할당한다.
* **자동 생성**: 대리 키 사용 방식
  * IDENTITY : DB에 생성을 위임한다.
  * SEQUENC: 데이터베이스 시퀀스를 사용한다.
  * TABLE: 키 생성 테이블을 사용한다.

기본 키 생성전략을 사용하기 위해서는 `persistence.xml`에 아래와 같은 설정을 추가해야 한다.

```xml
<property name="hibernate.id.new_generator_mappings" value="true" />
```

#### 4.6.1 기본 키 직접 할당 전략

지금까지 사용했던 `@Id`를 사용하는 전략이다. 반드시 코드에서 엔티티를 영속화 하기전에 기본키를 할당해 주어야 한다.

```java
Board board = new Board();
board.setId("id"); // 없으면 에러 발생
em.persist(board);
```

#### 4.6.2 IDENTITY 전략

데이터베이스에 기본 키 생성을 위임한다. 예를들어 MySql의 `AUTO_INCREMNT` 옵션을 통해 테이블의 기본키 자동생성을 사용하는 것이다. 위 자바 엔티티의 기본키로 사용하는 필드에 적용하기 위해서는 추가적으로 어노테이션이 필요하다.

```java
@Entity
public class Board{
    @Id
    @GeneratedValue(strategy = GenerationType.IDNTITY)
    private Long id;
}
```

앞서서 **엔티티의 영속상태에는 식별자가 반드시 필요**하다라고 언급했었다. 하지만 데이터베이스에 기본 키 생성을 위임하는 경우엔 데이터베이스에 저장하기 전까지는 기본 키(즉 식별자)를 알 수 없다. 따라서 IDENTITY 전략을 사용하는 경우엔 트랜잭션을 지원하는 쓰기지연이 동작할 수 없다.

#### 4.6.3 SEQUENCE 전략

시퀀스를 지원하는 Oracle과 같은 DB에서 사용가능한 전략이다. 시퀀스 생성이 선행되어야 한다.

```sql
CREATE TABLE BOARD (
	ID BIGINT NOT NULL PRIMARY KEY
    DATA VARCHAR(255)
)
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```

위와 같이 시퀀스를 데이터베이스에서 생성하면, 이제 JPA가 해당 시퀀스를 매핑할 수 있게 해주어야 한다.

```java
@Entity
@SequenceGenrator(
	name = "BOARD_SEQUENCE_GENERATOR"
	sequenceName="BOARD_SEQ" //DB의 시퀀스 이름과 일치시킨다.
	initialValue =1, allocationSiz=1)
public class Board{
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE
                   	generator = "BOARD_SEQUENCE_GENERATOR")
    private Long id;
}
```

위와 같이 엔티티 클래스 선언 위에 시퀀스 생성기를 위한 어노테이션을 사용해주고, 해당 시퀀스를 DB의 시퀀스와 매핑시킨다. 그리고 사용하기 원하는 필드에서 이를 적용한다. 이제 식별자는 `BOARD_SEQUENCE_GENERATOR`가 생성할 것이다. 시퀀스를 사용하는 전략은 엔티티 영속화 전에 DB의 시퀀스를 사용해서 식별자 값을 조회한다. 그 이후에 엔티티에 식별자를 할당하고 해당 엔티티를 컨텍스트에 저장한다.

##### `@SequenceGenerator`

| 속성                | 기능 | 기본값 |
| ------------------- | ---- | ------ |
| `name`              |식별자 생성기 이름|필수|
| `sequenceName`      | DB의 시퀀스 이름 |`hibernate_sequence`|
| `allocationSize`    | 시퀀스 한번 호출의 증가 값(성능최적화에 사용) | **50** |
| `initialValue`      | DDL 생성시에만 사용 | 1 |
| `catalog`, `schema` | DB catalog, schema 이름 |        |

> `allocationSize` 는 성능 최적화를 위해서, INSERT시마다 시퀀스를 조회하는 것을 피하기 위해서 사용하는 전략이다. 기본 값이 50이기 때문에 만약에 DB의 시퀀스 증가 값이 1이라면, 반드시 1로 설정해 주어야 한다.

#### 4.6.4 TABLE 전략

시퀀스를 지원하지 않는 데이터베이스에서 시퀀스 전략을 흉내낼 수  있는 방법이다. 먼저 키 생성 요오로 사용할 테이블이 필요하다.

```sql
CREATE TABLE MY_SEQUENCES (
	SEQUENCE_NAME VARCHAR(255) NOT NULL,
    NEXT_VAL BIGINT,
    PRIMARY KEY(SEQUENCE_NAME)
)
```

위와 같이 테이블을 만든 경우에 SEQUENCE_NAME 컬럼을 시퀀스 이름으로 사용하고, NEXT_VAL 컬럼을 시퀀스 값으로 사용한다.

```java
@Entity
@SequenceGenrator(
	name = "BOARD_SEQUENCE_GENERATOR"
	sequenceName="MY_SEQUENCES" //DB에 생성한 시퀀스 테이블과 일치시킨다.
	pkColumnValue ="BOARD_SEQ", allocationSiz=1)
public class Board{
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE //테이블 전략 사용
                   	generator = "BOARD_SEQUENCE_GENERATOR")
    private Long id;
}
```

TABLE 전략에서 키 생성 역할을 하는 것은 JPA에서 선언한 `@SequenceGenerator`이다. 테이블에는 속성으로 지정한 이름과, 증가 사이즈가 적용된다. 물론 그 과정속에서는 테이블을 조회하고, 다시 수정하는 데이터베이스와의 통신이 필요하다.

##### `@TableGenerator`

|속성|기능|기본값|
| ---- | ---- | ---- |
|`name`|식별자 생성기명|필수|
|`table`|키생성 테이블명|hibernate_sequence|
|`pkColumnName`|시퀀스 컬럼명|sequence_name|
|`valueColumnName`|시퀀스 값 컬럼명|netx_val|
|`pkColumnValue`|키 이름|엔티티 이름|
|`initialValue`|초기 값, 마지막으로 생성된 값이 기준|0|
|`allocationSize`|한번 호출에 증가하는 수|50|
|`catalog`,`schema`|DB catlog, schema명||
|`uniqueConstraints`(DDL)|자동생성시 제약조건||

#### 4.6.5 AUTO 전략

AUTO전략은 선택한 데이터베이스 방언에 따라서, 자동적으로 전략을 선택해주는 전략이다. 해당 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것이다. 아직 전략이 정해지지 않은 개발 초기단계나 프로토타입 개발시에 사용할 수 있다.

#### 4.6.6 기본 키 매핑 정리

각 전략결로 `em.persist()` 호출 시 일어나는 일을 정리하면 다음과 같다.

* **직접 할당**: `em.persist()` 호출 전 반드시 식별자 값을 코드에서 할당 해야 한다.
* **SEQUENCE**: 데이터베이스에서 식별자 값을 획득 (SELECT) 후 컨텍스트에 저장한다.
* **TABLE**: 시퀀스 생성용 테이블에서 값을 획득 한 후(SELECT + UPDATE)후 컨텍스트에 저장한다.
* **IDENTITY**: 엔티티를 저장(INSERT)후 식별자 값을 획득(SELECT)후 컨텍스트에 저장한다.

### 4.7 필드와 컬럼 매핑

#### `@Column`

엔티티 객체의 필드를 테이블의 컬럼에 매핑한다. `name` 과 `nullable`속성은 흔히 사용되고, 나머지는 잘 사용되지 않는다.

| 속성                      | 기능                                                         | 기본값                      |
| ------------------------- | ------------------------------------------------------------ | --------------------------- |
| `name`                    | 필드와 매핑할 테이블 컬럼명                                  | 객체 필드 명                |
| `insertable`              | 읽기전용 일때만 false옵션을 준다                             | true                        |
| `updatable`               | 읽기전용 일때만 false옵션을 준다                             | true                        |
| `table`                   | 하나의 엔티티를 두개 이상에 매핑할때 사용                    | 현재 클래스가 매핑된 테이블 |
| `nullable`(DDL)           | false로 설정하면 not null                                    | true                        |
| `unique`(DDL)             | `@Table`의 `uniqueConstraints`와 같음.                       |                             |
| `columnDefinition`(DDL)   | 컬럼 정보를 직접 입력한다                                    | JPA가 적절히 생성           |
| `length`(DDL)             | 문자 길이 제약조건 `String`만 사용                           | 255                         |
| `precision`, `scale`(DDL) | 아주 큰 숫자나 정밀한 소수를 다룰때, `BigDecimal`타입에 사용한다. | precision=19, scale=2       |

앞서서 살펴보지 않은 속성별 실제 DDL 생성 예시를 상펴보면 다음과 같다.

##### `columnDefinition`

```java
@Column(columnDefinition = "varchar(100) defulat 'EMPTY'")
private String data;

//DDL
DATA VARCHAR(100) DEFAULT 'EMPTY'
```

##### `precision, scale`

```java
@Column(precision = 10, scale = 2)
private BigDecima cal;

//DDL
CAL NUMERIC(10,2) //H2, PostgreSQL
CAL NUMBER(10,2) // 오라클
CAL DECIMAL(10,2)// MySQL
```

> `@Column` 어노테이션은 필드에 생략이 가능하다. 생략시에는 기본 속성을 가진 `@Column`이 부여된 것으로 판단한다. 단 자바의 기본 타입에는 `null`값이 입력 불가능 하다. 따라서 기본타입 필드에 `@Column`어노테이션을 사용하면 자동 생성된 DDL에서는 `nullable`속성의 기본 값인 `true`가 입력되어서 나중에 문제가 생길 수 있다. 따라서 기본타입 필드에 `@Column`을 사용하는 경우엔 반드시 `nullabe=false`로 지정해주는 것이 안전하다.

#### `@Enumerated`

자바의 `enum`타입을 매핑할 때 사용한다. 속성은 아래와 같다.

| 속성  | 기능                                                         | 기본값             |
| ----- | ------------------------------------------------------------ | ------------------ |
| value | `EnumType.ORDINAL`: `enum`순서를 저장 <br>`EnumType.STRING`: `enum` 이름을 데이터베이스에 저장 | `EnumType.ORDINAL` |

##### `@Enumerated`사용 예

```java
public enum RoleType {
    ADMIN, USER
}
```

위와 같은 `enum` 클래스가 존재할 때,

```java
public class Member {
...

    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    
...
}
```

`enum`을 타입으로 하는 필드에 어노테이션을 사용해 매핑한다.

```java
member.setRoleType(RoleType.ADMIN);
```

위와 같이 `enum`을 사용하여 엔티티의 값을 세팅하면, 데이터베이스 문자가 저장된다. 두가지 저장방식의 차이는 아래와 같다.

* **ORDINAL**: 데이터베이스 저장공간을 절약할 수 있지만, 이미 저장된 `enum`의 순서를 변경할 수 없다.
* **STRING**: 저장된 `enum`순서 변경이나 추가에 안전하지만, 저장되는 데이터 크기가 크다.

얻는 이득에 비해서 `enum`의 값 추가나 순서 변경에 따른 영향도가 더 크기 때문에 `EnumType.String`이 권장된다.

#### 4.7.3 `@Temporal`

날짜 타입(`java.util.Date`, `java.util.Calendar`)를 매핑할 때 사용한다. 속성은 아래와 같다.

| 속성  | 기능                                                         | 기본값 |
| ----- | ------------------------------------------------------------ | ------ |
| value | `TemporalType.DATE`: 날짜, 데이터베이스 **date** 타입과 매핑 <br>`TemporalType.TIME`:  시간, 데이터베이스 **time** 타입과 매핑 <br>`TemporalType.TIMESTAMP`: 날짜와 시간 데이터베이스 **timestamp**타입과 매핑 | 필수   |

##### `@Temporal`을 사용예

```java
@Temporal(TemporalType.DATE)
private Date date;

@Temporal(TemporalType.TIME)
private Date time;

@Temporal(TemporalType.TIMESTAMP)
private Date timestamp;

//DDL
date date,
time time,
timestamp timestamp,
```

`@Temporal`을 생략하면 자바의 `Date`와 가장 유사한 TIMESTAMP로 정의되는데, DB 벤더에 따라서 TIMESTAMP를 지원하지 않더라도 데이터베이스 방언을 이용해서 자동적으로 알맞은 타입으로 생성된다.

#### 4.7.4 `@Lob`

데이터베이스 BLOB과 CLOB 타입과 매핑된다. 매핑하련느 필드의 타입이 문자면, CLOB으로 매핑하고, 나머지는 BLOB으로 매핑한다.

* **CLOB**: `String`,`char[]`,`java.sql.CLOB`
* **BLOB**: `byte[]`, `java.sql.BLOB`

##### `@Lob`사용 예

```java
@Lob
private String lobString;

@Lob
private byte[] lobByte;

//DDL
//오라클
lobString clob,
lobByte blob,

//MySQL
lobString longtext,
lobByte longblob,

//PostgreSQL
lobString text,
lobByte old,
```

##### 4.7.5 `@Transient`

이 어노테이션이 붙은 필드는 매핑하지 않는다. 객체이 임시로 어떤 값을 보관하고 싶을 때 사용한다.

#### 4.7.6 `@Acess`

JPA의 엔티티 데이터 접근방식 지정에 사용된다.

* **필드 접근**(`AcessType.FIELD`): 필드에 직접 접근한다. `private`이어도 접근가능 하다.
* **프로퍼티 접근**(`AcessType.PROPERTY`): 접근자를 사용한다.

`@Access` 어노테이션은 엔티티 클래스 위에 선언할 수 있는데, 만약 사용하지 않으면 `@Id`의 위치를 기준으로 접근방식을 결정한다. `@Id`가 필드에 있으면 필드접근, getter 메소드 위에 있으면 프로퍼티 접근이다. 또한 각 필드마다 `@Acess`어노테이션을 필드나 메소드에 직접 사용하여 해당 값에만 접근방식을 지정할 수도 있다.

### 4.8 정리

* 어노테이션을 사용한 매핑을 사용해서 스키마 자동생성 기능을 사용할 수 있다.
* JPA의 기본키 매핑전략을 사용하면 데이터베이스에 따른 기본키 생성을 제어할 수 있다
* 연관관계에 대한 엔티티 매핑은 다음장에서 살펴본다.
