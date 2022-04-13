---
date: 2020-10-09 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 7. 재귀 호출 최적화
description: 7. 재귀 호출 최적화
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
category: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---
재귀는 문제를 해결하기 위한 강력하면서도 매혹적인 방법이다.

일반적으로 재귀를 적용해야할 경우는 처리해야 할 데이터의 양이 매우 큰 경우가 많다. 이 때 재귀를 그냥 활용하면 스택오버플로우가 발생할 수 있다.

> 스택 오버 플로우란 하나의 문제를 계속적으로 여러 개의 서브 문제로 분할하면서 해결하게 되는데 서브 문제로 분할을 할 수록 컴퓨터의 스택에 이전 단계의 서브 문제에 대한 상태 정보를 쌓아두게 된다. 어느 정도 스택에 상태 정보가 쌓이게 되면 결국 스택 공간이 모두 차버리고 이때 스택 오버 플로우가 발생할 수 있다.

이 때 테일-콜 최적화(TCO, Tail-Call Optimization)을 활용하면 대규모 입력에 대해서도 재귀를 가능하게 해준다. 또한 메모이제이션(Memoization) 기술을 활용하여 문제들을 빠르게 해결할 수 있다.

# 테일-콜 최적화 사용
재귀의 가장 큰 문제점은 입력 데이터가 매우 많은 경우, 스택 오버 플로우가 발생할 위험이 있다는 것이다. 이 때 TCO 기술을 활용하면 문제를 해결할 수 있다.

테일 콜이란 `마지막 오퍼레이션이 실행되는 위치에 있는 재귀 호출이 자기 자신을 호출하는 방법`을 말한다. 여기서 자기 자신을 호출한다는 것은 재귀 호출의 결과 부분에서 추가적인 연산이 가능하다는 의미다.

자바는 컴파일 레벨에서 TCO를 지원하지는 않지만 람다 표현식을 사용하면 쉽게 TCO를 구현할 수 있다. 이를 종종 트램펄린 호출이라고도 부른다.

## 최적화되지 않은 재귀로 시작하기

```java
public class Factorial {
    public static int factorialRec(final int number) {
        if (number == 1)
            return number;
        else 
            return number * factorialRec(number - 1);   //  팩토리얼을 구하기 위한 숫자를 갖고 있는 상태에서 factorialRec()의 호출에 대한 결과를 계속 기다리게 된다.
    }
}

System.out.println(factorialRec(20000));
```  

* 결과

```text
Exception in thread "main" java.lang.StackOverflowError
	at com.company.Main.factorialRec(Main.java:55)
	...

Process finished with exit code 1
```

위 코드에 number값을 큰 숫자를 입력하여 실행해보면 스택 오버 플로우가 날 수 밖에 없다. 왜냐하면 재귀가 완료될 때까지 연산의 부분 결과를 계속 스택(stack)에 보관하고 있게 되기 때문이다. 따라서 이 문제를 해결하기 위해서는 스택에 저장하지 않고 재귀를 사용할 수 있는 방법이 필요하다.

## 테일 재귀로 변경

```java
public static TailCall<Integer> factorialTailRec(final int factorial, final int number) {
    if (number == 1)
        return done(factorial);
    else
        return call(() -> factorialTailRec(factorial * number, number - 1));
}
```

마지막 오퍼레이션은 자기 자신에 대한 lazy 호출이며, 리턴 결과를 수행하기 위한 추가적인 연산은 없다. 따라서 팩토리얼 메서드인 TailRec()을 계속 호출하는 것보다 지연(lazy/later) 실행을 위해 이 부분을 람다 표현식으로 래핑한다.

## TailCall 함수형 인터페이스의 생성
위에서 본 코드는 factorialTailRec() 메서드를 호출할 때 TailCall의 인스턴스를 즉시 리턴한다.<br>
여기서 중요한 아이디어는 done() 메서드를 호출하면 재귀 종료 시그널을 보내고 call() 메서드를 계속 실행한다면 재귀 호출을 요청하지만 현재 스택 레벨에서 한 스텝 더 들어가게 된 후에 진행된다.

```java
@FunctionalInterface
public interface TailCall<T> {
    TailCall<T> apply();
    
    default boolean isComplete() {return false;}
    default T result() {throw new Error("not implemented");}
    default T invoke() {
        return Stream.iterate(this, TailCall::apply)    //  대기하고 있는 TailCall 인스턴스의 lazy 리스트를 생성
                     .filter(TailCall::isComplete)
                     .findFirst()
                     .get()
                     .result();
    }
}
```

