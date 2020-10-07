---
date: 2020-10-06 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 4. 리포지터리와 모델구현(JPA 중심)
description: 4. 리포지터리와 모델구현(JPA 중심)
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
category: ddd
tags:
  - ddd
  - 책 정리
paginate: true
comments: true
---
# JPA를 이용한 리포지터리 구현
## 모듈 위치
리포지터리 인터페이스는 애그리거트와 같이 도메인 영역에 속하고, 리포지터리를 구현한 클래스는 **DIP에 따라** InfraStructure 영역에 속한다.

> 간혹 리포지터리 구현 클래스는 domain.impl과 같은 패키지에 분리시키는 경우가 있는데 이는 리포지터리 인터페이스와 구현체를 분리시키기 위한 타협안이지 좋은 설계 원칙을 따르는 것은 아니다. 가능하면 리포지터리 구현 클래스를 InfraStructure 영역에 위치시켜서 InfraStructure에 대한 의존을 낮춰야 한다.

## 리포지터리 기본 기능 구현
리포지터리 기본 기능은 다음 두 가지이다.
* 아이디로 애그리거트 조회하기
* 애그리거트 저장하기

* 리포지터리 인터페이스 구현 예시

```java
public interface OrderRepository {
    public Order findById(OrderNo no);
    public void save(Order order);
}
```

인터페이스는 애그리거트 루트를 기준으로 작성한다. 주문 애그리거트로 예를 들면, Order 루트 엔티티를 비롯해 OrderLine, Orderer, ShippingInfo 등 다양한 객체를 포함하는데, 이 중 Order 루트 엔티티를 기준으로 리포지터리 인터페이스를 작성한다.

# 매핑 구현
## 엔티티와 밸류 기본 매핑 구현
애그리거트와 JPA 매핑을 위한 기본 규칙은 다음과 같다.
* 애그리거트 루트는 엔티티이므로 @Entity로 매핑 설정한다.
* 한 테이블에 엔티티와 밸류 데이터가 같이 있다면,
    * 밸류는 @Embeddable로 매핑 설정한다.
    * 밸류 타입 프로퍼티는 @Embedded로 매핑 설정한다.

* 주문 애그리거트 예시 코드

```java
@Entity
@Table(name = "od_order")
public class Order {
    // ...
    @Embedded
    private Orderer orderer;

    @Embedded
    private ShippingInfo shippingInfo;

    @Embedded
    private Receiver receiver;

}

@Embeddable
public class Orderer {
    //  MemberId에 정의된 컬럼 이름을 변경하기 위해
    //  @AttributeOverride 애노테이션 사용
    @Embedded
    @AttributeOverrides(
        @AttributeOverride(name = "id", column = @Column(name = "order_id"))
    )
    private MemberId memberId;

    @Column(name = "orderer_name")
    private String name;
}

@Embeddable
public class MemberId implements Serializable {
    @Column(name = "member_id")
    private String_id;
}

@Embeddable
public class ShippingInfo {
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "zipCode", column = @Column(name = "shipping_zipcode")),
        @AttributeOverride(name = "address1", column = @Column(name = "shipping_addr1")),
        @AttributeOverride(name = "address2", column = @Column(name = "shipping_addr2"))
    })
    private Address address;

    @Column(name = "shipping_message")
    private String message;
}

```

## 기본 생성자

```java
@Embeddable
public class Receiver {
    @Column(name = "receiver_name")
    private String name;
    @Column(name = "receiver_phone")
    private String phone;

    protected Receiver() {} //  JPA를 적용하기 위해 기본 생성자 추가

    public Receiver(String name, String phone) {
        this.name = name;    
        this.phone = phone;
    }

    // get 메서드 생략
}
```

Receiver 객체가 불변 타입이면 생성 시점에 필요한 값을 모두 전달받으므로 값을 변경하는 set 메서드를 제공하지 않는다. 이는 즉, Recevier 클래스에 파라미터가 없는 기본 생성자를 추가할 필요가 없음을 의미한다.

