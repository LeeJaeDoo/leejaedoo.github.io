---
date: 2020-10-06 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 4. 리포지터리와 모델구현(JPA 중심)
description: 4. 리포지터리와 모델구현(JPA 중심)
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
categories: ddd
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
밸류 컬렉션을 별도 테이블이 아닌 한 개 컬럼에 지정해야 할 때는 마찬가지로 AttributeConverter를 사용하면 밸류 컬렉션을 한 개 컬럼에 쉽게 매핑할 수 있다. 단, AttributeConverter를 사용하려면 밸류 컬렉션을 표현하는 새로운 밸류 타입을 추가해야 한다.

* ex.

```java
@Converter
public class EmailSetConverter implements AttributeConverter<EmailSet, String> {
    @Override
    public String converterToDatabaseColumn(EmailSet attribute) {
        if (attribute == null)  return null;
        return attribute.getEmails().stream()
                        .map(Email::toString)
                        .collect(Collectors.joining(","));
    }

    @Override
    public EmailSet convertToEntityAttribute(String dbData) {
        if (dbData == null) return null;
        String[] emails = dbData.split(",");
        Set<Email> emailSet = Arrays.stream(emails)
            .map(value -> new Email(value))
            .collect(toSet());

        return new EmailSet(emailSet);
    }
}
```

* 프로퍼티에 Converter 적용

```java
@Column(name = "emails")
@Convert(converter = EmailSetConverter.class)
private EmailSet emailSet;
```

## 밸류를 이용한 아이디 매핑
식별자는 최종적으로 문자열이나 숫자와 같은 기본 타입이기 때문에 다음과 같이 String이나 Long 타입을 이용해서 식별자를 매핑한다.(ex. @Id)

아니면 식별자 의미를 부각시키기 위해 식별자 자체를 밸류 타입으로 구현할 수도 있다. @Id 대신 **@EmbeddedId** 어노테이션을 사용하면 된다.

```java
@Entity
@Table(name = "purchase_order")
public class Order {
    @Embedded
    private OrderNo number;
    //...
}

@Embeddable
public class OrderNo implements Serializable {  //  식별자 타입은 Serializable 타입이어야 한다.
    @Column(name = "order_number")
    private String number;
    //...
}
```

밸류 타입을 식별자로 구현함으로써 **식별자에 여러 기능을 추가**할 수 있다는 장점이 생긴다.
## 별도 테이블에 저장하는 밸류 매핑
보통 애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류이다. 단지 별도 테이블에 데이터를 저장한다고 해서 다 엔티티인 것은 아니다.

밸류가 아니라 엔티티가 확실하다면 다른 애그리거트는 아닌지 확인해봐야 한다. 특히 자신만의 독자적인 라이프사이클을 갖는다면 다른 애그리거트일 가능성이 높다.

애그리거트에 속한 객체가 밸류인지 엔티티인지 구분하는 방법은 `고유 식별자를 갖는지 여부를 확인`하는 것이다. 하지만, 식별자를 찾을 때 매핑되는 테이블의 식별자를 애그리거트 식별자로 착각하면 안된다.

> 별도 테이블로 저장되고 테이블에 PK가 있다고 해서 테이블과 매핑되는 애그리거트 구성요소가 고유 식별자를 갖는 것은 아니다.

* @SecondaryTable을 이용한 밸류 매핑 설정

```java
@Entity
@Table(name = "article")
@SecondaryTable(
    name = "article_content",
    pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public Article {
    @Id
    private Long id;
    private String title;
    //...
    @AttributeOverrides({
        @AttributeOverride(name = "content",
            column = @Column(table = "article_content"))
        @AttributeOverride(name = "contentType",
            column = @Column(table = "article_content"))
    })
    private ArticleContent content;
    //...
}
``` 
@SecondaryTable의 name 속성은 `밸류를 저장할 테이블을 지정`한다.<br>
pkJoinColumns 속성은 `밸류 테이블에서 엔티티 테이블로 조인할 때 사용할 컬럼을 지정`한다.<br>
content 필드에 @AttributeOverride를 적용했는데 이 어노테이션을 사용해서 해당 밸류 데이터가 저장된 테이블 이름을 지정한다.

@SecondaryTable을 이용하면 아래와 같은 코드를 실행할 때 두 테이블을 조인해서 데이터를 조회한다.
```java
//  @SecondaryTable로 매핑된 article_content 테이블을 조인
Article article = entityManager.find(Article.class, 1L)
```

