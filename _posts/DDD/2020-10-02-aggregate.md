---
date: 2020-10-02 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 3. 애그리거트
description: 3. 애그리거트
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
category: ddd
tags:
  - ddd
  - 책 정리
paginate: true
comments: true
---
# 애그리거트
주요 도메인 개념 간의 관계를 파악하기 어렵다는 것은 곧 코드를 변경하고 확장하는 것이 어려워진다는 것을 의미한다. 상위 수준에서 모델이 어떻게 엮여 있는지 알아야 전체 모델을 망가뜨리지 않으면서 추가 요구사항을 모델에 반영할 수 있는데 세부적인 모델만 이해한 상태로는 코드를 수정하기가 두렵기 때문에 코드 변경을 최대한 회피하는 쪽으로 요구사항을 협의하게 된다.

복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들려면 상위 수준에서 모델을 조망할 수 있는 방법이 필요한데, 그 방법을 바로 `애그리거트`이다.

* 관련된 객체를 하나의 군으로 묶어준다. 애그리거트를 묶어서 바라보면 좀 더 상위 수준에서 도메인 모델 간의 관계를 파악할 수 있다.
* 일관성을 관리하는 기준이 된다. 일관성으로 관리하기 때문에 복잡한 도메인을 단순한 구조로 만들어주고, 복잡도가 낮아지는 만큼 도메인 기능 확장 및 변경에 드는 비용도 줄어든다.
* 한 애그리거트에 속한 객체는 유사하거나 동일한 LifeCycle을 갖는다.
* 한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다.

처음 도메인 모델을 만들기 시작하면 큰 애그리거트로 보이던 것들이 경험이 쌓이고 규칙을 제대로 이해할 수록 애그리거트의 크기는 줄어든다. 실제 두 개 이상의 엔티티로 구성되는 애그리거트는 드물다.
# 애그리거트 루트
애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요한데 이 책임을 지는 것이 애그리거트 루트 엔티티다. 애그리거트 루트 엔티티는 애그리거트의 대표 엔티티로 애그리거트에 속한 객체는 에그리거트 루트 엔티티에 직접 또는 간접적으로 속한다.
## 도메인 규칙과 일관성
애그리거트 루트의 핵심 역할은 `애그리거트의 일관성이 깨지지 않도록 하는 것`이다. 이를 위해 애그리거트 루트는 애그리거트가 제공해야 할 도메인 기능을 구현한다. 예를 들어, 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하는데 애그리거트 루트인 Order가 이 기능을 구현한 메서드를 제공한다.

애그리거트 루트가 아닌 다른 객체가 애그리거트에 속한 객체를 직접 변경하면 안된다. 이는 애그리거트 루트가 강제하는 규칙을 적용할 수 없어 모델의 일관성을 깨는 원인이 된다.

* 하위 객체에서 애그리거트에 속한 객체를 직접 변경하는 경우

```java
ShippingInfo si = order.getShippingInfo();
si.setAddress(newAddress);
```

위 코드는 애그리거트 루트인 Order에서 ShippingInfo를 가져와 직접 정보를 변경하고 있다. 주문 상태 없이 배송지 주소를 바로 변경할 수가 있는데 이는 논리적인 데이터 일관성을 깨뜨리는 행위가 된다.(ex. 마치 DB 데이터를 수정하는 것과 같다.)

* 해결 방안1

```java
ShippingInfo si = order.getShippingInfo();

// 주요 도메인 로직이 중복되는 문제
if (state != OrderState.PAYMENT_WAITING && state != OrderState.WAITING) {
    throw new IllegalArgumentException();
}
si.setAddress(newAddress);
```

위 코드 처럼 일관성을 지키기 위해 상태 확인 로직을 Application 서비스에 추가 구현할 수도 있지만, 이는 동일한 검사 로직이 여러 **Application 서비스에 중복 구현될 가능성**이 높아져 상황을 더 악화시킬 수도 있다.

불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하게 만들려면 도메인 모델에 대해 다음의 두 가지를 습관적으로 적용해야 한다.
* 단순히 필드를 변경하는 set 메서드를 공개(public) 범위로 만들지 않는다.
* 밸류 타입은 불변으로 구현한다.

