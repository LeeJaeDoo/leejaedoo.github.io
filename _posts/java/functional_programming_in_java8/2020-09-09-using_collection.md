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
위에서 알아본 람다 표현식의 중복에 따른 재사용 방법은 문제가 될 여지가 있다. 이때 렉시컬 스코프와 클로저를 통해 해결할 수 있다.

* 람다 표현식에서의 중복

```java
final Predicate<String> startsWithN = name -> startsWith("N");
final Predicate<String> startsWithB = name -> startsWith("B");

final long countFriendsStårtN = friends.stream().filter(startsWithN).count();
final long countFriendsStårtB = friends.stream().filter(startsWithB).count();
``` 

테스트하는 문자가 N과 B로 다르지만 이도 Predicate를 두 개 사용하는 중복이다.

* 렉시컬 스코프로 중복 제거하기

```java
public static Predicate<String> checkIfStartsWith(final String letter) {
    return name -> name.startsWith(letter);
}

final long countFriendsStartN = friends.stream().filter(checkIfStartsWith("N")).count();
final long countFriendsStartB = friends.stream().filter(checkIfStartsWith("B")).count();
```

비교할 목적으로 문자를 나중에 사용하기 위해 변수를 캐시해두거나 파라미터로 전달될 때까지 보관한다. 여기서는 name이 해당된다. checkIfStartsWith()는 결과로 함수를 리턴한다. 이것 역시 고차 함수이다.<br>
name은 람다 표현식에 전달되는 파라미터고 letter는 anonymous 함수의 범위에 있지 않기 때문에 람다 표현식의 정의에 대해 범위를 정하고 범위 안에서 변수 letter를 찾는다.

이를 `렉시컬 스코프(lexical scope)`라고 한다. 렉시컬 스코프는 사용한 하나의 컨텍스트에서 제공한 값을 캐시해 두었다가 나중에 다른 컨텍스트에서 사용하는 강력한 기술이다.

이렇게 렉시컬 스코프를 활용하여 코드의 중복성을 제거할 수 있다.

## 렉시컬 스코프
람다 표현식에서는 final이나 scope 내에 있는 effectively final인 지역 변수만을 액세스할 수 있다.<br>
람다 표현식은 정확한 시점에 호출될 수도 있지만 늦게, 혹은 멀티 스레드에서 여러 번 호출될 수도 있다. 레이스 컨디션을 피하기 위해 scope 내에서 액세스하는 지역 변수는 초기화된 후에 변경하면 안 된다. 이 변수에 대한 변경 시도는 컴파일 오류를 일으킨다.

final로 선언된 변수만 사용하면 문제를 해결할 수야 있지만 자바에서는 반드시 강제하는 건 아니다. 대신 두 가지 조건이 필요하다.

* 액세스 된 변수는 람다 표현식이 정의되기 전에 변수를 사용하는 메서드에서 초기화되어야 한다.
* 이 변수들의 값은 어느 곳에서도 변경할 수 없다.

로컬 상태를 캡처하는 람다 표현식을 사용하는 경우에, stateless 람다 표현식이 런타임 상수라는 점을 유의해야 한다. 로컬 상태를 계속 캡처하는 것은 추가적인 평가 비용이 들어간다.

