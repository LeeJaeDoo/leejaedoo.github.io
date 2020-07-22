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
따라서 런타임 예외는 예상하지 못했던 예외상황에서 발생하는 것이 아니기 때문에 굳이 catch나 throws 사용하지 않아도 되도록 만든 것이다.<br>
그러나, 체크 예외가 예외처리를 강제하는 것 때문에 예외 블랙홀이나 무책임한 throws 같은 코드가 남발되면서 최근에 자바 표준 스펙의 API들은 예상 가능한 예외상항을 다루는 예외는 체크 예외로 만들지 않는 경향이 있기도 하다.  
## 예외처리 방법
### 예외 복구
예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 방법이다.<br>
예외처리 코드를 강제하는 체크 예외들은 예외를 어떤 식으로든 복구할 가능성이 있는 경우 사용한다.
### 예외처리 회피
예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 방법이다. throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는(rethrow) 방식이다.<br>
즉, 예외를 자신이 처리하지 않고 회피하는 방법이다. 예외처리를 회피하려면 반드시 다른 오브젝트나 메소드가 예외를 대신 처리할 수 있도록 아래처럼 던져줘야 한다.
```java
public void add() throws SQLException {
    //  JDBC API
}

public void add() throws SQLException {
    try {
        // JDBC API
    } catch (SQLException) {
        //  로그 출력
        throw e;
    }
}
```
하지만, 콜백과 템플릿처럼 긴밀하게 역할을 분담하고 있는 관계가 아니라면 자신의 코드에서 발생하는 예외를 그냥 던져버리는 것은 무책임한 회피가 될 수 있다.<br>
예욀르 회피하는 것은 예외를 복구하는 것처럼 의도가 분명해야 한다. 콜백/템플릿처럼 긴밀한 관계에 있는 다른 오브젝트에게 예외처리 책임을 분명히 지게 하거나, 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분명한 확신이 있어야 한다.
### 예외 전환
예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 방법이다.<br>
하지만, 예외 회피와 달리, 발생한 예외를 그대로 넘기는 것이 아닌 적절한 예외로 전환하여 던진다는 차이가 있다.
#### 예외 전환의 2가지 목적

* 내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우에, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서다.

ex) 새로운 사용자를 등록할 때 같은 아이디를 사용하는 사용자가 있어 DB에러가 발생하게 되면 SQLException이 발생되지만, 그대로 밖으로 던지게 된다면 서비스 계층에서는 왜 SQLException이 발생했는지 쉽게 알 수 없기 때문에 DAO에서 SQLException의 정보를 해석하여 DuplicateUserIdException 같은 예외로 바꿔 던져주는 것이 좋다.

```java
public void add(User user) throws DuplicateUserIdException, SQLException {
    try {
        //  JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        //  그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch(SQLException e) {
        //  ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw DuplicateUserIdException();
        else
            throw e;    //  그 외의 경우는 SQLException 그대로
    }
}
```
보통 전환하는 예외에 발생한 예외를 담아서 `중첩 예외(nested exception)`로 만드는 것이 좋다. 중첩 예외는 getCause() 메소드를 이용해서 처음 발생한 예외가 무엇인지 확인할 수 있다. 중첩 예외는 아래와 같이 새로운 예외를 만들면서 생성자나 initCause() 메소드로 근본 원인이 되는 예외를 넣어주면 된다.

* 중첩 예외 1

```java
catch (SQLException e) {
    ...
    throw DuplicateUserIdException(e);
}
```

* 중첩 예외 2

```java
catch (SQLException e) {
    ...
    throw DuplicateUserIdException().initCause(e);
}
```
두 번째 전환 방법은 예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 것이다. 중첩 예외를 이용해 새로운 예외를 만들고 원인이되는 예외를 내부에 담아서 던지는 방식이 같다.<br>
하지만, 의미를 명확하게 하려고 다른 예외로 전환하는 것이 아니다. 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용한다.<br>