공개 set 메서드는 중요 도메인의 의미나 의도를 표현하지 못하고 도메인 로직이 도메인 객체가 아닌 Application 영역이나 Presentation 영역으로 분산되게 만드는 원인이 된다. 공개 set 메서드를 사용하지 않게 되면 의미가 드러나는 메서드를 사용해서 구현할 가능성이 높아진다.(ex. cancel이나 change)

공개 set 메서드를 만들지 않는 것의 연장으로 밸류는 불변 타입으로 구현한다. 밸류 객체의 값을 변경할 수 없으면 애그리거트 루트에서 밸류 객체를 구해도 값을 변경할 수 없기 때문에 애그리거트 외부에서 밸류 객체의 상태를 변경할 수 없게 된다.

* ex.

```java
Shippinfo si = order.getShippingInfo();
si.setAddress(newAddress);  //  ShippingInfo 밸류 객체가 불변이면, 이 코드는 컴파일 에러!
```

애그리거트 외부에서 내부 상태를 함부로 바꾸지 못하므로 애그리거트의 일관성이 깨질 가능성이 그만큼 줄어든다. 밸류 객체가 불변이면 밸류 객체의 값을 변경하는 방법은 새로운 밸류 객체를 할당하는 것뿐이다.

* 밸류 객체 값 변경 예시

```java
public Order {
    private ShippingInfo shippingInfo;
    public void changeshippingInfo(ShippingInfo newShippingInfo) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);
    }
    //  set 메서드의 접근 허용 범위는 private이다.
    private void setShippingInfo(ShippingInfo newShippingInfo) {
        //  밸류가 불변이면, 새로운 객체를 할당해서 값을 변경해야 한다.
        //  불변이므로 this.shippingInfo.setAddress(newShippedInfo.getAddress())와 같은 코드를 사용할 수 없다.
        this.shippingInfo = newShippingInfo;
    }
}
```

위 예시 코드처럼 불변 밸류 타입의 내부 상태를 변경하려면 애그리거트 루트를 통해서만 가능하다. 그러므로, 애그리거트 루트가 도메인 규칙을 올바르게만 구현하면 애그리거트 전체의 일관성을 유지할 수 있다.
## 애그리거트 루트의 기능 구현
애그리거트 루트는 애그리거트 내부의 다른 객체를 조합해서 기능을 완성한다.
## 트랜잭션 범위
트랜잭션 범위는 작을수록 좋다. 한개 테이블을 수정할 때에는 트랜잭션 충돌을 막기 위해 잠그는 대상이 한 개 테이블의 한 행으로 한정되지만, 세 개의 테이블을 수정하면 잠금 대상이 더 많아진다. 잠금 대상이 많아진다는 것은 그만큼 동시에 처리할 수 있는 트랜잭션 개수가 줄어든다는 것을 뜻하고 이는 전체적인 성능을 떨어뜨린다.

동일하게 한 트랜잭션에서는 한 개의 애그리거트만 수정해야 한다. 즉, 한 애그리거트에서 다른 애그리거트를 변경하지 않는다는 것을 의미한다.

한 애그리거트에서 다른 애그리거트까지 변경하게 된다면 자신의 책임 범위를 넘어 다른 애그리거트의 상태까지 관리하게되는데 이는 최대한 독립적이어야 하는 애그리거트의 결합도가 높이지게 되는 원인이 된다. 결합도가 높아지면 향후 수정 비용이 증가하기 때문에 애그리거트에서 다른 애그리거트를 수정하면 안된다.

부득이하게 한 트랜잭션으로 두 개 이상의 애그리거트를 수정해야 한다면 애그리거트에서 다른 애그리거트를 직접 수정하는 것이 아닌 `Application 서비스에서 두 애그리거트를 수정하도록 구현`해야 한다.

* Application 서비스에서 두 애그리거트 수정 구현 예시

```java
public class ChangeOrderService {
    //  두 개이상의 애그리거트를 변경해야 한다.
    //  Appliaction 서비스에서 각 애그리거트의 상태를 변경한다.
    @Transactional
    public void changeShippingInfo(OrderId id, ShippingInfo newShippingInfo, boolean useNewShippingAddrAsMemberAddr) {
        Order order = orderRepository.findbyId(id);
        if (order == null) throw new OrderNotFoundException();
        order.shipTo(newShippingInfo);
        if (userNewshippingAddrAsMemberAddr) {
            order.getOrderer().getCustomer().changeAddress(newShippingInfo.getAddress());
        }
    }
}
``` 

