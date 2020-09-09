---
date: 2020-09-10 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 3. String, Comparator 그리고 filter
description: 3. String, Comparator 그리고 filter
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
# 스트링 이터레이션
* chars() 메서드

```java
final String str = "w00t";

// 1.
str.chars().forEach(ch -> System.out.println(ch));  //  119 48 48 116

// 2.
str.chars().forEach(System.out::println);
```

chars() 메서드는 CharSequence 인터페이스로부터 파생한 String 클래스에 있는 새로운 메서드다. forEach()와 같은 내부 이터레이터를 사용하여 이터레이션하는 스트림을 리턴한다. 하지만 결과는 문자 대신 숫자가 출력된다.<br>
그 이유는 chars() 메서드가 Characters의 스트림 대신 문자를 표현하는 int의 스트림을 리턴하기 때문이다.(IntStream)<br>

**2번**은 메서드 레퍼런스를 사용하여 파라미터 라우팅을 할 수 있다. 메서드 레퍼런스는 클래스의 이름을 기반으로 하기 때문에 예를 들자면 name.toUppercase()가 곧 String::toUppercase로 변환될 수 있다. String::toUppercase는 자바 컴파일러에 의해 합성되는 메서드에 대한 파라미터이며, 이 메서드는 메서드 호출의 타깃으로 변환된다.(즉, parameter.toUppercase();와 같다. )<br>
다만 2번 예제에서는 PrintStream 클래스의 인스턴스로 정적 레퍼런스 System.out을 통해 액세스된다. 메서드를 위한 타깃을 이미 제공하기 때문에, 자바 컴파일러는 합성된 메서드의 파라미터를 레퍼런스하는 메서드의 인수로 사용한다. 즉, System.out.println(paramter);와 같은 코드가 된다.

* 숫자로 출력되는 문제 개선

```java

    public static class IterateString {
        private static void printChar(int aChar) {
            System.out.println((char)(aChar));
        }
    }

    public static void main(String[] args) {
        final String str = "w001";
        str.chars().forEach(IterateString::printChar);
    }
```
chars()의 결과는 int로 계속 사용되나 출력 시 printChar() 메서드에서 문자로 변환된다.

* 숫자로 출력되는 문제 개선2

```java
str.chars()
    .mapToObj(ch -> Character.valueOf((char)ch))
    .forEach(System.out::println);
```
처음부터 int가 아닌 문자로 처리하는 방법이다.

* 숫자만 필터링하여 출력 처리

```java
// 1.
str.chars()
   .filter(ch -> Character.isDigit(ch))
   .forEach(ch -> printChar(ch));

// 2.
str.chars()
   .filter(Character::isDigit)
   .forEach(IterateString::printChar);    
```
메서드 레퍼런스는 일반적인 파라미터 라우팅을 제거한다.
 
String::toUppercase는 인스턴스 메서드, Character::isDigit은 정적 메서드이다. 

> * String::toUppercase
> 
> ```java
> public final class String implements Serializable, Comparable<String>, CharSequence {
>   public String toUpperCase() {
>       return this.toUpperCase(Locale.getDefault());
>   }
> }
> ```
> 
> * Chracter::isDigit
> 
> ```java
>public final class Character implements java.io.Serializable, Comparable<Character> {
>   public static boolean isDigit(char ch) {
>          return isDigit((int)ch);
>   }
> }
> ``` 

구조는 같아보이지만 자바 컴파일러는 인스턴스/정적 메서드 여부를 체크하여 인스턴스 메서드이면 합성된 메서드의 파라미터를 호출되는 타깃으로 사용한다.(ex. parameter.toUppercase())<br>
반면에 정적 메서드이면 합성된 메서드에 대한 파라미터는 이 메서드의 인수로 라우팅된다.(ex. Character.isDigit(paramter))

이러한 파라미터 라우팅은 상당히 편리하지만 메서드의 충돌과 모호함이라는 문제를 갖고 있다. 인스턴스 메서드와 정적 메서드 모두 사용 가능하다면 컴파일러 입장에서는 어떤 메서드를 사용해야 할지 결정할 수 없기 때문에 컴파일 에러가 발생하게 된다.<br>
예를 들어, Double::toString 메서드 레퍼런스를 사용하게 된다면 컴파일러는 정적 메서드인 `public static String toString(double value)`와 `public String의 toString()` 인스턴스 메서드 중 어느 것을 사용해야 할지 결정하지 못하게 된다.

* ex.

```java
final List<Double> value = Arrays.asList(1.01, 2.02);
value.stream().map(Double::toString).forEach(System.out::println);
```  

* compile error

```text
Error:(23, 35) java: incompatible types: cannot infer type-variable(s) R
    (argument mismatch; invalid method reference
      reference to toString is ambiguous
        both method toString(double) in java.lang.Double and method toString() in java.lang.Double match)
```

# Comparator 인터페이스의 구현
Comparator 인터페이스는 자바 8에서 함수형 인터페이스로 바뀌면서 다양한 기법으로 사용된다.