여기서 중요한 것은 invoke() 메서드다. invoke() 메서드는 아래 두 가지를 책임져야 한다.
* 재귀 과정이 끝날 때까지 대기하고 있는 TailCall 재귀 메서드를 통해 반복적으로 이터레이션한다.
* 재귀 과정이 끝에 도달하면, 최종 결과(종단 TailCall 인스턴스의 result() 메서드에 있는)를 리턴해야 한다.


> * iterate() 내부 소스
>
> ```java
>    static <T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
>        Objects.requireNonNull(f);
>        Spliterator<T> spliterator = new AbstractSpliterator<T>(9223372036854775807L, 1040) {
>            T prev;
>            boolean started;
>
>            public boolean tryAdvance(Consumer<? super T> action) {
>                Objects.requireNonNull(action);
>                Object t;
>                if (this.started) {
>                    t = f.apply(this.prev);
>                } else {
>                    t = seed;
>                    this.started = true;
>                }
>
>                action.accept(this.prev = t);
>                return true;
>            }
>        };
>        return StreamSupport.stream(spliterator, false);
>    }
> ```

여기서 TailCall 객체를 사용하면 대기하고 있는 TailCall 인스턴스의 스트림으로 레이지 이터레이션(Lazy Iteration)을 쉽게 사용할 수 있다. 6장에서 알아본 기술을 활용하면 대기하고 있는 TailCall 인스턴스의 생성을 지연시킬 수 있다.

여러 개의 TailCall 객체에 대한 연속된 인스턴의 레이지 리스트를 생성하기 위해서 Stream 인터페이스의 iterate() 정적 메서드를 사용한다. 이 메서드는 초기 시드(seed) 값과 제너레이터(generator)를 갖고 있다. 현재 TailCall 인스턴스를 시드로 사용한다.<br>
제너레이터인 UnaryOperator는 현재 엘리먼트를 갖고 다음 엘리먼트를 생성한다. 그리고 나서 다음 대기하고 있는 TailCall 인스턴스를 리턴하는 제너레이터는 현재 TailCall의 apply() 메서드를 사용한다. 제너레이터를 생성하기 위한 목적을 위해 메서드 레퍼런스 TailCall::apply를 사용한다.

이터레이션이 시드부터 시작하도록 invoke() 메서드를 설계했다. 첫 번째 시드는 TailCall의 첫 번째 인스턴스다. 재귀 과정을 종료하는 시그널을 보내는 TailCall의 인스턴스를 찾을 때까지 제너레이터에 의해 생성되는 연속된 TailCall의 인스턴스를 이터레이션한다.

## TailCalls 컨비니언스 클래스 생성

```java
public class TailCalls {
    public static <T> TailCall<T> call(final TailCall<T> nextCall) {
        return nextCall;
    }

    public static <T> TailCall<T> done(final T value) {
        return new TailCall<T>() {

            @Override
            public boolean isComplete() {
                return true;
            }

            @Override
            public T result() {
                return value;
            }

            @Override
            public TailCall<T> apply() {
                throw new Error("not implemented");
            }
        };
    }
}
```

여기서 **call()** 메서드는 TailCall 인스턴스를 받아서 넘겨주는 역할을 한다. 재귀 호출(ex. factorialRailRect())이 done이나 call 중 하나를 호출하는 것과 같이 대칭적인 호출로 끝낼 수 있도록 해주는 컨비니언스 메서드이다.

**done()** 메서드에서는 TailCall의 특별한 버전을 리턴하며 재귀 과정이 종료되었음을 알려준다. 이 메서드에서 전달받은 값을 특정 인스턴스의 오버라이드 **result()** 메서드로 래핑한다.

특별한 버전의 **isComplete()** 메서드는 true 값을 리턴하도록 오버라이드 하여 재귀를 종료한다.

마지막으로 **apply()** 메서드가 TailCall의 종단 구현에서는 결코 호출되지 않기 때문에 예외를 발생시킨다. 이 예외는 재귀의 끝을 알려주는 것이다.

위와 같은 설계에서 invoke() 디폴트 메서드에서 있는 루프에서 레이지(lazy)하게 평가되기 때문에 간단한 재귀 코드에서 본 것처럼 스택 레벨이 쉽게 증가되지는 않는다.

## 테일-재귀 함수의 사용

* 사용 예제

```java
System.out.println(factorialTailRec(1,20000).invoke());
```  

첫 번째 인수 1은 팩토리얼의 초깃값이고, 20000은 팩토리얼을 계산하는 값이다. factorialTailRec()에 대한 호출은 주어진 숫자가 1인지 체크하여 아니면 call() 메서드를 사용하여 TailCall의 인스턴스를 합성하는 람다 표현식을 넘긴다.

이 합성된 인스턴스는 두 개의 인수를 사용하여 factorialTailRec()을 lazy하게 호출하며 인수는 각각 20000과 1이다.