> 도메인 이벤트를 사용하면 한 트랜잭션에서 한 개의 애그리거트를 수정하면서도 동기나 비동기로 다른 애그리거트의 상태를 변경하는 코드를 작성할 수 있다.

일반적으로 한 트랜잭션에서 한 개의 애그리거트를 변경하는 것을 권장하지만 다음과 같은 경우는 한 트랜잭션에서 두 개 이상의 애그리거트를 변경하는 것을 고려할 수 있다.

* 팀 or 조직의 표준에 따라 사용자 use case와 관련된 Application 서비스의 기능을 한 트랜잭션으로 실행해야 하는 경우가 있다. DB가 다른 경우 글로벌 트랜잭션을 반드시 사용하도록 규칙을 정하는 곳도 있다.
* 기술적 제약으로 인해 이벤트 방식을 도입할 수 없는 경우 한 트랜잭션에서 다수의 애그리거트를 수정해서 일관성을 처리해야 한다.
* UI 구현의 편리성

# 리포지터리와 애그리거트
애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하므로 객체의 영속성을 처리하는 리포지터리는 애그리거트 단위로 존재한다. 예를 들면 Order와 OrderLine을 물리적으로 각각의 별도 DB 테이블에 저장하더라도 Order가 애그리거트 루트고 OrderLine 애그리거트에 속하는 구성 요소이므로 Order를 위한 리포지터리만 존재한다.

애그리거트는 개념적으로 하나이므로 리포지터리는 애그리거트 전체를 저장소에 영속화해야 한다. 예를 들어, Order 애그리거트와 관련된 테이블이 세 개라면 리포지터리를 통해서 Order 애그리거트를 저장할 때 애그리거트 루트와 매핑되는 테이블뿐만 아니라 애그리거트에 속한 모든 구성요소를 위한 테이블에 데이터를 저장해야 한다.

애그리거트를 구하는 리포지터리 메서드는 완전한 애그리거트를 제공해야 한다. 즉, 아래 코드를 실행하면 order 애그리거트는 OrderLine, Orderer 등 모든 구성요소를 포함하고 있어야 한다.

```java
// 리포지터리는 완전한 order를 제공해야 한다.
Order order = orderRepository.findById(orderId);

// order가 온전한 애그리거트가 아니면
// 기능 실행 도중 NPE같은 문제가 발생한다.
order.cancel();
```
# ID를 이용한 애그리거트 참조
한 애그리거트에서 다른 애그리거트를 참조한다는 것은 애그리거트의 루트를 참조한다는 것과 같다.

* 필드로 직접 참조

```java
public class Member {
    ...
}

public class Orderer {
    private Member member;  //  회원 애그리거트 루트인 Member를 필드로 적접 참조 
    private String name;
    ...
}
```
필드를 이용해서 다른 애그리거트를 직접 참조하는 것은 개발자에게 구현의 편리함을 제공한다.

JPA를 사용하면 @ManyToOne, @OneToOne과 같은 애노테이션을 이용해서 연관된 객체를 로딩하는 기능을 제공하고 있으므로 필드를 이용해서 다른 애그리거트를 쉽게 참조할 수 있다.

ORM 기술 덕에 애그리거트 루트에 대한 참조를 쉽게 구현할 수 있고, 필드(또는 메서드)를 이용한 애그리거트 참조를 사용하면 다른 애그리거트의 데이터를 객체 탐색을 통해 조회할 수 있지만, 필드를 이용한 애그리거트 참조는 아래와 같은 문제를 일으킬 수 있다.

* 편한 탐색 오용

한 애그리거트에서 다른 애그리거트 객체 접근할 수 있으면 다른 애그리거트의 상태를 쉽게 변경할 수 있게 되고, 이는 한 애그리거트는 한 트랜잭션 범위로 한정해야 한다는 규칙을 깨기 쉬운 유혹의 원인이 된다.

* 성능에 대한 고민

특히 JPA를 사용할 경우 지연/즉시 로딩 방식 중 어느 방식이 현재 애그리거트의 기능에 적절할지 고민을 해야 한다.

* 확장 어려움

사용자가 늘고 트래픽이 증가하면서 자연스럽게 부하를 분산시키기 위해 하위 도메인별로 시스템을 분리하는 과정이 필요한데 도메인 간 의존성이 높았었다면 분리하는데 어려움이 따르게 된다.