> stateless 람다는 free variable을 참조하지 않는 람다 표현식을 말한다.(반대는 stateful 람다) 참조 : [https://dreamchaser3.tistory.com/5](https://dreamchaser3.tistory.com/5)

* 적용 범위를 좁히기 위한 리팩토링

위에서 본 예제와 같이 정적 메서드를 사용하는 것은 바람직하지 못하다. 필요한 곳에서만 사용되도록 함수의 범위를 좁게 만드는 것이 좋다.

```java
// 1.
final Function<String, Predicate<String>> startsWithLetter = (String letter) -> {
    Predicate<String> checkStarts = (String name) -> name.startsWith(letter);
    return checkStarts;
};

// 2.
final Function<String, Predicate<String>> startsWithLetter = (String letter) -> (String name) -> name.startsWith(letter);

// 3.
final Function<String, Predicate<String>> startsWithLetter = letter -> name -> name.startsWith(letter);

friends.stream().filter(startsWithLetter.apply("N")).forEach(System.out::println);
```

**1번**은 기존 startsWithLetter 정적 메서드를 대체하여 함수 안에 존재하도록 구현하였다.

**2번**은 1번을 간결하게 리팩토링 하였다. 

**3번**은 2번에 더해 타입을 제거하고 자바 컴파일러가 해당 타입을 컨텍스트에 따라 추론할 수 있도록 하였다.
# element 선택

* 명령형 스타일

```java
public static void pickName(final List<String> names, final String startingLetter) {
    String foundName = null;
    for (String name : names) {
        if (name.startsWith(startingLetter)) {
            foundName = name;
            break;
        }
    }
    (foundName != null) ? System.out.println(foundName) : System.out.println("No name found");
}
```

* 함수형 스타일

```java
public static void pickName(final List<String> names, final String startingLetter) {
    final Optional<String> foundName = names.stream()
                                            .filter(name -> name.startsWith(startingLetter))
                                            .findFirst();

    System.out.println(String.format("%s: %s"), startingLetter, foundName.orElse("No name found"));
}
```

filter() 메서드로 원하는 패턴과 매칭되는 모든 element를 가져오고 findFirst() 메서드로 컬렉션에서 첫 번째 값을 추출한다.
# 컬렉션을 하나의 값으로 reduce
element들을 비교하고 컬렉션에서 하나의 값으로 연산하는 기술에 대해 알아본다.

* 문자 전체 수 계산

```java
friends.stream().mapToInt(name -> name.length()).sum();
```

mapToInt()메서드로 이름을 길이로 한 번 변환 후, sum() 메서드로 그 값들을 합한다.<br>
mapToInt() 대신 mapToDouble(), mapToLong() 그리고 sum() 대신 max(), sorted(), average() 등을 사용할 수도 있다.

* 두 element를 비교하고 결과를 한 번에 얻어내는 연산

```java
// 1.
final Optional<String> aLongName = friends.stream()
                                          .reduce((name1, name2) -> 
                                                name1.length() >= name2.length() ? name1 : name2);

// 2.
final Optional<String> aLongName = friends.stream()
                                          .reduce("Steve", (name1, name2) -> 
                                                name1.length() >= name2.length() ? name1 : name2);
aLongName.ifPresent(name -> System.out.println(name));  
``` 

위 예제는 Strategy Pattern의 경량화 애플리케이션이다.<br>
이 람다 표현식은 BinaryOperator 함수형 인터페이스의 apply() 메서드 인터페이스를 따른다. 이는 reduce() 메서드가 받는 파라미터의 타입이다.<br>
reduce() 의 결과는 Optional이며 그 이유는 reduce()가 호출된 컬렉션 리스트가 빈 값일 수 있기 때문이다. 리스트가 오직 하나의 element만 갖고 있다면 reduce()는 그 element를 리턴하고 람다 표현식은 호출되지 않는다.

**2번**은 default 값으로 Steve를 설정한 예제다. default 값이 설정된 경우, 해당 컬렉션이 빈 값이더라도 이 때는 Optional을 리턴하지 않는다. 기본 값이 세팅돼있기 때문이다.
# element 조인
* 명령형 스타일

```java
// 1.
for(String name : friends) {
    System.out.println(name + ", );
}

// 2.
System.out.println(String.join(", " + friends));

// 3.
System.out.println(friends.stream().map(String::toUpperCase).collect(joining(", ")));
```

String의 join() 메서드는 StringJoiner를 호출하여 두 번째 매개변수인 가변인자 안에 있는 값들을 첫 번째 인수로 분리된 하나의 큰 스트링으로 합친다.
