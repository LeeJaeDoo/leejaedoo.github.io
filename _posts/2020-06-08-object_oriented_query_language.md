---
date: 2020-06-08 22:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 10. 객체지향 쿼리 언어
description: 10. 객체지향 쿼리 언어
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
JPQL은 가장 중요한 객체지향 쿼리 언어이다.

## 객체지향 쿼리 소개
EntityManager.find() 메소드를 사용하면 식별자로 엔티티를 조회할 수 있고, 그래프 탐색을 통해 연관된 엔티티까지 조회할 수 있다. 하지만 좀 더 복잡한 검색 방법이 필요하게 되면 SQL로 필요한 내용을 최대한 걸러내어 사용해야 한다.<br>
결과적으로, ORM을 사용하게 되면 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발하게 되므로 검색도 테이블이 아닌 엔티티 객체를 대상으로 할 수 있어야 한다. 이 때 JPQL을 사용할 수 있다.
* JPQL의 특징
    * 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리다.
    * SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.

JPQL을 사용하게 되면 JPA는 JPQL을 분석하여 적절한 SQL로 만들어 데이터베이스를 조회한다. 그리고 조회한 결과로 엔티티 객체를 생성해서 반환한다.

> JPQL은 한마디로 객체지향 SQL이다.

JPA는 JPQL뿐만 아니라 다양한 검색 방법을 제공한다.
* JPA가 공식 지원하는 기능
    * JPQL(Java Persistence Query Language)
    * Criteria Query : JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
    * Native SQL : JPA에서 JPQL 대신 직접 SQL을 사용할 수 있다.

* JPA가 공식 지원하진 않지만 알아둬야 할 기능
    * QueryDSL : Criteria Query처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음. 비표준 오픈소스 프레임워크.
    * JDBC, MyBatis

> Criteria, QueryDSL은 JPQL을 편하게 작성하도록 도와주는 빌더 클래스일 뿐이다. 따라서 JPQL을 이해해야한다.

### JPQL 소개
데이터베이스 방언만 변경하면 JPQL을 수정하지 않아도 자연스럽게 데이터베이스를 변경할 수 있다. 예로, 같은 SQL함수라도 데이터베이스 마다 사용 문법이 다른 데 JPQL을 사용하면 제공하는 표준화된 함수를 사용하면 선택한 방언에 따라 해당 데이터베이스에 맞춘 적절한 SQL 함수가 실행된다.

> JPQL은 SQL 보다 간결하다. 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL보다 코드가 간결하다.

```java
@Entity(name="Member")
public class Member {
    
    @Column(name = "name")
    private String username;
    //...
}

// 쿼리 생성
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```
em.createQuery() 메소드를 통해 실행할 JPQL과 반환할 엔티티의 클래스 타입인 Member.class 를 넘겨주고 getResultList() 메소드를 통해 JPA는 JPQL을 SQL로 변환하여 데이터베이스를 조회한다.

* 실행된 JPQL

```sql
select m
from Member as m
where m.username = 'kim'
```

* 실제 실행된 SQL

```sql
SELECT
    member.id AS id,
    member.age AS age,
    member.team_id AS team,
    member.name AS name
FROM
    Member member
WHERE
    member.name = 'kim'
```
### Criteria 쿼리 소개

> JPQL을 생성하는 빌더 클래스

* 장점
문자가 아닌 query.select(m).where(...) 처럼 프로그래밍 코드로 JPQL을 작성할 수 있다.<br>
코드로 JPQL을 작성하기 때문에 런타임이 아닌 `컴파일 시점에 오류를 발견`할 수 있다.
    * 컴파일 시점에 오류를 발견할 수 있다.
    * IDE를 사용하면 코드 자동완성을 지원한다.
    * 동적 쿼리를 작성하기 편하다.

```java
// JPQL
select m from Member as m where m.username = 'kim'

// Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// 루트 클래스(조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

// 쿼리 생성
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
```
위와 같이 Criteria를 활용하여 쿼리를 문자가 아닌 코드로 작성할 수 있다.<br>
아쉬운 점은 m.get("username") 처럼 필드 명을 코드가 아닌 문자로 작성한 것인데 `메타 모델(MetaModel)`을 활용하면 해결할 수 있다.<br>

```java
//  메타 모델 사용 전 -> 사용 후
m.get("username") -> m.get(Member_.username)
```
"username"대신 Member_.username이라는 코드로 활용할 수 있다. 하지만 복잡하고 장황해진다. 가독성도 떨어진다.

### QueryDSL 소개

> Criteria와 마찬가지로 JPQL 빌더 역할을 한다.

* 장점
코드 기반이면서 단순하고 사용하기 쉽다. 작성한 코드도 JPQL과 비슷해서 한눈에 들어온다.

> QueryDSL은 JPA 표준은 아니고 오픈소스 프로젝트이다.

```java
// 준비
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

// 쿼리, 결과조회
List<Member> members = query.from(member).where(member.username.eq("kim")).list(member);
```

queryDSL도 어노테이션 프로세서를 사용하여 쿼리 전용 클래스(Qclass)를 만들어야 한다.

### 네이티브 SQL 소개
JPA에서 SQL을 직접 사용할 수 있는 기능을 제공하는데 이를 네이티브 SQL이라고 한다.<br>
JPQL을 사용해도 가끔 특정 데이터베이스에 의존하는 기능을 사용해야 할 때가 있는데, (ex. Oracle CONNECT BY 기능이나 특정 데이터베이스에서만 동작하는 SQL 힌트) 이런 기능들은 JPQL로 구현할 수 없다.<br>
이처럼 SQL만 지원되고 JPQL에서는 지원되지 않는 기능을 사용해야 할 때 네이티브 SQL을 사용하면 된다.

> 단점 : 특정 데이터베이스에 의존하는 SQL을 작성해야 한다. 따라서 SQL 의존성이 높은 코딩을 할 수 밖에 없다.

```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```

네이티브 SQL은 em.createNativeQuery() 를 사용하면 된다.

### JDBC 직접 사용, MyBatis 같은 SQL Mapper 프레임워크 사용
JDBC나 MyBatis를 JPA와 함께 사용하면 `영속성 컨텍스트를 적절한 시점에 강제로 flush`해야 한다. 둘 모두 JPA를 우회해서 데이터베이스에 접근하기 때문에 JDBC나 MyBatis를 활용한 SQL은 JPA에서 인식하지 못하게 된다. 결과적으로 최악의 경우에는 `영속성 컨텍스트와 데이터베이스의 불일치 상태가 발생하여 데이터 무결성이 훼손`될 수 있다.<br>
> 이런 이슈를 해결하기 위해 `JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 flush`하여 `데이터베이스와 영속성 컨텍스트를 동기화`하면 된다.
> 스프링 AOP를 활용하여 JPA를 우회하여 데이터베이스에 접근하는 메소드를 실행할 때 마다 영속성 컨텍스트를 flush하면 위 이슈를 해결할 수 있다.
 