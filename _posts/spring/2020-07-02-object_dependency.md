---
date: 2020-07-02 18:40:40
layout: post
title: 토비의 스프링
subtitle: 1. 오브젝트와 의존관계
description: 1. 오브젝트와 의존관계
image: https://leejaedoo.github.io/assets/img/spring.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/spring.jpeg
category: spring
tags:
  - spring
  - 책 정리
paginate: true
comments: true
---
> DAO란 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트

# 초난감 DAO의 분리
## 관심사의 분리
관심사의 분리란 객체지향적으로 이야기해보면 관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것을 말한다.<ㅠr>
쉽게 말해 관심이 같은 것끼리 모으고 관심이 다른 것은 따로 떨어져 있게 하는 것이다. 관심사가 같은 것끼리 모으고 다른 것은 분리해줌으로써 같은 관심에 효과적으로 집중할 수 있게 된다.
## 중복 메소드의 추출. 리팩토링
책에서 예로 든 DB 커넥션을 가져오는 코드가 여럿 중복되어 있다는 것은 앞에서 말한 DB 연결이라는 관심사가 여기저기 흩어있음을 의미한다. 이는 곳 유지보수 및 변경사항이 발생될 때 큰 어려움을 야기시킨다.<br>
### 중복 코드의 메소드 추출
중복된 코드를 추출함으로써 관심의 종류에 따라 코드를 구분하도록 했기 때문에 한 가지 관심에 대한 변경이 일어날 경우, 그 관심에 집중되는 부분의 코드만 수정하면 된다.<br>
즉, 관심 내용이 독립적으로 존재하므로 수정도 간단해진다.
### 변경사항의 검증: 리팩토링과 테스트
중복된 코드를 추출하도록 코드를 수정했으므로 기능에 이상이 없는지 테스트가 필요하다. 작업의 기능에는 변화가 없으면서 구조만 변경하는 작업을 `리팩토링`이라 한다.
## 커넥션 독립: 상속을 통한 확장
같은 DAO 객체를 사용자 입맛에 맞게 유연하게 적용하기 위한 방법 중 하나로 `상속을 통한 확장`이 있다.

```java
public abstract class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }

    public void get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }    

    //  구현 코드는 제거되고 추상 메소드로 바뀜으로써 구현은 서브클래스가 맡는다.
    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```

