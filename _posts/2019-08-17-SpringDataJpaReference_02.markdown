---
layout: post
title: "Spring Data JPA - Reference Documentation 번역 - 5"
date: 2019-08-17
categories:
---
### 5. JPA Repositories

본 장에서는 JPA향 리포지토리의 특수성에 대해서 살펴본다. 본 장의 내용은 앞서 살펴본 "Working with Spring Data Repositories"를 기반으로 하고 있기 때문에, 해당 부분에서 설명한 핵심 컨셉에 대한 이해가 반드시 필요하다.

#### 5.1 Introduction

##### 5.1.2 Annotation-based Configuration

Spring Data JPA 리포지토리는 JavaConfig를 이용해서 활성화 될 수 있다.

```java
@Configuration
@EnableJpaRepositories
@EnableTransactionManagement
class ApplicationConfig {

  @Bean
  public DataSource dataSource() {

    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    return builder.setType(EmbeddedDatabaseType.HSQL).build();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setGenerateDdl(true);

    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
    factory.setJpaVendorAdapter(vendorAdapter);
    factory.setPackagesToScan("com.acme.domain");
    factory.setDataSource(dataSource());
    return factory;
  }

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

    JpaTransactionManager txManager = new JpaTransactionManager();
    txManager.setEntityManagerFactory(entityManagerFactory);
    return txManager;
  }
}
```

상기한 예제 클래스는 `spring-jdbc`의 `EmbeddedDatabaseBuilder` API를 사용하여서 내장 HSQL 데이터베이스와 연결한다. 그후 Spring Data 는 `EntityManagerFactory`를 만들고, Hibernate를 영속화 제공자로 사용한다. 마지막에 선언된 인프라 콤포넌트는 `JpaTransactionManager`이다. 마지막으로, 위 예제는 Spring Data 리포지토리들을 `@EnableJpaRepositories` 어노테이션을 통해 활성화 한다. 

##### 5.1.3 Bootstrap Mode

기본적으로, Spring Data JPA 리포지토리들은 스프링 Bean으로 싱글톤이며 초기화된 상태이다. 어플리케이션 기동 시점에서 해당 Bean들은 이미 JPA `EntityManager`들과 상호작용을 통해 검증과 메타정보 분석을 한다. Spring Framework는 JPA `EntityManagerFactory`를 백그라운드 스레드에서 초기화하는 것을 지원하는데, Spring Application을 기동하는 시간 중에 많은 부분을 차지하기 때문이다. 백그라운드 초기화를 효과적으로 하기 위해서 JPA 리포지토리들이 최대한 늦게 초기화 되도록 할 필요가 있다.

Spring Data JPA 2.1 부터, `BootstrapMode`를 아래와 같이 설정할 수 있다.

* `DEFAULT` (기본값) - 리포지토리들은 `@Lazy` 어노테이션이 붙지 않는 이상 최대한 빠르게 인스턴스화된다. 지연효과는 해당 리포지토리를 Bean으로 필요하는 다른 Bean이 없을 때만 효과가 있다.
* `LAZY` - 모든 리포지토리 bean들을 지연시키고, 초기화 지연 프록시들을 만들어 클라이언트 bean 주입한다. 만약 클라이언트 bean들이 단순히 리포지토리 bean을 필드에 저장만 하고 사용하지 않는 경우엔 리포지토리 bean들이 초기화 되지 않는 것을 의미한다. 해당 리포지토리와의 상호작용이 일어나는 순간에 리포지토리 bean은 초기화 될 것이다.
* `DEFERRED` - 근본적으로 `LAZY`와 동일한 모드이지만, 리포지토리의 초기화가 `ContextRefreshedEvent`의 응답이 이뤄질때 일어난다. 따라서 모든 리포지토리는 어플리케이션이 완전히 시작되기 전에 모두 검증된다.

#### 5.2 Persisting Entities

본 절에서는 Spring Data JPA에 entity를 영속화(저장) 하는지 알아본다

##### 5.2.1 Saving Entities

엔티티의 저장은 `CrudRepository.save(...)` 메서드를 통해서 할 수 있다. 해당 메서드는 주어진 엔티티를 기저에 있는 JPA `EntityManager`를 통해 영속화 하거나 머지한다. 만약 엔티티가 영속화 된 상태가 아니면, Spring Data JPA 는 `entityManager.persist(...)`를 호출해서 저장한다. 만약에 저장된 상태라면 `entityManager.merge(...)`를 호출한다.

###### Entity State-detection Strategy

Spring Data는 엔티티가 신규 엔티티인지 아닌지를 구별하기 위해 아래의 전략을 사용한다

