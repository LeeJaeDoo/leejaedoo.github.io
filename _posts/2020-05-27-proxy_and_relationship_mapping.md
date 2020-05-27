---
date: 2020-05-27 23:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 8. 프록시와 연관관계 관리
description: 8. 프록시와 연관관계 관리
image: https://leejaedoo.github.io/assets/img/jpa.png
optimized_image: https://leejaedoo.github.io/assets/img/jpa.png
category: jpa
tags:
  - jpa
  - orm
  - 책 정리
paginate: true
comments: true
---
* 프록시와 즉시/지연로딩
객체가 데이터베이스에 저장되면서 객체 그래프로 연관된 객체를 마음껏 탐색이 어려워졌다. 이 때 프록시를 사용함으로써 연관된 객체를 처음부터 데이터베이스에서 조회하는 것이 아닌, 실제 사용하는 시점에 데이터베이스에서 조회할 수 있다. 하지만 자주 함께 사용하는 객체들은 조인을 사용해서 함께 조회하는 것이 효과적이다.<br>
> JPA는 즉시/지연 로딩을 지원한다.

* 영속성 전이와 고아 객체
JPA는 연관된 객체를 함께 저장하거나 함께 삭제할 수 있는 영속성 전이와 고아 객체 제거라는 편리한 기능을 제공한다.

## 프록시
엔티티를 조회할 때 연관된 엔티티들은 비즈니스 로직에 따라 함께 사용되거나 사용되지 않을 수도 있다.

```java
@Entity
public class Member {
    
    private String username;

    @ManyToOne
    private Team team;

    public Team getTeam() {
        return team;
    }

    public String getUsername() {
        return username;
    }

    ...

}

@Entity
public class Team {
    
    private String name;

    public String getName() {
        return name;
    }

    ...
    
}
```
```java
public void printUserAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름: " + member.getUsername());
    System.out.println("소속팀: " + team.getName());
}

public String printUser(String memberId) {
    Member member = em.find(Member.class, memberId);
    System.out.println("회원 이름: " + member.getUsername());
}
```
여기서 printUser() 메소드는 회원 엔티티만 사용하므로 em.find()로 회원 엔티티를 조회할 때 연관된 팀 엔티티까지 데이터베이스에서 함께 조회되면 비효율적이게 된다.<br>
JPA는 이러한 문제를 `지연 로딩`이라는 기술을 활용하여 해결한다.
> 지연 로딩이란, 위 team.getName() 처럼 실제 팀 엔티티의 값을 사용하게 될 때 데이터베이스에서 조회하도록 하는 기술을 말한다.

그런데, 지연 로딩 기능을 사용하려면 `실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체`, 즉 `프록시 객체`가 사용 된다.

> 지연 로딩을 지원하기 위한 방법으로 `프록시를 사용`하는 방법과 `바이트 코드를 수정`하는 방법, 두 가지 방법이 있다.

### 프록시 기초
`EntityManager.find()`를 사용하면 영속성 컨텍스트에 엔티티가 없을 때, 데이터베이스를 직접 조회하게 된다.<br>
`EntityManager.getReference()`를 사용하면 엔티티가 실제 사용되는 시점에 데이터베이스를 조회해오게 된다. 이 메소드를 호출하게 되면 JPA는 데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않는다. 대신 **데이터베이스 접근을 위임한 프록시 객체를 반환**한다.

* 프록시의 특징
프록시 클래스는 `실제 클래스를 상속 받아서 만들어지므로` 실제 클래스와 겉모양이 같다. 따라서 사용자 입장에서는 실제 객체와 프록시 객체 구분없이 사용하면 된다.<br>
프록시 객체는 실제 객체에 대한 참조(target)을 보관한다. 그리고 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
![프록시 위임](../assets/img/proxy.jpg)

* 프록시 객체의 초기화
프록시 객체의 초기화란, member.getName() 처럼 실제 사용될 때 데이터베이스를 조회해서 `실제 엔티티 객체를 생성하는 행위`를 말한다.
```java
//MemberProxy 반환
Member member = em.getReference(Member.class, "id1");
member.getName();   //  1. getName();

class MemberProxy extends Member {
    
    Member target = null;   //  실제 엔티티 참조

    public String getName() {
        if (target == null) {
            //  2. 초기화 요청
            //  3. DB 조회
            //  4. 실제 엔티티 생성 및 참조 보관
            
            this.target = ...;
        }

        //  5. target.getName();
        return target.getName();
    }

}
```
![프록시 초기화](../assets/img/proxy_init.jpg)
1. member.getName() 호출을 통해 실제 데이터를 조회
2. 실제 엔티티가 생성되어있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청하게 되는데 이를 `프록시 초기화`라 한다.
3. 영속성 컨텍스트는 데이터베이스를 조회하여 실제 엔티티 객체를 생성한다.
4. 프록시 객체는 생성된 실제 엔티티 객체의 참조를 Member target 멤버변수에 보관한다.
5. 프록시 객체는 실제 엔티티 객체의 getName() 을 호출해서 결과를 반환한다.

