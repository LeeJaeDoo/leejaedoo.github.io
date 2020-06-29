---
date: 2020-06-28 18:40:40
layout: post
title: 자바 8 인 액션
subtitle: 6. 스트림으로 데이터 수집
description: 6. 스트림으로 데이터 수집
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---

* 통화별로 트랜잭션을 그룹화
명령형 프로그래밍

```java
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>(); //  그룹화한 트랜잭션을 저장할 맵을 생성

for (Transactin transaction : transactions) {   //  트랜잭션 리스트를 반복
    Currency currency = transaction.getCurrency();
    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);

    if (transactionForCurrency = null) {    //  현재 통화를 그룹화하는 맵에 항목이 없으면 항목을 만든다.
        transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
    }

    transactinsForCurrency.add(transaction);    //  같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랜잭션을 추가
}
``` 

함수형 프로그래밍
```java
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

> 함수형 프로그래밍에선 `무엇`을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다.

## 컬렉터란 무엇인가
multilevel로 그룹화를 수행할 때 명령형 코드는 가독성과 유지보수성이 크게 떨어진다. 하지만 함수형 코드는 필요한 컬렉터를 쉽게 추가할 수 있다.

### 고급 리듀싱 기능을 수행하는 컬렉터
컬렉터의 최대 강점은 collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이다.<br>
즉, 스트림에 collect를 호출하면 스트림의 요소에 컬렉터로 파라미터화된 리듀싱 연산이 수행된다. collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.<br>
![collector](../../assets/img/collector.jpeg)
보통 함수를 요소로 변환할 때는 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적한다.<br>
즉, Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할 지 결정된다.

### 미리 정의된 컬렉터
groupingBy와 같이 Collectors 클래스에서 제공하는 팩토리 메서드의 기능을 설명한다. 이를 미리 정의된 컬렉터라 표현한다.
* Collectors에서 제공하는 메서드의 기능
    * 스트림 요소를 하나의 값으로 리듀스하고 요약
    * 요소 그룹화
    * 요소 분할

## 리듀싱과 요약
Collector 팩토리 클래스로 만든 컬렉터 인스턴스로 무엇을 할 수 있을까.
* 개수 카운팅

```java
long howManyDishes = menu.stream().collect(Collectors.counting());

long howManyDishes = menu.stream().count();     //  위에서 더 간소화
```

### 스트림값에서 최댓값과 최솟값 검색
`Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드로 `스트림의 최댓값, 최솟값을 계산`할 수 있다.<br>

### 요약 연산
스트림에 있는 `객체의 숫자 필드의 합계나 평균 등을 반환하는 연산`에도 리듀싱 기능이 자주 사용된다. 이러한 연산을 요약(summarization) 연산이라 부른다.
Collectors 클래스는 `Collectors.summingInt`라는 특별한 요약 팩토리 메서드를 제공한다.
```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));   //  메뉴 리스트의 총 칼로리의 합을 계산한다.
```

> menu.stream().map(Dish::getCalories).reduce(0, Integer::sum)와의 차이는?

* summingInt 컬렉터의 누적 과정
![summingint](../../assets/img/summingint.jpeg)
칼로리로 매핑된 각 요리의 값을 탐색하면서 초깃값(0)으로 설정되어 있는 누적자에 칼로리를 더한다.

Collectors.averagingInt/Long/Double 으로 평균값 계산과 같은 요약 기능을 활용할 수 있다.
```java
double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

또한, `두 개 이상의 연산을 한번에 수행해야 될 때`는 팩토리 메서드 `summarizingInt`가 반환하는 컬렉터를 사용할 수 있다.
```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));

// 결과값 IntSummaryStatistics{count=9, sum=4200, min=120, average=466.666667, max=800}
```
위와 같이 메뉴에 있는 요소의 수, 칼로리 합계, 평균, 최댓값, 최솟값을 한 번에 계산할 수 있다.

### 문자열 연결
컬렉터의 `joining` 팩토리 메서드를 이용하면` 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결`해서 반환한다.
```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```
Dish 클래스에 name을 반환하는 toString 메서드를 포함하고 있다면 map처리과정을 생략할 수 있다.

> joining 메서드는 내부적으로 `StringBuilder`를 이용해서 문자열을 하나로 만든다.

### 범용 리듀싱 요약 연산
위의 모든 요약 연산들은 reducing 팩토리 메서드로도 구현할 수 있다. 위 처럼 특화된 팩토리 메서드를 사용한 이유는 프로그래밍적 편의성 때문이다.

* reducing 메서드를 활용한 칼로리 합계

```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```

reducing은 3개의 인수를 받는다.
* 0 : 리듀싱 연산의 시작값 혹은 스트림에 인수가 없을 때 반환 값
* Dish::getCalories : 변환 함수
* (i, j) -> i + j : 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator이다.

한 개의 인수를 받는 reducing 컬렉터는 시작값이 없으므로 빈 스트림이 넘겨졌을 때, 시작값이 설정되지 않게되므로 Optional<> 객체를 반환하게 된다.

> collect vs reduce
> collect 메서드 : 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드
> reduce 메서드 : 두 값을 하나로 도출하는 불변형 연산. 따라서 누적 연산으로 활용하게 되면 의미론적으로 잘못 사용한 것이다.

#### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행할 수 있다.
```java
int totalCalories = menu.stream().collect(reducing(0,       //  초깃값
                                          Dish::getCalories, // 변환 함수
                                          Integer::sum));   //  합계 함수
``` 
이전 예제의 람다 표현식 대신 Integer 클래스의 sum 메서드 레퍼런스를 활용하면 코드를 더 단순화 시킬 수 있다.
counting 컬렉터도 3개의 인수를 갖는 reducing 팩토리 메서드를 이용해서 구현할 수 있다.
```java
public static <T> Collector<T, ?, Long> counting() {
    return reducing(0L, e -> 1L, Long::sum);
}
```
스트림의 Long 객체 형식의 요소를 1로 변환한 다음에 모두 더할 수 있다.

> 제네릭 와일드 카드 ? 사용법
> 위 예제에서 ?는 컬렉터의 누적자 형식이 알려지지 않았음을, 즉 누적자의 형식이 자유로움을 의미한다.

한 개의 인수를 갖는 reduce 스트림과 같이 reduce(Integer:sum)도 빈 스트림과 같은 문제를 피할 수 있도록 Optional<Integer>를 반환한다.

#### 자신의 상황에 맞는 최적의 방법 선택
스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것 보다 컬렉터를 이용하는 코드가 더 복잡한 경우를 알 수 있었다. 대신, 재사용성과 커스터마이즈 기능을 통해 추상화와 일반화를 얻을 수 있었다.