* 정렬 - 오름차순

```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public int ageDifference(final Person other) {
        return age - other.age;
    }

    public String toString() {
        return String.format("%s - %d", name, age);
    }
}

final List<Person> people = Arrays.asList(
    new Person("John", 20),
    new Person("Sara", 21),
    new Person("Jane", 21),
    new Person("Greg", 35)
);

// 1.
List<Person> ascendingAge = people.stream()
            .sorted((person1, person2) -> person1.ageDifference(person2))
            .collect(Collectors.toList());

// 2.
List<Person> ascendingAge = people.stream()
            .sorted(Person::ageDifference)
            .collect(Collectors.toList());

ascendingAge.stream().forEach(System.out::println);
```
sorted() 메서드는 파라미터로 Comparator를 갖는다. Comparator가 함수형 인터페이스이기 때문에 람다 표현식을 인수로 쉽게 넘길 수 있다. sorted()의 실행 메커니즘은 reduce()와 흡사하다.

**1번**은 람다 표현식에서 두 개의 파라미터를 라우팅해야 한다. 첫 번째 파라미터는 ageDifference() 메서드에 대한 타깃이며 두 번째는 그 메서드에 대한 인수다.
**2번**은 메서드 레퍼런스를 활용하여 간결하게 표현하였다. 두 개의 파라미터를 갖는 람다 표현식을 메서드 레퍼런스로 표현할 때도 마찬가지로 자바 컴파일러는 첫 번째 파라미터는 ageDifference() 메서드의 타깃을 만들고 두 번째 파라미터는 그 메서드에 대한 파라미터로 사용한다.

* 정렬 - 내림차순

```java
// 1.
List<Person> ascendingAge = people.stream()
            .sorted((person1, person2) -> person2.ageDifference(person1))
            .collect(Collectors.toList());

// 2.
Comparator<Person> compareAscending = (person1, person2) -> person1.ageDifference(person2);
Comparator<Person> compareDescending = compareAscending.reversed();

List<Person> ascendingAge1 = people.stream()
    .sorted(compareDescending)
    .collect(Collectors.toList());
``` 
내림차순 표현은 **1번**예제와 같이 파라미터의 순서만 변경하면 된다.<br>
그렇다면 이를 메서드 레퍼런스로 구현하는 것은 어떨까. 아쉽게도 람다 표현식처럼 간단하게 구현할수는 없다. 왜냐하면 메서드 레퍼런스는 파라미터 순서가 위와 같은 라우팅 규칙을 따르지 않기 때문이다.

**2번**과 같이 reversed() 메서드를 활용하여 내림차순으로 구현할 수 도 있다. reversed() 메서드는 Comparator에 있는 디폴트 메서드다.

```java
people.stream()
      .min(Person::ageDifference)
      .ifPresent(youngest -> System.out.println(youngest));

people.stream()
      .max(Person::ageDifference)
      .ifPresent(youngest -> System.out.println(youngest));  
``` 
위 코드 처럼 가장 어리거나 많은 사람만 골라낼 수도 있다.
# 여러 가지 비교 연산
```java
// 1.
people.stream()
      .sorted((person1, person2) -> person1.getName().compareTo(person2.getName()));

// 2.
final Function<Person, String> byName = person -> person.getName();
people.stream().sorted(Comparator.comparing(byName));
```
**1번**은 이름을 오름차순으로 정렬한다.

**2번**은 1번을 Comparator 인터페이스에 있는 comparing() 메서드를 활용한다. comparing() 메서드는 Function 타입의 람다 표현식을 파라미터로 받는다.
```java
static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator)((Serializable)((c1, c2) -> {
        return ((Comparable)keyExtractor.apply(c1)).compareTo(keyExtractor.apply(c2));
    }));
}
```

* 중복 비교

```java
final Function<Person, Integer> byAge = person -> person.getAge();
final Function<Person, String> byTheirName = person -> person.getName();

people.stream().sorted(Comparator.comparing(byAge).thenComparing(byTheirName))
               .forEach(System.out::println);
```
두 개의 람다 표현식을 통해 중복 비교가 필요하다면 먼저 comparing() 메서드를 사용하여 Comparator 타입을 리턴 후, 리턴된 Comparator에서 thenComparing() 메서드를 호출해 나이와 이름 두 값에 따라 비교하는 복합 (composite) comparator를 생성한다.
# collect 메서드와 Collectors 클래스 사용하기
collect() 메서드는 컬렉션을 다른 형태, 즉 가변 컬렉션(mutable collection)으로 변경하는 reduce 오퍼레이션이다. collect() 메서드에서는 Collectors 클래스의 utiliy 메서드들과 조합하여 사용하면 더욱 편리하다.
# 디렉터리에서 모든 파일 리스트
# 디렉터리에서 선택한 파일 리스트
# flatMap을 사용하여 서브 디렉터리 리스트
# 파일 변경