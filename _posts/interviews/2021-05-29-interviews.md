---
date: 2021-05-29 20:40:40
layout: post
title: 면접 질문 리스트
subtitle: 면접 질문 리스트
description: 면접 질문 리스트
image: https://leejaedoo.github.io/assets/img/interview.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/interview.jpeg
category: interviews
tags:
- interviews 
paginate: true
comments: true
---

# 운영체제
## 프로세스
### 프로세스와 스레드의 차이는 무엇인가요?
### 교착상태란 무엇이며, 교착상태가 발생하기 위해서는 어떤 조건이 있어야 하나요?
### 교착상태의 해결법은 무엇인가요?
### 뮤텍스와 세마포어에 대해서 설명해 보시오.
### 경쟁 상태란 무엇인가요?
### 프로세스 혹은 스레드의 동기화란 무엇인가요?
### 사용자 수준의 스레드와 커널 수준의 스레드의 차이는 무엇인가요?
### CPU 스케줄링이란 무엇인가요?
### CPU 스케줄링 방법에는 대표적으로 어떤 것들이 있나요?
### 동기와 비동기, 블로킹과 넌블로킹의 차이는 무엇인가요?
#### Blocking
* 호출된 함수가 종료될 때 까지 함수의 제어권을 들고 있으면서 자신을 호출한 함수에게 제어권을 돌려주지 않는 상태
#### Non-Blocking
* 호출된 함수가 종료되지 않은 상태에서 함수의 제어권을 자신을 호출한 함수에게 돌려줌으로써 다른 일을 진행하게 해주는 상태
#### Synchronous
* 호출된 함수의 수행 결과 및 종료를 호출한 함수가 계속 신경쓰는 상태
#### Asynchronows
* 호출된 함수의 수행 결과 및 종료를 호출한 함수는 신경쓰지 않고 자기 할 일만 계속 하고 오직 호출된 함수만 관여하는 상태

### context switching이란?
* CPU가 한 개의 Task(Process/Thread)를 실행하고 있는 상태에서 Interrupt 요청에 의해 다른 Task로 실행이 전환되는 과정에서 기존의 Task 상태 및 Register 값들에 대한 정보(Context)를 저장하고 새로운 Task의 Context 정보로 교체하는 작업을 말한다.

## 메모리
### 프로세스에 할당되는 메모리의 각 영역에 대해서 설명해 주세요.
### 메모리 구조의 순서가 어떻게 되는가? CPU에서 가까운 순으로 말해보시오.
### 페이지와 세그멘테이션에 대해서 설명해 보시오.
### 외부 단편화란? 내부 단편화란?
### First Fit, Best Fit, Worst Fit에 대해서 설명해 보시오.
### 페이지 교체 알고리즘 종류에는 어떤 것들이 있나요?
# 네트워크
## 전산 기본
### OSI 7계층에 대해서 설명해주세요.
### TCP/IP 4계층에 대해서 설명해주세요.
### DNS가 무엇인가요?
### 도메인 이름으로 실제 IP를 어떻게 찾을 수 있는지 흐름을 설명해 주세요.
## TCP/UDP
### TCP와 UDP의 차이에 대해서 설명해 주세요.
### TCP 헤더에 대해서 설명해 주세요.
### MTU가 무엇인가요?
### 3-way hand shake, 4-way hand shake 흐름에 대해서 설명해주세요.
## HTTP
### HTTP 프로토콜에 대해서 아는대로 말해주세요.
### HTTP와 HTTPS 의 차이는 무엇인가요?
### HTTPS가 동작하는 방식에 대해서 설명해 주세요.
### HTTP 1.0과 1.1의 차이는 무엇인가요?
### HTTP2와 그 특징에 대해서 설명해 주세요.
### HTTP 헤더의 구조에 대해서 설명해 주세요.
### keep-alive 헤더에 대해서 설명해 주세요.
### HTTP GET과 POST의 차이는 무엇인가요?
### 쿠키와 세션에 대해서 설명해 주세요.
## 웹
### 웹브라우저에서 서버로 요청했을 때, 흐름을 설명해주세요.
### CORS란 무엇인가요?
### 웹 서버와 웹 어플리케이션 서버(WAS)의 차이는 무엇인가요?
### WAS의 동작 방식은?
### REST API에 대해서 설명해 주세요.
* HTTP URI를 통해 `resource`를, HTTP METHOD를 통해 `행위`를 `표현`한다. 

