---
layout: post
title: "Spring Data JPA - Reference Documentation 번역 - 4"
date: 2019-08-10
categories:
---

스프링 부트 레퍼런스를 백기선님 강의를 통해서 보는 것을 종료하고 주말에 이제부터 무엇을 공부할까 하다가 Spring Data JPA reference 문서를 보기로 했다. 그냥 보기만 하면 의미가 없어서 실습을 해야하는데 실습을 하기는 귀찮고 해서 그냥 레퍼런스 문서를 번역하는 수준에서 공부해보기로 한다. 

모든 내용을 다루지는 않고 kotlin 활용과 같이 스킵해도 되는 부분이나 커스터마이징 설정등은 나에게 의미가 없을 것으로 보아서 스킵한다. 오늘은 공통적인 Spring Data Repository에 대해서 살펴보았다.

---

### 4. Working with Spring Data Repositories

Spring Data repository 추상화의 목표는 다양한 영속 저장소를 위한 데이터 접근계층 구현에 뒤따르는 반복적인 코드의 양을 줄이는 것이다.

> Spring Data repository 문서와 개별 모듈
>
> 본 장은 Spring Data repository가 가진 핵심 컨셉과 인터페이스를 설명한다. 본 장에서 제공하는정보는 Spring Data Commons 모듈에서 발췌하였으며, Java Persistoence API(JPA) 모듈의 설정 및 샘플 코드를 사용한다. 개별 프로젝트에서 사용하기 위해서는 XML 네임스페이스 선언이나 타입 상속을해야만 한다. 개별 모듈에 대한 특정한 기능에 대해서 알기를 원하면 해당 장을 참고하라

#### 4.1 Core concepts

Spring Data repository 추상화의 가장 핵심에 위치한 인터페이스는 `Repository`이다. 해당 인터페이스는 도메인 클래스와 도메인클래스에 선언된 ID 타입을 type argument로 한다. 해당 인터페이스는 마커 인터페이스로서 이 기본 인터페이스를 상속하는 다른 인터페이스들과 타입을 개발자가 파악할 수 있도록 한다. `CrudRepository`는 이러한 상속 인터페이스중의 하나로 해당 인터페이스가 다루는 엔티티 클래스에 대한 세련된 CRUD 기능을 제공한다.

```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

  <S extends T> S save(S entity); //Entity를 저장한다.

  Optional<T> findById(ID primaryKey); //주어진 ID로 Entity를 조회한다.

  Iterable<T> findAll(); //모든 Entity를 반환한다.

  long count(); //총 Entity개수를 반환한다.

  void delete(T entity); //주어진 Entity를 삭제한다.

  boolean existsById(ID primaryKey); //Entity 존재 여부를 boolean으로 반환한다. 

  // … 이외 기능 생략
}
```

`CrudRepository`를 상속하는 `PagingAndSortingRepository` 는 CRUD기능에 더해서, 엔티티들을 페이지처리해서 접근하기 쉽도록 하는 기능을 제공한다.

```java
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

예를 들어 한 페이지에 20개의 엔티티가 포함되도록 하여, 두 번째 페이지를 조회하려면 아래와 같이 할 수 있다:

```java
PagingAndSortingRepository<User, Long> repository = // … get access to a bean
Page<User> users = repository.findAll(PageRequest.of(1, 20));
```

이러한 "query method'들에 더하여, `COUNT`나 `DELETE`와 같은 쿼리와 유사한 표현도 가능하다. 아래와같이 직접 `CrudRepository`를 상속하는 인터페이스에 count 쿼리표현을 사용할 수 있다.

```java
interface UserRepository extends CrudRepository<User, Long> {

  long countByLastname(String lastname);
}
```

아래는 삭제 쿼리 표현을 사용하는 예로 `delete`나 `remove`라는 접두어를 메소드에 붙이는 방법으로 사용할 수 있다.

```java
interface UserRepository extends CrudRepository<User, Long> {

  long deleteByLastname(String lastname);

