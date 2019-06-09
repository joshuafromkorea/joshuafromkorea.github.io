---
layout: post
title: "Java 8 in Action Chapter02 - 동작 파라미터화 코드 전달하기"
date: 2019-06-09
categories:
---

직접 코드를 작성하지 않던 *이것이 자바다* self-study와는 달리, 이 책은 먼저 책 내용을 정리하기전에 책에 나오는 예제코드를 수행하고, 추가적으로 learning test형식으로 코드를 만들어 테스트하면서 학습하고 있다.

덕분에 내용이 많이 어렵지만, 코드작성할 때 한번, 그리고 블로그에 남길 글을 작성할 때 한번해서, 총 두 번을 정독하는 셈이라서 내용을 이해하는데 큰 도움이 된다.

그걸 감안할지라도 이번 챕터에 나온 메서드 레퍼런스와, 생성자 레퍼런스는 원활하게 실무에서 사용할 수 있을지 아직 잘 모를정도로 약간 어렵고 생소하게 다가온다. 실제로 IPC포탈 프로젝트를 수행할 때에도 다른 람다 코드는 거부감 없이 받아들였는데, 메서드 레퍼런스와 생성자 레퍼런스가 복잡하게 되어있으면 은근히 이해하기 힘들었다.

저자는 해당 방식이 가독성이나 코드의 의미를 파악하기 쉽다고 말하였는데, 익숙함과 러닝커브의 벽은 역시 위대한것 같다.

