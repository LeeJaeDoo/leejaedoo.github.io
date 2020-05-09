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
`@MappedSuperclass`는 부모 클래스와 자식 클래스 모두 데이터베이스 테이블과 매핑하는 것이 아닌 `부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공할 때` 사용 된다.<br>
추상 클래스와 비슷한 개념으로, @Entity와 달리 실제 테이블과 매핑되지 않는다. 단순히 매핑 정보를 상속할 목적으로만 사용된다.
![mappedsuperclass](../assets/img/mappedsuperclass.jpg)
id, name 두 공통 속성을 부모 클래스로 모으고 객체 상속 관계로 만든다.
```java
@MappedSuperclass
public abstract class BaseEntity {

    @Id @GeneratedValue
    private Long id;
    private String name;
    ...
}

@Entity
public class Member extends BaseEntity {

    // ID 상속
    // NAME 상속
    private String email;
    ...
}

@Entity
public class Seller extends BaseEntity {

    // ID 상속
    // NAME 상속
    private String shopName;
    ...
}
```
* @AttributeOverrides/@AttributeOverride
부모로 부터 물려받은 매핑 정보를 재정의 한다.
```java
@Entity
@AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity { ... }
```
상속 받은 id 속성의 컬럼명을 MEMBER_ID로 재정의했다. 둘 이상을 재정의 할 때는 @AttributeOverrides를 사용한다.
```java
@Entity
@AttributeOverrides({
    @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
    @AttributeOverride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
public class Member extends BaseEntity { ... }
```
### 특징
* 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용한다.
* @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로 em.find()나 JPQL에서 사용할 수 없다.
* 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 킅래스로 만드는 것을 권장한다.
> 엔티티(@Entity)는 엔티티(@Entity)이거나 @MappedSuperclass로 지정한 클래스만 상속받을 수 있다.

## 복합 키와 식별 관계 매핑
### 식별 관계 vs 비식별 관계
데이터베이스 테이블 사이에 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.
#### 식별 관계
부모 테이블의 기본 키를 내려받아서 `자식 테이블의 기본 키 + 외래 키`로 사용하는 관계다.
![식별관계](../assets/img/identifying_relationship.jpg)

#### 비식별 관계
부모 테이블의 기본 키를 받아서 `자식 테이블의 외래 키`로만 사용하는 관계다.
![비식별관계](../assets/img/non_identifying_relationship.jpg)
비식별 관계는 외래 키에 NULL을 허용하는지에 따라 필수적/선택적 비식별 관계로 나뉜다.
* 필수적 비식별 관계(Mandatory)<br>
외래 키에 NULL을 허용하지 않는다. 연관관계를 필수적으로 맺어야 한다.
* 선택적 비식별 관계(Optional)<br>
외래 키에 NULL을 허용한다. 연관관계를 맺을지 말지 선택할 수 있다.

### 복합 키를 활용한 비식별 관계 매핑
JPA에서 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다.<br>
JPA는 영속성 컨텍스트에 엔티티를 보관할 때 엔티티의 식별자를 키로 사용한다. 그리고 식별자를 구분하기 위해 equals와 hashCode를 사용하여 동등성 비교를 한다. 그런데 식별자 필드가 하나일 때는 보통 자바의 기본 타입을 사용하므로 문제가 없지만, 식별자 필드가 2개 이상이면 별도의 식별자 클래스를 만ㄷ르고 그 곳에 equals와 hashCode를 구현해야 한다.<br>
JPA는 복합 키를 지원하기 위해 `@IdClass`와 `@EmbeddedId` 2가지 방법을 제공한다.
#### @IdClass
관계형 데이터베이스에 더 가까운 방법으로 아래와 같이 사용된다.
![복합키테이블](../assets/img/복합키.JPG)
```java
@Entity
@IdClass(ParentId.class)
public class Parent {
    @Id
    @Column(name = "PARENT_ID1")
    private String id1; //  ParentId.id1과 연결
    @Id
    @Column(name = "PARENT_ID2")
    private String id2; //  ParentId.id2과 연결

    private String name;
    ...
}
```
```java
public class ParentId implements Serializable {

    private String id1; //  Parent.id1 매핑
    private String id2; //  Parent.id2 매핑

    public ParentId() {}
    
    public ParentId(String id1, String id2) {
        this.id1 = id1;
        this.id2 = id2;
    }

    @Override
    public boolean equals(Object o) {...}

    @Override
    public int hashCode() {...}
}
```
@IdClass를 사용하기 위한 식별자 클래스는 다음 조건을 만족해야 한다.
* 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.(ex. Parent.id1과 ParentId.id1)
* Serializable 인터페이스를 구현해야 한다.
* equals, hashCode를 구현해야 한다.
* 식별자 클래스는 public 이어야 한다.

```java
Parent parent = new Parent();
parent.setId1("myId1"); //  식별자
parent.setId2("myId2"); //  식별자
parent.setName("parentName");
em.persist(parent);
```
복합 키를 사용한 엔티티를 저장하는 코드로, ParentId 식별자 클래스가 보이지 않는데 이는 em.persist()를 호출하면 영속성 컨텍스트에 엔티티를 등록하기 직전에 내부에서 Parent.id1, Parent.id2 값을 사용해서 식별자 클래스인 ParentId를 생성하고 영속성 컨텍스트의 키로 사용한다.

```java
ParentId parentId = new ParentId("myId1", "myId2");
Parent parent = em.find(Parent.class, parentId);
```
복합 키로 parent 엔티티를 조회해오는 코드다.

```java
@Entity
public class Child {
    
    @Id
    private String id;

    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID1",
            referencedColumnName = "PARENT_ID1"),
        @JoinColumn(name = "PARENT_ID2",
            referencedColumnName = "PARENT_ID2"),
    })
    private Parent parent;
}
```
부모 테이블의 기본 키 컬럼이 복합 키이므로 자식 테이블의 외래 키도 복합 키다. 따라서 외래 키 매핑 시, 여러 컬럼을 매핑해야 하므로 @JoinColumns 어노테이션을 사용하고 각각의 외래 키 컬럼을 @JoinColumn으로 매핑한다.
> 예제와 같이 @JoinColumn의 name 속성과 referencedColumnName 속성의 값이 같으면 referencedColumnName은 생략해도 된다.

## 참조
https://cyr9210.github.io/2019/11/18/JPA/ORM-JPA07/ 