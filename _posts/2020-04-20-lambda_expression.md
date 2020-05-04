---
date: 2020-04-20 18:40:40
layout: post
title: 자바 8 인 액션
subtitle: 3. 람다 표현식
description: 3. 람다 표현식
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
## 람다란?
메서드로 전달할 수 있는 익명 함수를 단순화 한 것이다.

### 특징
* 익명
보통의 메서드와 달리 이름이 없다.
* 함수
람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
* 전달
람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
* 간결성
익명 클래스처럼 자질구레한 코드 구현이 필요 없다.

> 람다 표현식을 활용함으로써 코드가 간결해지고 유연해진다.

### 람다의 구성요소
```java
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
* 파라미터 리스트
기존 Comparator의 compare 메서드의 파라미터인 `(Apple a1, Apple a2)`를 지칭한다.
* 화살표
`->`를 통해 파라미터 리스트와 바디를 구분한다.
* 람다의 바디
람다의 반환값에 대한 표현식이다.(ex. `a1.getWeight().compareTo(a2.getWeight())`)

### 람다의 문법
람다 표현식에는 return이 함축되어 있기 때문에 return문을 명시하지 않아도 된다.
* () -> {}<br>
파라미터가 없으며 `void`를 반환하는 람다 표현식이다. 이는 `public void run() {}`와 같이 바디가 없는 메서드와 같다.
* () -> "Hello"<br>
파마리터가 없으며 문자열을 반환하는 표현식이다. "Hello"는 구문이 아닌 표현식이기 때문에 세미콜론을 붙여선 안된다.
* () -> { return "Mario"; }<br>
파라미터가 없으며 (**명시적으로** return 문을 이용해서) 문자열을 반환하는 표현식이다.
```java
void lambdaTest() {
    return "Mario";
}
```
* (String s) -> { return "Alan" + s; } = (String s) -> "Alan" + s<br>
return은 흐름 제어문이기 때문에 {}로 감싸줘야 한다. return이 없다면 흐름 제어문이 아니므로 {}과 세미콜론을 빼야한다.

## 람다가 사용되는 경우
### 함수형 인터페이스
함수형 인터페이스란 정확히 하나의 추상 메서드를 지정하는 인터페이스이다. Comparator, Runnable 등이 있다.
> 많은 디폴트 메서드가 있다하더라도 추상 메서드가 하나이면 함수형 인터페이스이다.

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로, `전체 표현식을 함수형 인터페이스의 인스턴스로 취급`할 수 있다.

> Comparator는 `compare()`와 `equals()` 두 메서드가 존재하는데 함수형 인터페이스인 이유는 ?<br>
>-> 참고 :  [https://stackoverflow.com/questions/43616649/how-can-comparator-be-a-functional-interface-when-it-has-two-abstract-methods/43616695](https://stackoverflow.com/questions/43616649/how-can-comparator-be-a-functional-interface-when-it-has-two-abstract-methods/43616695)

### 함수 디스크립터
함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 `함수 디스크립터`라고 한다.
> 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있다.

#### 올바른 람다 표현식 사용 코드
```java
execute(() -> {});
public void execute(Runnable r) {
    r.run();
}
```
`() -> {}`의 시그니처는 `() -> void`이며 Runnable의 추상 메서드 run의 시그니처와 일치하다.
```java
public Callable<String> fetch() {
    return () -> "Tricky example ;-)";
}
```
Callable<String>의 시그니처는 `() -> String`이 되는데 return 형식도 이와 일치한다.
```java
Predicate<Apple> p = (Apple a) -> a.getWeight();
```
Predicate<Apple> p의 시그니처는 `Predicate<Apple>: (Apple) -> boolean`이지만 (Apple a) -> a.getWeight()의 시그니처는 `(Apple a) -> Integer`로 다르므로 유효한 람다 표현식이 아니다.

> @FunctionalInterface 란 함수형 인터페이스에 선언하는 어노테이션으로 만약 선언된 인터페이스에 추상 메서드의 개수가 한 개이상이라면 `Multiplenonoverriding abstract methods found in interface Foo` 오류가 나게 된다. 

## 람다의 활용: 실행 어라운드 패턴(Execute Around Pattern)
실행 어라운드 패턴이란 실제 데이터를 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다.
```java
public static String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();       //  실제 작업이 처리되는 구간
    }
}
```

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public static String processFile( BufferedReaderProcessor p ) throws IOException {
    ...
}
```
위와 같이 함수형 인터페이스를 활용하여 람다를 적용한다. 따라서 위와 같이 BufferedReader -> String 과 IOException 을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 구현해야 한다. 
```java
public static String processFile( BufferedReaderProcessor p ) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```
위와 같이 이제 BufferedReaderProcessor에 정의된 process 메서드의 시그니처(BufferedReader -> String)와 일치하는 람다를 전달할 수 있다. 이로써 processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있다.
```java
    String oneLine = processFile((BufferedReader br) -> br.readLine());

    String twoLine = processFile((BufferedReader br) -> br.readLine());
```
이제 람다를 이용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.

