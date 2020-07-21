---
date: 2020-07-22 18:40:40
layout: post
title: 토비의 스프링
subtitle: 4. 예외
description: 4. 예외
image: https://leejaedoo.github.io/assets/img/spring.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/spring.jpeg
category: spring
tags:
  - spring
  - 책 정리
paginate: true
comments: true
---
# 사라진 SQLException
## 초난감 예외처리
### 예외 블랙홀
* 초난감 예외처리 코드 1

```java
try {
    ...
} catch (SQLException e) {  //  예외를 잡고 아무것도 하지 않는다. 예외 발생을 무시해버리고 정상적인 상황인 것처럼
                            //  다음 라인으로 넘어가면 안된다.
}
```
위 처럼 예외가 발생했을 때 catch 블록을 써서 잡아내는 것은 좋지만, 잡아내놓고 아무것도 하지 않고 넘어가는 것은 매우 위험한 짓이다. 왜냐하면 프로그램 실행 중에 어디선가 오류가 있어서 예외가 발생했는데 그것을 무시하고 계속 진행해버리기 때문이다.

* 초난감 예외처리 코드 2

```java
} catch (SQLException e) {
    System.out.println(e);
}
```

* 초난감 예외처리 코드 3

```java
} catch (SQLException e) {
    e.printStackTrace();
}
```

위와 같은 처리는 개발 중에는 에러 로그를 파악하기 쉽지만, 운영서버에 올라가면 누군가 계속 모니터링 하지 않는한 폭탄으로 남아있게 된다.<br>
`예외는 처리돼야 한다.` 그런데 위 코드는 catch 블록을 이용해 메시지를 출력할 뿐 예외를 처리한 게 아니다.<br>
예외를 처리할 때 핵심 원칙은 모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 한다.
### 무의미하고 무책임한 throws
* 초난감 예외처리 4

```java
public void method1() throws Exception {
    method2();
    ...
}
public void method2() throws Exception {
    method3();
    ...
}
public void method3() throws Exception {
    ...
}
```

위와 같이 API 등에서 발생하는 예외를 일일이 catch하기 귀찮거나 매번 정확하게 예외 이름을 적어 선언하기 귀찮은 경우 모든 예외를 던져버리는 코드이다.<br>
이렇게 예외 처리를 할 경우 결과적으로 적절한 처리를 통해 복구될 수 있는 예외 상황도 수정할 수 있는 기회를 박탈당하게 된다.
## 예외의 종류와 특징

> 체크 예외 : 명시적인 처리가 필요한 예외

자바에서 throw를 통해 발생시킬 수 있는 예외는 크게 3가지가 있다.
### Error
java.lang.Error 클래스의 서브클래스들이다. 에러는 시스템에 비정상적인 상황이 발생했을 경우에 사용된다. 그래서 주로 자바 VM에서 발생시키는 것이고 애플리케이션 코드에서 잡으려고 하면 안된다. OutOfMemoryError나 ThreadDeath 같은 에러는 catch 블록으로 잡아봤자 아무런 대응 방법이 없기 때문이다.<br>
따라서 시스템 레벨에서 특별한 작업을 하는 것이 아니면 애플리케이션에서는 이런 에러에 대한 처리는 신경 쓸 필요가 없다.
### Exception과 체크 예외
java.lang.Exception 클래스와 그 서브클래스로 정의되는 예외들은 에러와 달리 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용된다.<br>
Exeption 클래스는 체크 예외와 언체크 예외로 구분된다.

* 체크 예외
    * Exception 클래스의 서브클래스이면서 RuntimeException 클래스를 상속하지 않는다.
* 언체크 예외
    * RuntimeException 클래스를 상속받는다.
    * RuntimeException은 Exception의 서브클래스이므로 Exception의 일종이긴 하지만 자바는 RuntimeException과 그 서브클래스는 특별하게 다룬다.
![Exception의 두 가지 종류](../../assets/img/exception.jpeg)
일반적으로 예외라고 하면 Exception 클래스의 서브클래스 중 RuntimeException 을 상속하지 않은 것만을 말하는 체크 예외라고 생각하면 된다. `체크 예외가 발생할 수 있는 메소드를 사용할 경우 반드시 예외를 처리하는 코드를 함께 작성`해야 한다. 그렇지 않으면 컴파일 에러가 발생한다.    
### RuntimeException과 언체크/런타임 예외
java.lang.RuntimeException 클래스를 상속한 예외들은 명시적인 예외처리를 강제하지 않기 때문에 언체크 예외라고 불린다. 또는 대표 클래스 이름을 따서 런타임 예외라고도 한다.<br>
에러와 마찬가지로 런타임 예외는 catch 문으로 잡거나 throws로 선언하지 않아도 된다.(명시적으로 throws로 선언해줘도 상관없다.)<br>
대표적으로 `NullPointerException`이나 `IllegalArgumentException` 등이 있다. 이런 예외는 코드에서 미리 조건을 주어 피할 수 있다. 피할 수 있지만 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것이 런타임 예외다.<br>
따라서 런타임 예외는 예상하지 못했던 예외상황에서 발생하는 것이 아니기 때문에 굳이 catch나 throwsㄹ르 사용하지 않아도 되도록 만든 것이다.<br>
그러나, 체크 예외가 예외처리를 강제하는 것 때문에 예외 블랙홀이나 무책임한 throws 같은 코드가 남발되면서 최근에 자바 표준 스펙의 API들은 예상 가능한 예외상항을 다루는 예외는 체크 예외로 만들지 않는 경향이 있기도 하다.  
## 예외처리 방법
### 예외 복구
### 예외처리 회피
### 예외 전환
## 예외처리 전략
### 런타임 예외의 보편화
### add() 메소드의 예외처리
### 애플리케이션 예외
## SQLException은 어떻게 됐나?
# 예외 전환
## JDBC의 한계
### 비표준 SQL
### 호환성 없는 SQLException의 DB 에러정보
## DB 에러 코드 매핑을 통한 전환
## DAO 인터페이스와 DataAccessException 계층구조
### DAO 인터페이스와 구현의 분리
### 데이터 액세스 예외 추상화와 DataAccessException 계층구조
## 기술에 독립적인 UserDao 만들기
### 테스트 보완
### DataAccessException 활용 시 주의사항
# 정리
