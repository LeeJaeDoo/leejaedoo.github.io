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

구조는 같아보이지만 자바 컴파일러는 인스턴스/정적 메서드 여부를 체크하여 인스턴스 메서드이면 합성된 메서드의 파라미터로 호출되는 타깃이 된다.(ex. parameter.toUppercase())<br>
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
# 여러 가지 비교 연산
# collect 메서드와 Collectors 클래스 사용하기
# 디렉터리에서 모든 파일 리스트
# 디렉터리에서 선택한 파일 리스트
# flatMap을 사용하여 서브 디렉터리 리스트
# 파일 변경