이러한 문제점들을 완화할 수 있는 방법으로 `ID를 이용해서 다른 애그리거트를 간접적으로 참조`하는 방법이 있다.

ID를 이용한 참조는 DB 테이블에서의 외래키를 사용해서 참조하는 것과 비슷하게 다른 애그리거트를 참조할 때 ID 참조를 사용한다. 단, 애그리거트 내의 엔티티를 참조할 때는 객체 레퍼런스로 참조한다.

`ID 참조`를 사용하면 모든 객체가 참조로 연결되지 않고 한 애그리거트에 속한 객체들만 참조로 연결된다. 이는 애그리거트의 경계를 명확히 하고 애그리거트 간 물리적인 연결을 제거하기 때문에 모델의 복잡도를 낮춰준다. 또한, 애그리거트 간의 의존을 제거하므로 응집도를 높여주는 효과도 있다.<br>
다른 애그리거트를 직접 참조하지 않기 때문에 지연/즉시 로딩 고민도 할 필요가 없다.

참조하는 애그리거트가 필요하면 Application 서비스에서 아이디를 이용해서 로딩하면 된다.

* ID를 이용한 간접 참조 예시

```java
public class ChangeOrderService {
    @Transactional
    public void changeShippingInfo(OrderId id, ShippingInfo newShippingInfo, boolean useNewShippingAddrAsMemberAddr) {
        Order order = orderRepository.findById(id);
        if (order == null) throw new OrderNotFoundException();
        order.changeShippingInfo(newShippingInfo);
        if (useNewshippingAddrAsMemberAddr) {
            //  ID를 이용해서 참조하는 애그리거트를 구한다.
            Customer customer = customerRepository.findById(order.getOrderer().getCustomerId());
            customer.changeAddress(newShippingInfo.getAddress());
        }
    }
}
``` 

Application 서비스에서 필요한 애그리거트를 로딩하므로 애그리서트 수준에서 지연 로딩을 하는 것과 동일한 결과를 만든다.

또한 ID를 이용한 참조 방식은 외부 애그리거트를 직접 참조하지 않기 때문에 애초에 한 애그리거트에서 다른 애그리거트의 상태를 수정하는 문제를 원천적으로 방지할 수 있다.

애그리거트 별로 다른 구현 기술을 사용하는 것도 가능해진다.
## ID를 이용한 참조와 조회 성능
하지만 다른 애그리거트를 ID로 참조하면 참조하는 여러 애그리거트를 읽어야 할 때 조회 속도가 문제가 될 수 있다.(ex. N + 1 문제)<br>
예를 들면 한 번에 모든 데이터를 가져올 수 있음에도 불구하고 주문마다 상품 정보를 읽어오는 쿼리를 실행하는 문제가 발생할 수 있다. 이는 더 많은 쿼리를 실행해서 전체 조회 속도가 느려지는 원인이 된다.

```java
Customer customer = customerRepository.findById(ordererId);
List<Order> orders = orderRepository.findByOrderer(ordererId);
List<OrderView> dtos = orders.stream()
                             .map(order -> {
                                ProductId prodId = order.getOrderLines().get(0).getProductId();
                                //  각 주문마다 첫 번째 주문 상품 정보 로딩 위한 쿼리 실행
                                Product product = productRepository.findById(prodId);
                                return new OrderView(order, customer, product);
                             }).collect(toList());
``` 

전체 조회 속도가 느려지는 문제를 개선하려면 조인을 사용해야 한다. 조인을 사용하는 가장 쉬운 방법은 ID 참조 방식을 객체 참조 방식으로 바꾸고 즉시 로딩을 사용하도록 매핑 설정을 바꾸는 것이다. 하지만, 이 방식은 애그리거트 간 참조를 ID 참조 방식에서 객체 참조 방식으로 다시 되돌리는 것이다.

ID 참조 방식을 사용하면서 N + 1 조회와 같은 문제를 해결하려면 전용 조회 쿼리를 사용하면 된다. 예를 들면 데이터 조회를 위한 별도 DAO를 만들고 DAO의 조회 메서드에서 세타 조인을 이용해서 한 번의 쿼리로 필요한 데이터를 로딩하면된다.