  List<User> removeByLastname(String lastname);
}
```

#### 4.2 Query Methods

기본적인 CRUD기능을 가진 리포지토리는 데이터 저장소에 대한 쿼리들을 가지고 있다. Spring Data를 통해서는 이러한 쿼리들을 만드는 것을 **4단계**를 통해서 할 수 있다.

1. `Repository`나 혹은 하위 인터페이스를 상속받는 타입의 인터페이스를 선언하고, 도메인으로 사용할 클래스와 ID를 type argument로 전달한다.

   ```java
   interface PersonRepository extends Repository<Person, Long> { … }
   ```

2. 쿼리 메소드를 인터페이스 내부에 선언한다.

   ```java
   interface PersonRepository extends Repository<Person, Long> {
     List<Person> findByLastname(String lastname);
   }
   ```

3. 스프링이 선언한 인터페이스에 대한 프록시를 생성할 수 있도록 Java 나 XML을 통해서 설정한다

   1. Java의 경어 config 클래스에 `@EnableJpaRepositories`를 선언한다.

      ```java
      import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
      
      @EnableJpaRepositories
      class Config {}
      ```

   2. XML의 경우엔 아래와 같이 설정해준다.

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:jpa="http://www.springframework.org/schema/data/jpa"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/data/jpa
           https://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
      
         <jpa:repositories base-package="com.acme.repositories"/>
      
      </beans>
      ```

4. 생성한 repository의 인스턴스를 주입받아서 사용한다.

   ```java
   class SomeClient {
   
     private final PersonRepository repository;
   
     SomeClient(PersonRepository repository) {
       this.repository = repository;
     }
   
     void doSomething() {
       List<Person> persons = repository.findByLastname("Matthews");
     }
   }
   ```

자 이제 4단계를 좀더 상세하게 살펴보도록 하자

#### 4.3 Defining Repository Interface

첫째로, 도메인 클래스에 특정한 리포지토리 인터페이스를 정의하자. 정의할 인터페이스는 `Repository`를 반드시 상속해야 하며, 도메인 클래스와 ID 타입으로 타입화 되어야 한다. 만약 해당 도메인 타입에 대한 기본적인 CRUD기능을 사용하고 싶을 때에는 `Repository`보다 `CrudRepository`를 사용하는 것이 낫다.

##### 4.3.1 Fine-tuning Repository Definition

일반적으로, 리포지토리 인터페이스는 `Repository` , `CrudRepository` , `PagingAndSortingRepository`중 하나를 상속하게 된다. Spring Data가 제공하는 인터페이스를 상속받고 싶지 않은 경우, `@RepositoryDefinition`이라는 어노테이션을 붙여서 리포지토리 인터페이스를 만들 수 있다. `CrudRepository`를 상속받게 되면, 엔티티들에 대한 조작을 위한 완전한 형태의 메소드들을 제공하게 된다. 만약 이러한 기능을 선택적으로 제공하고 싶을 때에는 `CrudRepository`의 메소드를 직접 생성한 리포지토리에 복사하면 된다.

아래의 예제가 `Repsitory`를 상속받은 인터페이스가 `CrudRepsitory`의 일부 기능을 선언하는 예이다.