invoke() 메서드에 대한 호출은 TailCall의 첫 번째 인스턴스를 시드로 사용하는 lazy collection을 생성하고, TailCall의 인스턴스를 종료하라는 시그널을 받을 때까지 컬렉션을 탐색한다. TailCall의 apply 메서드가 호출될 때 앞에서 언급한 두 개의 인수를 사용하여 factorialTailRec()에 대한 함수의 결과를 얻는다.

factorialTailRec()의 두 번째 호출은 done() 메서드에 대한 호출을 한다.

done()에 대한 호출은 TailCall의 특별한 인스턴스를 종료하고 재귀 과정을 종료하도록 시그널을 보낸다.

invoke() 메서드는 현대 연산의 마지막 결과를 리턴한다.

* 결과

```text
0

Process finished with exit code 0
```

초기 코드와 비교하면 스택 오버 플로우는 발생하진 않지만 위와 같이 0이 출력되게 되는데 이는 스택 오버 플로우가 아닌 **산술 오버 플로우**(표시할 숫자가 너무 커서 제대로 계산을 했어도 결과값을 저장하는 변수의 수용 한계를 넘을 때 발생)가 났기 때문이다.

## 산술 오버 플로우의 수정

```java
public class BigFactorial {
    private static TailCall<BigInteger> factorialTailRec(final BigInteger factorial, final BigInteger number) {
        if (number.equals(BigInteger.ONE)) {
            return done(factorial);
        } else {
            return call(() -> factorialTailRec(multiply(factorial, number), decrement(number)));
        }
    }

    public static BigInteger factorial(final BigInteger number) {
        return factorialTailRec(BigInteger.ONE, number).invoke();
    }

    public static BigInteger decrement(final BigInteger number) {
        return number.subtract(BigInteger.ONE);
    }

    public static BigInteger multiply(final BigInteger first, final BigInteger second) {
        return first.multiply(second);
    }

    public final static BigInteger ONE = BigInteger.ONE;
    public final static BigInteger FIVE = new BigInteger("5");
    public final static BigInteger TWENTY = new BigInteger("20000");

    //...
}

public static void main(String[] args) {
    System.out.println(factorial(BigFactorial.TWENTYK));
}
```

* 결과

```text
181920632023034513482764175686645876607160990147875264891806221863456946103855753445383609582775...
```

# 메모이제이션으로 성능 향상
재귀 방법을 사용할 때 발생할 수 있는 문제를 해결하는 방법과 함께 코드의 성능도 좀 더 향상시킬 수 있는 방법들이 있다.
## 최적화 문제
다이내믹 프로그래밍이라는 기술에서 재귀를 사용함으로써 매출에서 가장 큰 수익 찾기, 두 지점 간 최단 거리 구하기 같은 문제를 해결할 수 있다.

