---
date: 2020-10-09 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 6. 레이지
description: 6. 레이지
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
categories: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---

코드를 실행할 때 실행 순서가 됐을 때 바로 실행하기 보다 약간 지연(lazy)시키게 되면 성능 향상을 얻을 수 있는 경우가 있다. 즉시 실행되는 eager 방식은 간단하지만 lazy 방식은 효율적이다.

애플리케이션에 heavyweight한 객체가 사용된다면, 이 객체에 대한 생성 작업은 되도록 lazy하게 하고 싶을 것이다. 보통 lazy한 작업을 하기 위해 복잡한 처리가 필요한데 자바 8에서는 람다 표현식을 활용함으로써 lazy하면서도 빠르게 수행할 수 있게 해준다.
# 지연 초기화
객체 내부의 일부분이 헤비웨이트 리소스인 경우 그 객체의 생성을 늦추면 이점을 얻을 경우가 있다. 이런 작업은 전반적으로 객체 생성의 속도를 향상할 수 있고 프로그램은 사용하지도 않는 객체를 생성하기 위한 노력을 들이지 않아도 된다.
## 익숙한 방법
```java
public class Heavy {

    public Heavy() {
        System.out.println("Heavy created");
    }

    public String toString() {
        return "quite heavy";
    }
}

public class HolderNaive {
    private Heavy heavy;

    public HolderNaive() {
        System.out.println("Holder created");
    }

    public Heavy getHeavy() {
        if (heavy == null) {
            heavy = new Heavy();
        }
        return heavy;
    }
    //...
}

public static void main(String[] args) throws IOException {
    final HolderNaive holder = new HolderNaive();
    System.out.println("deferring heavy creation...");
    System.out.println(holder.getHeavy());
    System.out.println(holder.getHeavy());
}
```

```text
Holder created
deferring heavy creation...
Heavy created
quite heavy
quite heavy
```
위 예시 코드는 thread safety하지 않다.
## thread safety의 제공
위 코드를 보면 getHeavy() 메서드가 처음 호출될 때 Heavy 인스턴스가 생성된다. 그 후 재차 getHeavy() 메서드를 호출하면 이미 생성된 Heavy 객체가 리턴된다. 이는 우리가 원하는 결과지만 이 코드를 실행하면 race condition 상태가 되는 문제가 있다.