* 프록시의 특징
1. 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
2. 프록시 객체를 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다. `프록시 객체가 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근할 수 있다.`
3. 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시, 주의해서 사용해야 한다.
4. 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스 조회가 필요 없으므로 em.getReference()를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.
5. 초기화는 영속성 컨텍스트의 도움을 받아야 가능하다. 따라서 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태의 프록시를 초기화하면 문제가 발생한다, 하이버네이트는 org.hibernate.LazyInitializationException 예외를 발생시킨다.

> 프록시 객체 타입 체크 시 주의해야 하는 이유는?

* 준영속 상태와 초기화
```java
//  MemberProxy 반환
Member member = em.getReference(Member.class, "id1");
transaction.commit();
em.close();         //  영속성 컨텍스트 종료

member.getName();   //  준영속 상태 초기화 시도, 
                    //  org.hibernate.LazyInitializationException 예외 발생
```
em.close() 메소드로 영속성 컨텍스트가 종료됬으므로, member는 준영속 상태이다.

### 프록시와 식별자
엔티티를 프록시로 조회할 때 식별자(PK) 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관한다.
```java
Team team = em.getReference(Team.class, "team1");   //  식별자 보관
team.getId();   //  초기화되지 않음.
```
프록시 객체는 식별자 값을 가지고 있으므로 식별자 값을 조회하는 team.getId()를 호출해도 프록시를 초기화하지 않는다.<br>
단, 엔티티 접근 방식을 프로퍼티(@Access(AccessType.PROPERTY))로 설정한 경우에만 초기화하지 않는다.<br>
엔티티 접근 방식을 필드(@Access(AccessType.FIELD))로 설정하면 JPA는 getId()메소드가 id만 조회하는 메소드인지, 다른 필드까지 활용해서 어떤 일을 하는 메소드인지 알지 못하므로 프록시 객체를 초기화 한다.
> 단, `엔티티 접근 방식을 필드`로 설정했을 때 `연관관계를 설정할 때는 프록시를 초기화하지 않는다.`

### 프록시 확인
JPA가 제공하는 `PersistenceUnitUtil.isLoaded(Object entity)` 메소드로 프록시 객체의 초기화 여부를 확인할 수 있다. 이미 초기화 된 프록시 객체는 true, 아직 초기화되지 않았다면 false를 반환한다.
```java
boolean isLoaded = em.getEntityManagerFactory()
                     .getPersistenceUnitUtil().isLoaded(entity);
//  또는 boolean isLoad = emf.getPersistenceUnitUtil.isLoaded(entity);

System.out.println("isLoad = " + isLoad);   //  초기화 여부 확인
```
또 다른 방법으로 클래스명을 직접 출력해봄으로써 조회한 엔티티가 실제 엔티티인지, 프록시로 조회한 것인지 확인할 수 있다.
```java
System.out.println("memberProxy = " + member.getClass().getName());
//  결과 : memberProxy = jpabook.domain.Member_$$_javassist_0
```

* 프록시 강제 초기화
하이버네이트의 initialize() 메소드를 사용하여 프록시를 강제로 초기화할 수 있다.
```java
org.hibernate.Hibernate.initialize(order.getMember());  //  프록시 초기화
```
JPA의 표준에는 프록시 강제 초기화 메소드가 없기 때문에 member.getName()처럼 프록시의 메소드를 직접 호출함으로써 강제로 초기화할 수 있다.<br>
JPA의 표준은 단지 초기화 여부만 확인 할 수 있다.

## 즉시 로딩과 지연 로딩
주로 연관된 엔티티를 지연 로딩할 때 프록시 객체가 사용된다.<br>
JPA는 개발자가 연관된 엔티티의 조회 시점을 선택할 수 있도록 두 가지 방법을 제공한다.
* 즉시 로딩 : 엔티티를 조회할 때 연관된 엔티티도 함께 조회
ex. em.find(Member.class, "member1")를 호출할 때 회원 엔티티와 연관된 팀 엔티티도 함께 조회
> 설정 방법 : @ManyToOne(fetch = FetchType.EAGER)