- [**공부한 내용 Github 에서 보기**](https://github.com/joshuafromkorea/Java_8_In_Action_Study/tree/f7347a9928162b252d9348d8ea9fa673b9860a0d)

---

## Chapter 3 람다 표현식

* 전 챕터에서 마지막에 도달했던 **람다 표현식**이 어떻게 작동하는 가를 살펴본다.
  * 람다 표현식 생성 법
  * 람다 표현식 사용 법
  * 람다 표현식의 장점

### 3.1 람다란 무엇인가

* 람다의 특징은 아래 4가지로 정리할 수 있다.
  1. **익명**: 익명 클래스와 유사하게, 일반 메서드와 달리 이름이 없다.
  2. **함수**: 메서드처럼 특정 클래스에 ~~종속~~되지 않는다.
  3. **전달**: 람다 표현식을 **인수**로 전달하거나, **변수**로 저장할 수 있다.
  4. **간결성**: 이를 통해 매우 간결한 코드로 자바8 이전의 많은 내용을 커버할 수 있다.

* 지난 챕터에서 마지막에 사용한 람다 코드를 예제로, 람다의 구성요소 3가지를 살펴보자

	```java
(Apple apple)->"red".equals(apple.getColor())).size()
	```
	1. **파라미터 리스트**: `(Apple apple)` - 하나 이상으로 이뤄질 수 있으며, 바디에서 사용된다.
	2. **화살표(`->`)**:  파라미터 리스트와 바디를 구분한다
	3. **람다의 바디**: 파라미터 리스트로 할 동작, 람다의 **반환 값**에 해당하는 표현식이다.
	
* 위의 설명을 가지고 람다의 문법을 크게 두 가지로 도출할 수 있다.

  * `(parameters) -> expression`
    * 파라미터는 없을 수 있으며, `return`을 사용하지 않는 반환값이 있는 표현식이다.
  * `(parameters) -> { statemets; }`
    * 중괄호를 사용하며 값을 반환할 때는 `return`문을 포함해야 한다.

### 3.2 어디에, 람다를 어떻게 사용할까?

> 함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

#### 3.2.1 함수형 인터페이스

* 2장에서 만든 `Preadicate<T>`와 같은 인터페이스가, **함수형 인터페이스**이다.

  > 간단히 말해, **함수형 인터페이스**는 정확히 **하나의 추상 메서드**를 지정하는 인터페이스다.

* 람다 표현식의 역할은 바로, 이런 함수형 인터페이스의, 구현된 **인스턴스로 취급**되는 것이다.

  * 익명 내부클래스도 동일한 역할을 하지만, 람다표현식만큼 깔끔하지는 않다.

#### 3.2.2 함수 디스크럽터

* **함수 디스크럽터**란 추상메서드의 시그니처에 매칭되는 람다 표현식의 시그니처
  * 즉, `void XXX();` 형태의 메서드 시그니처는 `()->void` 로 표기할 수 있다.
    * 파라미터 리스트 없음, `void`를 반환함

### 3.3 람다 활용: 실행 어라운드 패턴

* I/O 기능을 작성할 때 빈번히 사용하는 자원을 열고 닫는 **실행 어라운드 패턴**에 람다활용을 적용해보자

###### `ExecuteAround.java`

```java
public  String processFile() throws IOException {
    try(BufferedReader br =
            new BufferedReader(new FileReader("/home/joshua/data.txt"))){

        return br.readLine();
    }
}
```

* 위 예제를 보면 `BufferedReader`에서 `readLine()`을 하는 형태로 미리 정의되어 있다.
  * 따라서 두 행을 읽거나 다른 작업을 하려면 반복되는 코드작성이 필요하다.

#### 3.3.1 1단계 동작 파라미터화를 기억하라

* 전 챕터에서 살펴본 동작 파라미터 전달을 람다를 통해 활용해 보자
  * 람다를 통해서 아래와 같이 클라이언트에서 동작을 전달하는 예제를 먼저 작성한다.

```java
 executeAround.processFileWithRamda((BufferedReader br) -> br.readLine())
```

#### 3.3.2 2단계: 함수형 인터페이스를 이용해서 동작 전달

* 람다는 앞에서 말한 것처럼 **함수형 인터페이스**자리에만 사용할 수 있다.
  * 따라서, 람다의 시그니쳐 (**즉, 람다 디스크립터**)와 일치하는 함수형 인터페이스를 만들어야 한다.

###### `ExecuteAround.java`

```java
@FunctionalInterface
public interface BufferedReaderProcessor{
    String process(BufferedReader br) throws IOException;
}
```

* 이제 이 인터페이스를 인수로 받는 동작을 실행할 코드만 있으면 된다.

#### 3.3.3 3단계: 동작 실행!

* 앞서 정의한 함수형 인터페이스를 실행하는 코드이다
###### `ExecuteAround.java`

```java
public String processFileWithRamda(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br =
                new BufferedReader(new FileReader("/home/joshua/data.txt"))){

        return p.process(br);
    }
}
```

#### 3.3.4 4단계: 람다 전달

```java
//한행을 처리
String oneLine = processFileWithRamda(BufferedReader br) -> br.readLine());

//두행을 처리
String oneLine = processFileWithRamda(BufferedReader br) -> br.readLine());
```

* 동작 파라미터를 람다로 전달하기 때문에, 하나의 실행 코드로, 여러 동작을 수행하게 되었다.

### 3.4 함수형 인터페이스 사용

* 자바는 기본적으로 "`java.util.function` 패키지로 여러가지 새로운 함수형 인터페이스를 제공한다."

#### 3.4.1 `Predicate`

* 제네릭 형식 `T`의 객체를 인수로 받아 `boolean`값을 반환한다.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T var1);
...
}
```

#### 3.4.2 `Consumer`

* 제네릭 형식 `T` 객체를 받아서 `void`를 반환한다: 어떤 동작을 수행하고 싶을 때 사용한다.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
...
}
```

#### 3.4.3 `Function`

* 제네릭 형식 `T`를 인수로 받아서 제네릭형식 `R`을 반환한다: **입출력**에 활용 한다.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
...
}
```

##### 기본형 특화

* 자바가 제공하는 함수형 인터페이스는 참조형<sup>reference type</sup>을 전제로 하기 때문에 기본형<sup>primitive type</sup>을 사용하면 **오토박싱**으로 인한 자원의 낭비가 발생할 수 있다.
  * 이를 위해 `Int`, `Long` 등의 접두사로 시작하는 기본형 특화된 함수형 인터페이스를 제공한다.

```java
@FunctionalInterface
public interface IntPredicate {
    boolean test(int value);
...
}
```

### 3.5 형식 검사, 형식 추론, 제약

> 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어있지 않다

* 이를 어떻게 검사하고 추론하는지 알아보자.

#### 3.5.1 형식 검사

* 람다가 소스코드의 **어떤 맥락**에서 사용되는 지를 보고 **형식**을 추론한다.
  1. 사용되는 **메서드의 선언**을 확인한다
  2. 메서드에서 **요구하는 형식**(대상 형식)을 기대한다
  3. 해당 함수형 인터페이스이의 **추상메서드**를 확인한다
  4. 추상메서드를 **함수 디스크립터**로 바꿔 비교한다
  5. 형식 검사가 완료 된다.

#### 3.5.2 같은 람다, 다른 함수형 인터페이스

> **대상 형식**이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

```java
Callable<Integer>  c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