> race condition이란? [http://blog.naver.com/PostView.nhn?blogId=winipe&logNo=150162868972&parentCategoryNo=&categoryNo=23&viewDate=&isShowPopularPosts=true&from=search](http://blog.naver.com/PostView.nhn?blogId=winipe&logNo=150162868972&parentCategoryNo=&categoryNo=23&viewDate=&isShowPopularPosts=true&from=search)

두 개 이상의 thread가 동시에 getHeavy() 메서드를 호출한다면 다중 Heavy 인스턴스를 갖게 되고 결국 thread당 하나씩의 인스턴스가 생성된다. 이런 side effect는 바람직하지 않다.

```java
public synchronized Heavy getHeavy() {
    if (heavy == null) {
        heavy = new Heavy();
    }

    return heavy;
}
```

getHeavy()를 synchronized 키워드로 마크해서 상호 배제를 보장하도록 한다. `동시에 이 메서드를 두 개 이상의 스레드가 호출한다면 상호 배제 때문에 하나의 스레드만이 메서드에 진입할 수 있고 다른 스레드들은 큐에 대기하며 차례를 기다리게 된다.` 메서드에 처음 진입한 스레드는 인스턴스를 생성한다. 다음 스레드가 이 메서드에 진입하면 이미 인스턴스가 생성됐다는 것을 알게 되고 간단하게 이미 생성되어 있는 인스턴스를 리턴한다.

하지만, race condition은 피했어도 getHeavy() 메서드에 대한 모든 호출은 동기화 오버헤드를 갖게 됐다. 스레드를 호출하는 것은 동시에 경쟁하는 스레드는 없다고 하더라도 메모리 장벽을 넘나드는 오버헤드를 갖는다.

사실 race condition은 heavy 레퍼런스가 처음 할당되는 경우에만 발생하기 때문에 발생할 빈도가 매우 낮다. 따라서 `동기화 방법이 오히려 오버헤드가 큰 방법`이다. 레퍼런스가 처음 생성될 때까지 thread safety가 필요하며 그 이후에는 레퍼런스에 대한 제약 없는 액세스를 해제한다.

## 인다이렉션(Indirection) 레벨 추가
인다이렉션은 Supplier<T> 클래스에서 왔다. 이것은 JDK의 함수형 인터페이스이며 get 추상 메서드를 갖고 있고, 인스턴스를 리턴한다. 즉, 입력과 같이 무엇인가를 기대하지 않고도 계속 주기만 하는 팩토리이다.

* Supplier 예제

```java
Supplier<Heavy> supplier = () -> new Heavy();
```

Supplier는 인스턴스를 리턴하기 때문에 일반적으로 사용해온 new를 사용하여 인스턴스를 초기화하는 대신 생성자 레퍼런스를 사용할 수 있다.<br>
생성자 레퍼런스는 메서드 레퍼런스와 비슷하지만, 메서드 대신 생성자에 대한 레퍼런스를 나타낸다.

* 생성자 레퍼런스 예제

```java
Supplier<Heavy> supplier = Heavy::new;
```

Supplier는 인스턴스를 지연시키고 캐시하는 것이 필요하다. 인스턴스 생성을 다른 함수에 이동시켜 이 작업을 할 수 있다.

* Supplier를 적용한 Holder 클래스 최종 예제

```java
public class Holder {
    private Supplier<Heavy> heavy = this::createAndCacheHeavy;

    private synchronized Heavy createAndCacheHeavy() {
        class HeavyFactory implements Supplier<Heavy> {
            private final Heavy heavyInstance = new Heavy();
            
            public Heavy get() {return heavyInstance;}
        }
        
        if(!(heavy instanceof HeavyFactory)) {
            heavy = new HeavyFactory();
        }
        
        return heavy.get();
    }
    
    public Holder() {
        System.out.println("Holder created");
    }

    public Heavy getHeavy() {
        return heavy.get();
    }
}
```

Holder 인스턴스가 생성될 때 Heavy 인스턴스는 생성되지 않는다. 이렇게 설계함으로써 레이지 초기화(Lazy Initialization)의 장점을 얻을 수 있다. 또한, thread safety에 대해 엄격하지 않은 해결책도 필요하다.

createAndCacheHeavy() 메서드는 synchronized로 마크함으로써 스레드가 동시에 이 메서드를 호출하는 것은 상호 배제된다.<br>
그러나, 이 메서드에는 첫 번째 호출에서 Supplier 레퍼런스 heavy를 직접 supplier인 HeavyFactor로 대체하여 Heavy의 인스턴스를 리턴한다.<br>
이렇게 처리함으로써 thread safety 문제를 해결할 수 있다.

만약 두 개의 스레드가 동시에 getHeavy() 메서드를 접근한다면 Supplier의 createAndCacheHeavy() 메서드가 둘 중 하나를 선택하게 되고 다른 스레드는 대기한다.<br>
진입한 **첫 번째 스레드**는 heavy가 heavyFactory의 인스턴스인지 체크하고 이 때 heavy는 디폴트 Supplier가 아니기 때문에 heavy를 heavyFactory 인스턴스로 바꾼다.<br>
마지막으로, HeavyFactory가 갖고 있는 Heavy 인스턴스를 리턴한다.<br>
진입하는 **두 번째 스레드**는 heavy가 HeavyFactory의 인스턴스인지를 다시 체크만하고 생성 부분은 첫 번째 스레드가 리턴한 같은 인스턴스를 리턴하기 때문에 넘어간다.<br>
여기서 Heavy 자체는 스레드 세이프라고 가정하고 Holder의 thread safety에만 집중한다.

인스턴스를 lazy하게 생성하기 떄문에 race condition에 대해 주의 깊게 보호할 필요가 없어진다.<br>
이제 heavy는 HeavyFactory로 교체되고 getHeavy() 메서드의 다음 호출이 직접 HeavyFactory의 get() 메서드를 호출하며 어떤 동기화 오버헤드도 발생시키지 않는다.

이로써 lazy initialization을 설계했고 null 체크도 피했다. 게다가 lazy 인스턴스 생성의 thread safety도 보장했다.<br>
이것은 간단하며 가상 proxy 패턴(virtual proxy pattern)의 경량화 구현이 된다. 
# 레이지 이밸류에이션
람다 표현식을 사용하여 함수 이밸류에이션(function evaluation)을 지연시킬 수도 있다.

이전 섹션에서는 heavyweight 객체의 생성을 지연시키는 방식을 소개했다면 이번 섹션에서는 **실행하는 메서드를 지연시키는 방법**을 알아본다.

자바는 이미 논리 operation을 실행할 때 lazy execution을 사용한다. 이러한 쇼트 서킷으로부터 프로그램은 불필요한 서술문이나 함수의 평가를 피하고 성능 향상의 이점을 갖는다.<br>

# 스트림의 레이지 강화
# 무한, 그리고 레이지 컬렉션의 생성
# 정리
