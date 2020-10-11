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
        return Stream.iterate(this, TailCall::apply)
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

여기서 TailCall 객체를 사용하면 대기하고 있는 TailCall 인스턴스의 스트림으로 레이지 이터레이션(Lazy Iteration)을 쉽게 사용할 수 있다. 6장에서 알아본 기술을 활용하면 대기하고 있는 TailCall 인스턴스의 생성을 지연시킬 수 있다.

여러 개의 TailCall 객체에 대한 연속된 인스턴스의 레이지 리스트를 생성하기 위해서 Stream 인터페이스의 iterate() 정적 메서드를 사용한다. 이 메서드는 초기 시드(seed) 값과 제너레이터(generator)를 갖고 있다. 현재 TailCall 인스턴스를 시드로 사용한다.<br>
제너레이터인 UnaryOperator는 현재 엘리먼트를 갖고 다음 엘리먼트를 생성한다. 그리고 나서 다음 대기하고 있는 TailCall 인스턴스를 리턴하는 제너레이터는 현재 TailCall의 apply() 메서드를 사용한다. 제너레이터를 생성하기 위한 목적을 위해 메서드 레펄너스 TailCall::apply를 사용한다.

