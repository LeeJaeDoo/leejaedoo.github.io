---
date: 2020-11-09 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 8. 람다 표현식의 조합
description: 8. 람다 표현식의 조합
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
categories: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---
독립적인 연산을 scatter하고 최종 솔루션을 구하기 위해 각각의 결과를 gather하는 역할을 하는 맵리듀스(MapReduce)패턴을 통해 함수 조합에 대해 알아본다.
# 함수 조합의 사용
함수형 스타일로 프로그래밍하게 되면 고차 함수를 조합하고 불변성과 함수들의 사용을 가능한 최대로 활용할 수 있다. OOP와 자바에서 새롭게 제공하는 함수형 스타일을 함께 사용하면 기존의 개발 경험을 더욱 향상할 수 있다.

객체를 연속된 함수에 넘기는 것처럼, 원하는 결과를 얻기 위해 연속된 함수에게 전달되면서 새로운 객체로 변형된다.

이를 활용한 예제를 알아본다. 외부 이터레이터를 통해 리스트를 정려했고 변경할 수 있는 Collection으로 업데이트했다. 객체를 변경하는 대신, 현재의 주식 시세표 리스트를 $100 이상의 가격으로 된 주식 시세표의 리스트로 필터링한다. 그리고 나서 리스트를 정렬하고 마지막으로 결과를 리포트한다.

```java
public class Tickers {
    public static final List<String> symbols = Arrays.asList(
        "AMD", "HPQ", "IBM", "TXN", "VMW", "XRX", "AAPL", "ADBE",
        "AMZN", "CRAY", "CSCO", "SNE", "GOOG", "INTC", "INTU",
        "MSFT", "ORCL", "TIBX", "VRSN", "YHOO" 
    );
}

final BigDecimal HUNDRED = new BigDecimal("100");
System.ut.println("Stocks priced over $100 are " +
    Tickers.symbols
           .stream()
           .filter(symbol -> YahooFinance.getPrice(symbol).compareTo(HUNDRED) > 0)
           .sorted()
           .collect(joining(", ")));
```
일련의 오퍼레이션들이 체인으로 묶여서 동작한다. 조합 함수를 사용하게 되면 심볼 리스트는 아래와 같은 순서대로 처리된다.

1. 원래의 심볼 리스트는 `수정하지 않고 그대로 둔 상태에서 리스트를 심볼의 필터링된 스트림으로 변경`한다.
2. 그리고 나서 정렬된 심볼의 스트림으로 바꾼다.
3. 마지막으로 출력을 위해 최종 스트림에 있는 심볼들을 서로 연결시킨다.(chaining)

위와 같이 함수의 조합을 오퍼레이션의 체인으로 묶는 기능은 코드를 좀 더 이해하기 쉽게 해주고 가변성을 줄이며 오류가 발생할 확률을 낮춰주고 또한 쉽게 병렬화할 수 있도록 해준다.   
# 맵리듀스의 사용
맵리듀스 패턴에서는 아래와 같이 두 가지의 오퍼레이션을 사용한다.
1. 컬렉션에 있는 각 엘리먼트에 대해 실행하는 오퍼레이션(중간 연산 - ex. peek(), filter())
2. 최종 결과를 위해 기존의 결과를 조합하는 오퍼레이션(최종 연산 - ex. collect(), count())

이 패턴은 간단하면서도 멀티코어 프로세서를 활용하는 강력한 기능을 갖고 있다.

JVM은 멀티코어 프로세서의 활용을 극대화하기 위한 기능을 제공한다.

먼저 주식 시세표 심볼 리스트에서 $500 보다 작은 주식 중 가장 가격이 높은 주식을 선택해보는 예제를 알아본다.

* 연산을 위한 준비 예제

```java
public class StockInfo {
    public final String ticker;
    public final BigDecimal price;
    public StockInfo(final String symbol, final BigDecimal thePrice) {
        ticker = symbol;
        price = thePrice;
    }

    public String toString() {
        return String.format("ticker: %s price: %g", ticker, price);
    }
}

public class StockUtil {
    public static StockInfo getPrice(final String ticker) {
        return new StockInfo(ticker, YahooFinance.getPrice(ticker));
    }

    public static Predicate<StockInfo> isPriceLessThan(final int price) {
        return stockInfo -> stockInfo.price.compareTo(BigDecimal.valueOf(price)) < 0;
    }

    public static StockInfo pickHigh(final StockInifo stock1, final StockInfo stock2) {
        return stock1.price.compareTo(stock2.price) > 0 ? stock1 : stock2;
    }
}
```