## 함수형 인터페이스 사용
다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.
### Predicate
`java.util.function.Predicate<T>` 인터페이스는 test라는 추상 메서드를 정의하며, test는 제네릭 형식 T의 객체를 인수로 받아 boolean을 반환한다.
별도의 정의 없이 바로 사용할 수 있다. T 형식의 객체를 사용하는 boolean 표현식이 필요한 상황에서 Predicate 인터페이스를 사용할 수 있다.
* ex.
```java
@FunctionalInterface
public interface Perdicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for(T s: list) {
        if(p.test(s))   results.add(s);
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```
### Consumer
`java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다. T 형식이 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있다. 예를 들면 forEach 메서드를 정의할 때 Consumer를 활용할 수 있다.
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}    

public static <T> void forEach(List<T> list, Consumer<T> c) {
    for (T i : list) {
        c.accept(i);
    }
}

forEach(
    Arrays.asList(1, 2, 3, 4, 5),
    (Integer i) -> System.out.println(i);   //  Consumer의 accept 메서드를 구현하는 람다
);
```

### Function
`java.util.function.Function<T, R>` 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 apply라는 추상 메서드를 정의한다. 입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
    for (T s : list) {
        result.add(f.apply(s));
    }
    return result;
}

List<Integer> l = map(
                        Arrays.asList("lambdas", "in", "action"),
                        (String s) -> s.length()    //  Function의 apply메서드를 구현하는 람다
                     );  
```

>자바의 함수형 인터페이스를 활용할 때 제네릭 파라미터에 의해 오토박싱이 적용될 경우 소모되는 비용을 줄이기 위해 자바 8에서는 기본형에 특화된 인터페이스를 제공한다.
* 기본형 특화 인터페이스
<table>
  <thead>
    <tr>
      <th>함수형 인터페이스</th>
      <th>함수 디스크립터</th>
      <th>기본형 특화</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Predicate< T ></td>
      <td>T -> boolean</td>
      <td>IntPredicate, LongPredicate, DoulePredicate</td>
      <td></td>
    </tr>
    <tr>
      <td>Consumer< T ></td>
      <td>T -> void</td>
      <td>IntConsumer, LongConsumer, DoubleConsumer</td>
      <td>추상 메서드 accept를 정의한다.</td>
    </tr>
    <tr>
      <td>Function< T, R ></td>
      <td>T -> R</td>
      <td>IntFunction< R >, IntToDoubleFunction, IntToLongFunction, LongFunction< R >, LongToDoubleFunction, LongToIntFunction, DoubleFunction ...</td>
      <td>Function< Integer, Integer >로 사과의 무게 정보를 추출할 수 있다.</td>
    </tr>
    <tr>
      <td>Supplier< T ></td>
      <td>() -> T</td>
      <td>BooleanSupplier, IntSupplier, LongSupplier, DoubleSupplier</td>
      <td>추상 메서드 get을 정의한다.</td>
     </tr>
     <tr>
      <td>UnaryOperator< T ></td>
      <td>T -> T</td>
      <td>IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator</td>
      <td></td>
     </tr>
     <tr>
      <td>BinaryOperator< T ></td>
      <td>(T, T) -> T</td>
      <td>IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator</td>
      <td>IntBinaryOperator는 applyAsInt의 추상 메서드를 갖는다.</td>
     </tr>
     <tr>
      <td>BiPredicate< L, R ></td>
      <td>(L, R) -> boolean</td>
      <td></td>
      <td></td>
     </tr>
     <tr>
      <td>BiConsumer< T, U ></td>
      <td>(T, U) -> void</td>
      <td>ObjIntConsumer< T >, ObjLongConsumer< T >, ObjDoubleConsumer< T ></td>
      <td></td>
     </tr>
     <tr>
      <td>BiFunction< T, U, R ></td>
      <td>(T, U) -> R</td>
      <td>ToIntBiFunction< T, U >, ToLongBiFunction< T, U >, ToDoubleBiFunction< T, U ></td>
      <td></td>
     </tr>
  </tbody>
</table>
* 람다와 함수형 인터페이스 예제
<table>
  <thead>
    <tr>
      <th>사용 사례</th>
      <th>람다 예제</th>
      <th>대응하는 함수형 인터페이스</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>boolean 표현</td>
      <td>(List< String > list) -> list.isEmpty()</td>
      <td>Predicate< List < String > ></td>
    </tr>
    <tr>
      <td>객체 생성</td>
      <td>() -> new Apple(10)</td>
      <td>Supplier< Apple ></td>
    </tr>
    <tr>
      <td>객체에서 소비</td>
      <td>(Apple a) -> System.out.println(a.getWeight())</td>
      <td>Consumer< Apple ></td>
    </tr>
    <tr>
      <td>객체에서 선택/추출</td>
      <td>(String s) -> s.length()</td>
      <td>Function< String, Integer > or ToIntFuncton< String ></td>
     </tr>
    <tr>
      <td>두 값 조합</td>
      <td>(int a, int b) -> a * b</td>
      <td>IntBinaryOperator</td>
     </tr>
    <tr>
      <td>두 객체 비교</td>
      <td>(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())</td>
      <td>Comparator< Apple > or BiFunction< Apple, Apple, Integer > or ToIntBiFunction< Apple, Apple ></td>
     </tr>
  </tbody>
  </table>
  
* 예외, 람다, 함수형 인터페이스의 관계
함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 즉, 예외를 덩지는 람다 표현식을 만드려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch 블록으로 감싸야 한다.
  
## 형식 검사/추론, 제약
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다. 따라서 람다 표현식을 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.
  
### 형식 검사
#### 확인 과정
```java
  List<Apple> heavierThan150g = filter(inventory, (Apple a) -> a.getWeight() > 150);