![상속](../../assets/img/상속.jpeg)
사진처럼 UserDao 클래스를 추상클래스로 만들고, 각 사용자들은 자신의 입맛에 맞게 UserDao 클래스를 상속받아 서브클래스 형태로 원하는 형식으로 구현할 수 있다.<br>
이제 UserDao 클래스는 단순히 변경이 용이한 수준을 넘어 확장이 가능해진다.<br>
이렇게 슈퍼클래스에 기본적인 로직의 흐름을 만들고 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하는 방법을 `템플릿 메소드 패턴`이라고 부른다.<br>
또한 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 `팩토리 메소드 패턴`이라고 한다.  
![팩토리메소드패턴](../../assets/img/팩토리메소드.jpeg)
```java
public class AUserDao extends UserDao {
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // 구현 코드
    }
}
public class BUserDao extends UserDao {
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // 구현 코드
    }
}
```
# DAO의 확장
하지만 상속을 활용함으로써 생긴 한계점도 있다. 자바는 다중 상속이 불가하기 때문에 이미 UserDao 클래스가 다른 목적으로 상속받고 있다면 활용하기 어렵다.<br>
또한 상속을 통한 상하위클래스의 관계는 생각보다 밀접하기 때문에 상속을 통해 열심히 기능을 분리하고 확장성을 줬음에도 불구하고 서브클래스는 슈퍼클래스의 기능을 직접 사용할 수 있다.<br>
즉, 슈퍼클래스 내부의 변경이 있을 때마다 서브클래스도 함께 수정하거나 다시 개발해야 하는 상황이 생길 수 있다.
## 클래스의 분리
상속 관계가 아닌 완전히 분리된 독립적인 클래스 관계로 만들 수 있다.
![클래스분리](../../assets/img/클래스분리.jpeg)
UserDao 클래스 안에는 new 키워드를 통해 새로 생성한 SimpleConnectionMaker 클래스 객체를 만들어 사용한다.
```java
public class UserDao {
    private SimpleConnectionMaker simpleConnectionMaker;
    
    public UserDao() {
        simpleConnectionMaker = new SimpleConnectionMaker();
    }

    public void add(User user) {...}

    public void get(String id) {...}
}
```
다만, 이렇게 클래스를 분리함으로써 사용자 입맛에 맞게 확장하여 구현할 수가 없게되었다.
## 인터페이스 도입
인터페이스를 도입하여 두 클래스가 기존처럼 긴밀하지는 않으면서 중간에서 추상적이면서 느슨한 연결고리를 갖게함으로써 해결할 수 있다.<br>
추상화란 어떤 것들의 공통적인 성격을 뽑아내어 분리하는 작업을 말한다.
![인터페이스](../../assets/img/인터페이스.jpeg)
인터페이스는 자신을 구현한 클래스에 대한 구체적인 정보는 모두 감춰버린다. 인터페이스는 기능만 정의해놓 것이다. 구현 방법은 나타나있지 않다. 구현 클래스에서 알아서 결정할 일이다.
```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    public UserDao userDao () {
        connectionMaker = new DConnectionMaker();   //  생성한 클래스 명이 노출된다.
    }

    public void add(User user) {...}

    public void get(String id) {...}    
}
```
인터페이스를 통해 구현하는 클래스에 대한 정보는 대부분 사라졌지만 아직까지 남아있는 부분이 있다. 여전히 필요할 때마다 UserDao 클래스 생성자를 수정해야 하는 부분이 얽혀있게 된다. 독립적이지 못하다는 얘기다.
## 관계 설정 책임의 분리
외부에서 만들어 준 객체를 가져오는 방법이 있다.
![불필요한의존관계](../../assets/img/불필요한의존관계.jpeg)
결과적으로 UserDao 객체가 동작하려면 특정 클래스와 관계를 맺어야 한다. 하지만 클래스 사이에 관계가 아닌 객체 사이에 동적인 관계가 만들어지게 된다.
```java
public UserDao(ConnectionMaker connectionMaker) {
    this.connectionMaker = connectionMaker; 
}
```
UserDao의 클라이언트로 UserDao와 ConnectionMaker 객체 간의 관계를 맺는 책임을 넘겨줌으로써 즉, UserDao 객체의 클라이언트로 관계설정의 책임이 넘겨졌다.
```java
public class UserDaoTest {
    public static void main(String [] args) {
        ConnectionMaker connectionMaker = new DConnectionMaker();   //  클라이언트에서 UserDao가 사용할 ConnectionMaker 구현 클래스를 결정하고 객체를 생성한다.
        UserDao userDao = new UserDao(connectionMaker); 
    
    }
}
```
![관계설정책임](../../assets/img/관계설정책임.jpeg)
상속을 통한 확장 방법보다 깔끔하고 유연한 방법으로 UserDao와 ConnectionMaker 클래스들을 분리하고 서로 영향을 주지 않으면서 필요에 따라 자유롭게 확장할 수 있는 구조가 되었다. 
## 원칙과 패턴
### 개방 폐쇄 원칙
객체지향 설계 원칙 중 하나로, `클래스나 모듈은 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다.`는 의미다.
> 객체지향 설계 원칙(SOLID)
> 참조 : [http://www.nextree.co.kr/p6960/](http://www.nextree.co.kr/p6960/)
### 높은 응집도와 낮은 결합도
#### 높은 응집도
응집도가 높다는 것은 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 의미다. 또한 변화가 일어날 때 해당 모듈에 변하는 부분이 크다는 의미이기도 하다. 즉, 변경이 일어날 때 모듈에 많은 변화가 일어나면 응집도가 높은 것이다.
#### 낮은 결합도
책임과 관심사가 다른 모듈 또는 객체는 느슨한 연결 관계를 맺어야 한다. 결합도가 낮아지면 변화에 대응하는 속도가 높아지고 구성이 깔끔해진다. 또한 확장하기에도 편리하다.
### 전략 패턴
전략 패턴은 자신의 기능 context에서 필요에 따라 변경이 필요한 알고리즘을 인터페이를 통해 통째로 외부로 분리시키고, 이를 구현한 체계적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있는 디자인 패턴이다.<br>
여기서 UserDaoTest-UserDao-ConnectionMaker 구조로 설명할 수 있다. 컨텍스트(UserDao)를 사용하는 클라이언트(UserDaoTest)는 컨텍스트가 사용할 전략(ConnectionMaker를 구현한 클래스)을 컨텍스트의 생성자를 통해 제공해주는 것이 일반적이다.

# 제어의 역전(IoC)