일반적으로 체크 예외를 계속 throws를 사용해 넘기는 건 무의미하다. 메소드 선언은 지저분해지고 아무런 장점이 없다. DAO에서 발생한 SQLException이 웹 컨트롤러 메소드까지 명시적으로 전달된다고 해서 아무런 효율이 없다.<br>
어차피 복구가 불가능한 예외라면 가능한 한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws 선언이 들어가지 않도록 해줘야 한다.

## 예외처리 전략
### 런타임 예외의 보편화
catch블록이나 throws를 사용하여 예외처리를 강제하는 것은 예외가 발생할 가능성이 있는 API 메소드를 사용하는 개발자의 실수를 방지하기 위한 배려라고 볼 수도 있지만, 귀찮기도 하다.<br>
차라리 애플리케이션 차원에서 예외상황을 미리 파악하고, 예외가 발생하지 않도록 차단하는 게 좋다. 또는 프로그램의 오류나 외부 환경으로 인해 예외가 발생하는 경우라면 빨리 해당 요청의 작업을 취소하고 서버 관리자나 개발자에게 통보해주는 편이 낫다.<br>
자바의 환경이 서버로 이동하면서 체크 예외의 활용도와 가치는 점점 떨어지고 있기 때문에 대응이 불가능한 체크 예외라면 빨리 런타임 예외로 전환해서 던지는 게 낫다.<br>
최근에는 API가 발생시키는 예외를 체크 예외보다 언체크 예외로 정의하는 것이 일반화되고 있다. 따라서 언체크 예외라도 필요에 따라서 catch 블록으로 처리할 수 있지만 애개 복구 불가능한 상황이고 결국엔 RuntimeException 등으로 포장해서 던져야할테니 아예 API 차원에서 런타임 예외를 던지도록 만드는 것이다. 
### add() 메소드의 예외처리
앞에서 봤던 DuplicateUserIdException은 굳이 체크 예외로 둘 필요는 없다. 잡을 수 있다면 런타임 예외로 만드는 게 낫다.

* 아이디 중복 시 사용하는 예외

```java
public class DuplicateUserIdException extends RuntimeException {
    public DuplicateUserIdException(Throwable cause) {
        super(cause);
    }
}
```

이로써 DuplicateUserIdException 외에 시스템 예외에 해당하는 SQLException은 언체크 예외가 됐다. 따라서 메소드 선언의 throws에 포함시킬 필요가 없다. 반면에 역시 언체크 예외로 만들어지긴 했지만 add() 메소드를 사용하는 쪽에서 아이디 중복 예외를 처리하고 싶은 경우 활용할 수 있음을 알려주도록 DuplicateUserIdException을 메소드의 throws 선언에 포함시킨다.

* 예외 처리 전략을 적용한 add()

```java
public void add() throws DuplicateUserIdException {
    try {
        //  JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        //  그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch (SQLException e) {
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw new DuplicateUserIdException(e);  //  예외 전환
        else
            throw new RuntimeException(e);          //  예외 포장
    }
}
```

이런 식으로 런타임 예외를 일반화하여 사용하면 주의해야할 점으로, 컴파일러가 예외처리를 강제하지 않기 때문에 예외상황을 충분히 고려해야 한다.
### 애플리케이션 예외
런타임 예외 중심의 전략은 낙관적인 예외처리 기법이라고 할 수 있다. 어차피 런타임 예외는 시스템 레벨에서 알아서 처리될 것이고, 꼭 필요한 경우는 런타임 예외라도 잡아서 복구하거나 대응해줄 수 있으니 문제될 것이 없다는 낙관적인 태도를 기반으로 한다.<br>
반면, 시스템 또는 외부의 예외상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외가 있는데 이를 `애플리케이션 예외`라고 한다.

* 애플리케이션 예외를 사용한 코드

```java
try {
    BigDecimal balance = account.withdraw(amount);
    ..
    // 정상적인 처리 결과를 출력하도록 진행
} catch (InsufficientBalanceException e) {  //  체크 예외
    //  InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
    BigDecimal availFunds = e.getAvailFunds();
    ...
    //  잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
}
```
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