하지만, JPA의 @Entity와 @Embeddable로 클래스를 매핑하려면 기본 생성자를 제공해야 한다. 왜냐하면 하이버네이트와 같은 JPA 프로바이더는 DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본 생성자를 사용해서 객체를 생성한다. 따라서 이런 기술적 제약 때문에 불변 타입은 기본 생성자가 필요 없음에도 불구하고 위와 같이 추가해야 한다.

기본 생성자는 JPA 프로바이더가 객체를 생성할 때만 사용한다. 기본 생성자를 다른 코드에서 사용하면 값이 없는 온전하지 못한 객체를 만들게 된다. 이런 이유로 다른 코드에서 기본 생성자를 사용하지 못하도록 protected로 선언한다. 

> 하이버네이트는 클래스를 상속한 프록시 객체를 이용해서 지연 로딩을 구현한다. 이 경우 프록시 클래스에서 상위 클래스의 기본 생성자를 호출할 수 있어야 하므로 `지연 로딩 대상이 되는 @Entity와 @Embeddable의 기본 생성자는 private이 아닌 protected로 지정`해야 한다.

## 필드 접근 방식 사용
JPA는 필드와 메서드의 두 가지 방식으로 매핑을 처리할 수 있다. 메서드 방시을 사용하려면 get/set 메서드를 구현해야 한다.

앞에서 얘기했지만 엔티티에 프로퍼티를 위한 public get/set 메서드를 추가하면 도메인의 의도가 사라지고 객체가 아닌 데이터 기반으로 엔티티를 구현할 가능성이 높아지기 때문에  특히 set 메서드는 내부 데이터를 외부에서 변경할 수 있는 수단이 되기 때문에 캡슐화를 깨는 원인이 된다.

엔티티가 객체로서 제 역할을 하려면 외부에 set 메서드 대신 의도가 잘 드러날 수 있는 메소드를 구현해야 한다.(ex. setShippingInfo() -> changeShippingInfo())

> 불변 타입의 밸류 타입을 구현할 때는 set 메서드 자체가 필요하지 않는데 JPA의 구현 방식 때문에 public set 메서드를 추가하는 것도 좋지 않다.

엔티티를 객체가 제공할 기능 중심으로 구현하도록 유도하려면 JPA 매핑 처리를 프로퍼티 방식이 아닌 **필드 방식**으로 선택해서 불필요한 get/set 메서드를 구현하지 말아야 한다.

```java
@Entity
@Access(AccessType.FIELD)   //  필드 방식 선택
public class Order {
    @EmbeddedId
    private OrderNo number;
    
    @Column(name = "state")
    @Enumerated(EnumType.STRING)
    private OrderState state;

    // cancel(), changeShippingInfo() 등 도메인 기능 구현
    // 필요한  get 메서드 제공 
}
```

> @Access가 선언되어 있지 않으면, @Id나 @EmbeddedId 위치로 접근 방식을 선택한다. 필드에 위치하면 **필드 접근 방식**, get 메서드에 위치하면 메서드 접근 방식을 선택한다.

## AttributeConverter를 이용한 밸류 매핑 처리
두 개 이상의 프로퍼티를 가진 밸류 타입을 한 개 컬럼에 매핑해야 할 경우 @Embeddable로는 처리할 수 없다. AttributeConverter를 사용해서 변환처리 할 수 있다. AttributeConverter는 JPA 2.1에 추가된 인터페이스로 다음과 같이 밸류 타입과 컬럼 데이터 간의 변환 처리를 위한 기능을 정의하고 있다.

```java
public interface AttributeConverter<X, Y> {
    public Y convertToDatabaseColumn(X attribute);
    public X convertToEntityAttribute(Y dbData);
}
```

