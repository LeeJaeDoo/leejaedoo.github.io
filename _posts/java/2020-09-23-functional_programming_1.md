---
date: 2020-09-23 15:40:40
layout: post
title: 자바 8 인 액션
subtitle: 14. 함수형 프로그래밍 기법
description: 14. 함수형 프로그래밍 기법
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
categories: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
# 함수는 모든 곳에 존재한다.
함수형 프로그래밍은 함수를 마치 일반값처럼 사용해서 인수로 전달하거나, 결과로 반환받거나, 자료구조에 저장할 수 있음을 의미힌다. 일반값처럼 취급할 수 있는 함수를 `일급 함수(first-class function)`라고 한다.<br>
바로 자바 8의 특징 중 하나가 일급 함수를 지원한다는 점이다. **메서드 레퍼런스**나 **람다 표현식**으로 직접 함수값을 표현하여 메서드를 함수값으로 사용할 수 있다.
## 고차원 함수
지금까지는 1, 2장에서 본 것 처럼 함수값으로 `Apple::isGreenApple`를 전달해서 동작 파라미터화를 달성하는 용도로만 사용했다. 이는, 함수값 활용의 일부에 불과했다.<br>
`Comparator.comparing` 처럼 함수를 인수로 받아서 다른 함수로 반환하는 정적 메서드 형태로도 활용할 수 있다.
자바 8에서는 함수를 인수로 전달하거나 결과로 반환하고, 지역 변수로 할당하거나, 구조체로 삽입할 수 있으므로 자바 8의 함수도 고차원 함수라고 할 수 있다.

스트림 연산과 마찬가지로 고차원 함수를 적용할 때도 전달되는 함수에 부작용이 없어야 하며, 부작용을 포함하는 함수를 사용하면 문제가 발생한다. 고차원 함수나 메서드를 구현할 때 어떤 인수가 전달될지 알 수 없으므로 인수가 부작용을 포함할 가능성을 염두에 두어야 한다.<br>
함수를 인수로 받아 사용하면서 코드가 정확히 어떤 작업을 수행하고 프로그램의 상태를 어떻게 바꿀지 예측하기 어려워진다. 또한 디버깅도 어려워진다. 따라서 인수로 전달된 함수가 어떤 부작용을 포함하게 될지 정확하게 문서화하는 것이 좋다. 
## 커링
**커링**이란 함수를 모듈화하고 코드를 재사용하는 데 도움을 주는 기법이다.

* 변환 패턴 적용 전 예제

```java
static double converter(double x, double f, double b) {
    return x * f + b;
}
```

* 적용 후

```java
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}

DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);

double gbp = convertUSDtoGBP.applyAsDouble(1000);
```

x, f, b라는 세 인수를 converter 메서드로 전달하지 않고 f, b 두 가지 인수로 함수를 요청하고 있다. 그리고 반환된 함수에 인수 x를 이용해서 `x * f + b`라는 결과를 얻는다. 이런 방식으로 변환 로직을 재활용할 수 있으며 다양한 변환 요소로 다양한 함수를 만들 수 있다.
# 영속 자료구조
영속 자료구조란 함수형 프로그램에서의 함수형 자료구조, 불변 자료구조 등을 지칭한다.

함수형 메서드에서는 전역 자료구조나 인수로 전달된 구조를 갱신할 수 없다. 자료구조를 바꾼다면 같은 메서드를 두 번 호출했을 때 결과가 달라지면서 참조 투명성에 위배되고 인수를 결과로 단순하게 매핑할 수 있는 능력이 사라지기 때문이다. 
# 스트림과 게으른 평가
스트림은 단 한 번만 소비할 수 있어 재귀적으로 정의할 수 없다.
## 자기 정의 스트림

* 소수를 생성하는 재귀 스트림 예제

```java
// 1.숫자 스트림 생성
static IntStream numbers() {
    return IntStream.iterate(2, n -> n + 1);
}

// 2.머리 획득
static int head(IntStream numbers) {
    return numbers.findFirst().getAsInt();
}

// 3.꼬리 필터링
static IntStream tail(IntStream numbers) {
    return numbers.skip(1);
}

// 4.재귀적으로 소수 스트림 생성
static IntStream primes(IntStream numbers) {
    int head = head(numbers);
    return IntStream.concat(
        IntStream.of(head),
        primes(tail(numbers).filter(n -> n % head != 0))
    );
}
```
### 나쁜 소식
하지만 예상과 다르게 4번 코드를 실행하게 되면 `Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed` 라는 에러가 발생하는데 이는 2번과 3번에서 사용한 **findFirst**와 **skip**이 최종 연산이기 때문에 최종 연산을 스트림에 호출하게 되면서 스트림이 이미 완전 소비된 상태에서 계속 호출이 되면서 발생되게 된다.

### 게으른 평가
또 한가지 문제가 있다. 정적 메서드 IntStream.concat은 두 개의 스트림 인스턴스를 인수로 받는다. 두 번째 인수가 primes를 직접 재귀적으로 호출하면서 무한 재귀에 빠질 수가 있는데 이를 `lazy evaluation`이란 제한을 둠으로써 `재귀적 정의를 허용하지 않는다`라는 자바 8의 스트림 규칙이 발생하지 않게 된다. 즉, 소수를 처리할 필요가 있을 때만 스트림을 실제로 평가한다.

