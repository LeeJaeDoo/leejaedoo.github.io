---
date: 2020-04-19 15:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 4. 엔티티 매핑
description: 4. 엔티티 매핑
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
JPA를 올바르게 사용하려면 아래와 같은 매핑 어노테이션들을 활용하여 엔티티와 테이블이 정확히 매핑시켜야 한다.
* 객체와 테이블 매핑 : @Entity, @Table 등
* 기본 키 매핑 : @Id
* 필드와 컬럼 매핑 : @Column
* 연관관계 매핑 : @ManyToOne, @JoinColumn

## @Entity
테이블과 매핑할 클래스에는 @Entity 어노테이션을 필수로 붙여야 한다. @Entity가 붙은 클래스는 JPA가 관리하는 것으로, 엔티티라 부른다.

### 속성 - name
```java
    @Entity(name = "member_1")
    public class Member {
        ...
    }
```
JPA에서 사용할 엔티티 이름을 지정한다. 기본값은 클래스 이름이지만 다른 패키지에 동명의 엔티티 클래스가 있을 경우, name 속성으로 별도의 엔티티 이름을 지정하여 충돌을 피할 수 있다.
> 하지만, 이렇게 엔티티 이름이 충돌하는 경우가 발생한다면 설계에서 잘못 됬다고 볼 수 있다.

### @Entity 적용 시, 주의사항
* 기본 생성자는 필수(parameter가 없는 public or protected 생성자) -> 자바에서 자동 생성됨.
* final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
* 저장할 필드에 final을 사용하면 안된다.

## @Table
엔티티와 매핑할 테이블을 지정한다. 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

### 속성
<table>
  <thead>
    <tr>
      <th>속성</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>매핑할 테이블 이름</td>
      <td>엔티티 이름을 사용한다.</td>
    </tr>
    <tr>
      <td>catalog</td>
      <td>catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.</td>
      <td></td>
    </tr>
    <tr>
      <td>schema</td>
      <td>schema 기능이 있는 데이터베이스에서 schema를 매핑한다.</td>
      <td></td>
    </tr>
    <tr>
      <td>uniqueConstraints(DDL)</td>
      <td>DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다.</td>
      <td></td>
     </tr>
  </tbody>
</table>

> @CreatedDate vs @Temporal ?

## DDL 생성 기능
```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME", nullable = false, length = 10)   // 추가
    private String username;
    ...
}
```
@Column 속성
<table>
  <thead>
    <tr>
      <th>속성</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>매핑할 컬럼 이름</td>
      <td>필드명을 사용한다.</td>
    </tr>
    <tr>
      <td>nullable</td>
      <td>false 설정 시 not null 제약 조건을 추가한다.</td>
      <td>true</td>
    </tr>
    <tr>
      <td>length</td>
      <td>문자의 크기를 지정할 수 있다.</td>
      <td>255</td>
    </tr>
  </tbody>
</table>

> 이러한 DDL 생성 기능들은 JPA 실행 로직에는 영향을 주지 않기 때문에 스키마 자동 생성 기능을 사용하지 않는다면 사용할 이유는 없지만 사용하게 되면 개발자가 엔티티만 보고도 다양한 제약 조건을 파악할 수 있는 장점이 있다.

## 기본 키 매핑
### JPA의 기본 키 생성 전략
직접 할당 방식과 자동 생성 방식이 있다. 직접 할당 방식은 `@Id` 선언을 통해 애플리케이션에서 직접 할당한다. 자동 생성 방식은 데이터베이스 벤더마다 지원하는 방식이 다르기 때문에 JPA에서는 자동 생성에 대한 다양한 방식을 제공한다. @Id에 `@GeneratedValue`를 추가하고 설정 옵션에 따라 자동 생성 방식을 셜정할 수 있다.
> 자동 생성 방식을 사용하기 위해 yml에서 `hibernate.use-new-id-generator-mappings` 설정이 필요하다.

> spring boot 1.5 부터는 use-new-id-generator-mappings: false가 default로, 기본 키 생성 전략이 AUTO인 경우 IDENTITY를 따라간다.
> spring boot 2.0 부터는 use-new-id-generator-mappings: true가 default로, 기본 키 생성 전략이 AUTO인 경우 TABLE을 따라간다.

