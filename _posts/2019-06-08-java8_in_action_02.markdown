---
layout: post
title: "Java 8 in Action Chapter02 - 동작 파라미터화 코드 전달하기"
date: 2019-06-08
categories:
---

드디어 *Java 8 In Action*을 폈다. 이책을 공부하기전에 자바를 한번 정리해야겠다고 생각해서 *이것이 자바다*를 먼저 스터디 하기 시작한지 거의 7개월 만이다. 이미 실무에서 선배들이 작성했던 코드를 보고 응용하거나, 재활용하면서 자바8의 람다나, 스트림을 접한적이 있다.

*이것이 자바다*를 공부하면서, 제대로 알지 못하고 사용했던 것들에 대해서 정리 했던 것처럼, 본 책을 공부하면서 이제 나도 면접에가서 자바8을 할줄안다고 자신있게 말할 수 있는 개발자가 되야겠다.

Chapter1은 자바에 대한 개괄과 자바의 발전 및 자바8에 특징에 대한 설명으로 블로그에서 다루는 것은 Skip한다.

* [**공부한 내용 Github 에서 보기**](https://github.com/joshuafromkorea/Java_8_In_Action_Study/tree/149bcc9f4ae63d7445f57078656d1e3b28de2c87)

---

## Chapter 2 동작 파라미터화 코드 전달하기

* **동작 파라미터화**: "아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록"
  * 자주 바뀌는 요구사항에 효과적으로 대응 가능
  * 한가지 구현으로 다양한 기능을 수행하게 할 수 있다.

### 2.1 변화하는 요구사항에 대응하기

#### 2.1.1 첫 번째 시도: 녹색 사과 필터링

* 농장 재고 목록에서 녹색사과만 필터링 하는 기능

###### `AppleFilter.java`

```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<Apple>();
    for(Apple apple : inventory) {
        if("green".equals(apple.getColor())){
            result.add(apple);
        }
    }
    return result;
}
```

#### 2.1.2 두번째 시도: 색을 파라미터화

* 색이 변할 때마다, 코드를 새로 복붙해서 만드느니, 색을 파라미터로 받도록 변경

###### `AppleFilter.java`

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, String color){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(color.equals(apple.getColor())){
            result.add(apple);
        }
    }
    return result;
}
```

* 무게로도 필터를 해야하는 **새로운 요구사항**이 생겨버림, 무게로 필터링하는 새로운 메서드 생성

###### `AppleFilter.java`

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(weight < apple.getWeight()){
            result.add(apple);
        }
    }
    return result;
}
```

* 결국 요구사항을 파라미터화 하는 것도 **DRY**<sup>don't repeat again</sup>을 어길 수 밖에 없게 만든다.

### 2.2 동작 파라미터화

* 요구사항에 따른 선택 조건을 추상화 → **사과**의 **어떤속성**에 기초해서 **불린<sup>boolean</sup>**을 반환
* 선택 조건을 **결정하는** 인터페이스 (a.k.a Predicate)를 만들고, 이를 구체적으로 구현한다.

###### `AppleFilterBehavParam.java`

```java
public interface ApplePredicate {
    boolean test (Apple apple);
}


public static class AppleHeavyPredicate implements ApplePredicate{

    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}
public static class AppleGreenPredicate implements ApplePredicate{

    @Override
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}

public static class AppleRedAndHeavyPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return "red".equals(apple.getColor())
        &&apple.getWeight()>150;
    }
}
```

#### 2.2.1 네 번째 시도: 추상적 조건으로 필터링

* 이제, 위에서 만든 Predicate들을 유동적으로 적용할 수 있는 필터링 메서드를 만들 수 있다

###### `AppleFilterBehavParam.java`

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }
    return result;
}
```

##### 코드/동작 전달하기

> 이제 Apple 속성과 관련한 모든 변화에 대응할 수 있는 유연한 코드를 준비 한 것이다.

* `filterApples()`에 파라미터로 넘겨주는 `ApplePredicate` 넘겨줄 때 코드(동작)를 넘겨주는 것이다.
  * 구체적으로 말해, 각 구현체들이 구현하고 있는 `test()` 메서드가 전달되는 것이다.

###### `AppleFilterTest.java`

```java
public void 동작파라미터화_방식(){
    //기존 녹색사과만_필터링()의 테스트와 비교
    assertThat(legacyFilter.filterApplesByColor(inventory,"green").size())
            .isEqualTo(behaviorParamFilter.filterApples(inventory, 
               									new AppleGreenPredicate()).size());
    
     //전략패턴을 사용해서 두가지 비교를 한번에 하는 predicate 파라미터화 전달
    assertThat(behaviorParamFilter.filterApples(inventory, 
                                                new AppleRedAndHeavyPredicate())
               										.size()).isEqualTo(1);
}
```

##### 한 개의 파라미터, 다양한 동작

> 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다.

* 하나의 메서드가, 어떠한 타입의 동작을 감싸는 객체를 전달하느냐에 따라 수행하는 것이 달라진다.

### 2.3 복잡한 과정 간소화

* 요구사항에 따른 수행코드 복사붙여넣기나, 파라미터 추가는 하지 않아도 되지만 여전히 번거롭다
  * `ApplePredicate`타입을 구현하는 구현체를 계속 만들어주어야 하기 때문이다.

#### 2.3.2 다섯 번째 시도: 익명 클래스 사용

* **익명 구현 클래스**를 통해서, 인터페이스 구현 및 인스턴스화 작업을 한번에 할 수 있다.
###### `AppleFilterTest.java`
```java
@Test
public void 익명클래스_동작파라미터화_방식(){
    assertThat(filterApples(inventory, new ApplePredicate() {
        @Override
        public boolean test(Apple apple) {
            return "red".equals(apple.getColor());
        }
    }).size()).isEqualTo(legacyFilter.filterApplesByColor(inventory,"red").size());
}
```

* 익명 클래스는 분명한 장점에 비해, 치명적인 단점이 있다.
  * 결국 반복적인 익명 구현 행위는 동일하게 코드양을 불린다
  * 가독성이 떨어지는 코드가 장황하게 펼쳐져서 유지보수에 좋지 않다.

#### 2.3.3 여섯 번째 시도: 람다 표현식 사용

* 람다 표현식을 사용하면, 반복적이긴 하지만, 가독성 좋은 코드를 더 짧게 작성할 수 있다.
###### `AppleFilterTest.java`
```java
@Test
public void 람다표현식_동작파라미터화_방식(){
    assertThat(filterApples(inventory,
                            (Apple apple)->"red".equals(apple.getColor())).size())
        .isEqualTo(legacyFilter.filterApplesByColor(inventory,"red").size());
}
```

#### 2.3.4 일곱 번째 시도: 리스트 형식으로 추상화

* 현재 우리가 동작파라미터를 전달받기위해 작성한 필터는 `Apple`타입만 받을 수 있다. 이를 개선해보자.

###### `AppleFilterBehavParam.java`

```java
//Apple 이외의 모든 타입에 대해서 동일한 역할을 할 수 있는 필터를 만든다.
public static <T> List<T> filter(List<T> inventory, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for(T t : inventory){
        if(p.test(t)){
            result.add(t);
        }
    }
    return result;
}
```