```java
@Repository
public class JpaOrderViewDao implements OrderViewDao {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<OrderView> selectByOrder(String ordererId) {
        String selectQuery =
            "select new com.myshop.order.application.dto.OrderView(o, m, p) " +
            "from Order o join o.orderLines ol, Member m, Product p " +
            "where o.orderer.memberId.id = :ordererId " +
            "and o.orderer.memberId = m.id " +
            "and ol.productId = p.id " +
            "order by o.number.number desc";

        TypedQuery<OrderView> query =
            em.createQuery(selectQuery, OrderView.class);
        query.setParameter("ordererId", ordererId);
        return query.getResultList();
    }
}
```

> JPA를 사용하면 각 객체 간 모든 연관을 지연/즉시로딩으로 어떻게든 처리하고 싶은 욕구가 생길텐데 이는 실용적이지 않다. ID를 참조하여 애그리거트를 참조해도 한 번의 쿼리로 필요한 데이터를 로딩하는 것이 가능하다.

하자민, 만약 애그리거트마다 다른 저장소를 사용하는 경우라면 한 번의 쿼리로 관련 애그리거트를 조회할 수 없으므로 성능을 높이기 위해 캐시를 적용하거나 조회 전용 저장소를 구성한다. 코드는 복잡해지지만 시스템의 처리량을 높일 수 있다. 
# 애그리거트 간 집합 연관
애그리거트 간 1:N, M:N 연관에 대해 살펴본다. 대표적인 예로 카테고리와 상품 간의 연관을 들 수 있다.
## 1:N
애그리거트 간 1:N 관계는 Set과 같은 컬렉션을 활용할 수 있다.

```java
public class Category {
    private Set<Product> products;  //  다른 애그리거트에 대한 1:N 연관
    //...
    public List<Product> getProducts(int page, int size) {
        List<Product> sortedProducts = sortById(products);
        return sortedProducts.subList((page - 1) * size, page * size);
    }
}
```
하지만 위 처럼 도메인 객체 내에 연관을 맺게 되면 해당 객체가 불릴때마다 Category에 속한 모든 Product를 조회하게 되면서 성능에 심각한 문제를 야기시킨다. 따라서 개념적으로는 애그리거트 간에 1:N 연관이 있다고 하더라도 `성능상 문제로 인해 애그리거트 간의 1:N 연관을 실제 구현에 반영하는 경우는 드물다.`

카테고리에 속한 상품을 구할 필요가 있다면 상품 입장에서 자신이 속한 카테고리를 N:1로 연관지어 구하면 된다.

```java
public class Product {
    // ...
    private CateogryId category;
    //...
}

public class ProductListService {
    public Page<Product> getProductOfCategory(Long categoryId, int page, int size) {
        Category category = categoryRepository.findById(categoryId);
        checkCategory(category);
        List<Product> products = productRepository.findByCategoryId(category.getId(), page, size);
        int totalCount = productRepository.countByCategoryId(category.getId());
        return new Page(page, size, totalCount, products);
    }
}
```

따라서 상품 입장에서 자신이 속한 카테고리를 N:1로 연관짓고 Application 서비스를 활용하여 해당 카테고리 식별자에 따른 Product 목록을 구한다.
## M:N
M:N연관은 개념적으로 양쪽 애그리거트에 컬렉션으로 연관을 만든다. 상품이 여러 카테고리에 속할 수 있다고 가정하면 카테고리와 상품은 M:N 연관을 맺는다. M:N도 1:N과 마찬가지로 실제 요구사항을 고려해서 M:N 연관을 구현에 포함시킬지 여부를 결정해야 한다.<br>
예를 들면, 보통 상품 목록 페이지를 보여줄 때 각 상품 별 모든 카테고리 정보를 다 보여주진 않는다. 상품 상세 화면에서 주로 카테고리 정보를 보여주게 된다. 이 요구사항을 고려하면 카테고리 -> 상품의 연관은 필요하지 않다. 상품 -> 카테고리 연관만 구현하면 된다.<br>
즉, 개념적으로는 상품과 카테고리의 양방향 M:N 연관이 존재하지만 실제 구현에서는 상품 -> 카테고리의 단방향 M:N 연관만 적용하면 된다.

RDBMS를 이용해서 M:N 연관을 구현하려면 조인 테이블을 사용한다. JPA를 이용하면 다음과 같이 매핑 설정을 사용해서 ID 참조를 이용한 M:N 단방향 연관을 구현할 수 있다.