### API Gateway란 무엇인가요?
### API Gateway가 다운되면 모든 API를 사용 못할지도 모르는데, 어떤 방안을 마련해야 할까요?
### 웹 브라우저에 google.com을 입력하면 일어나는 과정은?
# 데이터베이스
## 전산 기본
### JOIN에 대해서 설명해 주세요.
### 내부 조인과 외부 조인의 차이는 무엇인가요?
### 정규화에 대해서 설명해 주세요.
### 파티셔닝과 샤딩에 대해서 설명해 주세요.
### ORM이란 무엇인가요?
### NoSQL이란 무엇인가요?
### 스키마란 무엇인가요?
### 물리적 조인 방식의 종류는?
### DB Optimizer란?
## 인덱스
### 인덱스란 무엇인가요? 어떻게 동작 하나요?
### 인덱스의 알고리즘에는 어떤 것들이 있나요?
### Table Full Scan과 Index Range Scan 을 설명해주세요.
## 트랜잭션
### 트랜잭션이란 무엇인가요? 4가지 원칙을 포함해서 설명해 주세요.
### 트랜잭션의 격리 수준(Transaction Isolation Level)과 각 수준에서 발생할 수 있는 문제들에 대해 말해보세요.
#### READ UNCOMITTED
#### READ COMITTED(= NON REPEATABLE READ)
#### REPEATABLE READ
#### SERIALIZABLE
### 공유 락과 배타 락의 차이는 무엇인가요?
### 데드락이란 무엇이며, 어떻게 발생할까요?
# 알고리즘
## 전산 기본
### 빅오 표기법에 대해서 설명해주세요
### 팩토리얼(factorial)을 구현해 보세요(손코딩).
### 피보나치 수열 구현 방식 세 가지를 말해보시고, 시간복잡도와 공간복잡도를 설명해 주세요.
### BFS/DFS 차이는 무엇인가요?
### Prim 알고리즘에 대해서 설명해 주세요.
### 다익스트라 알고리즘에 대해서 설명해 주세요.
### 은행원 알고리즘에 대해서 설명해 주세요.
## 정렬
### 정렬의 종류에는 어떤 것들이 있나요?
#### 선택 정렬 - O(N^2)
* 최소값을 찾아 맨 앞 숫자와 교환하면서 정렬하는 방식

#### 삽입 정렬 - O(N^2)
* 타겟 위치 숫자와 앞 자리 숫자들과 비교하여 작으면 교환하면서 정렳하는 방식

#### 퀵 정렬 - O(NlogN)

#### 카운팅 정렬 - O(N)
* 해당 숫자와 일치하는 배열의 index에 해당 숫자 개수를 카운팅 하면서 정렬하는 방식

### 54321 배열이 있을 때, 어떤 정렬을 사용하면 좋을까요?
### 랜덤으로 배치된 배열이 있을때, 어떤 정렬을 사용하면 좋을까요?
### 자릿수가 모두 같은 수가 담긴 배열이 있을 때, 어떤 정렬을 사용하면 좋을까요?
### 두 개의 스택으로 큐처럼 동작하는 클래스를 정의해보세요.
# 자료구조
## 전산 기본
### Array와 LinkedList의 차이점에 대해서 설명해 주세요.
#### Array
* 메모리 상에 순서대로 데이터를 저장
* 검색이 빠름
* insert, update, delete에 취약함

#### LinkedList
* 포인터를 활용한 위치에 대한 참조 형태로 데이터를 저장
* 검색은 느림
* insert, update, delete가 빠름

