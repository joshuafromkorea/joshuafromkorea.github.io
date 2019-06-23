---
layout: post
title: "Java 8 in Action Chapter05 - 스트림 활용"
date: 2019-06-15
categories:
---

4장에서 소개한 스트림의 연산들에 대한 활용을 다룬다, 이미 예제코드를 작성해보고, 일주일이나 지난 오늘(23일)에나 되서야 내용을 책을 보면서 다시 정리했다. 람다보다도 스트림이 더 실무에서 더 활용할일이 많은 것처럼 보인다. 점점 자바 8을 이해하는 개발자가 되가고 있다.

- [**공부한 내용 Github 에서 보기**](https://github.com/joshuafromkorea/Java_8_In_Action_Study/tree/85c0465d8b47e666137bc23097bb804f7c3cb477)

---

## Chapter 5 스트림 활용

> 스트림 API가 지원하는 연산을 이용해서 필터링, 슬라이싱, 매핑, 검색, 매칭, 리듀싱 등 다양한 데이터 처리 질의를 표현할 수 있다.

### 5.1 필터링과 슬라이싱

#### 5.1.1 프레디케이트로 필터링

* `filter()` 메서드는 인수로 `Predicate<>`타입을 받는데, 이는 `boolean`을 반환하는 함수를 말한다. 
* 아래는 채식메뉴를 반환하는 소스코드로, `Dish.isVegeterian` 함수를 통해 반환 받은 `boolean`을 사용한다.

```java
@Test
public void 프레디케이트_필터링(){
    List<Dish> vegetarianMenu = menu.stream()
            .filter(Dish::isVegeterian)
            .collect(toList());
    assertThat(vegetarianMenu.size()).isEqualTo(3);
}
```

#### 5.1.2 고유 요소 필터링

* `distinct()` 메서드는 객체 내부의 `hashCode()`, `equals()`로 객체의 중복여부를 판단하고 필터링한다.

```java
@Test
public void 고유요소_필터링_짝수(){
    List<Integer> numbers = Arrays.asList(1,2,2,4,5,6,8,8,9,10);
    List<Integer> evenNumbers = numbers.stream()
            .filter(i->i%2==0)
        	.distinct() //2와 8이 중복되어 각자 하나씩 남게 된다.
            .collect(toList());
    assertThat(evenNumbers.size()).isEqualTo(5);
}
```

#### 5.1.3 스트림 축소

* `limit(n)` 메서드는 인수로 받은 n개의 개수로 스트림을 선착순으로 축소한다.

```java
@Test
public void 스트림_축소(){
    List<Dish> firstThreeMenus = menu.stream()
            .filter(Dish::isVegeterian)
            .limit(2)
            .collect(toList());
    firstThreeMenus.stream().forEach(System.out::println);
}
```

#### 5.1.4 요소 건너뛰기

* 처음 n개의 요소를 제외한 나머지를 반환하는 `skip(n)`메서드도 제공한다.

```java
@Test
public void 요소건너뛰기(){
    List<Dish> exceptFIrstTwoMenu = menu.stream()
            .skip(2)
            .collect(toList());
    assertThat(exceptFIrstTwoMenu.size()).isEqualTo(7);
}
```

* 중요한 것은 `limit(n)`, `skip(n)` 다른 연산과 상호 보완적으로 동작한다.
  * 아래의 연산에서는 처음 요리 두개를 건너뛰는 것이 아니라, 300칼로리가 넘는 요리들 중 처음 두 개를 건너뛴다.

```java
 List<Dish> dishes = menu.stream()
     		.filter(d->d.getCalories() > 300)
            .skip(2)
            .collect(toList());
```

### 5.2 매핑

* 매핑연산은 특정 데이터 묶음 들에서, 한 종류의 데이터들만 추출하는 형태의 연산이다. SQL에서 특정 테이블에서 특정 열만 가져오는 것과 유사하다고 할 수 있다.

#### 5.2.1 스트림의 각 요소에 함수 적용하기

> 스트림은 함수를 인수로 받는 `map`메서드를 지원한다. 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.

```java
@Test
public void 요리명_추출하기(){
    List<String> menuNames = menu.stream()			// Stream<Dish>
                                .map(Dish::getName) // Stream<String>
                                .collect(toList());
    assertThat(menuNames.size()).isEqualTo(9);
    assertThat(menuNames.get(0)).isEqualTo("pork");
}
```

* `map`은 최종연산이 아니기 때문에, 새로운 요소를 가지는 스트림으로 바뀌는 것이다.
* `map()`메서드는 스트림의 제네릭 타입이 가진 함수를 활용할 수 있다. 아래의 예제는 스트링 배열의 각 요소가 가진 길이를 반환하는 소스코드다

```java
@Test
public void 단어길이_출력(){
    List<String> words = Arrays.asList("Java", "8", "In", "Action");
    words.stream()
            .map(String::length)
            .forEach(System.out::println);
}
//출력결과
4
1
2
6
```

#### 5.2.2 스트림 평면화

* 위에서 사용한 `words`에 있는 단어들을 모두 고유의 문자만을 담은 `List`로 반환하는 것을 생각해보자

```java
words.stream()
    .map(word->word.split(""))
    .distinct()
    .collect(toList());
```

* 위와 같은 코드로 해결할 수 있을 것으로 보이지만, 실상은 `List<String[]`가 반환되어 버린다.
  * `split("")`의 결과가 `String[]`이기 때문이다.

##### `map`과 `Arrays.stream` 활용

* 배열 처리를 위한 유틸 클래스인 `Arrays`에는 스트림을 만들 수 있는 `Arrays.stream()`메서드가 존재한다.

```java
@Test
public void Array_stream_활용(){
    List<Stream<String>> streamList = words.stream()
            .map(word->word.split(""))
            .map(Arrays::stream) //Stream<Stream<String>> 이 생성됨!!!
            .distinct()
            .collect(toList());
    Stream<String> stream1 =  streamList.get(0);
}
```

* 위의 결과도 우리가 원하는 `List<String>`을 반환하지 않고, `List<Stream<String>>`을 반환한다

##### `flatMap` 사용

* 이 문제의 해결책은 `flatMap`메서드이다.

```java
@Test
public void Flatmap_으로_해결(){
    words.stream()
            .map(w->w.split(""))
            .flatMap(Arrays::stream)
            .distinct()
            .forEach(System.out::println);
}
```

> `flatMap`은 배열을 스트림이 아니라 **스트림의 컨텐츠**로 매핑한다. 즉 `map(Arrays::stream)`과 달리 `flatMap`은 하나의 평면화된 스트림을 반환한다.

* 요약하면, 스트림의 각 값을 다른 스트림으로 만들어서, 모든 스트림을 하나의 스트림으로 연결해야 할때는 `flatMap`을 사용하면 된다.

### 5.3 검색과 매칭

> 특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다.

* 검색과 매칭 연산은 최종 연산이다!!
* 검색과 매칭연산은 **쇼트 서킷** 기법을 활용한다.
  * 자바의 `&&`나 `||`와 같은 기법으로 불린 판단시 스마트하게 로직을 처리하는 방법

#### 5.3.1 프레디케이트가 적어도 한 요소와 일치 하는지 확인

* `anyMatch()`메서드에 인수로 전달된 프레디케이트와 같은 값이 하나라도 있는지 찾는다.

```java
@Test
public void anyMatch_테스트(){
    assertThat(menu.stream().anyMatch(Dish::isVegeterian)).isEqualTo(true);
}
```

#### 5.3.2 프레디케이트가 모든 요소와 일치하는 지 검사

* `allMatch()` 에 인수로 전달된 프레디케이트와 모든 요소가 일치하는지 검사한다.

```java
@Test
public void allMatch_테스트(){
    assertThat(menu.stream().allMatch(x->x.getCalories()>0)).isEqualTo(true);
    assertThat(menu.stream().allMatch(x->x.getCalories()>150)).isEqualTo(false);
}
```

##### `noneMatch` 

* `allMatch()`와 정 반대의 연산을 수행한다.

```java
@Test
    public void noneMatch_테스트(){
        assertThat(menu.stream().noneMatch(x->x.getCalories()>1000)).isEqualTo(true);
    }
```

#### 5.3.3 요소 검색

* `findAny()`메서드는 스트림에서 임의의 요소를 `Optional`로 감싸서 반환한다.

```java
@Test
public void findAny(){
    Optional<Dish> dish = menu.stream()
                    .filter(Dish::isVegeterian)
                    .findAny(); //임의의 채식 메뉴를 선택
    assertThat(dish.get().isVegeterian()).isEqualTo(true);
}
```

##### `Optional`이란

> 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다. `null`은 쉽게 에러를 일으킬 수 있으므로 자바 8 라이브러리 설계자는 `Optional<T>`라는 기능을 만들었다.

#### 5.3.4 첫 번째 요소 찾기

* `findFirst()`를 활용해서, 순차적인 요소의 첫 번째 값을 찾을 수도 있다.

```java
@Test
public void findFirst(){
    Optional<Dish> dish = menu.stream()
                        .filter(d->{
                            System.out.println("checking "+ d.getName());
                            return d.isVegeterian();
                        })
                        .findFirst();
    assertThat(dish.get().getName()).isEqualTo("french fries");
}
```

* 위의 소스코드를 수행하면 `filter`메서드가 전체 메뉴를 먼저 채식메뉴로 필터링 하는 것이 아니라, 첫 번째 채식메뉴를 만날 때까지만 수행함을 알 수 있다.

### 5.4 리듀싱

* 스트림 내부의 요소를 모두 모아서 결과를 내는 질의를 **리듀싱 연산**이라고 한다.

#### 5.4.1 요소의 합

* 먼저 스트림을 사용하지 않는 일반적인 for-each 루프를 통한 숫자 합을 구하는 코드를 보자

```java
int sum = 0;
for(int x : numbers){
    sum+= x;
}
```

* 해당 연산에서 필요한 부분들을 살펴보면 두 가지가 필요하다.
  * sum 변수의 **초깃값 0**
  * 모든 요소롤 조합하는 **연산 (+=)**
* 이와 같이 **리듀싱**연산도 **초깃 값**과 두 값을 조합해서 새로운 값을 만드는 **`BinaryOperator<T>`** 필요로 한다. 

```java
int sum = numbers.stream().reduce(0, (a,b)->a+b);
```

##### 초깃값 없음

* `reduce()`함수중에는 초기 값을 필요로 하지않는 메서드도 있다, 단 이 경우 연산의 결과가 없어 새로운 값이 나오지 않을 수 있기 때문에, `Optional`로 감싸서 반환한다.

#### 5.4.2 최댓값과 최솟값

* 두 요소를 가지고 식을 수행하는 함수를 인수로 받는 `reduce()`를 활용해서 최댓값과 최솟값을 구할 수 있다.
  * `Integer.max(a,b)` 메서드는 a,b중의 큰수를 반환하는 정적 메서드 레퍼런스를 활용한다.

```java
@Test
public void 최대값_구하기(){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
    Optional<Integer> sum = numbers.stream()
                                    .reduce(Integer::max);
    assertThat(sum.get()).isEqualTo(10);
}
```

##### 스트림 연산: 상태 없음과 상태 있음

* 스트림의 연산은 내부적으로 상태를 갖는 연산과, 상태를 같지 않는 연산이 있다.
  * `reduce`, `sum`,`max`등의 연산은 연산내부에서 값을 다룰 상태가 있다.
  * `sorted`나 `distinct`도 과거 이력을 담은 상태를 가지고 있다.

##### 스트림 연산 정리

| 연산        | 형식                          | 변환형식      | 함수형 인터페이스        | 함수 디스크럽터 |
| ----------- | ----------------------------- | ------------- | ------------------------ | --------------- |
| `filter`    | 중간                          | `Stream<T>`   | `Predicate<T>`           | T -> boolean    |
| `distinct`  | 중간<br>(상태 있는 언바운드)  | `Stream<T>`   |                          |                 |
| `skip`      | 중간<br/>(상태 있는 바운드)   | `Stream<T>`   | `Long`                   |                 |
| `limit`     | 중간<br/>(상태 있는 바운드)   | `Stream<T>`   | `Long`                   |                 |
| `map`       | 중간                          | `Stream<R>`   | `Function<T,R>`          | T -> R          |
| `flatMap`   | 중간                          | `Stream<R>`   | `Function<T, Stream<R>>` | T -> Stream<R>  |
| `sorted`    | 중간<br/>(상태 있는 언바운드) | `Stream<R>`   | `Comparator<T>>`         | (T, T) -> int   |
| `anyMatch`  | 최종                          | `boolean`     | `Predicate<T>`           | T -> boolean    |
| `noneMatch` | 최종                          | `boolean`     | `Predicate<T>`           | T -> boolean    |
| `allMatch`  | 최종                          | `boolean`     | `Predicate<T>`           | T -> boolean    |
| `findAny`   | 최종                          | `Optional<T>` |                          |                 |
| `findFirst` | 최종                          | `Optional<T>` |                          |                 |
| `forEach`   | 최종                          | `void`        | `Consumer<T>`            | T -> void       |
| `collect`   | 최종                          | R             | `Collector<T,A,R>`       |                 |
| `reduce`    | 최종<br/>(상태 있는 바운드)   | `Optional<T>` | `BinaryOperator<T>`      | (T, T) -> T     |
| `count`     | 최종                          | `long`        |                          |                 |

### 5.5 실전 연습

##### 5.5.1 거래자와 트랜잭션

* 실전연습에 사용할 `Trader` 클래스와, `Transaction` 클래스 그리고, `List<Transaction`의 정의다

###### `Trader.java`

```java
public class Trader {
    private final String name;
    private final String city;

    public Trader(String name, String city) {
        this.name = name;
        this.city = city;
    }

    public String getName() {
        return name;
    }

    public String getCity() {
        return city;
    }

    @Override
    public String toString() {
        return "Trader:" + this.name + " in " +this.city;
    }
}
```

###### `Transaction.java`

```java
public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Trader getTrader() {
        return trader;
    }

    public int getYear() {
        return year;
    }

    public int getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "{"
                 + trader +
                ", " + year +
                ", " + value +
                '}';
    }
}
```

```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
                new Transaction(brian, 2011, 300),
                new Transaction(raoul, 2012, 1000),
                new Transaction(raoul, 2011, 400),
                new Transaction(mario, 2012, 710),
                new Transaction(mario, 2012, 700),
                new Transaction(alan, 2012, 950)
        	);
```

##### 1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오

```java
@Test
public void question_1(){
    System.out.println("Question 1");
    List<Transaction> sorted2011s =transactions.stream()
                                    .filter(x->x.getYear()==2011)
                                    .sorted(comparing(Transaction::getValue))
                                    .collect(toList());


    assertThat(sorted2011s.size()).isEqualTo(2);
    assertThat(sorted2011s.get(0).getValue()).isEqualTo(300);
    assertThat(sorted2011s.get(1).getValue()).isEqualTo(400);
    sorted2011s.stream().forEach(System.out::println);
}
```

##### 2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오

```java
@Test
public void question_2(){
    System.out.println("Question 2");
    List<String> cities = transactions.stream()
                            .map(Transaction::getTrader)
                            .map(Trader::getCity)
                            .distinct()
                            .collect(toList());
    cities.stream().forEach(System.out::println);
}
```

##### 3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오
```java
@Test
public void question_3(){
    System.out.println("Question 3");
    List<Trader> traderFromCambridge = transactions.stream()
                                        .map(Transaction::getTrader)
                                        .distinct()
                                        .filter(x->x.getCity().equals("Cambridge"))
                                        .sorted(comparing(Trader::getName))
                                        .collect(toList());
    traderFromCambridge.stream().forEach(System.out::println);
}
```

##### 4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오
```java
@Test
public void question_4(){
    System.out.println("Question 4");
    List<String> sortedTraders = transactions.stream()
                                .map(Transaction::getTrader)
                                .sorted(comparing(Trader::getName))
                                .map(Trader::getName)
                                .distinct()
                                .collect(toList());
    sortedTraders.stream().forEach(System.out::println);
}
```

##### 5. 밀라노에 거래자가 있는가?
```java
@Test
public void question_5(){
    System.out.println("Question 5");
    boolean isThereTraderInMilan = transactions.stream()
                                        .map(Transaction::getTrader)
                                        .anyMatch(x->"Milan".equals(x.getCity()));
    System.out.println(isThereTraderInMilan);
}
```

##### 6. 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오
```java
@Test
public void question_6(){
    System.out.println("Question 6");
    List<Transaction> transactionByTraderInMilan 
        = transactions.stream()                                                   							.filter(x->"Cambridge".equals(x.getTrader().getCity()))
                    .collect(toList());
    transactionByTraderInMilan.stream().forEach(System.out::println);
}
```

##### 7. 전체 트랜잭션 중 최댓값은 얼마인가?
```java
@Test
public void question_7(){
    System.out.println("Question 7");
    //전체 트랜잭션 중 최댓값은 얼마인가?
    int biggest = transactions.stream()
                            .map(Transaction::getValue)
                            .reduce(Integer.MIN_VALUE, Integer::max);
    System.out.println(biggest);
}
```

##### 8. 전체 트랜잭션 중 최솟값은 얼마인가?

```java
@Test
public void question_8(){
    System.out.println("Question 8");
    int smallest = transactions.stream()
            .map(Transaction::getValue)
            .reduce(Integer.MAX_VALUE, Integer::min);
    System.out.println(smallest);
}
```

### 5.6 숫자형 스트림

```java
int calories = menu.stream()
    				.map(Dish::getCalories)
    				.reduce(0, Integer::sum);
```

* 위의 예제에 대해서 아직 살펴보지 못한 부분이 있다 바로 박싱과 언박싱의 문제이다.

#### 5.6.1 기본형 특화 스트림

> 자바 8에서는 세 가지 기본형 특화 스트림을 제공한다.

* `int`요소에 특화된 `IntStream` - `mapToInt()`를 통해 변환한다.
* `double`요소에 특화된 `DoubleStream` - `mapToDouble()`을 통해 변환한다.
* `long`요소에 특화된 `LongStream` - `mapToLong()`을 통해 변환한다.

##### 숫자 스트림으로 매핑

```java
int calories = menu.stream()
        .mapToInt(Dish::getCalories)
        .sum();
```

* 숫자 스트림은 박싱된 객체를 요소로 하지 않는 것 뿐만 아니라, 스트림 인터페이스 차원에서 `max`, `min`,`average`등의 유틸리티 메서드도 지원한다.

##### 객체 스트림으로 복원하기

* 간단하게 `boxed()` 메서드를 호출하면 된다.

```java
@Test
public void 객체_스트림_복원(){
    IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
    IntStream intStream2 = menu.stream().mapToInt(Dish::getCalories);
    System.out.println(intStream2.sum());
    Stream<Integer> stream = intStream.boxed();
    System.out.println(stream.reduce(0, Integer::sum));
}
```

##### 기본값: OptionalInt

* 앞서서 최종연산에서 `Optional<T>`을 반환하는 것을 살펴보았다. 기본형 특화스트림을 위해서 이와 유사한 `OptionalInt`, `OptionalLong`, `OptionalDouble`을 제공한다.

```java
OptionalDouble averageCalory = menu.stream()
        .mapToInt(Dish::getCalories)
        .average();
System.out.println(averageCalory.getAsDouble());
```

#### 5.6.2 숫자 범위

* 특정 범위의 숫자를 스트림으로 만들기 위해서 숫자형 특화스트림에는 `range`와 `rangeClosed`라는 정적메서드가 있다.
  * 둘의 차이는 시작값과 종료값 인수가 스트림에 포함되느냐 여부이다 (`rangeClosed`가 포함)

```java
@Test
public void rangeClosed(){
    IntStream stream = IntStream.rangeClosed(1,100);
    System.out.println(stream.sum());  //5050
}
```

#### 5.6.3 숫자 스트림 활용: 피타고라스 수

* A x A + B x B = C x C 를 만족하는 세 개의 정수를 찾는 로직을 스트림을 활용해서 만들어보자

##### 좋은 필터링 조합

* A와 B가 주어진다고 했을 때, C에 걸리는 제약 조건은 A x A + B x B의 제곱근이 정수여야 한다는 것이다. 여기서 아래의 `filter`식을 도출할 수 있다.

```java
filter(b -> Math.sqrt(a*a + b*b)%1 == 0)
```

##### 집합 생성

* 위의 필터를 통해서 A값과 B값을 바탕으로 C값의 조합을 필터할 수 있으니 이를 이제 집합으로 바꿔주는 식이 필요하다

```java
stream.filter(b -> Math.sqrt(a*a + b*b)%1 == 0)
    .map(b->new int[]{a, b, (int) Math sqrt(a * a + b * b)});
```

##### b 값 생성

* A값과 B값은 주어져야 한다, 먼저 `IntStream.rangeClosed`를 사용해서 B값을 생성해보자
  * `mapToObj()` 메서드는 `boxed()`와 `map()`을 연달아 호출하는 효과를 가진다.

```java
IntStream.rangeClosed(1, 100)
    .filter(b -> Math.sqrt(a*a + b*b)%1 == 0)
    .mapToObj(b->new int[]{a, b, (int) Math sqrt(a * a + b * b)});
```

##### a값 생성

* A 값도  동일하게 생성해주면 된다, 단 여기서 기존의 B값의 시작 값을 A로 바꿔주어야 한다.

```java
IntStream.rangeClosed(1, 100).boxed()
    .flatMap(a ->
             IntStream.rangeClosed(a, 100)
    	`			.filter(b -> Math.sqrt(a*a + b*b)%1 == 0)
    				.mapToObj(b->new int[]{a, b, (int) Math sqrt(a * a + b * b)})
            );