문제점이 있다면, 위 예시에서 article 테이블의 데이터만 필요한 상황에서 article_content 테이블의 데이터까지 조인하여 데이터를 읽어오게 되는데 이는 원하는 결과가 아니다.<br>
해결방법으로 지연 로딩 방식을 활용할 수 있지만 **엔티티가 아닌 모델을 엔티티로 만들게 됨으로** 좋은 방법은 아니다.

> 이럴때는 조회 전용 쿼리를 활용하는 것이 좋다.(JPQL 활용)

## 밸류 컬렉션을 @Entity로 매핑하기
개념적으로는 밸류 타입이지만 구현 기술의 한계 혹은 팀 표준 때문에 @Entity를 사용해야 할 떄도 있다.

JPA는 @Embeddable 타입의 클래스 상속 매핑을 지원하지 않기 때문에 다른 방법을 사용해야 하는데 바로 `@Inheritance`와 `@DiscriminatorColumn` 어노테이션을 활용할 수 있다.

상위 클래스에는 `@Inheritance`를 적용하고 strategy는 `SINGLE_TABLE`을 사용, `@DiscriminatorColumns`을 이용해서 타입을 구분하는 용도로 사용할 컬럼을 지정한다.

* 상위 클래스 예시

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DisCriminatorColumn(name = "image_type")
@Table(name = "image")
public abstract class Image {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "image")
    private Long id;

    @Column(name = "image_path")
    private String path;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "upload_time")
    private Date uploadTime;

    protected Image() {}
    public Image(String path) {
        this.path = path;
        this.uploadTime = new Date();
    }

    protected String getPath() {
        return path;
    }

    public Date getUploadTime() {
        return uploadTime;
    }

    public abstract String getURL();
    public abstract boolean hasThumbnail();
    public abstract String getThumbnailURL();
}
```

* 하위 클래스 예시

```java
@Entity
@DiscriminatorValue("II")
public class InternalImage extends Image {
    //...
}

@Entity
@DiscriminatorValue("EI")
public class ExternalImage extends Image {
    //...
}
```

상위 클래스를 상속받는 하위 클래스는 위와 같이 @Entity와 @Discriminator를 사용해서 메핑을 설정한다.

위에서 예시를 든 Image는 @Entity로 설정했고 Image는 Product 엔티티와 일대다관계를 맺게 된다.(@OneToMany) 이로써 Image는 밸류이기 때문에 독자적인 라이프사이클을 갖지 않고 Product에 완전히 의존하게 된다.

* Product 엔티티

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;
    private String name;

    @Converter(converter = MoneyConverter.class)
    private Money price;
    private String detail;

    @OneToMany(casecade = {CascadeType.PERSIST, CascadeType.REMOVE},
            orphanRemoval = true)
    @JoinColumn(name = "product_id")
    @OrderColumn(name = "list_idx")
    private List<Image> images = new ArrayList<>();

    //...

    public void changeImage(List<Image> newImage) {
        images.clear();
        images.addAll(newImages);
    }
}
```

**cascade 속성**을 이용하여 Product와 함께 저장되고 삭제되도록 설정할 수 있다. **orphanRemoval도 true로 설정**한다.

하지만, changeImage() 메서드를 보면 List타입의 images 컬렉션을 변경하기 위해 clear() 후 addAll() 을 활용하고 있는데 이는 clear() 메서드가 호출될 때 select 쿼리로 대상 엔티티를 로딩 후, 각 개별 엔티티에 대해 delete 쿼리를 실행한다.<br>
즉, seletc 쿼리 조회 후 네 번의 delete 쿼리가 실행되면서 성능에 문제가 생길 여지가 있다. 

애그리거트의 특성을 유지하면서 이 문제를 해결하기 위해서는 결과적으로 위와 같은 방식의 상속을 포기하고 `@Embeddable로 매핑된 단일 클래스로 구현`해야 한다. 물론, 이 경우 타입에 따른 분기처리 코드가 필요함에 따라 코드 유지보수에 어려움이 생긴다는 단점이 있다.

```java
@Embeddable
public class Image {
    @Column(name = "image_type")
    private String imageType;
    @Column(name = "image_path")
    private String path;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "upload_time")
    private Date uploadTime;

    //...

    public boolean hasThumbnail() {
        //  성능을 위해 다형을 포기하고 if-else로 분기처리
        if (imageType.equals("II")) {
            return true;
        } else {
            return false;
        }
    }
}
```

> 코드 유지보수와 성능, 두 가지 측면을 고려하여 상황에 맞는 구현 방식을 선택해야 한다.