### 스택과 큐에 대해서 설명해 주세요.
### 해시테이블에 대해서 설명해 주세요.
## 트리
### 포화(Perfect) 이진트리, 완전(Complete) 이진트리, 정(Full) 이진트리의 차이점에 대해 각각 설명해주세요.
### 그래프와 트리의 차이점에 대해서 설명해 주세요.
### 힙 자료구조에 대해 설명해 주세요.
### 힙의 삽입과 삭제는 어떻게 이루어지나요?
### 레드 블랙 트리에 대해 설명해주세요.
### 레드 블랙 트리의 삽입과 삭제 과정에 대해서 말해보세요.
### B-Tree에 대해서 설명해 주세요.
### 최소 신장 트리에 대해서 설명해 주세요.
# 프로그래밍
## 전산 기본
### 객체지향이 무엇인가요? 절차지향과의 차이점은 뭐죠?
### 객체지향 SOLID 원칙에 대해서 설명해 주세요.
#### SRP - 단일 책임 원칙
#### OCP - 개방 폐쇄 원칙
#### LIP - 리스코드 치환 원칙
#### ISP - 인터페이스 분리 원칙
#### DIP - 의존관계 역전 원칙
### 객체지향 4가지 특징에 대해서 설명해 주세요.
### 대표적인 객체지향 언어에는 어떤 것들이 있나요?
### 데이터 타입과 변수의 차이는 무엇인가요?
### 함수형 프로그래밍에 대해서 설명해 주세요.
### AOP란 무엇인가요?
### 컴파일러와 인터프리터의 차이는 무엇인가요?
### 오버로딩과 오버라이딩의 차이는 무엇인가요?
### 1급 객체에 대해서 설명해 주세요.
# JAVA
## 기본
### JAVA 메모리 영역에 대해 설명해 주세요.
* java 메모리 영역은 자바 애플리케이션을 실행할 때 사용되는 데이터 적재 영역을 말한다.

#### Method Area
* JVM이 구동될 때 생성되며 모든 thread에서 접근이 가능하다.
* 클래스 정보, static 변수 등이 저장된다.

#### Heap
* 몇 개의 thread가 존재하든 상관 없이, 단 하나의 Heap 메모리만 존재한다.
* stack에서 참조하는 참조형 데이터 타입을 갖는 객체나 배열의 참조 값이 저장된다.
* 참조하는 변수나 필드가 없게 되면 GC에 의해 제거된다.
* 런타임 시 할당된다.

#### Stack(= Call Stack)
* 각 thread 마다 하나씩 존재하며, thread가 시작될 때 할당된다.
* 지역 변수, 파라미터, 리턴 값, 연산 등에 사용되는 임시 값들이 저장된다.

#### PC Register
* thread가 생성될 때 마다 생성되면 현재 thread가 실행되는 부분의 `주소`와 `명령`을 저장하는 영역

#### Native Method Stack
* java외 다른 언어로 작성된 네이티브 코드를 위한 메모리 영역

### Java 접근 제어자에 대해서 각각 설명해 주세요.
### JVM의 구조에 대해서 설명해 주세요.
### Garbage Collector 에 대해서 설명해 주세요. 어떻게 동작하나요?
### GC의 종류에 대해서 말해보세요.
### Java 버전 별 특성에 대해서 아는대로 말해주세요.
### Java는 Call By Value일까요, Call By Reference 일까요?
### String, StringBuffer, StringBuilder의 특징과 차이점은?
### 자바의 데이터 타입인 Primitive Type(기본형) 에 대해 설명해 보세요.
### Thread를 구현하기 위한 인터페이스, 클래스는 무엇이 있고 그 것들의 특성과 차이점은?
### 자바 코드의 실행 과정을 설명해주세요.
### 리플렉션(Reflection)이란 무엇인가요?
### Stream API란 무엇인가요?
### Lambda란 무엇인가요?
### 함수형 인터페이스란 무엇인가요?
### JVM 기동시 주로 사용되는 옵션들을 아는대로 말해보세요.
### foreach를 사용할 수 있는 자료구조는 어떤 인터페이스를 상속받고 있나요?
### iterator와 iterable 차이는 무엇인가요?
### synchronized 키워드에 대해 설명해 주세요.
### volatile 키워드에 대해 설명해 주세요.
### final 키워드에 대해서 설명해주세요. 각각의 쓰임에 따라 어떻게 동작하나요?
### HashTable vs HashMap vs ConcurrentHashMap
### Fail-Safe Iterator vs Fail-Fast Iterator
#### Fail-Fast Iterator
* Multi Thread 환경에서 반복문이 도는 중에 데이터가 변경될 경우 ConcurrentModificationException 예외를 뱉고 STOP
* 중간에 fail이 발생될 경우 도중에 STOP
* ex. HashMap, ArrayList

#### Fail-Safe Iterator
* Multi Thread 환경에서 반복문이 도는 중에 데이터가 변경될 경우 동시성을 보장하며 예외 없이 끝까지 돔. 최대한
* 중간에 fail이 발생되더라도 STOP 없이 최대한 많은 개체를 성공시킴
* ex. ConcurrentHashMap, CopyOnWriteArrayList