> 다이내믹 프로그래밍이란? - 알고리즘 기법 중에 하나로 주어진 문제에 대한 최종적인 솔루션을 구하기 위해 원래 하나였던 문제를 서브 문제로 분할해 해결하는 방법이다.  
> [https://ratsgo.github.io/data%20structure&algorithm/2017/11/15/dynamic/](https://ratsgo.github.io/data%20structure&algorithm/2017/11/15/dynamic/)

다이내믹 프로그래밍을 단순한 재귀 방법으로 구현하게 된다면 입력 데이터가 많아질수록 실행 시간이 기하급수적으로 증가하게 되는데 이 때 `메모이제이션(Memoization)`으로 해결할 수 있다.<br>
메모이제이션 기술을 활용하면 기존에 한 번이라도 수행한 연산을 저장했다가 다시 필요할 때 재활용하기 때문에 재연산할 필요가 없어지면서 입력 데이터 대비 기하급수적으로 늘어나던 연산 시간을 비례하는 정도로만 늘어나게 해준다.

## 다양한 크기에 장작을 팔아 최대 이익을 구하는 프로그램

* 가격표

<table>
  <thead>
    <tr>
      <th>인치</th>
      <th>가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>10</td>
      <td>15$</td>
    </tr>
    <tr>
      <td>9</td>
      <td>9$</td>
    </tr>
    <tr>
      <td>8</td>
      <td>8$</td>
    </tr>
    <tr>
      <td>7</td>
      <td>1$</td>    
    </tr>
    <tr>
      <td>6</td>
      <td>2$</td>
    </tr>
    <tr>
      <td>5</td>
      <td>2$</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2$</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1$</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1$</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2$</td>
    </tr>
  </tbody>
</table>

* 코드 예제

```java
public class RodCutterBasic {
    final List<Integer> prices;

    public RodCutterBasic(List<Integer> prices) {
        this.prices = prices;
    }

    public int maxProfit(final int length) {
        int profit = (length <= prices.size()) ? prices.get(length - 1) : 0;
        for (int i = 1; i < length; i++) {
            int priceWhenCut = maxProfit(i) + maxProfit(length - i);
            if (profit < priceWhenCut) profit = priceWhenCut;
        }

        return profit;
    }
}

public static void main(String[] args) {
    final List<Integer> priceValues = Arrays.asList(2, 1, 1, 2, 2, 2, 1, 8, 9, 15);

    RodCutterBasic rodCutterBasic = new RodCutterBasic(priceValues);

    System.out.println(rodCutterBasic.maxProfit(22));
}
```

이 코드를 연산하는데 상당한 시간이 걸리게된다. 이유는 이 연산의 시간 복잡도가 O(2<sup>n</sup>-1)로 지수 형태로 증가하기 때문이다.

따라서 메모이제이션을 적용하여 실행 속도를 향상시켜본다.
## 결과를 메모이제이션하기
메모이제이션은 재귀 과정이 연산을 계속 반복해서 실행하는 과정을 빠르게 해주는 간단한 방법이다. 메모이제이션을 사용하면 처음으로 연산하는 경우에만 연산을 실행하고, 이전에 한 번이라도 연산을 한 적이 있다면 과거의 연산 결과를 사용한다.<br>
결국 새로운 연산이 발생하면 결과를 저장해두고 그 다음 호출에서 같은 입력의 경우에는 저장해 둔 결과를 재사용한다. 이 기술은 연산이 주어진 입력에 대해 같은 결과를 리턴하는 경우에는 유용하다.

위에서 본 로드-커팅 문제에서는 주어진 길이를 더 작은 길이로 자른 서브 길이에 대한 수식을 찾을 때 해당 길이의 수익이 이미 계산되어 있다면 그에 대한 연산은 하지 않아도 된다. 따라서 프로그램의 실행 속도가 향상되고 수익값을 찾는 작업처럼 반복되는 호출은 해쉬맵(hashmap)을 참조해서 빠르게 실행된다.

* 기존 maxProfit() 수정

```java
    public int maxProfit(final int rodLength) {
        BiFunction<Function<Integer, Integer>, Integer, Integer> compute =
            (func, length) -> {     //  //  메모이제이션 버전에 대한 레퍼런스(func)와 연산에 사용할 파라미터(length)
                int profit = (length <= prices.size()) ? prices.get(length - 1) : 0;
                for (int i = 1; i < length; i++) {
                    int priceWhenCut = func.apply(i) + func.apply(length - i);  //  메모이제이션 레퍼런스에 대한 호출을 재귀로 사용한다. 
                                                                                //  이 것은 값이 캐시되어 있거나 메모이제이션되어 있으면 재빨리 리턴한다.
                                                                                //  그렇지 않으면 람다 표현식을 호출해서 길이에 대한 연산을 재귀 방식으로 리턴한다.
                    if (profit < priceWhenCut) profit = priceWhenCut;
                }
                return profit;
            };
        return callMemoized(compute, rodLength);
    }
```

람다 표현식(compute 변수)에서 태스크를 수행하고 

* 메모이제이션 구현

```java
public class Memoizer {
    public static <T, R> R callMemoized(final BiFunction<Function<T, R>, T, R> function, final T input) {
        Function<T, R> memoized = new Function<T, R>() {

            private final Map<T, R> store = new HashMap<>();

            @Override
            public R apply(final T input) {
                return store.computeIfAbsent(input, key -> function.apply(this, key));
            }
        };

        return memoized.apply(input);
    }
}
```

callMemorized()에서 주어진 입력에 대한 솔루션을 이미 계산해서 갖고 있는지를 체크하는 함수를 구현한다.

computeIfAbsent()를 이용해서 input을 키로 갖는 값이 존재하는지 체크하고 있다면 리턴, 없다면 원래 계획된 함수(function.apply(this, key))를 호출하고 메모이제이션 함수에 대한 레퍼런스를 보내서 해당 함수가 그 다음 연산을 수행할 수 있도록 한다.

돌려보면 이전 버전은 약 45초가 소요되던 것이 메모이제이션 버전에서는 0.15초도 걸리지 않는다.

# 정리
재귀는 프로그래밍에서 상당히 유용한 기법이지만, 재귀를 간단하게 구현하기란 어렵다. 따라서 함수형 인터페이스, 람다 표현식, 무한 스트림 등을 활용하면 테일-콜 최적화를 설계하는 데 도움이 되고 더욱 효과적으로 재귀를 활용할 수 있다. 더 나아가 재귀와 메모이제이션 기술을 혼합하면 오버랩되는 재귀의 성능도 개선할 수 있다.