```

* `flatMap`의 역할은 `IntStream.rangeClosed`으로 만들어진 A의 스트림과, 다른 두 개의 숫자들의 스트림을 평탄화 시키는 것이다.

##### 개선할 점?

* `Mat.sqrt`가 두번 호출되는 것을 방지하기 위해선, 원하는 조건의 수를 먼저 만들고 그 뒤에 필터하는 것으로 개선할 수 있다. 아래의 코드가 최종 결과물이다.

```java
@Test
public void 피타고라스_수(){
    Stream<double[]> pythagoreanTriples =
            IntStream.rangeClosed(1,100).boxed()
            .flatMap(a ->
                    IntStream.rangeClosed(a,100)
                    .mapToObj(
                    b-> new double[]{a,b,Math.sqrt(a*a+b*b)})
                    .filter(t->t[2] %1 ==0));
}
```

### 5.7 스트림 만들기

#### 5.7.1 값으로 스트림 만들기

* `Stream.of` 정적메서드를 활용해서, 값을 스트림으로 만들 수 있다.

```java
@Test
public void 값으로_스트림(){
    Stream<Integer> numbers = Stream.of(1,2,3,4,5,6,7,8,9,10);
    System.out.println(numbers.mapToInt(x->x).sum());
}
```

* `Stream.empty()` 메서드는 빈 스트림을 반환한다.

#### 5.7.2 배열로 스트림 만들기

* `Arrays.stream` 정적메서드를 사용해서, 배열을 스트림으로 만들 수 있다.

```java
@Test
public void 배열로_스트림(){
    int[] numbers = {1,2,3,4,5,6,7,8,9,10};
    int sum= Arrays.stream(numbers).sum();
}
```

#### 5.7.4 함수로 무한 스트림 만들기

* 스트림 인터페이스는 함수에서 스트림을 만들 수 있는 `interate`와 `generate`정적메소드를 제공한다
* 두 메서드를 활용해서, 크기가 고정되지 않은 무한스트림을 만들 수 있으며, 일반적으로 `limit()`과 함께 유한한 값을 생성하는데 활용한다.

##### `iterate`

* `Stream.iterate()` 정적 메서드는, 초깃값과 람다를 인수로 받아서 값을 끊임 없이 생성한다.

```java
@Test
public void 무한스트림_iterate(){
    IntStream stream = IntStream.iterate(0, n->n+1)
            .limit(100);
    stream.forEach(System.out::println);
}
```

* `iterate`의 원리를 호라용해서 피보나치 수열을 만들 수도 있다.

```java
@Test
public void 피보나치_수열_출력(){
    Stream.iterate(new int[]{0,1}, a-> new int[] {a[1],a[0]+a[1]})
            .limit(20)
            .map(t->t[0])
            .forEach(System.out::println);
}
```

##### `generate`

* `generate`는 생성된 값을 연속적으로 계산하는게 아니라, `Supplier<T>` 함수형 인터페이스를 인수로 받아 새로운 값을 생성한다.

```java
@Test
public void 무한스트림_generate(){
    DoubleStream.generate(Math::random)
            .limit(5)
            .forEach(System.out::println);
}
```

