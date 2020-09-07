---
date: 2020-09-09 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 2. 컬렉션의 사용
description: 2. 컬렉션의 사용
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
# 리스트를 사용한 Iteration
* for 루프 활용 코드

```java
final List<String> friends = Arrays.asList("Brian", "Nate", "Neal", "Raju", "Sara", "Scott");

// 1.
for(int i = 0; i < friends.size(); i++) {
    System.out.println(friends.get(i));
}

// 2.
for(String name : friends) {
    System.out.println(name);
}
```

* 함수형 스타일

```java
// 1.
friends.forEach(new Consumer<String>() {
    @Override
    public void accept(final String name) {
        System.out.println(name);
    }
});

// 2.
friends.forEach((final String name) -> System.out.println(name));

// 2-1
friends.forEach((final String name) -> System.out.println(name));

// 2-2
friends.forEach((name) -> System.out.println(name));

// 3.
friends.forEach(System.out::pringln);
```
**1번**은 Consumer의 anonymous instance를 파라미터로 넘긴다. forEach() 메서드는 컬렉션에서 각 element에 대해 주어진 Consumer의 accept() 메서드를 호출하고 원하는 작업을 수행한다.<br>
기존 for 루프를 새로운 내부 iterator로 변경했다. 이로써 각 element에서 해야할 작업에만 집중할 수 있게 되었다.<br>
한 가지 아쉬운 점은 코드가 장황해졌다.

**2번**에서는 1번에서의 장황한 코드가 대폭 개선된다. forEach() 메서드는 람다 표현식 혹은 코드 블록을 인수로 받는 고차 함수이며 리스트에 있는 각 element의 내용을 처리한다. 이 람다 표현식은 호출하는 순서와 상관없이 lazy(실행 순서를 변경할 수 있다는 의미)하게 실행되며 따라서 **병렬화**처리도 가능해졌다.

**2-1번**같은 경우에 자바 컴파일러는 변수 name에 저장되어 있는 내용을 보고 name 파라미터가 String 타입이라는 것을 알 수 있다. 호출된 메서드 (forEach() 메서드)의 시그니처를 찾아 파라미터로 받은 함수형 인터페이스를 분석한다. 인터페이스의 추상 메서드를 보고 예상되는 파라미터 수와 타입을 결정한다.
람다 표현식이 `다중 파라미터`를 갖는다면 타입 인터페이스를 사용하지만 이 경우에는 모든 파라미터에 대한 타입 정보를 설정하거나 모두 설정하지 않아야 한다.

**2-2번**과 같이 `싱글 파라미터`를 갖는 람다 표현식에서는 **파라미터의 타입을 추론할 수 있는 경우**에 한해, 파라미터 주변의 괄호를 생략할 수 있도록 자바 컴파일러가 처리한다.<br>
한 가지 문제가 있는데 타입을 선언하지 않게되면서 name 변수가 final 선언하지 않고 컴파일러가 추론하게 되면 람다 표현식 내에서 파라미터의 타입은 추론할 수 있지만, `람다 표현식 내부에서 파라미터들이 변경되어도 상관없는지`에 대한 부분까지는 알 수가 없게 된다. 

**3번**은 메서드 레퍼런스를 활용했다.

이 처럼 내부 iteration 버전은 다른 버전보다 간결해지고 코드 작업 시 우리가 각 element를 사용해서 하고자 하는 목적에 초점을 맞출 수 있다. 이것이 `서술적`이다.<br>
하지만 제약도 있다. 내부 iteration은 중간에 멈출 수 없다.(해결은 가능하다.)

# 리스트 변형

* 명령형 스타일

```java
final List<String> uppercaseNames = new ArrayList<String>();

for(String name : friends) {
    uppercaseNames.add(name.toUpperCase());
}
```

* 람다 표현식 사용

```java
// 1.
friends.stream()
       .map(name -> System.out.print(name + " "));
System.out.println();

// 2.
friends.stream()
       .map(name -> name.length())
       .forEach(count -> System.out.print(count + " "));  
```

**1번**에 stream() 메서드는 스트림 인스턴스에 대한 컬렉션을 매핑한다. map() 메서드는 람다 표현식의 실행 결과를 취합하여 결과 컬렉션으로 리턴한다. 마지막으로 결과물의 element들을 forEach() 메서드를 사용하여 출력한다.<br>