> lazy evaluation이란? [https://dororongju.tistory.com/137](https://dororongju.tistory.com/137)
## 게으른 스트림 만들기
자바 8의 스트림은 요청할 때만 값을 생성하는 블랙박스와 같다. 스트림에 일련의 연산을 적용하면 연산이 수행되지 않고 일단 저장된다. 그리고나서 스트림에 **최종 연산**을 적용해서 실제 계산을 해야하는 상황에서만 실제 연산이 이뤄진다.<br>
이러한 `게으른 특성` 때문에 각 연산 별로 스트림을 탐색할 필요 없이 한 번에 여러 연산을 처리할 수 있다.

* 기본적인 연결 리스트

```java
interface MyList<T> {
    T head();

    MyList<T> tail();

    default boolean isEmpty() {
        return true;
    }
}

class MyLinkedList<T> implements MyList<T> {
    private final T head;
    private final MyList<T> tail;
    public MyLinkedList(T head, MyList<T> tail) {
        this.head = head;
        this.tail = tail;
    }

    public T head() {
        return head;
    }

    public MyList<T> tail() {
        return tail;
    }

    public boolean isEmpty() {
        return false;
    }
}

class Empty<T> implements MyList<T> {
    public T head() {
        throw new UnsupportedOperationException();
    }

    public MyList<T> tail() {
        throw new UnsupportedOperationException();
    }
}

MyList<Integer> i = new MyLinkedList<>(5, new MyLinkedList<>(10, new Empty<>()));
``` 

* 기본적인 게으른 리스트

```java
import java.util.function.Supplier;

class LazyList<T> implements MyList<T>{
    final T head;
    final Supplier<MyList<T>> tail;
    public LazyList(T head, Supplier<MyList<T>> tail) {
        this.head = head;
        this.tail = tail;
    }

    public T head() {
        return head;
    }

    public MyList<T> tail() {
        return tail.get();      //  위의 head와 달리 tail에서는 Supplier로 게으른 동작을 만듦.
    }

    public boolean isEmpty() {
        return false;
    }
}
```

Supplier<T>를 이용해서 게으른 리스트를 만들게 되면 tail 모두 메모리에 존재하지 않게 할 수 있다.(게으른 특성을 활용)

일반적으로 eagerly 기능 보다 게으른 편이 성능이 좋은 편이다. 모든 값을 계산해버리는 것 보다는 요청했을 때 값을 계산하는 것이 여러 면에서 더 좋다.(물론 현실은 다른 경우가 있다.)

이처럼 게으른 자료구조는 강력한 프로그래밍 도구다. 
# 패턴 매칭
일반적으로 함수형 프로그래밍에서 또 하나의 특징으로 (구조적인) `패턴 매칭(pattern matching)`을 들 수 있다.
## 방문자 디자인 패턴
자바에서는 `방문자 디자인 패턴(visitor design pattern)`으로 자료형을 언랩할 수 있다. 특히 특정 데이터 형식을 '방문' 하는 알고리즘을 캡슐화하는 클래스를 따로 만들 수 있다.

* 방문자 패턴 예제

```java
class BinOp extends Expr {
    ...
    public Expr accept(SimplifyExprVisitor v) {
        return v.visit(this);
    }
}

public class SimplifyExprVisitor {
    ...
    public Expr visit(BinOp e) {
        if ("+".equals(e.opname) && e.right instanceof Number && ...) {
            return e.left;
        }
        return e;
    }
} 
```
SimplifyExprVisitor를 인수로 받는 accept를 BinOp에 추가한 다음 BinOp 자신을 SimplifyExprVisitor로 전달한다. 그러면 SimplifyExprVisitor는 BinOp 객체를 언랩할 수 있다.
## 패턴 매칭의 힘
자바는 패턴 매칭을 지원하지 않지만 람다를 활용하면 흉내는 낼 수 있다.

* 패턴 매칭 적용 전 예제

```scala
Expr simplifyExpression(Expr expr) {
    if (expr instanceof BinOp
        && ((BinOp)expr).opname.equals("+"))
        && ((BinOp)expr).right instanceof Number
        && ...  //  코드가 깔끔하지 못하다.
        && ... ) {
        return (BinOp)expr.left;
    }
}
```

* 스칼라 패턴 매칭 예제

```scala
def simplifyExpression(expr: Expr): Expr = expr match {
    case BinOp("+", e, Number(0)) => e   // 0 더하기  
    case BinOp("*", e, Number(1)) => e   // 1 곱하기 
    case BinOp("/", e, Number(1)) => e   // 1로 나누기 
    case _ => expr   //  expr을 단순화할 수 있다.
}
```

### 자바로 패턴 매칭 흉내 내기
자바 8의 람다를 이용한 패턴 매칭 흉내는 단일 수준의 패턴 매칭만 지원한다.

```java
interface TriFunction<S, T, U, R> {
    R apply(S s, T t, U u);
}

static <T> T patternMatchExpr(Expr e, TriFunction<String, Expr, Expr, T> binopcase, Function<Integer, T> numcase, Supplier<T> defaultcase) {
    return (e instanceof BinOp) ?
             binopcase.apply(((BinOp)e).opname, ((BinOp)e).left, ((BinOp)e).right) :
             (e instanceof Number) ?
             numcase.apply(((Number)e).val) :
             default.get();
}

public static Expr simplify(Expr e) {
    TriFunction<String, Expr, Expr, Expr> binopcase =   //  BinOp 표현식 처리
        (opname, left, right) -> {
            if ("+".equals(opname)) {
                if (left instanceof Number && ((Number) left).val == 0) { return right; }
                if (right instanceof Number && ((Number) right).val == 0) { return left; }
            }
            if ("*".equals(opname)) {
                if (left instanceof Number && ((Number) left).val == 1) { return right; }
                if (right instanceof Number && ((Number) right).val == 1) { return left; }
            }
            return new BinOp(opname, left, right);
        };
    Function<Integer, Expr> numcase = val -> new Number(val);   //  숫자 처리
    Supplier<Expr> defaultcase = () -> new Number(0);           //  수식을 인식할 수 없을 때 기본 처리

    return patternMatchExpr(e, binopcase, numcase, defaultcase);    //  패턴 매칭 적용
} 

Expr e = new BinOp("+", new Number(5), new Number(0));
Expr match = simplify(e);
System.out.println(match);  //  5 출력 
```
# 기타 정보
## '같은 객체를 반환함'은 무엇을 의미하는가?
`참조 투명성(referentially transparent)`이란 '인수가 같다면 결과도 같아야 한다'라는 규칙을 만족함을 의미한다.<br>
하지만 여기서 한가지 궁금점이 있는데 동등성은 만족해도 동일성을 만족하지 못하는 경우가 있다면 이도 참조 투명성을 만족하다고 할 수 있느냐다. 결과부터 말하자면 만족한다고 볼 수 있다.

함수형 프로그래밍에서는 데이터가 변경되지 않으므로 같다는 의미 == 이 아니라 구조적인 값이 같다는 것을 의미한다. 
## 콤비네이터
함수형 프로그래밍에서는 두 함수를 인수로 받아 다른 함수를 반환하는 등 함수를 조합하는 고차원 함수를 많이 사용하게 된다. 이처럼 함수를 조합하는 기능을 `콤비네이터`라고 부른다.(ex. CompletableFuture.thenCombine)

* 함수 조합 개념

```java
static <A, B, C> Function<A, C> compose(Function<B, C> g, Function<A, B> f) {
    return x -> g.apply(f.apply(x));
}
```

compose함수는 f와 g를 인수로 받아서 f의 기능을 적용한 다음 g의 기능을 적용하는 함수를 반환한다. 이 함수를 활용하면 콤비네이터로 내부 반복을 수행하는 동작을 정의할 수 있다.<br>
여기에 repeat라는 함수를 만들어 데이터를 받아서 f에 연속적으로 n번 적용하는 루프가 있다고 가정한다.

* 반복 함수 예제

```java
repeat(3, (Integer x) -> 2 * x);
System.out.println(repeat(3, (Integer x) -> 2 * x).apply(10));
```

repeat() 메서드는 다음 처럼 구현될 수 있다.

```java
static <A> Function<A, A> repeat(int n, Function<A, A> f) {
    return n == 0 ? x -> x : compose(f, repeat(n - 1, f));
}
```

이 개념을 활용하면 반복 과정에서 전달되는 가변 상태 함수형 모델 등 방복 기능을 좀 더 다양하게 활용할 수 있다.
# 요약
* **일급 함수**란 인수로 전달하거나, 결과로 반호나하거나, 자료구조에 저장할 수 있는 함수다.
* 고차원 함수란 한 개 이상의 함수를 인수로 받아서 다른 함수를 반환하는 함수다. 자바에서는 comparing, andThen, compose 등의 고차원 함수를 제공한다.
* 커링은 함수를 모듈화하고 코드를 재사용할 수 있도록 지원하는 기법이다.
* 영속 자료구조는 갱신될 때 기존 버전의 자신을 보존한다. 결과적으로 자신을 복사하는 과정이 필요하지 않다.
* 자바의 스트림은 스스로 정의할 수 없다.
* 게으른 리스트는 자바 스트림보다 비싼 버전으로 간주할 수 있다. 게으른 리스트는 데이터를 요청했을 때 Supplier를 이용해서 요소를 생성한다. Supplier는 자료구조의 요소를 생성하는 역할을 수행한다.
* 패턴 매칭은 자료형을 언랩하는 함수형 기능이다. 자바의 switch 문을 일반화할 수 있다.
* 참조 투명성을 유지하는 상황에서는 계산 결과를 캐시할 수 있다.
* 콤비네이터는 둘 이상의 함수나 자료구조를 조합하는 함수형 개념이다.