* 지연 로딩 : 연관된 엔티티를 실제 사용할 때 조회
ex. member.getTeam().getName() 처럼 조회한 팀 엔티티를 실제 사용하는 시점에 JPA가 SQL을 호출해서 팀 엔티티를 조회
> 설정 방법 : @ManyToOne(fetch = FetchType.LAZY)

### 즉시 로딩
```java
@Entity
public class Member {
    //  ...
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //  ...
    
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam();   //  객체 그래프 탐색
```
em.find(Member.class, "member1") 로 회원을 조회하는 순간 팀도 함께 조회한다. 이 때 회원, 팀 두 테이블을 모두 조회해야 하므로 쿼리를 2번 실행할 것 같지만, 대부분의 JPA 구현체는 `즉시 로딩을 최적화하기 위해 가능하면 조인 쿼리를 사용`한다.
```sql
SELECT
    M.MEMBER_ID AS MEMBER_ID,
    M.TEAM_ID AS TEAM_ID,
    M.USERNAME AS USERNAME,
    T.TEAM_ID AS TEAM_ID,
    T.NAME AS NAME
FROM
    MEMBER M LEFT OUTER JOIN TEAM T
        ON M.TEAM_ID = T.TEAM_ID
WHERE
    M.MEMBER_ID = 'member1'
```
sql상으로 회원과 팀을 조인하여 쿼리 한 번으로 조회한다. 이후 member.getTeam() 을 호출하면 이미 로딩된 팀1 엔티티를 반환한다.
* NULL 제약 조건과 JPA 조인 전략
즉시 로딩 실행 시, 외부 조인(LEFT OUTER JOIN)이 사용되고 있다. 회원 테이블에 TEAM_ID 외래 키는 NULL 값을 허용하고 있다. 따라서 팀에 소속되지 않은 회원이 있을 가능성이 있다. 팀에 소속하지 않은 회원과 팀을 내부 조인하면 팀은 물론이고 회원 데이터도 조회할 수 없다.<br>
JPA는 이런 상황을 고려하여 외부 조인을 사용한다. 하지만 성능과 최적화 면에서 내부 조인이 유리하다. 내부 조인을 사용하려면 선행 조건으로 `외래 키에 NOT NULL 조건을 설정`함으로써 내부 조인만 사용해도 문제가 없다.<br>
JPA에게는 `@JoinColumn(nullable = false)`를 설정하여 NOT NULL 조건을 설정했음을 먼저 알려주게 되면, JPA는 외부 조인 대신 내부 조인을 사용하게 된다.
```java
@Entity
public class Member {
    //  ...
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID", nullable = false)
    private Team team;
    //  ...
}
```


> nullable 설정에 따른 조인 전략
> * @JoinColumn(nullable = true) : NULL 허용(기본 값). 외부 조인 사용
> * @JoinColumn(nullable = false) : NULL 허용하지 않음. 내부 조인 사용<br>
>또는 `@ManyToOne.optional = false`로 설정해도 내부 조인을 사용할 수 있다.
>```java
>@Entity
>public class Member {
>   //  ...
>   @ManyToOne(fetch = FetchType.EAGER, optional = false)
>   @JoinColumn(name = "TEAM_ID")
>   private Team team;
>   //  ...
>}
>```

### 지연 로딩
```java
@Entity
public class Member {
    //  ...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //  ...
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam();   //  객체 그래프 탐색(프록시 객체)
team.getName();                 //  팀 객체 실제 사용
```
em.find(Member.class, "member1")를 호출하면 회원만 조회하고 팀은 조회하지 않는다. 대신, `조회한 회원의 team 멤버변수에 프록시 객체를 넣어둔다.`<br>
반환된 팀 객체는 프록시 객체이다. 이 프록시 객체는 실제 사용될 때 까지 데이터 로딩을 미룬다. 그래서 지연 로딩이라 한다.
이처럼 실제 데이터가 필요한 순간이 되어서야 데이터베이스를 조회해서 프록시 객체를 초기화한다.
em.find(Member.class, "member1") 호출 시 아래와 같은 sql이 실행된다.
```sql
SELECT * FROM MEMBER
WHERE MEMBER_ID = 'member1'
```
team.getName() 호출로 프록시 객체가 초기화 되면서 아래와 같은 sql이 실행된다.
```sql
SELECT * FROM TEAM
WHERE TEAM_ID = 'team1'
```

> 그렇다면 지연 로딩은 쿼리가 2번 실행되는것인가?

* 지연 로딩
연관된 엔티티를 프록시로 조회한다. 프록시를 실제 사용할 때 초기화하면서 데이터베이스를 조회한다.
* 즉시 로딩
연관된 엔티티를 즉시 조회한다. 하이버네이트는 가능하면 sql 조인을 통해 한 번 조회한다.