```java
@Entity
@Table(name = "pd_mall_product")
public class Product {
    @EmbeddedId
    private ProductId id;
    
    @ElementCollection
    @CollectionTable(name = "pd_mall_product_category",
        joinColumns = @JoinColumn(name = "product_id))
    private Sets<CategoryId> categoryIds;
    //...
}
```

이 매핑은 카테고리 ID 목록을 보관하기 위해 밸류 타입에 대한 컬렉션 매핑을 이용했다. 이 매핑을 사용하면 다음과 같이 JPQL의 member of 연산자를 이용해서 특정 Category에 속한 Product 목록을 구하는 기능을 구현할 수 있다.

```java
@Repository
public class JpaProductRepository implements ProductRepository {
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Product> findByCategoryId(CategoryId categoryid, int page, int size) {
        TypedQuery<Product> query = entityManager.createQuery(
            "select p from Product p " + 
            "where :catId member of p.categoryIds order by p.id.id desc", Product.class);
        query.setParameter("catId", categoryId);
        query.setFirstResult((page - 1) * size);
        query.setMaxResults(size);
        return query.getResultList();
    }
}
```

> **@CollectionTable vs @JoinTable**
> - @CollectionTable은 1:N관계에서 모든 프로퍼티가 아닌 필요한 프로퍼티만 가지고 와서 CollectionTable을 구성할 때 먼저 @ElementCollection을 선언 후 사용된다.<br>
> - @JoinTable은 M:N관계에서 중간 매핑 테이블을 정의할 때 사용한다.
> 참조 : [https://jogeum.net/7](https://jogeum.net/7)

# 애그리거트를 팩토리로 사용하기
예를 들어, 특정 상점에서 더 이상 상품 등록을 할 수 없도록 차단한 상태라고 할 때 상품 등록 기능을 아래와 같이 Application 서비스에 구현할 수 있을 것이다.

* 도메인 로직 처리가 Application 서비스에 노출된 예제 코드

```java
public class RegisterProductService {
    public ProductId registerNewProduct(NewProductRequest req) {
        Store accout = accountRepository.findStoreById(req.getStoreId());
        checkNull(account);
        if (account.isBlocked()) {throw new StoreBlockedException();}
        ProductId id = productRepository.nextId();
        Product product = new Product(id, account.getId(), /*...*/);
        productRepository.save(product);
        return id;
    }
}
```

위 코드는 Store가 Product를 생성할 수 있는 지에 대한 여부를 판단하고 Product를 생성하는 것은 논리적으로 하나의 도메인 기능인데 이를 Application 서비스에서 구현하고 있다.<br>
이 도메인 기능을 넣기 위한 별도의 도메인 서비스나 팩토리 클래스를 만들 수도 있지만 이 기능을 구현하기에 더 좋은 장소는 Store 애그리거트이다. Product를 생성하는 기능을 Store 애그리거트에 다음과 같이 옮길 수 있다.

```java
public class Store extends Member {
    public Product createProduct(ProductId newProductId, /*...*/) {
        if (isBlocked()) throw new StoreBlockedException();
        return new Product(newProductId, getId(), /*...*/);
    }
}
```

Store 애그리거트의 createProduct()는 Product 애그리거트를 생성하는 **팩토리 역할**을 한다. 이제 Application 서비스에서는 팩토리 기능을 이용하여 Product를 생성하면 된다.

```java
public class RegisterProductService {
    public ProductId registerNewProduct(NewProductRequest req) {
        Store account = accountRepository.findStoreById(req.getStoreId());
        checkNull(account);
        ProductId id = productRepository.nextId();
        Product product = accout.createProduct(id, /*...*/);
        productRepository.save(product);
        return id;
    }
}
```

이로써 Product 생성 가능 여부를 확인하는 도메인 로직을 변경해도 도메인 영역의 Store만 변경하면 되고 Application 서비스는 영향을 받지 않는다. 또한 도메인 응집도도 높아졌다.

이게 바로 애그리거트를 팩토리로 사용할 때 얻을 수 있는 장점이다.

> 애그리거트가 갖고 있는 데이터를 이용하여 다른 애그리거트를 생성해야 한다면 애그리거트에 팩토리 메서드를 구현하는 것을 고려하는 것이 좋다.
