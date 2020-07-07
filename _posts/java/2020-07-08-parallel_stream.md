---
date: 2020-07-07 18:40:40
layout: post
title: 자바 8 인 액션
subtitle: 7. 병렬 데이터 처리와 성능
description: 7. 병렬 데이터 처리와 성능
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
## 병렬 스트림
컬렉션에 parallelStream을 호출하면 `병렬 스트림`이 생성된다. 병렬 스트림이란, `각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림`이다.<br>
따라서 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.
* 1부터 n부터까지의 합을 구하는 코드 : 일반 스트림

```java
public static long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1)   //  무한 자연수 스트림 생성
                 .limit(n)                  //  n개 이하로 제한
                 .reduce(0L, Long::sum);    //  모든 숫자를 더하는 스트림 리듀싱 연산
}
```

* 1부터 n부터까지의 합을 구하는 코드 : 전통적인 자바 코드

```java
public static long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i<= n; i++) {
        result += i;
    }
    return result;
}
```
### 순차 스트림을 병렬 스트림으로 변환하기
```java
public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
                 .limit(n)
                 .parallel()                //  스트림을 병렬 스트림으로 변환
                 .reduce(0L, Long::sum);
}
```
parallel 메서드를 통해 스트림이 여러 청크로 분할되게 되고, 마지막 리듀싱 연산을 통해 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐 전체 스트림의 리듀싱 결과를 도출한다.

parallel과 sequential 메서드를 통해 어떤 연산을 병렬로 실행할지, 순차로 실행할지 제어할 수 있다.
```java
stream.parallel()
      .filter(...)
      .sequential()
      .map(...)
      .parallel()
      .reduce();  
```
parallel과 sequential 두 메서드 중 `최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다.` 위 코드는 parallel이 마지막 호출되었으므로 위 파이프라인은 병렬로 실행된다.
![병렬리듀싱연산](../../assets/img/parallel_reducing.jpeg)

* 병렬 스트림에서 사용하는 thread pool 설정

병렬 스트림은 내부적으로 `ForkJoinPool`을 사용한다. 기본적으로 ForkJoinPool은 프로세서 수, 즉 Runtime.getRuntime().availableProcessors()가 반환하는 값에 상응하는 thread를 갖는다.

### 스트림 성능 측정
자신의 기기에서 지원하는 코어 수 등에 따라서 성능 속도는 달라질 수 있다.<br>
고전적인 for 루프는 저수준으로 동작하며 기본 값을 박싱하거나 언박싱할 필요가 없으므로 수행속도가 빠를 수 있다. 따라서 병렬 버전이 순차 버전보다 느리게 동작할 수 있다. 이유가 뭘까?
* iterate가 박싱된 객체를 생성하므로 이를 다시 언박싱하는 과정이 필요했다.
* iterate는 병렬로 실행될 수 있도록 독립적인 청크로 분할하기 어렵다.
    * iterate 연산은 이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 청크로 분할이 어렵다. 
![iterate](../../assets/img/iterate.jpeg)
위와 같은 상황에서는 리듀싱 연산이 수행되지 않는다. 리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가 준비되지 않았으므로 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수가 없기 때문이다.<br>
iterate같은 경우는 스트림이 병렬로 처리되도록 지시했고 각각의 합계가 다른 thread에서 수행되었음에도 불구하고 순차처리 방식으로 처리되기 때문에 thread를 할당하는 오버헤드만 증가하게 될 뿐이다.<br>
따라서 iterate와 같은 병렬과는 거리가 먼 방식을 사용하면 오히려 프로그램의 성능이 더 나빠질 수도 있다.
#### 더 특화된 메서드 사용
LongStream.rangeClosed라는 메서드를 활용할 수 있다. 이는 iterate에 비해 아래와 같은 장점이 있다.
* 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라진다.
    * 특화되지 않은 스트림을 처리할 때는 오토박싱/언박싱 등의 오버헤드를 수반하기 때문에 iterate 보다 처리 속도가 빠르다.