* **Id-Property 조사** (**기본값**): 기본적으로 Spring Data JPA는 주어진 엔티티의 식별자 프로퍼티를 조사한다. 만약 해당 식별자 프로퍼티가 `null`이면, 엔티티는 새로운 것으로 간주된다. 
* **`Persistable`구현**: 만약에 엔티티가 `Persistable`을 구현하고 있을 경우 Spring Data JPA는 `isNew(...)` 메서드를 호출하여서 신규 엔티티 여부를 검증한다.

#### 5.3 Query Methods

본 절에서는 Spring Data JPA를 통해서 어떻게 쿼리를 만드는지 살펴본다.

##### 5.3.1. Query Lookup Strategies

JPA 모듈은 수동으로 `String` 형태로 작성된 쿼리나 메서드 이름을 통해서 유도한 쿼리 정의를 지원한다.

`IsStartingWith`, `StartingWith`, `StartsWith`, IsEndingWith", `EndingWith`, `EndsWith`, `IsNotContaining`, `NotContaining`, `NotContains`, `IsContaining`, `Containing`, `Contains`와 같은 제한자를 가진 유도 쿼리들에게 주어지는 문자열은 정제될 수 있는데, 예를들어 `LIKE` 구문에 와일드카드로 주어지는 문자들이 무시된다는 의미이다. 이스케이프 문자의 경우 `@EnableJpaRepositories` 어노테이션의 `escapeCharacter` 어트리뷰트를 통해서 세팅할 수 있다.

###### Decalred Queries

메서드 이름을 통해 유도되는 쿼리는 매우 편리하지만, 떄로는 메서드 이름 파싱이 원하는 키워드를 지원하지 않거나, 메서드 이름이 지저분해지는 결과를 얻을 수 있다. 따라서 네이밍 기법을 사용하거나 쿼리 메서드에 `@Query` 어노테이션을 사용할 수도 있다.

##### 5.3.2 Query Creation

일반적으로 앞선 장의 "Query methods" 에서 설명한 대로 JPA 에서도 쿼리 생성 매커니즘은 동작한다.

```java
public interface UserRepository extends Repository<User, Long> {

  List<User> findByEmailAddressAndLastname(String emailAddress, String lastname);
}
```

위의 쿼리 메서드는 `select u from User u where u.emailAddress = ?1 and u.lastname = ?2`로 번역되어서 실행되게 된다. Spring Data JPA는 앞서서 "Property Expression"에서 설명한대로 프로퍼티를 확인한다.

아래의 표는 JPA에서 지원하는 메서드의 이름이 어떻게 쿼리 키워드로 변환되는 가를 보여준다.

| Keyword             | Sample                                                       | JPQL snippet                                                 |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `And`               | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is,Equals`         | `findByFirstname`,`findByFirstnameIs`,<br>`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`           | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`          | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`     | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`       | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`  | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`             | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`            | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`            | `findByAgeIsNull`                                            | `… where x.age is null`                                      |
| `IsNotNull,NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`              | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`           | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`      | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`        | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`        | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`           | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`               | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`             | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`              | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`             | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`        | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

##### 5.3.3 Using JPA Named Queries

본절의 예제에서는 `@NameQuery` 어노테이션을 사용하는데, 이 경우에는 JPA query 언어를 사용해야만 한다. 만약 SQL문법을 사용하고 싶을 경우엔 `@NamedNativeQuery`를 사용할 수 있다. 다만 이경우에는 네이티브 SQL을 사용하는 대신에 벤더 독립성을 잃어버리게 된다.

###### Annotation-based Configuration

어노테이션 기반 JPA Named 쿼리 사용은 별도의 설정 파일을 가지지 않아도 되기 때문에 XML설정에 비해서 장점이 있다. 다만 도메인 클래스에 어노테이션을 통해 정의하기 때문에 쿼리의 변경에 따라 매번 컴파일 해주어야 하는 단점이 있다.

```java
@Entity
@NamedQuery(name = "User.findByEmailAddress",
  query = "select u from User u where u.emailAddress = ?1")
public class User {

}
```

Named 쿼리를 사용하기 위해서는 아래와 같이 리포지토리 인터페이스에 정의해주면 된다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  List<User> findByLastname(String lastname);

  User findByEmailAddress(String emailAddress);
}
```

Spring Data는 위 예제의 두번째 메서드를 보고 엔티티 클래스에 정의된 `@NamedQuery` 어노테이션을 찾아서 해당 쿼리를 실행한다.

##### 5.3.4 Using `@Query`

Named 쿼리를 `@Query` 어노테이션을 사용해서 메서드에 종속시키는 것도 가능하다. 이 경우 도메인 클래스를 영속화 정보와 분리할 수 있고, 쿼리를 리포지토리 인터페이스에 종속시킬 수 있는 장점이 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);
}
```