```
1. filter의 정의를 확인한다.
    filter(inventory, `(Apple a) -> a.getWeight() > 150`); <-> filter(List<Apple> inventory, `Predicate<Apple> p`)
2. 대상 형식 확인 후, 대상 형식(Predicate<Apple> 인터페이스)의 추상 메서드가 무엇인지 확인한다.
    boolean test(Apple apple)
3. 추상 메서드는 Apple을 인수로 받아 boolean을 반환하는 test메서드임을 확인한다.
4. 함수 디스크립터는 Apple -> boolean 이므로 람다의 시그니처와 일치함을 확인한다. 람다도 Apple을 인수로 받아 boolean을 반환하므로 코드 형식 검사가 성공적으로 완료된다.

### 같은 람다, 다른 함수형 인터페이스
대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.(ex. Callable <-> PrivilegedAction)
> 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.
```java
//  Predicate는 boolean 반환값을 갖는다.
Predicate<String> p = s -> list.add(s);
//  Consumer는 void 반환값을 기대하지만 List의 add메서드와 같이 boolean을 반환해도 유효하다.
Consumer<String> b = s -> list.add(s); 
```
> 할당문 컨텍스트 메서드 호출 컨텍스트(파라미터, 반환값), 형변환 콘텍스트 등으로 람다 표현식의 형식을 추론할 수 있다.

### 형식 추론
자바 컴파일러는 대상 형식을 통해 함수 디스크립터를 파악하여 람다의 시그니처까지도 추론할 수 있다.

### 지역 변수 사용
람다 표현식은 외부에서 정의된 변수, 즉 `자유 변수(free variable)`를 활용할 수 있다. 자유 변수를 활용함으로써 인수를 자신의 바디 밖에서도 사용할 수 있다. 이를 `람다 캡처링(cpaturing lambda)`이라고 부른다.
> 하지만, `final`과 같이 한 번만 할당 할 수 있는 지역 변수만 캡처 할 수 있다.

#### 지역 변수의 제약
람다도 thread에서 실행된다면, 변수를 할당한 thread가 사라져 변수 할당이 해제되었음에도 람다를 실행하는 thread에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값이 할당되어야 한다.

* 클로저(clojure)
함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 람다와 익명 클래스 모두 클로저처럼 자신의 외부 영역의 변수에 접근할 수 있다. 따라서 람다가 정의된 메서드의 지역 변숫값은 `final`변수여야 지역 변수의 값을 변경할 수 없게 된다.

## 메서드 레퍼런스
메서드 레퍼런스를 이용하면 기존의 메서드 정의를 재활용하여 람다처럼 활용할 수 있다. 때로는 람다 표현식보다 메서드 레퍼런스를 활용하는 것이 더 가독성이 좋으며 자연스러울 수 있다.
```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
->
inventory.sort(Comparator.comparing(Apple::getWeight()));
```
실제 메서드 레퍼런스는 `(Apple a) -> a.getWeight()`를 `Apple::getWeight()`로 축약한 것이다. 즉, 메서드 레퍼런스는 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있게 하는 문법이다.
### 메서드 레퍼런스 3가지 유형
#### 정적 메서드 레퍼런스
예를 들면 `Integer::parseInt`로 표현할 수 있다.
```java
(args) -> ClassName.staticMethod(args)
->
ClassName::staticMethod
```
#### 다양한 형식의 인스턴스 메서드 레퍼런스
예를 들면 `String::length`로 표현할 수 있다. 
```java
(arg0, rest) -> arg0.instanceMethod(args)
->
ClassName::instanceMethod
```
#### 기존 객체의 인스턴스 메서드 레퍼런스
예를 들면 Transaction 객체를 할당 받은 expensiveTransaction 지역 변수가 있고, Transaction객체에는 getValue 메서드가 있다면 `expensiveTransaction::getValue`로 표현할 수 있다.
```java
(args) -> expr.instanceMethod(args)
->
expr::instanceMethod
```

### 생성자 레퍼런스
ClassName::new처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 레퍼런스를 만들 수 있다.
```java
Function<Integer, Apple> c2 = Apple::new;
->
Function<Integer, Apple> c2 = (weight) -> new Apple();
```

## 람다 표현식을 조합할 수 있는 유용한 메서드
간단한 여러 개의 람다 표현식을 조합해 복잡한 람다 표현식도 만들 수 있다.
### Comparator 조합
#### 역정렬
```java
Inventory.sort(comparing(Apple::getWeight).reversed());
```
인터페이스 자체에서 주어진 reverse라는 디폴트 메서드를 활용하여 별도의 Comparator 인스턴스 생성 없이 비교자의 순서를 뒤바꿔줄 수 있다.

#### Comparator 연결
같은 무게의 사과인 경우 정렬 조건을 추가하려면 다음과 같이 처리할 수 있다.
```java
inventory.sort(comparing(Apple::getWeight)
         .reversed()
         .thenComparing(Apple::getCountry));   
