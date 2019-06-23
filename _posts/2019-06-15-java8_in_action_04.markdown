---
layout: post
title: "Java 8 in Action Chapter04 - 스트림 소개"
date: 2019-06-15
categories:
---

4장과 5장은 소스코드를 한 패키지에 넣어놨을 만큼 서로 결합도가 높은 내용이다. 스트림을 드디어 알게 되었다 현재 소속된 프로젝트에서 개발자들이 사용했던 스트림 관련 코드를 이해할 수 있게 되었고, 나도 동일하게 반복문을 사용한 외부반복으로 처리하던 것을 간결하고 명료한 스트림 연산을 쓸 수 있게 되었다.

- [**공부한 내용 Github 에서 보기**](https://github.com/joshuafromkorea/Java_8_In_Action_Study/tree/85c0465d8b47e666137bc23097bb804f7c3cb477)

---

## Chapter 4 스트림 소개

> 거의 모든 자바 애플리케이션은 컬렉션을 **만들고 처리하는 과정**을 포함한다

* 기존 자바 7까지는 컬렉션을 사용했지만, 완벽한 컬렉션 관련 연산을 지원하기엔 부족했다.
  * 컬렉션의 **그룹화**, 컬렉션 내부의 **검색**에 있어서 마치 SQL 수준의 간결성과 편의성을 가지기 힘들었다
  * 또한 병렬 처리에 있어서, 복잡한 코드와 반복적인 코드를 요구 했다.
* 자바 8은 **스트림**을 통해서 이 문제를 해결했다.

### 4.1 스트림이란 무엇인가?

1. **선언형**으로 컬렉션 연산을 처리할 수 있게 해준다.
   * 반복문이나 조건문으로 컬렉션 외부에서 동작을 구현할 필요 없이, 람다를 활용해 **선언형**으로 처리한다.
2. **투명하게** 병렬로 연산을 처리할 수 있게 해준다.
   * 파이프 라인을 만들어서, 병렬 처리를 이뤄낸다.

앞으로 사용할 예제는 다음과 같다. 

###### `Dish.java`

```java
public class Dish {
    private final String name;
    private final boolean vegeterian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegeterian, int calories, Type type) {
        this.name = name;
        this.vegeterian = vegeterian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegeterian() {
        return vegeterian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return name;
    }

    public enum Type{
        MEAT, FISH, OTHER
    }
}
```

###### `MenuBuilder.java`

```java
public class MenuBuilder {
    private final static List<Dish> menu =Arrays.asList(
            new Dish("pork", false, 800, Dish.Type.MEAT),
            new Dish("beef", false, 700, Dish.Type.MEAT),
            new Dish("chicken", false, 400, Dish.Type.MEAT),
            new Dish("french fries", true, 530, Dish.Type.OTHER),
            new Dish("rice", true, 350, Dish.Type.OTHER),
            new Dish("season fruit", true, 120, Dish.Type.OTHER),
            new Dish("pizza", false, 550, Dish.Type.OTHER),
            new Dish("prawns", false, 300, Dish.Type.FISH),
            new Dish("salmon", false, 450, Dish.Type.FISH)

            );

    public static List<Dish> getMenu(){
        return menu;
    }
}
```

### 4.2 스트림 시작하기

> **스트림이란** '**데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**'로 정의할 수 있다.

* **연속된 요소**
  * 컬렉션이 시공간의 복잡성을 생각해야 하는 **자료구조**라면, 스트림은 자료들에 대한 **계산**이 주가된다.
* **소스**
  * 스트림은, 컬렉션, 배열, I/O 자원등의 데이터를 소스로하여, 해당 데이터를 **소비<sup>consume</sup>**한다.
* **데이터 처리 연산**
  * 데이터베이스에서 지원하는 연산과 유사한 연산을 메소드를 호출하는 방식을 통해 지원한다.
* **파이프라이닝**
  * 스트림 연산을 서로 **연결**하는 방식으로 커다란 파이프라인을 구성한다.
* **내부반복**
  * 반복자를 이용하는 기존의 처리는 외부에서 연산을 처리한다면, 스트림은 내부에서 연산을 처리한다.

위의 스트림에 대한 정의들을 코드로 간단히 살펴보면 다음과 같다.

```java
List<String> threeHighCaloricDishNames = 
    menu.stream() //소스에서 스트림을 얻는다
    .filter(d->d.getCalories() > 300) //파이프라인 연산을 만들어 고칼로리 필터링
    .map(Dish::getName) // 이름만 추출하는 질의어
    .limit(3) // 선착순으로 3개만 선택
    .collect(toList()); // 결과를 도출하기 위한 저장
```

사용된 스트림의 메소드들을 간단히 살펴보면 아래와 같다.

* `filter`: **람다**를 인수로 받아, 특정 요소를 제외 시킨다.
* `map`: **람다**를 활용해서, 한 요소를 다른 요소로 변환하거나 정보를 추출한다.
* `limit`: 스트림의 크기를 축소 한다.
* `collect`: 스트림을 다른 형식으로 변환 한다.

### 4.3 스트림과 컬렉션

> 데이터를 **언제** 계산하느냐가 컬렉션과 스트림의 가장 큰 차이라고 할 수 있다.

* 컬렉션은 연산을 위해서 항상 **모든 값**을 메모리에 저장하는 것을 필요로 한다.
  * 이에 비해서 스트림은 **요청할 때만 요소를 계산**하는 자료구조다
* 결과적으로 스트림은 소스에서 **생산**된 스트림을 **소비**하는 과정을 지니는 연산을 하게 된다.
  * 또한 스트림은 게으르게 만들어지는 컬렉션으로, 사용자가 요청할 때만 연산한다.

좀더 자세히 차이를 살펴보자

#### 4.3.1 딱 한 번만 탐색할 수 있다!

* 반복자를 사용한 구현이 한번만 탐색할 수 있는 것처럼, 스트림도 한 번만 탐색가능하다.
  * 스트림은 이러한 탐색을 **소비**라고 표현하는데, 한번 소비되면 자시한번 소스에서 새로운 스트림을 만들어야 한다.

#### 4.3.2 외부 반복과 내부 반복

* 컬렉션은 사용자가 직접 요소를 (for-each)등을 사용해서 **외부 반복**해야 한다.
  * 스트림 라이브러리는 명령을 내리면 반복을 **내부 반복**으로 처리해서 반환해준다.

### 4.4 스트림 연산

> 스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있다.

* **중간 연산**: 다른 스트림 연산과 연결할 수 있는 스트림연산
* **최종 연산**: 스트림을 닫아서 소비를 종료하는 연산

#### 4.4.1 중간 연산

> 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다. 중간 연산의 중요한 특징은 ***최종 연산*을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는 다는 것**, 즉 게으르다는 것이다.

* 아래의 예시가 **게으르다**라는 스트림 연산의 특징을 잘 보여줄 수 있는 예시코드이다.

```java
@Test
public void 중간연산_게으름_테스트(){
    List<String> names = menu.stream()
            .filter(d->{
                System.out.println("filtering " + d.getName());
                return d.getCalories() >300;
            })
            .map(d->{
                System.out.println("mapping " + d.getName());
                return d.getName();
            })
            .limit(3)
            .collect(toList());
    System.out.println(names);
}
```

* `.limit(3)` 연산으로 인해, `.filter()`연산이나 `.map()`연산 대상이 3가지 요소만을 대상으로 하도록 축소된다 테스트를 수행하면 아래의 콘솔 결과로 확인할 수 있다.

```shell
filtering pork
mapping pork
filtering beef
mapping beef
filtering chicken
mapping chicken
[pork, beef, chicken]
```

#### 4.4.2 최종 연산

> 최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 List, Integer, void 등의 **스트림 이외의 결과**가 반환된다.

#### 4.4.3 스트림 이용하기

> 스트림 이용과정은 다음과 같이 세 가지로 요약할 수 있다.

* 질의를 수행할 (컬렉션 같은) **데이터 소스**
* 스트림 파이프라인을 구성할 **중간 연산**연결

* 스트림 파이프라인을 **실행**하고 결과를 만들 **최종 연산**

앞으로 자주 살펴보게 될 스트림 API들을 살펴보자

##### 중간연산

| 연산       | 연산의 인수      | 함수 디스크럽터 |
| ---------- | ---------------- | --------------- |
| `filter`   | `Predicate<T>`   | T -> boolean    |
| `map`      | `Function<T, R>` | T -> R          |
| `limit`    |                  |                 |
| `sorted`   | `Comparator<T>`  | (T, T) -> int   |
| `distinct` |                  |                 |

##### 최종연산

| 연산      | 목적                                                         |
| --------- | ------------------------------------------------------------ |
| `forEach` | 스트림의 각 요소를 소비하면서 람다를 적용한다. `void`를 반환한다. |
| `count`   | 스트림의 요소 개수를 반환한다. `long`을 반환한다.            |
| `collect` | 스트림을 리듀스해서, 리스트, 맵, 정수 형식의 컬렉션을 만든다. |