###### Using Advanced `LIKE` Expression

`@Query`를 이용한 쿼리 정의는 `LIKE` 표현법도 아래와 같이 지원한다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.firstname like %?1")
  List<User> findByFirstnameEndsWith(String firstname);
}
```

###### Native Queries

`@Query` 어노테이션의 `nativeQuery` 속성을 `true`로 하면 네이티브 SQL 쿼리도 실행할 수 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
}
```

##### 5.3.5. Using Sort

쿼리 결과에 대한 정렬을 하고 싶다면, `PageRequest` 혹은 `Sort`를 직접 사용해서 할 수 있다. `Sort`의 `Order`에서 사용한 프로퍼티가 실제 도메인 모델과 일치해야 하는데, 해당 프로퍼티가 쿼리에서 사용한 프로퍼티와 일치해야 한다. 

그러나, `Sort`를 `@Query`와 함께 사용할 경우에는 `ORDER BY`문에서 함수를 사용할 수 있게 해주는데, `Order` 인스턴스 대신에, `JpaSort.unsafe`를 사용하면 된다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.lastname like ?1%")
  List<User> findByAndSort(String lastname, Sort sort);

  @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
  List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);
}

repo.findByAndSort("lannister", new Sort("firstname"));     			//1        
repo.findByAndSort("stark", new Sort("LENGTH(firstname)"));     		//2      
repo.findByAndSort("targaryen", JpaSort.unsafe("LENGTH(firstname)")); 	//3
repo.findByAsArrayAndSort("bolton", new Sort("fn_len"));				//4
```

1. 도메인에 존재하는 `firstname`을 파라미터로 넘겨주어서 정상이다
2. 함수를 넘겨줬으므로 Exception을 발생시킨다.
3. `unsafe` 한 `Order`를 사용했으므로 정상이다.
4. `@Query`에 정의된 alias를 넘겨줬으므로 정상이다.

##### 5.3.6. Using Named Parameters

기본 적으로 Spring Data는 위치-기반 파라미터 바인딩을 사용하는데, 이 방법은 파라미터의 위치를 리팩토링할때에 에러가 발생하기 쉽다. 이 이슈를 해결하기 위해서 `@Param` 어노테이션을 활용해서 메소드에 사용한 파라미터에 정확한 이름을 부여해서 `@Query`에서 사용할 수 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
}
```

##### 5.3.7. Using SpEL Expressions

Spring Data JPA 1.4부터 `@Query`에서 제한된 SpEL 템플릿 표현식을 지원한다. Spring Data는 `entityName`이라는 변수를 사용하는데, `select x from #{#entityName} x`처럼 사용할 수 있다. `entityName`의 자리에 해당 리포지토리에 타입 파라미터로 넘겨진 도메인을 치환시킨다.

```java
@Entity
public class User {

  @Id
  @GeneratedValue
  Long id;

  String lastname;
}

public interface UserRepository extends JpaRepository<User,Long> {

  @Query("select u from #{#entityName} u where u.lastname = ?1")
  List<User> findByLastname(String lastname);
}
```

이렇게 하면 향후 엔티티 이름이 변경되는 경우에도 대응할 수 있다.

`#{#entityName}`을 활용할 수 있는 또다른 예시는, 제네릭을 사용하는 리포지토리의 경우이다.

```java
@MappedSuperclass
public abstract class AbstractMappedType {
  …
  String attribute
}

@Entity
public class ConcreteType extends AbstractMappedType { … }

@NoRepositoryBean
public interface MappedTypeRepository<T extends AbstractMappedType>
  extends Repository<T, Long> {

  @Query("select t from #{#entityName} t where t.attribute = ?1")
  List<T> findAllByAttribute(String attribute);
}

public interface ConcreteRepository
  extends MappedTypeRepository<ConcreteType> { … }
```

위의 예제에서 `MappedTypeRepository`는 `AbstractMappedType`을 상속하는 몇가지의 도메인의 부모 인터페이스 역할을 하게 된다. 또한 내부에 정의된 `findAllByAttribute(...)`는 구체화된 하위 리포지토리에서 사용될 수 있다. 따라서 `ConcreteRepository`에서 해당 메서드를 호출하면 `select t from ConcreteType t where t.attribute =?1`이 실행된다.

##### 5.3.8. Modifying Queries

앞서 설명한 모든 예제들은 엔티티들에 접근하기 위한 쿼리를 선언하는 예제들이었다. `@Modifying` 어노테이션을 사용하면 update 쿼리를 작성할 수 있다.

```java
@Modifying
@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
int setFixedFirstnameFor(String firstname, String lastname);
```

###### Derived Delete Queries

