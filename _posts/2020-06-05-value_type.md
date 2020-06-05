---
date: 2020-06-04 22:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 9. 값 타입
description: 9. 값 타입
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
* 엔티티 타입
    * @Entity로 정의하는 객체
    * 식별자를 통해 지속해서 추적할 수 있다.
    
* 값 타입
    * int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입
    * 식별자가 없고 숫자나 문자같은 속성만 있으므로 추적할 수 없다.
    
* 값 타입 3가지 종류
    * 기본값 타입(Basic Value Type)
        * 자바 기본 타입(ex. int, double)
        * 래퍼 클래스(ex. Integer)
        * String
    * 임베디드 타입(Embedded Type - 복합 값 타입)
    * 컬렉션 값 타입(Collection Value Type)
    
## 기본 값 타입
```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;

    private String name;    //  값 타입

    private int age;        //  값 타입
    ...
}
```

member 엔티티는 id라는 식별자 값도 갖고 생명주기도 있지만, 값 타입인 name, age 속성은 식별자 값도 없고 생명주기도 member 엔티티에 의존한다. 따라서 회원 엔티티 인스턴스를 제거하면 name, age 값도 제거된다.<br>
그리고 값 타입은 공유하면 안된다.

## 임베디드 타입(복합 값 타입)
직접 정의한 새로운 값 타입을 말한다.
```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;
    private String name;

    //  근무 기간
    @Temporal(TemporalType.DATE)    java.util.Date startDate;
    @Temporal(TemporalType.DATE)    java.util.Date endDate;

    //  집 주소 표현
    private String city;
    private String street;
    private String zipcode;

    //...
}
```

위 처럼 회원이 상세한 데이터를 그대고 갖고 있는 것은 객체지향적이지 않으며 응집력만 떨어진다. 대신에 근무 기간, 주소 같은 타입이 있다면 코드가 더 명확해질 것이다.

```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;
    private String name;

    @Embedded Period wordPeriod;    //  근무 기간
    @Embedded Address homeAddress;  //  집 주소
    ...
}

@Embeddable
public class Period {
    
    @Temporal(TemporalType.DATE)    java.util.Date startDate;
    @Temporal(TemporalType.DATE)    java.util.Date endDate;
    //..

    public boolean isWork(Date date) {
        //.. 값 타입을 위한 메소드를 정의할 수 있다.
    }
}

@Embeddable
public class Address {
    
    @Column(name="city")    //  매핑할 컬럼 정의 가능
    private String city;
    private String street;
    private String zipcode;
}
```

위와 같이 소스를 짜게되면서 회원 엔티티가 더욱 의미있고 응집력 있게 되었다. 새로 정의한 값 타입들은 재사용도 가능하고 응집도도 높다.<br>
임베디드 타입을 사용하기 위해 다음과 같은 2가지 어노테이션을 사용해야 한다.
* @Embeddable : 값 타입을 `정의`하는 곳에 표시
* @Embedded : 값 타입을 사용하는 곳에 표시

> 임베디드 타입은 기본 생성자가 필수다.

임베디드 타입을 포함한 모든 값 타입은 엔티티의 생명주기에 의존하므로 엔티티와 임베디드 타입의 관계를 UML로 표현하면 컴포지션(Composition) 관계가 된다.

> 하이버네이트에서는 임베디드 타입을 `컴포넌트(Component)`라 한다.

### 임베디드 타입과 테이블 매핑
![회원-테이블매핑](../assets/img/embedded.jpg)
임베디드 타입 덕분에 객체와 테이블을 세밀하게 매핑하는 것이 가능하다. 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.<br>
ORM을 사용하지 않고 개발하면 테이블 컬럼과 객체 필드를 대부분 1:1 로 매핑한다. 이제 테이블 하나에 여러 클래스를 매핑하는 반복적인 작업은 JPA에 맡기고 더 세밀한 객체지향 모델을 설계하는데 집중할 수 있다.

### 임베디드 타입과 연관관계
임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있다.
> 엔티티는 공유될 수 있으므로 `참조`한다고 표현하고, 값 타입은 특정 주인에 소속되고 논리적인 개념상 공유되지 않으므로 `포함`한다고 표현했다.

```java
@Entity
public class Member {

    @Embedded Address address;          //  임베디드 타입 포함
    @Embedded PhoneNumber phoneNumber;  //  임베디드 타입 포함
    //...
}

@Embeddable
public class Address {
    
    String street;
    String city;
    String state;
    @Embedde Zipcode zipcode;   //  임베디드 타입 포함
}

@Embeddable
public class Zipcode {
    
    String zip;
    String plusFour;

    @ManyToOne
    PhoneServiceProvider provider;  //  엔티티 참조
    ...
}

@Entity
public class PhoneServiceProvider {
    
    @Id
    String name;
    ...
}
```