## ID 참조와 조인 테이블을 이용한 단방향 M-N 매핑
애그리거트 간 집합 연관은 성능상의 이유로 피하는 것이 좋지만, 요구사항을 구현하는데 집합 연관을 사용해야 한다면 ID 참조를 이용한 단방향 집합 연관을 적용해 볼 수 있다.

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;
    
    @ElementCollection
    @CollectionTable(name = "product_category",
        joinColumns = @JoinColumn(name = "product_id"))
    private Set<CategoryId> categoryIds;
    //...
}
```

ID 참조를 이용한 애그리거트 간 단방향 M:N 연관은 밸류 컬렉션 매핑과 동일한 방식이다. 차이점이 있다면, 집합의 값에 밸류 대신 연관을 맺는 식별자가 온다는 점이다.

@ElementCollection을 이용하기 때문에 Product를 삭제할 때 매핑에 사용된 조인 테이블의 데이터도 함께 삭제된다. 애그리거트를 직접 참조하는 방식을 사용했다면 영속성 전파나 로딩 전략을 고민해야 하는데 ID 참조 방식을 사용함으로써 이런 고민을 할 필요가 없다.
# 애그리거트 로딩 전략
JPA 매핑을 설정할 때 애그리거트 루트를 로딩하면 루트에 속한 모든 객체가 완전한 상태여야 함을 기억해야 한다.

조회 시점에 애그리거트를 완전한 상태가 되도록 하려면 애그리거트 루트에서 연관 매핑의 조회 방식을 즉시 로딩으로 설정하면 된다.<br>
즉시 로딩으로 설정하면 애그리거트 루트를 로딩하는 시점에 애그리거트에 속한 모든 객체를 함께 로딩할 수 있는 장점이 있지만, 문제가 될 수도 있다.

* 즉시 로딩이 문제가 될 수 있는 예제

```java
@Entity
@Table(name = "product")
public class Product {
    //...
    @OneToMany(
        casecade = {CascadeType.PERSIST, CascadeType.REMOVE},
        orphanRemoval = true,
        fetch = FetchType.EAGER
    )
    @JoinColumn(name = "product_id")
    @OrderColumn(name = "list_idx")
    private List<Image> images = new ArrayList<>();

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "product_option",
        joinColumns = @JoinColumn(name = "product_id")
    )
    @OrderColumn(name = "list_idx")
    private List<Option> options = new ArrayList<>();

    //...
}
```

특히 위와 같은 컬렉션에 대해 로딩 전략을 즉시 로딩으로 설정하면 문제가 될 수 있다.<br>
위 코드에 따라 Product를 조회하면 Product, Image, Option을 위한 테이블을 조인한 쿼리가 실행된다.

```sql
select
    p.product_id, ..., img.product_id, img,image_id, img.list_idx, img.image_id, ...,
    opt.product_id, opt.option_title, opt.option_value, opt.list_idx
from
    product p
    left outer join image img on p.product_id=img.product_id
    left outer join product_option opt on p.product_id=opt.product_id
where p.product_id=?
``` 

위와 같이 카타시안 조인이 사용되는데 이는 쿼리 결과에 중복을 발생시킨다. 중복 조회되는 행의 개수가 많아지면서 오히러 조회 성능이 나빠지는 문제가 발생할 수 있다.

애그리거트는 개념적으로 하나여야 하지만, 루트 엔티티를 로딩하는 시점에 애그리거트에 속한 객체를 모두 로딩해야 하는 것은 아니다.<br>
애그리거트가 오나전해야 하는 이유는 아래와 같다.
* 상태를 변경하는 기능을 실행할 때 애그리거트 상태가 완전해야 하기 때문에
* Presentation 영역에서 애그리거트의 상태 정보를 보여줄 때 필요하기 떄문에

이 중 두 번째 이유는 별도의 조회 전용 기능을 구현하는 방식을 사용하는 것이 유리할 때가 많기 때문에 애그리거트의 완전한 로딩과 관련된 문제는 상태 변경과 더 관련이 있다. 상태 변경 기능을 실행하기 위해 조회 시점에 즉시 로딩을 이용해서 애그리거트를 완전한 상태로 로딩할 필요는 없다. 실제 상태가 변경되는 시점에 필요한 구성요소만 로딩할 수 있는 지연 로딩을 활용해도 문제가 되지 않는다.

* 지연 로딩 예제

```java
@Transactional
public void removeOptions(ProductId id, int optIdxToBeDeleted) {
    //  Product를 로딩. 컬렉션은 지연 로딩으로 설정했다면, Option은 로딩되지 않음.
    Product product = productRepository.findById(id);

    // 트랜잭션 범위이므로 지연 로딩으로 설정한 연관 로딩 가능
    product.removeOption(optIdxToBeDeleted); 
}