> 참고 : [https://junhyunny.blogspot.com/2019/12/hibernate.html](https://junhyunny.blogspot.com/2019/12/hibernate.html)

#### 직접 할당
기본 키를 애플리케이션에서 직접 할당한다. (ex. 해당 필드에 @Id 선언) 아래와 같은 자바 타입에서만 @Id 선언이 가능하다.
* 자바 기본타입
* 자바 Wrapper 타입
* String, java.util.Date, java.sql.Date, java.math.BigDecimal, java.math.BigInteger

#### 자동 생성
대리 키를 사용하는 방식으로 생성된다.

##### IDENTITY
기본 키 생성을 데이터베이스에 위임한다. 주로 MySQL, PostgreSQL, SQL Server, DB2 에서 사용된다.
```sql
CREATE TABLE BOARD (
    ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    DATA VARCHAR(255)
)
```
```java
@Entity
public class Board {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    ...
}
```

IDENTITY 전략은 데이터를 데이터베이스에 INSERT한 후에 기본 키 값을 조회할 수 있기 때문에 엔티티에 식별자 값을 할당하려면 JPA는 추가로 데이터베이스를 조회해야 한다.
> 영속 상태의 엔티티는 반드시 식별자가 있어야 하기 때문에, IDENTITY 전략을 사용하게 되면 em.persist()가 호출되는 즉시, INSERT SQL이 데이터베이스에 전달된다. 따라서 이 전략은 `트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다.`

##### SEQUENCE
데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다. 이 전략은 시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2 에서 사용할 수 있다.
```sql
CREATE TABLE BOARD (
    id BIGINT NOT NULL PRIMARY KEY,
    data VARCHAR(255)
)

// 시퀀스 생성
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```
```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ", //  매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1
)
public class Board {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BOARD_SEQ_GENERATOR)
    private Long id;
    ...

}
```
* @SequenceGenerator
<table>
  <thead>
    <tr>
      <th>속성</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>식별자 생성기 이름</td>
      <td>필수</td>
    </tr>
    <tr>
      <td>sequenceName</td>
      <td>데이터베이스에 등록되어 있는 시퀀스 이름</td>
      <td>hibernate_sequence</td>
    </tr>
    <tr>
      <td>initialValue</td>
      <td>DDL 생성 시에만 사용됨. 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다.</td>
      <td>1</td>
    </tr>
    <tr>
      <td>allocationSize</td>
      <td>시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)</td>
      <td>50</td>
    </tr>
    <tr>
      <td>catalog, schema</td>
      <td>데이터베이스 catalog, schema 이름</td>
      <td></td>
    </tr>
  </tbody>
</table>

SEQUENCE 전략도 IDENTITY 전략과 마찬가지로 식별자를 조회하기 위해서는 em.persist() 먼저 호출해야 하지만 내부 동작 방식은 IDENTITY 전략과 다르다. em.persist()가 호출 될 때, `먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회`한다. 그리고 조회한 식별자를 엔티티에 할당 후 엔티티를 영속성 컨텍스트에 저장한다. 이후 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장한다. 반대로 IDENTITY전략은 먼저 엔티티를 데이터베이스에 저장한 후 식별자를 데이터베이스에서 다시 조회하여 엔티티의 식별자에 할당한다.

SEQUENCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다. 따라서 IDENTITY 전략과 마찬가지로 2번 통신한다.
1. 식별자를 구하기 위해 데이터베이스 시퀀스를 조회
2. 조회한 시퀀스를 기본 키 값으로 사용해 데이터베이스에 저장.

##### TABLE 
키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다. 이 전략은 별도의 테이블을 사용하므로 모든 데이터베이스에 적용할 수 있다. TABLE 전략을 사용하려면 아래와 같이 키 생성 용도로 사용할 테이블을 만들어야 한다.
```sql
CREATE TABLE MY_SEQUENCES (
    sequence_name VARCHAR(255) NOT NULL,    //  시퀀스 이름
    next_val BIGINT,                        //  시퀀스 값
    PRIMARY KEY (SEQUENCE_NAME)
)
```
```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", allocationSize = 1
)
public class Board {
    @Id @GeneratedValue(strategy = GenerationType.TABLE, generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```