#### 3.5.3 형식 추론

* 람다 표현식을 바탕으로, 람다 파라미터 리스트의 형식도 추론할 수 있다.

```java
(Apple a1, Apple a2) -> a1.geWeight().compareTo(a2.getWeight());// 형식추론 없음
(a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); // 형식 추론함
```

#### 3.5.4 지역 변수 사용

* 람다 표현식안에서, 외부에 정의된 **자유 변수**를 활용하는 것을 **람다 캡처링**<sup>capturing lamda</sup>라고 부른다.

##### 지역 변수의 제약

* **람다 캡처링**에 사용되는 지역변수는 **`final`**키워드로 선언되던지, 실질적으로 `final`이어야 한다.
  * 람다 캡처링은 자유 변수의 복사본을 제공하는 것이기 때문에, 값이 바뀌지 않는 것이 보장되어야 한다.

### 3.6 메서드 러퍼런스

> **메서드 레퍼런스를** 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

#### 3.6.1 요약

* 메서드 레퍼런스는 메서드 앞에 구분자 `::`를 붙이는 방법으로 사용한다
  * 예를들어 `Apple::getWeight`은  `Apple`클래스에 정의된 `getWeight`라는 이름의 메서드 레퍼런스다
* 이는 실제로 메서드를 호출하는 것이 아니라, 람다표현식을 축약해서 전달하는 것이다.
  * 이렇게 메서드명을 사용하는 목적은 **가독성을 높이는 것**이다.

##### 메서드 레퍼런스를 만드는 방법

1. **정적 메서드 레퍼런스**: `Integer`의 `parseInt`는**`Integer::parseInt`**로 표현한다
2. **인스턴스 메서드 레퍼런스**: `String`의 `length`는 **`String::length`**로 표현한다.
3. **기존 객체의 인스턴스 메서드 레퍼런스**
   * `Transaction` 객체를 항당받은 `expensiveTransaction` 지역변수가 있다
   * `Transaction` 클래스에는 `getValue`라는 이름의 메서드가 있다.
   * **`expensiveTransaction::getValue`**로 표현할 수 있다.

###### `MethodReferenceTest.java`

```java
@Test
public void 메서드_레퍼런스_만들기_테스트(){
    List<String> str1 = str;
    str1.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

    List<String> str2 = str;
    str1.sort(String::compareToIgnoreCase);

    assertThat(str1.equals(str2));
}
```

#### 3.6.2 생성자 레퍼런스

* 메서드에 더해서, **생성자**조차도 레퍼런스를 만들 수 있다. 정적메서드를 만드는 것과 유사하다.

###### `ConstructorReference.java`

```java
@Test
public void 디폴트_생성자_레퍼런스_만들기(){
    Supplier<Apple> c1 = () ->new Apple();
    Apple apple1 = c1.get();
    assertThat(apple1.getColor()).isEqualTo("default color");

    Supplier<Apple> c2 = Apple::new;
    Apple apple2 = c2.get();
    assertThat(apple2.getColor()).isEqualTo(apple1.getColor());
}
```

* 파라미터가 있는 생성자가 오버로딩 되어있어도 동일한 표현식으로 사용 가능하다.

```java
@Test
public void 한개의_파라미터를_갖는_생성자_레퍼런스_만들기(){
    Function<Integer, Apple> c1 = (weight) -> new Apple(weight);
    Function<Integer, Apple> c2 = Apple::new;

    assertThat(c1.apply(100).getWeight()).isEqualTo(c2.apply(100).getWeight());
}
```