값 타입인 Address가 값 타입인 Zipcode를 포함하고, 값 타입인 PhoneNumber가 엔티티 타입인 PhoneServiceProvider를 참조한다.

### @AttributeOverride: 속성 재정의
임베디드 타입에 정의한 매핑정보를 재정의하기 위해 사용되는 어노테이션이다.

```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;
    private String name;

    @Embedded Address homeAddress;
    @Embedded Address companyAddress;

}
```
위와 같이 같은 Address 타입을 갖고 있게되면서 테이블에 매핑하는 컬럼명이 중복되게 될 때 @AttributeOverride를 사용하여 재정의 해줄 수 있다.

```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    private Long id;
    private String name;

    @Embedded Address homeAddress;

    @Embedded
    @AttributeOverride({
        @AttributeOverride(name="city", column="@Column(name = "COMPANY_CITY")),
        @AttributeOverride(name="street", column="@Column(name = "COMPANY_STREET")),
        @AttributeOverride(name="zipcode", column="@Column(name = "COMPANY_ZIPCODE"))
    })
    Address companyAddress;
}
```
```sql
CREATE TABLE MEMBER (
    COMPANY_CITY varchar (255),
    COMPANY_STREET varchar (255),
    COMPANY_ZIPCODE varchar (255),
    city varchar(255),
    street varchar(255),
    zipcode varchar(255).
    ...
)
```
> @AttributeOverride를 너무 많이 사용하면 엔티티 코드가 지저분해진다.
> @AttributeOverride는 엔티티에 설정해야 한다. 임베디드 타입을 가지고 있어도 엔티티에 설정해야 한다.

### 임베디드 타입과 null
임베디드 타입이 null이면 매핑한 컬럼 값은 모두 null이 된다.

## 값 타입과 불변 객체
### 값 타입 공유 참조
임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다. 때문에 값을 복사하여 사용해야 한다.
### 값 타입 복사
```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

//  회원1의 address 값을 복사해서 새로운 newAddress 값을 생성
Address newAddress = address.clone();

newAddress.setCity("NewCity");
member2.setHomeAddress(newAddress);
```

회원1의 주소 인스턴스를 복사해 사용하므로써 의도된대로 영속성 컨텍스트는 회원2의 주소만 변경된 것으로 판단하여 회원2에 대해서만 UPDATE SQL을 실행한다.<br>
이렇게 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.

> 문제는 객체 타입이다. 임베디드 타입처럼 직접 정의한 값 타입은 `자바의 기본 타입이 아니라 객체 타입`이다.

객체 타입은 객체에 값을 대입하면 항상 참조 값을 전달한다. 따라서 `참조 값을 넘겨 받은 두 객체는 같은 인스턴스를 공유하게 된다.`

> 객체의 공유 참조는 피할 수 없다.

근본적인 해결책은 단순한 방법으로, 객체의 값을 수정하지 못하게 막으면 된다. (ex. Address객체의 setCity() 같은 수정자 메소드를 모두 제거)

### 불변 객체
값 타입은 부작용이 발생하면 안된다. 객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원천 차단할 수 있다. 따라서 값 타입은 될 수 있으면 불변 객체(Immutable Object)로 설계해야 한다.<br>
불변 객체의 값은 조회할 수 있지만 수정할 수 없다. 불변 객체도 결국 객체이기 때문에 인스턴스의 참조 값 공유를 피할 수는 없지만, 참조 값을 공유해도 수정할 수 없기 때문에 부작용이 발생할 수 없다.

> 생성자로만 값을 설정하고 수정자를 만들지 않음으로써 불변 객체를 생성할 수 있다.

```java
@Embeddable
public class Address {
    
    private String city;

    protected Address();    //  JPA에서 기본 생성자는 필수다.

    //  생성자로 초기 값을 설정한다.
    public Address(String city) {
        this.city = city;
    }

    //  접근자(Getter)는 노출한다.
    public String getCity() {
        return city;
    }

    //  수정자(Setter)는 만들지 않는다.
}
``` 

즉, 불변 객체를 활용함으로써 객체 타입의 부작용을 해결할 수 있다.

## 값 타입의 비교
* 동일성(Identity) 비교 : 인스턴스의 참조 값을 비교. == 사용
* 동등성(Equality) 비교 : 인스턴스의 값을 비교. equals() 사용

Address 값 타입을 a == b로 동일성 비교하면 둘은 서로 다른 인스턴스이므로 결과는 거짓이 나오지만, 값 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야하기 때문에 a.equals(b)를 사용해서 동등성 비교를 해야 한다.
> Address의 equals() 재정의는 필요하다.

> 자바에서 equals()를 재정의하면 hashCode()도 재정의하는 것이 안전하다. 그렇지 않으면 해시를 사용하는 컬렉션(HashSet, HashMap)이 정상 동작하지 않는다.