TABLE 전략은 시퀀스 대신 테이블을 사용한다는 점만 제외하면 SEQUENCE 전략과 내부 동작방식이 같다.
* @TableGenerator
<table>
  <thead>
    <tr>
      <th>속성</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>식별자 생성기 이름</td>
      <td>필수</td>
    </tr>
    <tr>
      <td>table</td>
      <td>키생성 테이블명</td>
      <td>hibernate_sequences</td>
    </tr>
    <tr>
      <td>pkColumnName</td>
      <td>시퀀스 컬럼명</td>
      <td>sequence_name</td>
    </tr>
    <tr>
      <td>valueColumnName</td>
      <td>시퀀스 값 컬럼명</td>
      <td>next_val</td>
    </tr>
    <tr>
      <td>pkColumnValue</td>
      <td>키로 사용할 값 이름</td>
      <td>엔티티 이름</td>
    </tr>
    <tr>
      <td>initialValue</td>
      <td>초기 값, 마지막으로 생성된 값이 기준</td>
      <td>0</td>
    </tr>
    <tr>
      <td>allocationSize</td>
      <td>시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)</td>
      <td>50</td>
    </tr>
    <tr>
      <td>catalog, schema</td>
      <td>데이터베이스 catalog, schema 이름</td>
      <td></td>
    </tr>
    <tr>
      <td>uniqueConstraint(DDL)</td>
      <td>유니크 제약 조건을 지정할 수 있다.</td>
      <td></td>
    </tr>
  </tbody>
</table>
TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고 다음 값으로 증가시키기 위해 UPDATE 쿼리를 사용한다. 이 전략은 SEQUENCE 전략과 마찬가지로 데이터베이스에 한 번 더 통신하게 되는 단점이 있다. 최적화하려면 마찬가지로 @TableGenerator.allocationSize를 사용한다.

> * SEQUENCE/TABLE 전략의 allocationSize 기본 값이 50인 이유는?<br>
>   * SEQUENCE 전략의 최적화를 위해<br>
>   시퀀스에 접근하는 횟수를 줄이기 위해 allocationSize를 사용한다. 예를 들어 allocationSize 값이 50이면 시퀀스를 한번에 50 증가 시킨 후 1 ~ 50 까지는 메모리에서 식별자를 할당한다. 그리고 51이 되면 시퀀스 값을 100으로 증가시킨 후 51 ~ 100 까지 메모리에서 식별자를 할당한다.<br>
>   이 방법은 시퀀스 값을 선점하므로 jvm이 동시에 동작하더라도 기본 키 값이 충돌하지 않는 장점이 있다. 다만 시퀀스 값이 한 번에 많이 증가한다.
>   * TABLE 전략의 최적화를 위해<br>
>   값을 조회하게 되면 SELECT 쿼리 사용 후 다음 값을 증가시키기 위해 UPDATE 쿼리가 사용된다. 이는 SEQUENCE 전략과 비교해서 데이터베이스와 한 번 더 통신한다는 단점이 있다. 따라서 마찬가지로 allocationSize 기본 값을 50으로 줘서 이를 최소화하게 된다. 

##### AUTO
GenerationType.AUTO는 선택된 데이터베이스 방언데 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다. (ex. Oracle - SEQUENCE, MySQL - IDENTITY)
> AUTO를 적용하게 되면 데이터베이스를 변경해도 코드 수정이 필요 없다.