* 이런 **생성자 레퍼런스**는 전달가능한 일급객체이기 때문에, 다른 메서드에 전달할 수도 있다.

```java
private List<Apple> map(List<Integer> list, Function<Integer, Apple>f){
    List<Apple> result = new ArrayList<>();
    for(Integer e : list){
        result.add(f.apply(e));
    }
    return result;
}

List<Integer> weights = Arrays.asList(7,3,4,10) ;
List<Apple> apples = map(weights, Apple::new);
apples.stream().forEach(x-> System.out.println(x.getWeight()));
```

* `map`이라는 이름의 메서드는 `Integer`가 담긴 리스트와, `Function`타입의 함수형 인터페이스를 받는다.
  * 여기에 두번재 파라미터로 `Apple` 객체의 생성자 레퍼런스를 전달하면 객체를 담은 리스트를 반환한다.

### 3.7 람다, 메서드 레퍼런스 활용하기!

* 지금까지 배운 람다와 메서드레퍼런스를 활용해서 기존의 사과정렬 코드를 개선해보자

#### 3.7.1 1단계: 코드전달

* 자바8의 `List` 는 정렬과 관련된 메서드가 있으므로 코드를 전달받을 정렬메서드는 구현할 필요가 없다.

###### `List.java`

```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

* `sort` 메서드가 인수로 받는 `Comparator` 구현객체에 따라 전략이 달라지므로 이를 먼저 구현한다.

###### `AppleComparator.java`

```java
public class AppleComparator implements Comparator<Apple> {
    @Override
    public int compare(Apple o1, Apple o2) {
        return ((Integer) o1.getWeight()).compareTo(o2.getWeight());
    }
}
```

* 그리고 이 코드를 이제 `sort`에 전달하면 된다.

###### `AppleComparatorTest.java`

```java
@Test
public void 코드_전달(){
    printSortedApples(inventory);
    inventory.sort(new AppleComparator());
    System.out.println("after sort");
    printSortedApples(inventory);
}
```

#### 3.7.2 2단계: 익명 클래스 사용

* 직접 `Comparator`를 구현해서 클래스를 작성하는 것보다, 아래처럼 익명클래스가 편리함을 이미 알아봤다.

###### `AppleComparatorTest.java`

```java
@Test
public void 익명_클래스(){
    printSortedApples(inventory);
    inventory.sort(new Comparator<Apple>() {
        @Override
        public int compare(Apple o1, Apple o2) {
            return ((Integer) o1.getWeight()).compareTo(o2.getWeight());
        }
    });
    System.out.println("after sort");
    printSortedApples(inventory);
}
```

#### 3.7.3 3단계: 람다 표현식 사용

> **함수형 인터페이스**를 기대하는 곳 어디에서나 람다 표현식을 사용할 수 있음을 배웠다.

* 즉, 익명 클래스가 사용되고 있는 `Comparator`가 함수형 인터페이스라면 람다표현식이 사용가능하다.

###### `Comparator.java`

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
...
}
```

* 함수형 인터페이스인 `Comparator.java`가 가진 추상 메서드의 시그니처로 람다 시그니처를 정의한다
  * `int compare(T o1, T o2)`  => `(T, T) -> int` => `(Apple, Apple) -> int`

* 따라서 익명클래스 대신에, 람다 표현식을 넘겨주어 코드가 간결해질 수 있다.
###### `AppleComparatorTest.java`
```java
public void 람다_표현식(){
    printSortedApples(inventory);
    inventory.sort((o1, o2) -> ((Integer)o1.getWeight()).compareTo(o2.getWeight()));
    System.out.println("lamda after");
    printSortedApples(inventory);
}
```
> `Comaprator`는 `Comparable`키를 추출해서 `Comparator` 객체로 만드는 `Function` 함수를 인수로 받는 정적 메서드 `comparing`을 포함한다.

* 위 명제에서 간소화 할것을 발견할 수 있다.
  * **`Comparator`** 객체: `sort`가 필요로 하는 인수이다.
  * **`Function`**함수: 람다 형식 전달 가능