```java
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

> `@NoRepositoryBean` 어노테이션이 붙은 모든 리포지토리 인터페이스는 Spring Data가 런타임에 인스턴스로 생성하지 않는다.

##### 4.3.2 Null Handling of Repository Methods

Spring Data 2.0 이후부터는, 리포지토리의 CRUD 메서드가, 값이 존재하지 않을 가능성을 내포하는 Java 8의 `Optional`을 사용하여 값을 리턴한다. 이외에도 Spring Data는 아래의 wrapper type을 쿼리메소드를 통해 반환하는 기능을 제공한다.

- `com.google.common.base.Optional`
- `scala.Option`
- `io.vavr.control.Option`
- `javaslang.control.Option` (`deprecated`)

또한, 쿼리 메소드는 wrapper 타입을 사용하지 않도록 설정할 수 있다. 이 경우에는 값이 존재하지 않을 경우엔 `null`을 리턴하게 된다. 만약 리포지토리 메서드가 컬렉션이나, 유사 컬렉션, wrapper나 stream을 반환하는 경우엔 절대로 `null`을 반환하지 않고 대신에 해당 타입의 빈 상태를 반환한다.

###### Nullability Annotation

어노테이션을 사용하여서, Null 값에 대한 제약을 리포지토리 메소드에 사용할 수 있다. 아래의 어노테이션들은 사용하기 간편한 opt-in `null` 검사 기능을 런타임에 제공한다.

- [`@NonNullApi`](https://docs.spring.io/spring/docs/5.1.9.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html): 패키지 레벨에 사용되어서, 파라미터들과 반환값들이 `null`을 받거나 생산하지 않는 다는 것을 선언한다.
- [`@NonNull`](https://docs.spring.io/spring/docs/5.1.9.RELEASE/javadoc-api/org/springframework/lang/NonNull.html): 파라미터나 반환값에 사용되어서, 해당 값들이 `null`이 아니어야 함을 명시한다.
- [`@Nullable`](https://docs.spring.io/spring/docs/5.1.9.RELEASE/javadoc-api/org/springframework/lang/Nullable.html): `null`이 될 수 있는 파라미터나 반환 값에 사용된다.

런타임에서 null여부를 체크하는 제한을 쿼리메소드에 사용하기 위해서는, 스프링의 `@NonNullApi`를 패키지 레벨에 아래와 같이 사용하여야 한다.

```java
@org.springframework.lang.NonNullApi
package com.acme;
```

일단 "non-null"이 기본값으로 세팅되면, 리포지토리 쿼리 메소드들은 런타임 시점에 널 여부를 검증하게 된다. 만약 쿼리 실행 결과가 정의된 제약을 위반하게 되면, 예외가 던져진다. 예를들어 메서드가 `null`을 반환하지 않는 것으로 선언된 상태에서 `null`을 반환하는 경우이다. 만약 해당 메소드에만`null`을 사용하기를 다시 원한다면, `@Nullable`을 개별 메소드에 선언하면 된다. 앞서 언급한 wrapper 클래스나 empty 컬렉션들의 반환은 동일하게 동작한다.

위에서 언급한 `null`관련 테크닉을 정리한 아래 예제를 참고하라.

```java
@org.springframework.lang.NonNullApi
package com.acme; //패키지 레벨에 null불가 제약이 선언되었다.                                                   
import org.springframework.lang.Nullable;

interface UserRepository extends Repository<User, Long> {

  User getByEmailAddress(EmailAddress emailAddress);
  //만약 위 메서드가 아무런 값을 반환하지 않으면, EmptyResultDataAccessException이 발생한다.
  //만약 eamilAddress 값으로 null이 전달되면, IllegalArgumentException이 발생한다.

  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAdress);
  //만약 위 메서드가 아무런 값을 반환하지 않으면, null이 반환된다.
  //emailAddress로 null이 전달 될 수 있다.

  Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
  //만약에 위 메서드가 아무런 값을 반환하지 않으면, Optional.empty()가 반환된다.
  //만약에 emailAddress가 null이면 IllegalArgumentException이 발생한다.
}
```

##### 4.3.3 Using Repositories with Multiple Spring Data Modules

하나의 어플리케이션에서 하나의 Spring Data 모듈만 사용할 경우 모든 리포지토리 인터페이스가 해당 모듈의 종속적이 되기 때문에 간단하다. 때때로, 어플리케이션이 하나 이상의 Spring Data 모듈을 사용할 수가 있다. 이경우에, 리포지토리는 영속화 기술에 비 종속적으로 정의 되어야만 한다. Spring Data가 하나 이상의 리포지토리 팩토리를 감지하게 될 경우, **엄격한 설정**모드에 돌입하게 된다. 해당 모드에서는 리포지토리의 상세와 도메인 클래스를 바탕으로 리포지토리와 연동할 스프링 모듈을 결정한다.

1. 만약 리포지토리 정의가 특정 모듈의 리포지토리를 상속할 경우, 해당 특정 모듈의 연결된다.
2. 만약 도메인클래스가 특정 모듈의 국한된 어노테이션을 사용할 경우, 해당 특정 모듈에 연결된다. Spring Data 모듈은 서드파티 어노테이션이나 자체 어노테이션을 모두 취급한다.

아래의 예제가 특정 모듈 인터페이스를 보여준다. (이 경우 JPA)

```java
interface MyRepository extends JpaRepository<User, Long> { }

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
  …
}