## 필드와 컬럼 매핑 : 레퍼런스
### @Column
객체 필드를 테이블 컬럼에 매핑한다.
<table>
  <thead>
    <tr>
      <th>속성</th>
      <th>기능</th>
      <th>기본 값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>필드와 매핑할 테이블의 컬럼 이름</td>
      <td>객체의 필드 이름</td>
    </tr>
    <tr>
      <td>insertable(거의 사용 X)</td>
      <td>엔티티 저장 시 false로 설정한 경우 해당 필드는 데이터베이스에 저장되지 않는다.(읽기 전용일 때 사용)</td>
      <td>true</td>
    </tr>
    <tr>
      <td>updatable(거의 사용 X)</td>
      <td>엔티티 수정 시 false로 설정한 경우 해당 필드는 데이터베이스에 수정하지 않는다.(읽기 전용일 때 사용)</td>
      <td>true</td>
    </tr>
    <tr>
      <td>table(거의 사용 X)</td>
      <td>하나의 엔티티를 두 개 이상의 엔티티에 매핑할 때 사용. 지정한 필드를 다른 테이블에 매핑할 수 있다.</td>
      <td>현재 클래스가 매핑된 테이블</td>
    </tr>
    <tr>
      <td>nullable(DDL)</td>
      <td>false 설정 할 경우 DDL 생성 시에 not null 제약 조건을 추가한다.</td>
      <td>true</td>
    </tr>
    <tr>
      <td>unique(DDL)</td>
      <td>한 컬럼에 유니크 제약조건을 걸 때 사용.</td>
      <td></td>
    </tr>
    <tr>
      <td>columnDefinition(DDL)</td>
      <td>데이터베이스 컬럼 정보를 직접 줄 수 있다.</td>
      <td>필드의 자바 타입과 벙언 정보를 사용해서 적절한 컬럼 타입을 생성한다.</td>
    </tr>
    <tr>
      <td>length(DDL)</td>
      <td>문자 길이 제약조건, String 타입에만 사용.</td>
      <td>255</td>
    </tr>
    <tr>
      <td>precision, scale(DDL)</td>
      <td>BigDecimal 타입에서 사용한다. 아주 큰 숫자, 정밀한 소수를 다룰때만 사용한다.</td>
      <td>precision=19, scale=2</td>
    </tr>
  </tbody>
</table>

#### nullable
```java
int data;       //  not null로 설정됨.

Integer data;   //  nullable = true로 설정됨.

@Column
int data        // nullalbe = true로 설정되기 때문에 nullable = false 설정이 필요함.
```

> int data; 에 @Column을 선언하게 되면 nullable = true가 기본 값이므로 `nullable = false`를 선언하는 것이 안전하다.

### @Enumerated
자바의 enum 타입을 매핑할 때 사용한다.
#### 속성
##### EnumType.ORDINAL(기본 값)
enum 순서를 데이터베이스에 저장한다.(ex. ADMIN은 0, USER는 1)
* 장점 : 데이터베이스에 저장되는 데이터 크기가 작다.
* 단점 : 이미 저장된 enum의 순서를 변경할 수 없다.

##### EnumType.STRING
enum 이름을 데이터베이스 저장한다.(ex. ADMIN은 ADMIN, USER는 USER)
* 장점 : 저장된 enum의 순서가 바뀌거나 enum이 추가되어도 안전하다.
* 단점 : 데이터베이스에 저장되는 데이터 크기가 ORDINAL에 비해 크다.

### @Temporal
날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용한다. 해당 어노테이션을 생략하게 되면 자바의 Date와 유사한 timestamp or datetime으로 정의된다.
* datetime: MySQL
* timestamp: H2, Oracle, PostgreSQL

#### 속성
기본 값 없이 필수로 지정해야 한다.
##### Temporal.DATE
날짜, 데이터베이스 date 타입과 매핑(ex. 2020-10-11)
##### Temporal.TIME
시간, 데이터베이스 time 타입과 매핑(ex. 11:24:23)
##### Temporal.TIMESTAMP
날짜와 시간, 데이터베이스 timestamp 타입과 매핑(ex. 2020-10-11 11:23:45)

### @Lob
데이터베이스의 BLOB, CLOB 타입과 매핑한다.
#### 속성
* CLOB : String, char[], java.sql.CLOB
* BLOB : byte[], java.sql.BLOB

### @Transient
매핑하고 싶지 않는 필드에 선언한다. 객체에 임시로 어떤 값을 저장하고 싶을 때 사용한다.

### @Access
JPA가 엔티티 데이터에 접근하는 방식을 지정한다. 해당 어노테이션을 설정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다.(생략 가능하다.) @Id 설정 방식에 따라 필드/프로퍼티 접근 방식을 함께 사용할 수 있다.
#### 속성
##### 필드 접근 - AccessType.FIELD
필드에 직접 접근하는 방식이다. 필드 접근 권한이 private여도 가능하다. 또는 @Id가 필드에 있으면 필드 접근 방식으로 설정된다.
##### 프로퍼티 접근 - AccessType.PROPERTY
접근자(getter)를 사용하여 접근한다. 또는 @Id가 프로퍼티에 있으면 프로퍼티 접근 방식으로 설정된다.