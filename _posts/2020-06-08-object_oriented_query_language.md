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
    