* 명령형 스타일 예제

```java
final List<StockInfo> stocks = new ArrayList<>();
for (String symbol : Tickers.symbols) {
    stock.add(StockUtil.getPrice(symbol));
}

final List<StockInfo> stocksPriceUnder500 = new ArrayList<>();
final Predicate<StockInfo> isPriceLessThan500 = StockUtil.isPriceLessThan(500);
for (StockInfo stock : stocks) {
    if (isPriceLessThan500.test(stock)) {
        stocksPriceUnder500.add(stock);
    }
}

StockInfo highPriced = new StockInfo("", BigDecimal.ZERO);
for (StockInfo stock : stocksPricedUnder500) {
    highPriced = StockUtil.pickHigh(highPriced, stock);
}

System.out.println("High priced under $500 is " + highPriced);
```

* 개선된 명령형 스타일 예제

```java
StockInfo highPriced = new StockInfo("", BigDecimal.ZERO);
final Predicate<StockInfo> isPriceLessThan500 = StockUtil.isPriceLessThan(500);

for (String symbol : Tickers.symbols) {
    StockInfo stockInfo = StockUtil.getPrice(symbol);

    if (isPriceLessThan500.test(stockInfo)) {
        highPriced = StockUtil.pickHigh(highPriced, stockInfo);
    }
}

System.out.println("High priced under $500 is " + highPriced);
```

* 함수형 스타일 예제

```java
public static void findHighPriced(final Stream<String> symbols) {
    final StockInfo highPriced =
        symbols.map(StockUtil::getPrice)
               .filter(StockUtil.isPriceLessThan(500)) 
               .reduce(StockUtil::pickHigh)
               .get();

    System.out.println("High priced under $500 is " + highPriced);  
}
```

# 병렬화 적용
코드를 병렬화 처리하기 위해선 태스크를 다중 스레드로 나누고, 결과를 모은 후에 순차적 과정으로 옮겨야 한다. 이 작업을 하는 동안 `race condition`이 발생하지 않도록 해야 한다. `다른 스레드가 데이터를 업데이트하는 동안 스레드들이 충돌하거나 데이터를 망치면 안된다.`

여기서 병렬화 처리를 위해 고려해야할 두 가지 문제가 있다.

1. 어떻게 처리해야 하는가.
2. 적절하게 하려면 어떻게 해야하는가.

1번은 스레드를 관리해주는 라이브러리의 도움을 받으면된다. 레이스 컨디션은 공유되고 있는 가변성에서 대부분 발생한다. 다중 스레드들이 같은 시각에 객체나 변수를 업데이트하려고 하면 스레드 세이프티를 보장해야 한다. 이 문제는 함수형 스타일과 불변성 원칙을 따르면 해결된다.

stream()과 parallelStream()은 같은 리턴 타입을 갖지만 리턴하는 인스턴스는 전혀 다르다.<br>
parallelStream()의 리턴 인스턴스는 다중 스레드에서 병렬로 실행할 수 있는 map()이나 filter()와 같은 메서드이며 내부적으로 스레드 풀에 의해 관리된다. 장점은 순차 버전을 병렬 버전으로 쉽게 바꿀 수 있다는 점이다.

stream()을 호출할 지, parallelStream()을 호출할 지 결정할 때 몇 가지 이슈를 고려해야 한다.

1. 람다 표현식을 동시에 실행하고 싶은가?
2. 코드는 사이드 이펙트나 레이스 컨디션 없이 독립적으로 실행 가능한가?
3. 솔루션이 올바르다는 것은 동시에 실해되도록 스케줄된 람다 표현식의 실행 순서와는 독립적이어야 한다. 예를 들어, forEach() 메서드에 대한 호출과 람다 표현식을 통한 결과 출력을 병렬화하는 것은 맞지 않다. 병렬화된 코드에서는 어떤 메서드가 먼저 실행하고, 어떤 메서드가 나중에 실행되는지와 같은 실행 순서를 예측할 수 없기 때문이다.<br>
한편, map()과 filter()와 같은 메서드는 병렬화하기에 적합한 예이다.

# 정리
람다 표현식은 함수를 조합해서 오퍼레이션의 체인이 되도록 하며, 이 체인은 풀어야 할 문제를 객체 변형의 결합 집합으로 바꾸어 준다.<br>
게다가 불변성을 보장하고 사이드 이펙트를 막아주기 때문에 체인 오퍼레이션의 부분들에 대한 실행을 쉽게 병렬화하고 속도 향상의 장점을 얻을 수 있다.