타입 파라미터 X는 **밸류 타입**, Y는 **DB 타입**이다. convertToDatabaseColumn() 메서드는 `밸류 타입을 DB 컬럼 값으로 변환`하고, convertToEntityAttribute() 메서드는 `DB 컬럼 값을 밸류로 변환`한다.

```java
@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Integer> {
    
    @Override
    public Integer convertToDatabaseColumn(Money money) {
        if (money == null)  return null;
        else    return money.getValue();
    }
    
    @Override
    public Money convertToEntityAttribute(Integer value) {
        if (value == null)  return null;
        else    return new Money(value);
    }
    
}
``` 

**autoApply를 true로 지정**하면 위 코드에서는 모델에 출현하는 모든 Money 타입의 프로퍼티에 대해 MoneyConverter를 자동으로 적용한다.<br>
**autoApply가 false인 경우** 프로퍼티 값을 변환할 때 아래와 같이 사용한 컨버터를 직접 지정할 수 있다.

```java
public class Order {
    @Column(name = "total_amounts)
    @Converter(converter = MoneyConverter.class)
    private Money totalAmounts;
    // ... 
}
```
## 밸류 컬렉션: 별도 테이블 매핑
Order 엔티티는 한 개 이상의 OrderLine을 가질 수 있다. OrderLine에 순서가 있다면 아래와 같이 List타입을 사용하여 OrderLine 타입의 컬렉션을 프로퍼티로 갖게 된다.

밸류 타입의 컬렉션은 별도 테이블에 보관한다. 아래와 같이 ORDER_LINE 테이블은 외부키를 이용하여 PURCHASE_ORDER 테이블을 참조한다.<br>
이 외부키는 컬렉션이 속할 엔티티를 의미한다. List 타입의 컬렉션은 인덱스 값이 필요하므로 ORDER_LINE 테이블에는 인덱스 값을 저장하기 위한 컬럼(line_idx)도 존재한다.

밸류 컬렉션을 별도 테이블로 매핑할 때는 **@ElementCollection**과 **@CollectionTable**을 함께 사용한다.  
```java
@Entity
@Table(name = "purchase_order")
public class Order {
    //...
    @ElementCollection
    @CollectionTable(name = "order_line",
                     joinColumns = @JoinColumn(name = "order_number"))
    @OrderColumn(name = "line_idx")
    private List<OrderLine> orderLines;
    //...
}

@Embeddable
public class OrderLine {
    @Embedded
    private ProductId productId;

    @Column(name = "price")
    private Money price;

    @Column(name = "quantity")
    private int quantity;

    @Column(name = "amounts")
    private Money amounts;

    //...
}
```

OrderLine의 매핑을 함께 표시했는데 OrderLine에 List의 인덱스 값을 저장하기 위한 프로퍼티는 존재하지 않는다. 왜냐하면 List 타입 자체가 인덱스를 갖고 있기 때문이다. JPA는 @OrderColumn 애노테이션을 이용해서 지정한 컬럼에 리스트의 인덱스 값을 저장한다.

**@CollectionTable**은 밸류를 저장할 테이블을 지정할 때 사용한다. name은 속성으로 테이블 명을 지정하고 joinColumns 속성은 외부키로 사용하는 컬럼을 지정한다. 위에서는 외부키가 한 개이지만, 두 개 이상인 경우는 @joinColumn의 배열을 이용해서 외부키 목록을 지정한다.
## 밸류 컬렉션: 한 개 컬럼 매핑
밸류 컬렉션을 별도 테이블이 아닌 한 개 컬럼에 지정해야 할 때는 
## 밸류를 이용한 아이디 매핑
## 별도 테이블에 저장하는 밸류 매핑
## 밸류 컬렉션을 @Entity로 매핑하기
## ID 참조와 조인 테이블을 이용한 단방향 M-N 매핑
# 애그리거트 로딩 전략
# 애그리거트의 영속성 전파
# 식별자 생성 기능