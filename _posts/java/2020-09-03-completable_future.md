---
date: 2020-09-03 15:40:40
layout: post
title: 자바 8 인 액션
subtitle: 11. CompletableFuture: 조합할 수 있는 비동기 프로그래밍
description: 11. CompletableFuture: 조합할 수 있는 비동기 프로그래밍
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
Future 인터페이스와 Future를 구현하는 새로운 클래스 CompletableFuture를 이용해서 병렬성이 아닌 동시성을 이용해야 하는 상황, 즉 하나의 CPU 사용을 가장 극대화할 수 있도록 느슨하게 연관된 여러 작업을 수행해야 하는 상황이라면 원격 서비스 결과를 기다리거나, 데이터베이스 결과를 기다리면서 스레드를 블록하는 상황을 해결할 수 있다.
# Future
Future 인터페이스는 자바 5부터 제공되어 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있는 기능이다.<br>
비동기 계산을 모델링하는 데 Future를 이용할 수 있으며, Future는 계산이 끝났을 때 결과에 접근할 수 있는 레퍼런스를 제공한다.<br>
시간이 걸릴 수 있는 작업을 Future 내부로 설정하면 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있다.<br>
Future는 저수준의 스레드에 비해 직관적으로 이해하기 쉽다는 장점이 있다.<br>
Future를 이용하려면 시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼 다음에 ExecutorService에 제출해야 한다.

* Future로 오래 걸리는 작업을 비동기적으로 실행하기

```java
ExecutorService executor = Executor.newCachedThreadPool();  //  스레드 풀에 태스크를 제출하려면 ExecutorService를 만들어야 한다.
Future<Double> future = executor.submit(new Callable<Double>() {    //  Callable을 ExecutorService로 제출한다.
    public Double call() {
        return doSomeLongComputation(); //  시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행한다.
    }
});

doSomethingElse();  //  비동기 작업을 수행하는 동안 다른 작업을 한다.

try {
    Double result = future.get(1, TimeUnit.SECONDS);    //  비동기 작업의 결과를 가져온다. 결과가 준비되어 있지 않으면 호출 스레드가 블록된다. 하지만 최대 1초까지만 기다린다.
} catch (ExecutionException ee) {
    //  계산 중 예외 발생
} catch (InterruptedException ie) {
    //  현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te) {
    //  Future가 완료되기 전에 타임아웃 발생
}
```
![Future 비동기](../../assets/img/future.jpg)
ExecutorService에서 제공하는 스레드가 시간이 오래 걸리는 작업을 처리하는 동안 우리 스레드로 다른 작업을 동시에 실행할 수 있다.<br>
다른 작업을 처리하다가 시간이 오래 걸리는 작업의 결과가 필요한 시점이 되었을 때 Future의 get 메서드로 결과를 가져올 수 있다. get 메서드를 호출했을 때 이미 계산이 완료되어 결과가 준비되었다면 즉시 결과를 반환하지만 결과가 준비되지 않았다면 작업이 완료될 때까지 우리 스레드를 블록시킨다.

이같은 방식의 문제점은 `오래 걸리는 작업이 영원히 끝나지 않는다면` 문제가 발생할 수 있다. 작업이 끝나지 않는 문제가 있을 수 있으므로, get 메서드를 오버로드해서 우리 스레드가 대기할 최대 타임아웃 시간을 설정하는 것이 좋다.
## Future 제한
Future 인터페이스가 갖고 있는 메서드들 만으로는 간결한 동시 실행 코드를 구현하기 충분치 않다. 따라서 아래와 같은 선언형 기능이 필요하다.

* 두 개의 비동기 계산 결과를 하나로 합친다. 두 가지 계산 결과는 서로 독립적일 수 있으며 또는 두 번째 결과가 첫 번째 결과에 의존하는 상황일 수 있다.
* Future 집합이 실행하는 모든 태스크의 완료를 기다린다.
* Future 집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다.(ex. 여러 태스크가 다양한 방식으로 같은 결과를 구하는 상황)
* 프로그램적으로 Future를 완료시킨다.(즉, 비동기 동작에 수동으로 결과 제공)
* Future 완료 동작에 반응한다.(즉, 결과를 기다리면서 블록되지 않고 결과가 준비되었다는 알림을 받은 다음에 Future의 결과로 원하는 추가 동작을 수행할 수 있음.)

따라서 자바 8에서 제공하는 CompletableFuture 클래스(Future 인터페이스를 구현한 클래스)를 활용하면 위에서 설명한 기능을 선언형으로 이용할 수 있게 해준다.<br>
Stream과 CompletableFuture는 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용한다. 따라서 Future와 CompletableFuture의 관계를 Collection과 Stream의 관계에 비유할 수 있다.  
## CompletableFuture로 비동기 애플리케이션 만들기

* 고객에게 비동기 API를 제공하는 방법을 배운다(온라인상점을 운영하고 있는 독자에게 유용한 기술)
* 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법을 배운다. 두 개의 비동기 동작을 파이프라인으로 만드는 방법과 두 개의 동작 결과를 하나의 비동기 계산으로 합치는 방법을 살펴본다.
* 비동기 동작의 완료에 대응하는 방법을 배운다.(ex. 모든 상점에서 가격 정보를 얻을 때까지 기다리는 것이 아닌 각 상점에서 가격 정보를 얻을 때마다 즉시 최저가격을 찾는 애플리케이션을 갱신하는 방법을 설명한다. 그렇지 않으면 서버가 다운되는 등 문제가 발생했을 때 사용자에게 검은 화면만 보여주게 될 수 있다.)

> #### 동기 API와 비동기 API
> `동기 API`에서는 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행한다. 호출자와 피호출자가 각각 다른 스레드에서 실행되는 상황이었더라도 호출자는 피호출자의 동작 완료를 기다렸을 것이다. 이처럼 동기 API를 사용하는 상황을 `블록 호출(blocking call)`이라고 한다.<br>
> 반면 `비동기 API`에서는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다. 이와 같은 상황은 `비블록 호출(non-blocking call)`이라고 한다. 다른 스레드에 할당된 나머지 계산 결과는 콜백 메서드를 호출해서 전달하거나 호출자가 '계산 결과가 끝날 때까지 기다림' 메서드를 추가로 호출하면서 전달된다. 주로 I/O 시스템 프로그래밍에서 이와 같은 방식으로 동작을 수행한다. 즉, 계산 동작을 수행하는 동안 비동기적으로 디스크 접근을 수행한다. 그리고 더 이상 수행할 동작이 없으면 디스크 블록이 메모리로 로딩될 때까지 기다린다.
# 비동기 API 구현
## 동기 메서드를 비동기 메서드로 변환
## 에러 처리 방법
# 비블록 코드 만들기
## 병렬 스트림으로 요청 병렬화하기
## CompletableFuture로 비동기 호출 구현하기
## 더 확장성이 좋은 해결 방법
## 커스텀 Executor 사용하기
# 비동기 작업 파이프라인 만들기
## 할인 서비스 구현
## 할인 서비스 사용
## 동기 작업과 비동기 작업 조합하기
## 독립 CompletableFuture와 비독립 CompletableFuture 합치기
## Future의 리플렉션과 CompletableFuture의 리플렉션
# CompletableFuture의 종료에 대응하는 방법
## 최저가격 검색 애플리케이션 리팩토링
## 응용
# 요약