interface UserRepository extends MyBaseRepository<User, Long> {
  …
}
```

위의 예제의 `MyRepository`와 `UserRepository`는 `JpaRepository`를 상속받고 있다. 따라서 이 둘은 Spring Data JPA 모듈의 리포지토리가 된다.

아래의 예제는 제네릭 인터페이스를 사용하는 리포지토리 예제이다.

```java
interface AmbiguousRepository extends Repository<User, Long> {
 …
}

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
  …
}

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> {
  …
}
```

위의 예제처럼 특정 모듈에 종속되지 않은 `Repository`와 `CrudRepository`를 상속받는 경우엔, 하나의 Spring Data 모듈만 사용할 경우엔 전혀 문제가 되지 않는다. 그러나 만약 다중 모듈을 사용하게 되면 위 예제의 `AmbiguousRepository`와 `AmbiguousUserRepository`는 어떠한 모듈과 바인딩 될지가 결정될 수 없다.

아래의 예제는 어노테이션을 사용한 도메인 클래스를 사용하는 리포지토리 예제이다.

```java
interface PersonRepository extends Repository<Person, Long> {
 …
}

@Entity
class Person {
  …
}

interface UserRepository extends Repository<User, Long> {
 …
}

@Document
class User {
  …
}
```

`Person`의 경우 JPA 어노테이션인 `@Entity`를 사용하고 있기 때문에, 이를 타입으로 받는 `PersonRepository`는 Spring Data JPA 모듈을 사용하게 된다. 만약 `UserRepository`가 `User`를 타입으로 받는 경우엔 해당 도메인이 Spring Data MongoDB의 `@Document` 어노테이션을 사용하고 있으므로 MongoDB에 바인딩 된다.

아래의 예제는 혼합된 어노테이션을 도메인 클래스에 사용하는 잘못된 예제이다.

```java
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}