* 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다. 예를 들어, 1 ~ 20의 숫자 범위를 각각 1 ~ 5, 6 ~ 10, 11 ~ 15, 16 ~ 20 범위의 숫자로 분할할 수 있다.

LongStream.rangeClosed를 활용하면 실질적으로 리듀싱 연산이 병렬로 수행된다. 올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘할 수 있다.
    
### 병렬 스트림의 올바른 사용법
공유된 상태를 바꾸는 알고리즘을 사용할 때 병렬 스트림을 사용하면 문제가 발생한다.
```java
public static long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total = 0;
    public void add(long value) { total += value; }
}
```
위 같은 코드는 본질적으로 순차 실행할 수 있도록 구현되어 있으므로 병렬로 실행하게 되면 올바른 값을 얻을 수 없게 된다. 여러 스레드에서 동시에 total += value 를 실행하면서 문제가 발생되기 때문이다.<br>
따라서 병렬 스트림과 병렬 계산에는 공유된 가변 상태를 피해야 한다.
### 병렬 스트림 효과적으로 사용하기
* 확신이 서지 않을 때는 직접 측정해서 사용하라.
    * 병렬 스트림이 순차 스트림보다 항상 성능이 좋은 것이 아니기 때문에 모를 때는 직접 성능 체크해보는 것이 정확하다.
* 박싱을 주의해서 사용하라.
    * 오토박싱/언박싱은 성능을 크게 저하시킬 수 있는 요소다. 기본형 특화 스트림(ex. IntStream, LongStream, DoubleStream)을 활용하여 박싱 동작을 피할 수 있다.
* 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있음을 주의하라.
    * limit이나 findFirst같이 요소의 순서에 의존하는 연산을 병렬 스트림에 활용하게 되면 비싼 비용을 치뤄야 한다.
* 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.
* 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.
    * 소량의 데이터를 처리하는 상황에서는 병렬화 과정에서 생기는 부가 비용을 상쇄할 수 있을 만큼의 이득을 얻지 못한다.
* 스트림을 구성하는 자료구조가 올바른지 확인하라.
    * 예를 들면, ArrayList가 LinkedList보다 효율적으로 분할할 수 있다. LinkedList는 분할하려면 모든 요소를 탐색해야 하지만 ArrayList는 요소를 탐색하지 않고도 리스트를 분할할 수 있다.
* 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
    * 예를 들어, SIZED 스트림은 정확히 같은 크기의 두 스트림으로 분할되므로 효과적으로 스트림을 병렬처리 할 수 있으나 필터 연산은 스트림의 길이를 예측할 수 없으므로 효과적으로 병렬 처리 할 수 있을지 알 수 없게 된다.
* 최종 연산의 병합 과정(ex. Collector의 combiner 메서드) 비용을 살펴봐라.
    * 병합 과정의 비용이 비싸다면, 병렬 스트림으로 얻은 성능의 이익이 서브스트림의 부분 결과를 합치는 과정에서 상쇄될 수 있다.

* 스트림 소스와 분해성

<table>
  <thead>
    <tr>
      <th>소스</th>
      <th>분해성</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ArrayList</td>
      <td>훌륭함</td>
    </tr>
    <tr>
      <td>LinkedList</td>
      <td>나쁨</td>
    </tr>
    <tr>
      <td>IntStream.range</td>
      <td>훌륭함</td>
    </tr>
    <tr>
      <td>Stream.iterate</td>
      <td>나쁨</td>
    </tr>
    <tr>
      <td>HashSet</td>
      <td>좋음</td>
    </tr>
    <tr>
      <td>TreeSet</td>
      <td>좋음</td>
    </tr>
  </tbody>
</table>

## 포크/조인 프레임워크
### RecursiveTask 활용
### 포크/조인 프레임워크를 제대로 사용하는 방법
### 작업 훔치기
## Spliterator
### 분할 과정
### 커스텀 Spliterator 구현하기
## 정리

