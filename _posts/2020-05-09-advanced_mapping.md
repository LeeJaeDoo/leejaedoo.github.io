---
date: 2020-05-09 15:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 7. 고급 매핑
description: 7. 고급 매핑
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
## 상속 관계 매핑
ORM에서 이야기하는 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 슈퍼/서브 타입 관계를 매핑하는 것이다.<br>
슈퍼/서브 타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법을 선택할 수 있다.
* 각각의 테이블로 변환 - 조인 전략
* 통합 테이블로 변환 - 단일 테이블 전략
* 서브타입 테이블로 변환 - 구현 클래스마다 테이블 전략

### 조인 전략(Joined Strategy)
엔티티를 각각 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략이다.
![조인테이블전략](../assets/img/jointable.jpg)
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
        
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;    //  이름 
    private int price;      //  가격

    ...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
    
    private String artist;
    
    ...
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
    
    private String director;    //  감독  
    private String actor;       //  배우
}

@Entity
@DiscriminatorValue("B")
@PrimaryKeyJoinColumn(name = "BOOK_ID") //  ID 재정의
public class Book extends Item {

    private String author;  //  작가
    private String isbn;    //  ISBN
    ...
}
```
<table>
  <thead>
    <tr>
      <th>어노테이션</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>@Inheritance(strategy = InheritanceType.JOINED)</td>
      <td>상속 매핑은 부모 클래스에 @Inheritance를 사용해야 한다. 그리고 strategy로 매핑 전략을 지정한다.</td>
      <td>SINGLE_TABLE</td>
    </tr>
    <tr>
      <td>@DiscriminatorColumns(name = "DTYPE")</td>
      <td>부모 클래스에 구분 컬럼명(DTYPE)을 지정한다. @DiscriminatorColumns로 줄여도 된다.</td>
      <td>DTYPE</td>
    </tr>
    <tr>
      <td>@DiscriminatorValue("M")</td>
      <td>엔티티 저장할 때 구분 컬럼에 입력할 값을 지정한다. 영화 엔티티 저장 시 구분 컬럼인 DTYPE값으로 M이 저장된다.</td>
      <td>엔티티 이름</td>
    </tr>
    <tr>
      <td>@PrimaryKeyJoinColumn(name = "BOOK_ID")</td>
      <td>기본 값으로 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용하는데, 자식 테이블의 기본 키 컬럼명을 변경하고 사용한다.</td>
      <td></td>
    </tr>
  </tbody>
</table>

### 특징
JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이크를 포함한 몇 구현체는 구분 컬럼(@DiscriminatorColumn)없이도 동작한다.
#### 장점
* 테이블이 정규화된다.
* 외래 키 참조 무결성 제약조건을 활용할 수 있다.
* 저장공간을 효율적으로 사용한다.

#### 단점
* 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
* 조회 쿼리가 복잡하다.
* 데이터를 등록할 INSERT SQL을 두 번 실행한다.

### 단일 테이블 전략
테이블을 하나만 사용하고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분한다. 조회할 때 조인을 사용하지 않으므로 일반적으로 가장 빠르다.<br>
주의할 점은 `자식 엔티티가 매핑할 컬럼은 모두 null을 허용해야 한다.` 예를 들어 Book 엔티티를 저장하면 ITEM 테이블의 AUTHOR, ISBN 컬럼만 사용하고 다른 엔티티와 매핑된 ARTIST, DIRECTOR, ACTOR 컬럼은 사용하지 않으므로 null이 입력되기 떄문이다. 
![싱글테이블전략](../assets/img/singletable.jpg)
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)   //  default가 SINGLE_TABLE이므로 strategy 생략 가능
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;    //  이름 
    private int price;      //  가격

    ...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {...}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {...}

@Entity
@DiscriminatorValue("B")
public class Book extends Item {...}

```

### 특징
구분 컬럼을 꼭 사용해야 한다. 따라서 @DiscriminatorColumn을 꼭 설정해야 한다.<br>
@DiscriminatorValue를 지정하지 않으면 기본으로 엔티티 이름을 사용한다.(ex. Movie, Album, Book)
#### 장점
* 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
* 조회 쿼리가 단순하다.
#### 단점
* 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
* 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 그러므로 상황에 따라서는 조회 성능이 오히려 느려질 수 있다.

### 구현 클래스마다 테이블 전략
자식 엔티티마다 테이블을 만들고 각각의 자식 테이블마다 필요한 컬럼이 모두 있다. 일반적으로 추천되지 않는 전략이다.
![구현클래스마다테이블전략](../assets/img/tablefperconcreteclass.jpg)
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;    //  이름 
    private int price;      //  가격
    ...
}

@Entity
public class Album extends Item {...}

@Entity
public class Movie extends Item {...}

@Entity
public class Book extends Item {...}

```

### 특징
구분 컬럼이 사용되지 않는다. DBA나 ORM 전문가 모두 추천하지 않는 전략으로 조인이나 단일 테이블 전략을 고려하자.
#### 장점
* 서브 타입을 구분해서 처리할 때 효과적이다.
* not null 제약조건을 사용할 수 있다.
#### 단점
* 여러 자식 테이블을 함께 조회할 때 성능이 느리다.(SQL에 UNION을 사용해야 한다.)
* 자식 테이블을 통합해서 쿼리하기 어렵다.

## @MappedSuperclass