## 클래스와 객체
### Wrapper Class란 무엇인가요?
### 클래스, 객체, 인스턴스 차이에 대해서 설명해 주세요.
### 직렬화(Serialization)과 역직렬화(Deserialization)에 대해서 설명해 주세요.
### Java Generic에 대해서 설명해 주세요.
### equals와 ==의 차이는 무엇인가요?
### hashCode란 무엇인가요?
### 문자열을 리터럴(string = "abcd")로 할당하는 것과 객체(string = new String("abcd"))로 할당하는 방식의 차이가 무엇인가요?
### 순수 추상 클래스와 인터페이스의 차이는 무엇인가요?
### 본인 관점에서, 인터페이스는 주로 어떨 때 사용하나요?
## 자료형, 자료구조
### Java의 Collection에 대해서 설명해 주세요.
### Array와 ArrayList의 차이점은 무엇인가요?
### char type과 string type으로 나뉘어져 있는 이유는 무엇인가요?
### HashMap과 HashTable의 특성과 차이점은?
# Spring Framework
## 기본
### Spring이란 무엇인가요?
### Spring, Spring MVC, Spring Boot의 차이점에 대해 각각 설명해 주세요.
### Spring 버전 별 특성에 대해서 아는대로 답변해 주세요.
### Spring Framework의 생명 주기에 대해서 말해주세요.
### Bean이란 무엇인가요?
### Bean과 Component의 차이점은?
### Interceptor와 Filter의 차이점을 말해주세요.
### IOC와 DI에 대해서 설명해주세요.
### Container란 무엇인가요?
### VO, DTO, DAO에 대해서 각각 설명해 주세요.
### Checked Exception과 Unchecked Exception에 대해 설명해주세요. 스프링 트랜잭션 추상화에서 rollback 대상은 무엇일까요?
### 생성자 주입과 필드 주입의 차이점은?
## MVC
### MVC에 대해서 설명해 주세요.
### Servlet이 무엇인가요? (사실 이건 Java 섹션에 있는게 맞음..)
### Dispatcher-Servlet이란 무엇인가요?
### Spring MVC에서 HTTP 요청이 들어왔을 때의 흐름을 설명해 주세요.
## Spring Batch
### 일반적인 reader/processor/writer 구현방식과 하나의 tasklet 기반의 구현 방식(simpleJob)의 차이점은?
* chunk단위의 트랜잭션 처리가 필요 없는 경우, 간단한 로직으로 구현이 가능한 경우, 데이터 양이 많지 않은 경우에 주로 하나의 tasklet을 커스텀하게 활용하는 방식을 활용.
* 그 외에는 reader/processor/writer 별로 구현 로직을 분할하여 chunk 단위로 트랜잭션을 관리하도록 함으로써 대용량 처리에 용이하게 구현.

# 인프라/클라우드/DevOps
## 기본
### 도커란 무엇인가요?
### 리버스 프록시란?
### 로드 밸런서란?
# ETC
## 전산 기본
### TDD란 무엇인가요?
### 프레임워크와 라이브러리 차이는 무엇인가요?
### Monolitc Architecture, Micro Service Architecture에 대해 각각 설명해 주세요.
### 애자일 방법론이란?
## 디자인 패턴
### 디자인 패턴이란 무엇인가요?
### singleton 패턴에 대해서 설명해주세요.(생각보다 어려움)
### strategy(전략) 패턴에 대해서 설명해주세요.
### builder(빌더) 패턴에 대해서 설명해주세요.
### factory method(팩토리 메서드) 패턴에 대해서 설명해주세요.
### facade(퍼사드) 패턴에 대한 예를 들어주세요.
## MyBatis
### #과 $의 차이점은?

# 참고자료
* [https://velog.io/@hygoogi/%EA%B8%B0%EC%88%A0-%EB%A9%B4%EC%A0%91-%EC%A7%88%EB%AC%B8-%EB%AA%A8%EC%9D%8C](https://velog.io/@hygoogi/%EA%B8%B0%EC%88%A0-%EB%A9%B4%EC%A0%91-%EC%A7%88%EB%AC%B8-%EB%AA%A8%EC%9D%8C)
* [https://yadon079.github.io/2021/cs/backend-developer-interview](https://yadon079.github.io/2021/cs/backend-developer-interview)