**2번**처럼 map() 메서드를 사용시 입력/출력 element의 타입이 다를 수 있다.

람다 표현식 버전은 명시적 변경(explicit mutation)이 없으며 간결하다. 더 이상 비어 있는 컬렉션의 초기화 작업이나 가비지 변수가 필요하지 않다.

* 메서드 레퍼런스 사용

```java
friends.stream()
       .map(String::uoUpperCase)
       .forEach(name -> System.out.println(name)); 
```

메서드 레퍼런스를 활용하면 코드를 더 간결하게 구현할 수 있다. 자바 컴파일러는 람다 표현식 뿐만 아니라 함수형 인터페이스의 구현이 필요한 코드에 대해 메서드의 레퍼런스를 사용할 수 있다.<br>
위에서 활용한 toUpperCase()는 합성된 메서드에 전달된 파라미터이며 함수형 인터페이스의 추상 메서드 구현에 해당한다. String::toUpperCase라는 파라미터 레퍼런스가 암묵적으로 제공된다.

## 메서드 레퍼런스는 언제 사용하는 것이 좋을까
메서드 레퍼런스는 `람다 표현식이 짧아서 간단하게 만들 수 있을 때`, 혹은 `인스턴스 메서드나 정적 메서드를 직접 호출하는 경우`에 유용하다. 즉, 람다 표현식을 사용할 때 `파라미터를 전달하지 않는 경우라면` 메서드 레퍼런스를 사용할 수 있다.(?)<br>
메서드 레퍼런스를 사용할 때 내부적으로 약간의 컴파일러의 도움이 필요하다. 타깃 객체와 파라미터는 합성된 메서드에 전달된 파라미터로부터 파생됐다.<br>
하지만 파라미터를 인수로 전달하기 전에 파라미터를 처리해야 하거나 혹은 리턴하기 전에 호출의 결과를 사용해야 하는 경우에는 사용할 수 없다.

# element 찾기

filter() 메서드를 활용하면 컬렉션에서 직접 element를 탐색, 선택할 수 있다. 

* 명령형 코드

```java
final List<String> startsWithN = new ArrayList<String>();

for(String name : friends) {
    if (name.startsWith("N")) {
        startsWithN.add(name);
    }
}
```

* 함수형 코드

```java
final List<String> startWithN =
    friends.stream()
           .filter(name -> name.startsWith("N"))
           .collect(Collectors.toList()); 
```

filter() 메서드가 리턴한 결과 컬렉션의 element는 입력 컬렉션에 있는 엘리먼트의 서브셋이다. 즉, filter() 메서드에 의해 처리된 결과 컬렉션의 element들은 모두 입력 컬렉션의 element들 중 하나라는 의미다.

# 람다 표현식의 재사용성
람다 표현식은 간결하지만 코드 안에서 중복해서 사용되기 쉽다.

* 중복 사용된 람다 표현식

```java
final long countFriendsStartN = 
    friends.strema()
           filter(name -> name.startWith("N")).cont();
final long countEditorsStartN = 
    editors.strema()
           filter(name -> name.startWith("N")).cont();
final long countComradesStartN = 
    comrades.strema()
           filter(name -> name.startWith("N")).cont();
```

이처럼 중복 사용될 경우 람다 표현식을 변수에 저장해서 재사용이 가능하다.<br>
filter() 메서드는 **java.util.function.Predicate** 함수형 인터페이스에 대한 레퍼런스를 받는다. 람다 표현식에서 Predicate의 test() 메서드의 구현을 합성한다.

* Predicate 함수형 인터페이스를 사용하여 변수 처럼 활용

```java
final Predicate<String> startsWithN = name -> name.startWithN("N");

final long countFriendsStartN = 
    friends.stream()
           .filter(startsWithN)
           .count();  
final long countEditorsStartN = 
    editors.stream()
           .filter(startsWithN)
           .count();  
final long countComradesStartN = 
    comrades.stream()
           .filter(startsWithN)
           .count();  
```

# 렉시컬 스코프와 클로저 사용하기
# element 선택
# 컬렉션을 하나의 값으로 reduce
# element 조인