Spring Data JPA는 메소드 이름으로 유도된 삭제 쿼리를 제공하여서, JPQL로 아래와같이 명시하지 않아도 되게 해준다.

```java
interface UserRepository extends Repository<User, Long> {

  void deleteByRoleId(long roleId);

  @Modifying
  @Query("delete from User u where user.role.id = ?1")
  void deleteInBulkByRoleId(long roleId);
}
```

위의 예제의 쿼리와 `deleteByRoleId(...)`는 동일한 결과를 낼 것 같이 보이지만, 매우 중요한 차이가 존재한다. 예제의 쿼리는 데이터베이스에 오직 하나의 쿼리를 실행하는데, 이경우 로드된 `User` 인스턴스의 라이프사이클 콜백을 실행하지 못하게 된다. 따라서 라이프사이클 쿼리가 실행되길 원한다면 `deleteByRoleId(...)`를 실행해야 한다. 

##### 5.3.9. Applying Query Hints

리포지토리에 선언된 쿼리에 힌트를 적용하기 위해서, `@QueryHints` 어노테이션을 사용할 수 있다. 해당 어노테이션의 어트리뷰트로는 `@QueryHint` 어노테이션과 페이징 처리에 사용하는 추가 카운트 쿼리에 힌트를 사용할지 여부를 결정하는 불리언 값이 제공할 수 있다.

```java
public interface UserRepository extends Repository<User, Long> {

  @QueryHints(value = { @QueryHint(name = "name", value = "value")},
              forCounting = false)
  Page<User> findByLastname(String lastname, Pageable pageable);
}
```

위의 예제는 실제 쿼리에는 `@QueryHint`를 적용하지만, 총 페이지를 계산하는 카운트 쿼리에는 힌트를 적용하지 않는다.

##### 5.3.11. Projections

Spring Data 쿼리 메서드는 일반적으로 리포지토리가 관리하는 도메인의 하나 혹은 다수의 인스턴스를 리턴한다. 하지만, 때로는 해당 타입들의 특정 어트리뷰트만을 목표로 하는 것이 바람직한 경우도 있다. Spring Data는 이러한 도메인의 일부분많을 선별하는 모델링을 제공한다.

아래와 같은 예제가 있다고 할때,

```java
class Person {

  @Id UUID id;
  String firstname, lastname;
  Address address;

  static class Address {
    String zipCode, city, street;
  }
}

interface PersonRepository extends Repository<Person, UUID> {

  Collection<Person> findByLastname(String lastname);
}
```

자 이제, `Person`에서 이름 속성만을 탐색하기 원한다고 해보자. 이를 위해 Spring Data는 어떠한 도구를 제공하는지 본 장의 나머지 부분을 통해 설명한다.

###### Interface-based Projections

가장 손쉬운 방법은 아래의 예제 처럼 읽으려는 프로퍼티만에 접근하는 인터페이스를 선언하는 것이다.

```java
interface NamesOnly {

  String getFirstname();
  String getLastname();
}
```

여기서 주목할 점은 상기 인터페이스에 정의된 프로퍼티와 목표로하는 도메인 프로퍼티가 정확히 일치한다는 것이다. 이렇게 함으로서 아래와 같은 쿼리메서드를 작성할 수 있게 된다.

```java
interface PersonRepository extends Repository<Person, UUID> {

  Collection<NamesOnly> findByLastname(String lastname);
}
```

쿼리 실행 엔진은 리턴되는 각각의 엘리먼트를 위해, 앞서 선언한 인터페이스의 프록시 인스턴스를 런타임시에 생성하고, 해당 인스턴스에 대한 메서드 호출을 포워딩 해준다.

이 구조는 재귀적으로도 이뤄질 수 있는데, 만약 `Person` 객체안의 `Address` 객체중 일부만 포함시키고 싶다면, 아래와 같이 해주면 된다.

```java
interface PersonSummary {

  String getFirstname();
  String getLastname();
  AddressSummary getAddress();

  interface AddressSummary {
    String getCity();
  }
}
```

###### Class-based Projections (DTOs)

또 다른 방법은 탐색에 필요한 프로퍼티를 가진 DTOs 타입을 사용하는 것이다. 이러한 DTOs들은 인터페이스를 통한 방법과 동일하게 사용할 수 있지만, 프록시를 사용하지 안흔ㄴ 다는 점이 다르다.

```java
class NamesOnly {

  private final String firstname, lastname;

  NamesOnly(String firstname, String lastname) {

    this.firstname = firstname;
    this.lastname = lastname;
  }

  String getFirstname() {
    return this.firstname;
  }

  String getLastname() {
    return this.lastname;
  }

  // equals(…) and hashCode() implementations
}
```