@Entity
public class Product {
    @ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(name = "product_option",
        joinColumns = @JoinColumn(name = "product_id"))
    @OrderColumn(name = "list_idx")
    private List<Option> options = new ArrayList<>();

    public void removeOption(int optIdx) {
        //  실제 컬렉션에 접근할 때 로딩
        this.options.remove(optIdx);
    }
}
```

일반적인 애플리케이션은 상태 변경 기능 실행 빈도보다 조회 기능 실행 빈도가 훨씬 높다. 따라서 상태 변경을 위한 지연 로딩을 사용할 때 발생하는 추가 쿼리로 인해 실행 속도가 저하되는 것은 크게 문제가 되지 않는다.
  
이런 이유로 애그리거트 내 모든 연관을 즉시 로딩으로 할 필요는 없다. `지연 로딩은 동작 방식이 항상 동일하기 때문에 즉시 로딩 처럼 경우의 수를 따질 필요가 없다는 장점`이 있다.

> 즉시 로딩은 @Entity나 @Embeddable에 대해 다르게 동작하고, JPA 프로바이더에 따라 구현 방식이 달라질 수 있다.

물론, 지연 로딩은 즉시 로딩보다 쿼리 실행 횟수가 많아질 수 있기 때문에 애그리거트에 맞게 즉시 로딩과 지연 로딩을 선택해야 한다.
# 애그리거트의 영속성 전파
애그리거트가 완전한 상태여야 한다는 의미는 `조회/저장/삭제가 하나로 처리`돼야함을 의미한다.

@Embeddable 매핑 타입의 경우 함께 저장되고 삭제되므로 cascade 속성을 추가할 필요는 없다.<br>
반면, 애그리거트에 속한 @Entity 타입에 대한 매핑은 cascade 속성을 사용해서 저장과 삭제 시에 함께 처리되도록 설정해야 한다. @OneToOne, @OneToMany는 cascade 속성의 기본값이 없으므로 cascade 속성 값(CascadeType.PERSIST, CascadeType.REMOVE)을 설정한다.

```java
@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
    orphanRemoval = true)
@JoinColumn(name = "product_id")
@OrderColumn(name = "list_idx")
private List<Image> images = new ArrayList<>();
```
# 식별자 생성 기능
식별자는 크게 세 가지 방식 중 하나로 생성한다.
## 사용자가 직접 생성
   * ex. 이메일 주소
   
## 도메인 로직으로 생성
식별자 생성 규칙이 있는 경우, 엔티티를 생성할 때 이미 생성한 식별자를 전달하므로 엔티티가 식별자 생성 기능을 제공하는 것보다는 별도 서비스로 식별자 생성 기능을 분리해야 한다. 식별자 생성 규칙은 곧 도메인 규칙이므로 도메인 서비스에서 구현한다.

```java
public class ProductIdService {
    public ProductId nextId() {
        //  정해진 규칙으로 식별자 생성
    }
}
```

Application 서비스는 위와 같은 도메인 서비스를 이용하여 식별자를 구하고 엔티티를 생성할 수 있다.

```java
public class CreateProductService {
    @Autowired 
    private ProductIdService idService;
    @Autowired 
    private ProductRepository productRespository;

    @Transactional
    public ProductId createProduct(ProductCreationCommend cmd) {
        //  Application 서비스는 도메인 서비스를 이용해서 식별자를 생성
        ProductId id = productIdService.nextId();
        Product product = new Product(id, cmd.getDetail(), cmd.getPrice(), /*...*/);
        productRepository.save(product);
        return id;
    }
}
```

리포지터리에서도 식별자 생성 규칙을 구현할 수 있다.

```java
public interface ProductRepository {
    //... save() 등 다른 메서드

    //  식별자를 생성하는 메서드
    ProductId nextId();
}
```
nextId() 메서드를 구현하는 클래스에서 알맞게 구현할 수 있다.
## DB를 이용한 일련번호 사용
* @GeneratedValue 어노테이션 활용
* insert 쿼리 실행 후 DB에 생성된 식별자를 활용

```java
articleRepository.save(article);    //  실행 시점에 식별자 생성
return article.getId(); //  저장 이후 식별자 사용 가능
```
JPA는 저장 시점에 생성한 식별자를 @Id로 매핑한 프로퍼티/필드에 할당한다.