```java
a1.getWeight().compareTo(a2.getWeight());
Comparator.comparing((Apple a) -> a.getWeight()); //위아래 내용은 모두 같은 Comparator를 반환
```

#### 3.7.4 4단계: 메서드 레퍼런스 사용

* 위에서 간단하게 바꾼 람다표현식은 `Apple` 의 인스턴스 메서드를 사용하는 것이다.
  * 따라서 메서드 레퍼런스로 전달이 가능하다!!
###### `AppleComparatorTest.java`
```java
@Test
public void 메소드_레퍼런스(){
    printSortedApples(inventory);
    inventory.sort(Comparator.comparing(Apple::getWeight));
    System.out.println("method reference after");
    printSortedApples(inventory);
}
```

### 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

> 자바 8 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 메서드를 포함한다.

* 위의 `Comparator.comparing`이 한 예가 될 수 있다.

* 이하의 코드들은 모두 `LamdaUtilMethodTest.java`에 있다.

#### 3.8.1 `Comparator` 조합

##### 역정렬

* `reversed()` 라는 디폴트 메서드를 사용하면, 이미 정렬 된 것을 역정렬 할 수 있다.

```java
@Test
public void Comparator_역정렬(){
    printSortedApples(inventory);
    inventory.sort(Comparator.comparing(Apple::getWeight).reversed());
    System.out.println("sort then reverse");
    printSortedApples(inventory);
}
```

##### `Comparator` 연결

* `thenComparaing()`메서드는 정렬 후에, 또 정렬을 할 수 있게 만들어준다.

```java
@Test
public void Comparator_조합(){
    printSortedApples(inventory);
    inventory.sort(Comparator.comparing(Apple::getWeight)
                                                    .thenComparing(Apple::getColor));
    System.out.println("sort then sort again with color");
    printSortedApples(inventory);
}
```

#### 3.8.2 `Predicate` 조합

* `Predicate` 인터페이스는 복잡한 조건식을 만들 수 있도록 `negate`, `and`, `or` 세가지의 메서드를 제공한다.

```java
@Test
public void Predicate_조합(){
    //기본 Predicate
    Predicate<Apple> redApple = (Apple a) -> a.getColor() == "red";
    List<Apple> redApples = filter(inventory, redApple);
    System.out.println("red apples:");
    printSortedApples(redApples);
	
    //negate()을 사용한 조건 반전
    Predicate<Apple> notRedApple = redApple.negate();
    List<Apple> nonRedApples = filter(inventory, notRedApple);
    System.out.println("non red apples");
    printSortedApples(nonRedApples);
	
    //and()와 or()를 사용한 조건 조합하기
    Predicate<Apple> redAndHeavyOrJustGreenApple = redApple
        .and(a -> a.getWeight()>150).or(a -> a.getColor()=="green");
    List<Apple> redAndHeavyOrJustGreenApples 
        = filter(inventory, redAndHeavyOrJustGreenApple);
    System.out.println("red and heavy or just green");
    printSortedApples(redAndHeavyOrJustGreenApples);
}
```

#### 3.8.3 `Function` 조합

* `Function` 인터페이스에는 자신의 타입을 반환하는 `andThen()` 과 `compose()` 메소드를 제공한다.
  * `andThen()`: 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환한다.
  * `compose()`: 인수로 주어진 함수를 먼저 실행한 다음에, 그 결과를 외부함수에 인수로 제공한다.
* 실제로 보면 어떻게 동작하는지 알 수 있다.

```java
@Test
public void Function_조합(){
    Function<Integer, Integer> f = x-> x+1;
    Function<Integer, Integer> g = x-> x*2;
    Function<Integer, Integer> andThen = f.andThen(g);
    Function<Integer, Integer> compose = f.compose(g);

    // 2 + 1 = 2
    assertThat(f.apply(2)).isEqualTo(3);
    // 2 * 2 = 4
    assertThat(g.apply(2)).isEqualTo(4);

    // (2+1) * 2 = 6
    assertThat(andThen.apply(2)).isEqualTo(6);

    //  2 * 2 + 1 = 5
    assertThat(compose.apply(2)).isEqualTo(5);
}
```



