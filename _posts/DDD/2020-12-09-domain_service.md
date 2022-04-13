---
date: 2020-12-09 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 7. 도메인 서비스
description: 7. 도메인 서비스
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
categories: ddd
tags:
  - ddd
  - 책 정리
paginate: true
comments: true
---
# 여러 애그리거트가 필요한 기능
한 애그리거트에 넣기 애매한 도메인 기능(ex. 주문 결제 금액 계산)을 특정 애그리거트에서 억지로 구현하게 되면, 해당 애그리거트는 자신의 책임 범위를 넘어서는 기능을 구현하게 되기 때문에 코드가 길어지고, 외부에 대한 의존이 높아지게 된다. 즉, 코드를 복잡하게 만들어 유지보수하기 어려워지게 된다.

게다가 애그리거트의 범위를 넘어서는 도메인 개념이 애그리거트에 숨어들어서 명시적으로 드러나지 않게 된다. 이럴 때 도메인 서비스라는 개념을 도입하여 별도로 구현하여 해결할 수 있다.

# 도메인 서비스
 
응용 영역의 서비스가 응용 로직을 다룬다면, 도메인 서비스는 도메인 로직을 다룬다.

도메인 서비스와 다른 도메인 영역의 애그리거트나 밸류 같은 다른 구성요소와 비교를 해본다면, 도메인 서비스는 상태 없이 로직만 구현한다. 도메인 서비스를 구현하는데 필요한 상태는 애그리거트나 다른 방법으로 전달받는다.

도메인 서비스는 도메인의 의미가 드러나는 용어를 타입과 메서드 이름으로 갖는다.

* 할인 계산 도메인 서비스 로직 예제

```java
@DomainService
public class DiscountCalculationService {

    public Money calculateDiscountAmounts(
        List<OrderLine> orderLines,
        List<Coupon> coupons,
        MemberGrade grade
    ) {
        Money couponDiscount = coupons.stream()
                                      .map(coupon -> calculateDiscount(coupon))
                                      .reduce(Money(0), (v1, v2) -> v1.add(v2));
        
        Money membershipDiscount = calculateDiscount(orderer.getMember().gerGrade());
  
        return couponDiscount.add(membershipDiscount);
    }

    private Money calculateDiscount(Coupon coupon) {
        //...
    }
    
    private Money calculateDiscount(MemberGrade grade) {
        //...
    }
}

@Entity
public class Order {
    
    public void calculateAmounts(
        DiscountCalculationService disCalSvc,
        MemberGrade grade
    ) {
        Money totalAmounts = getTotalAmounts();
        Money discountAmounts = disCalSvc.calculateDiscountAmounts(this.orderLines, this.coupons, grade);
        this.paymentAmounts = totalAmounts.minus(discountAmounts);
    }

    //...

}

@ApplicationService
public class OrderService {
    
    private DiscountCalculationService discountCalculationService;
    
    @Transactional
    public OrderNo placeOrder(OrderRequest orderRequest) {
        OrderNo orderNo = orderRepository.nextId();
        Order order = createOrder(orderNo, orderRequest);
        orderRepository.save(order);
        //  응용 서비스 실행 후 표현 영역에서 필요한 값 리턴
        return orderNo;
    }

    private Order createOrder(OrderNo orderNo, OrderRequest orderRequest) {
        Member member = findMember(orderRequest.getOrdererId());
        Order order = new Order(orderNo, orderRequest.getOrderLines(), 
                                orderRequest.getCoupons(), createOrderer(member), 
                                orderRequest.getShippingInfo());
        order.calculateAmounts(this.discountCalculationService, member.getGrade());
    
        return order;
    }
    //...
}
```

> 애그리거트 객체에 도메인 서비스를 전달하는 것은 응용 서비스 책임이다.

## 도메인 서비스 객체를 애그리거트에 주입하지 않기

```java
public class Order {
    
    @Autowired
    private DiscountCalculateService discountCalculateService;
    
    //...
}
```

위처럼 DI를 위해 애그리거트 루트 엔티티에 도메인 서비스에 대한 참조를 필드로 추가한다면, 문제가 있다.(필자의 의견)<br>
도메인 객체는 필드(프로퍼티)로 구성된 데이터와 메서드를 이용한 기능을 이용해서 개념적으로 하나의 모델을 표현하게되는데 discountCalculateService 필드는 데이터 자체와는 관련이 없다. Order 객체를 DB에 보관할 때 다른 필드와 달리 저장 대상도 아니다.<br>
또 Order가 제공하는 모든 기능에서 discountCalculateService를 필요로 하는 것도 아니다. 일부 기능만을 위해 굳이 도메인 서비스 객체를 애그리거트에 DI할 이유는 없다.<br>
이는 프레임워크의 기능을 사용하고 싶은 개발자의 욕심일 뿐이다.


> 도메인 서비스는 도메인 로직을 수행하지 응용 로직을 수행하지는 않는다. 트랜잭션 처리와 같은 로직은 응용 로직이므로 도메인 서비스가 아닌 응용 서비스에서 처리해야 한다.

* 응용 서비스와 도메인 서비스의 구분법

애그리거트의 상태를 변경하거나 상태 값을 계산하는 경우 도메인 로직이다. 도메인 로직이면서 한 애그리거트에 넣기 적합하지 않으므로 도메인 서비스로 구현하게 된다.

## 도메인 서비스의 패키지 위치
도메인 서비스는 도메인 로직을 실행하므로 도메인 서비스의 위치는 다른 도메인 구성 요소와 동일한 패키지에 위치한다.

도메인 서비스의 개수가 많거나 엔티티아 밸류와 같은 다른 구성요소와 명시적으로 구분하고 싶다면 domain 패키지 밑에 domain.model, domain.service, domain.repository와 같이 하위 패키지를 구분해도 된다.

## 도메인 서비스의 인터페이스와 클래스
도메인 서비스의 로직이 고정되어 있지 않은 경우 도메인 서비스 자체를 인터페이스로 구현하고 이를 구현한 클래스를 둘 수도 있다.
 
특히 도메인 로직을 외부 시스템이나 별도 엔진을 이용해서 구현해야 할 경우에 인터페이스와 클래스를 분리하게 된다. 

![도메인서비스 인터페이스](../../assets/img/domain_service_interface.jpg)

위 그림처럼 도메인 서비스의 구현이 특정 구현 기술에 의존적이거나 외부 시스템의 API를 실행한다면 도메인 영역의 도메인 서비스는 인터페이스로 추상화해야 한다. 이를 통해 도메인 영역이 특정 구현에 종속되는 것을 방지할 수 있고 도메인 영역에 대한 테스트가 수월해진다.