@Entity
@Document
class Person {
  …
}
```

위의 예제는 `Person` 클래스가 JPA와 Spring Data MongoDB 어노테이션을 동시에 사용하고 있다. Spring Data는 더이상 리포지토리를 분간할 수 없기 때문에, 어떻게 동작할지 보장할 수 없다.

리포지토리 타입에 대한 상속정보와 도메인 클래스의 어노테이션 정보는 리포지토리가 어떠한 Spring Data 모듈과 연결될지를 결정하는 중요한 설정이다. 하나의 도메인 타입에 여러 모듈의 어노테이션을 사용하는 것은 가능하고 해당 도메인 타입을 다중의 영속화 기술에 사용하는 것은 가능하지만, 리포지토리가 어떠한 모듈에 종속될지를 Spring Data가 결정하지 못한다.

리포지토리를 구별하는 마지막 방법은 기본 패키지에 설정하는 것이다. 기본 패키지는 리포지토리 인터페이스 정의를 스캔하는 시작점으로 아래와 같이 설정할 수 있다.

```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
interface Configuration { }
```

#### 4.4 Defining Query Methods

리포지토리 프록시가 저장소에 특화된 쿼리를 메소드 이름에서 유도하는데에는 두 가지의 방법이 있다.

* 메소드 이름에서 직접적으로 유도한다
* 직접 작성한 쿼리를 사용한다.

실제 저장소에 따라 가능한 선택지가 달라지지만, 어떠한 쿼리가 생성되느냐에 대한 전략이 반드시 존재한다. 이제부터 가능한 선택지들을 살펴보도록 하자

##### 4.4.1 Query Lookup Strategies

쿼리를 분석하기 위한 가능한 전략 선택지는 세가지가 존재한다. XML 설정을 통해서는 `query-lookup-stratgy` 어트리뷰트에 명시하는 방법으로 전략을 선택할 수 있으며, 자바 설정에서는 `Enable*Repositories` 어노테이션의 어트리뷰트로 `queryLookupStratgy`를 넘겨주어서 설정할 수 있다. 일부 설정의 경우 저장소에 따라 사용하지 못할 수도 있다.

* `CREATE` 의 경우엔 쿼리 메소드 이름을 통하여서 저장소에 특화된 쿼리를 만들어낸다. 주어진 메소드 이름에서 잘 알려진 접두사를 제거하고, 나머지 부분을 파싱하는 방법이 일반적으로 쓰인다. 해당 내용은 4.4.2 Section에서 좀더 자세히 살펴볼 수 있다.
* `USE_DECLARED_QUERY`의 경우엔 선언된 쿼리를 찾고, 없다면 예외를 반환한다. 어노테이션을 통해 소스 어딘가에 선언되거나, 또한 다른방식으로 선언될 수 있다. 
* `CREATE_IF_NOT_FOUND`(기본값) 위의 두 전략이 섞인 것으로, 먼저 선언된 쿼리를 찾고 없는 경우에 이름 기반으로 쿼리를 만들어준다. 

##### 4.4.2 Query Creation

Spring Data 리포지토리 인프라스터럭쳐에 내장된 쿼리 생성 메커니즘은 해당 리포지토리의 엔티티를 관리하는 쿼리를 생성하는데 매우 유용한다. 먼저 메서드 이름이 가진 잘 알려진 접미사들 (`find..By`, `read..By`, `query..By`, `get..By`)를 벗겨낸뒤에, 나머지 부분을 파싱하기 시작한다. 이어지는 부분은 `Distinct`와 같은 추가적인 표현들을 포함할 수 있다. 다만, `first..By`의 경우엔 해당 기준으로 나오는 엔티티의 시작 값을 의미한다. 가장 기본적인 수준에서 엔티티의 속성에 대한 상태를 정의할 수 있으며, 이러한 정의를 `And` 와 `Or`로 연결할 수 있다. 아래의 예제가 몇 가지의 예제를 보여준다.

```java
interface PersonRepository extends Repository<User, Long> {

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // 쿼리에 Dinstinct 플래그를 넣어준다.
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // 특정 프로퍼티의 상태를 조회할 때 대소문자를 무시한다.
  List<Person> findByLastnameIgnoreCase(String lastname);
  // 모든 프로퍼티에 상태 조회에 대해서 대소문자를 무시한다.
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // 정렬된 결과를 도출한다. order by절
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```

실제 메소드를 통해 유도되는 쿼리는 쿼리를 실행하는 저장소에 따라 달라진다. 다만 공통적으로 적용되는 부분이 있다:

* 메소드 표현들은 일반적으로 엔티티의 프로퍼티를 다루고, 연산자(`AND`,`OR`)를 통해 연결하는 방식으로 사용한다. 또한 `Between` , `LessThan`, `GreaterThan` , `Like`과 같은 표현도 사용할 수 있다. 이러한 표현들은 데이터 저장소에 따라 달라지기 때문에, 해당 모듈의 레퍼런스 문서를 참고하는게 좋다.
* `IgnoreCase` 플래그의 경우 개별 프로퍼티에 적용되거나, `*AllIgnoreCase`와 같이 사용해서 모든 프로퍼티에 사용할 수 있다. 이 경우에도 데이터 저장소에 따라 달라지기 때문에 각 모듈의 쿼리메소드 section을 참고하라
* `OrderBy`를 추가하여 정렬을 할 수 있는데, 여기에 `Asc`혹은 `Desc`를 붙여서 정렬 방식을 정의할 수 있다. 동적인 정렬을 위해서는 4.4.4 section을 참고하라.

##### 4.4.3 Property Expressions

프로퍼티에 대한 표현은 리포지토리에서 관리하는 엔티티에 대한 표현으로 국한된다. 쿼리가 만들어지는 시점에, 해당 도메인 클래스에 선언된 프로퍼티만 파싱된 프로퍼티가 될 수 있도록 해야한다. 그러나 객체가 프로퍼티라면 해당 객체안의 프로퍼티를 사용할 수도 있다. 아래의 예제를 보자

```java
List<Person> findByAddressZipCode(ZipCode zipCode);
```

`Person` 클래스는 `Address`라는 클래스를 프로퍼티로 가지고 있고, `Address`안에 `ZipCode`가 프로퍼티로 존재한다. 이경우에는 위의 쿼리를 사용하면 `x.address.zipCode`를 조건에 사용하여 검색하게 된다. 실제로는 `AddressZipCode`라는 이름의 프로퍼티를 먼저 찾게되고, 만약 발견되면 그 프로퍼티를 사용한다. 만약에 발견되지 않으면 이를 카멜표기법 기준으로 분리한 뒤에 찾게 된다.

대부분의 경우엔 이 방식이 통할 수 있지만 이 알고리즘이 원하지 않는 방향으로 작동할 수도 있다. 마약 `Person` 클래스가 `addressZip`이라는 프로퍼티를 가지는 경우, 해당 프로퍼티 내부에서 `code`라는 프로퍼티를 찾을 것이기 때문에 실패할 것이다.

이러한 모호성을 피하기 위해서 카멜표기법에 `_`를 추가하는 방법으로 명시적으로 탐색포인트를 정할 수 있다. 이경우 아래와같이 표기하게 된다.

```java
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

`_` 문자를 예약어와 준하게 사용하기 때문에, 나머지의 경우엔 철저하게 Java 명명법을 따르기를 권고한다. 따라서 프로퍼티 이름으로는 절대로 `_`를 사용하지말고 카멜표기법만 사용해야 한다.

##### 4.4.4 Special parameter handling

쿼리의 파라미터와 정의된 메서드의 파라미터를 다루는 것에 대해서는 앞선 예제들에서 이미 살펴봤다. 이와 별개로 `Pageable`과 `Sort`가 동적인 페이징 처리와 정렬을 위해서 사용될 수 있다. 

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

첫 번째 메서드는 `Pageable` 인스턴스를 쿼리메서드에 전달해서, 동적으로 페이징 기법을 정적으로 정의된 쿼리에 전달한다. `Page`객체는 해당 요소가 가진 전체의 페이지를 알고 있다. 전달된 `Pageable` 객체를 통해서 전체 갯수를 세는 쿼리를 동작시켜 얻어내는 방식을 사용하기 때문에, 이에 따른 비용증가가 발생하기 때문에 `Slice`를 대신 사용할 수 있다. `Slice`의 경우엔 다음 요소가 존재하는 지만을 알고 있기 때문에 큰 결과를 다루는데에는 좀더 나을 수 있다.

정렬 옵션 또한 `Pageable` 인스턴스를 통해 처리되는데, 만약 정렬만을 사용해할 경우에는 `Sort`를 대신 사용할 수도 있다. 이 경우엔 전체 쿼리가 정해진 범위의 엔티티만을 살펴보도록 한다.

##### 4.4.5 Limiting Query Results

`first` 혹은 `top` 키워드를 통해 쿼리 메소드의 결과 값 갯수를 제한할 수 있다. 또한 해당 키워드에 숫자를 더하게 되면 한 개 이상의 값을 반환하도록 할 수 있다. 기본적으로 숫자를 사용하지 않으면 한 개를 요청한 것으로 해석한다.

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

제한 표현법은 `Dinstinct` 키워드와 동시에 사용할 수 있고, `Optional`을 사용해서 결과값을 감싸도록 할 수도 있다. 

##### 4.4.6 Streaming query results

Java 8에서 지원하는 `Stream<T>`을 리턴 타입으로 하는 경우에 증가하는 방식으로 쿼리를 처리할 수 있다. 

```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```

##### 4.4.7 Async query results

리포지토리 쿼리는 비동기적으로 실행 될 수 있다. 따라서, 스프링의 `TaskExecutor`에서 실제 쿼리의 실행이 된 직후에 메서드 결과가 반환된다. 비동기 쿼리 실행은 Reactive한 쿼리 실행과 다르며, 동시에 사용되어서는 안된다.

```java
@Async
Future<User> findByFirstname(String firstname);               

@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

#### 4.5 Creating Repository Instances

본 절에서는, 정의된 리포지토리에 대한 인스턴스와 빈 정의를 생성한다. 스프링이 제공하는 설정기능을 통해 스프링 데이터 모듈을 불러오는 방식으로 실행되는데, Java Configuration을 사용하는 방법을 추천한다.

##### 4.5.2 JavaConfig

각 저장소에 따라 달라지는 `@Enable*Repositories`어노테이션을 통해서 리포지토리 인프라스트럭쳐를 활성화 시킬 수 있다. 

```java
@Configuration
@EnableJpaRepositories("com.acme.repositories")
class ApplicationConfiguration {

  @Bean
  EntityManagerFactory entityManagerFactory() {
    // …
  }
}
```

##### 4.6.2. Customize the Base Repository

모든 리포지토리들에 대한 동작에 대한 변조를 하고 싶을 경우엔, 영속화 기술에 특정하는 리포지토리의 기본 클래스를 구현하는 방법으로 할 수 있다. 이렇게 구현된 클래스는 리포지토리 프록시의 기본 클래스로 동작하게 된다.

```java
class MyRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> {

  private final EntityManager entityManager;

  MyRepositoryImpl(JpaEntityInformation entityInformation,
                          EntityManager entityManager) {
    super(entityInformation, entityManager);

    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }

  @Transactional
  public <S extends T> S save(S entity) {
    // implementation goes here
  }
}
```

위와 같이 기본 클래스를 생성한 뒤에는, Spring Data 인프라가 해당 클래스를 기본 클래스로 사용하게 해주어야 한다. 아래와 같이 JavaConfig에 명시해주면 된다.

```java
@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```