```
thenComparing 메서드를 추가할 수 있다.

### Predicate 조합
Predicate 인터페이스는 복잡한 조합이 가능하도록 negate, and, or 세가지 메서드를 제공한다.
```java
//  기존 프레디케이트 객체 redApple의 결과를 반전시킨 객체를 만든다.
Predicate<Apple> notRedApple = redApple.negate();

// 프레디케이트 메서드를 연결하여 더 복잡한 프레디케이트 객체를 만든다.
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150)
                                                    or(a -> "green".equals(a.getColor()));
```

### Function 조합
Function 인터페이스는 Function 인스턴스를 반환하는 andThen, compose 두 가지 디폴트 메서드를 제공한다.
> andThen 메서드는 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환한다.
> compose 메서드는 인수로 주어진 함수를 먼저 실행한 뒤 그 결과를 외부 함수의 인수로 제공한다.(andThen의 반대개념)

```java
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline =
    addHeader.andThen(Letter::checkSpelling)
             .andThen(Letter::addFooter);
``` 

## 정리
* `람다 표현식`은 익명 함수의 일종으로 이름은 없지만 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다. 이로 인해 간결한 코드를 구현할 수 있다.
* `함수형 인터페이스`는 하나의 추상 메서드만을 정의하는 인터페이스로, 이 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
* 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며, 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
* 실행 어라운드 패턴을 람다를 통해 활용하면 더 유연하고 재사용성이 높은 코드를 구현할 수 있다.
* `메서드 레퍼런스`를 통해 기존의 메서드 구현을 재사용하고 직접 전달할 수 있고 람다 표현식 보다 더 간결하게 코드를 구현할 수도 있다.
* Comparator, Predicate, Funcion 같은 함수형 인터페이스는 람다 표현식을 조합하여 다양한 디폴트 메서드를 제공한다.