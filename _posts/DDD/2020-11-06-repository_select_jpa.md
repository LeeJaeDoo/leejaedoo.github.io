---
date: 2020-11-06 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 5. 리포지터리의 조회 기능(JPA 중심)
description: 5. 리포지터리의 조회 기능(JPA 중심)
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
categories: ddd
tags:
  - ddd
  - 책 정리
paginate: true
comments: true
---
# 검색을 위한 스펙
검색 조건이 다양해지면 더 이상 find 메서드로는 정의할 수 없다. 이 때, 스펙(Specification)을 이용하여 문제를 해결할 수 있다.

스펙(Specification)은 애그리거트가 특정 조건을 충족하는지 여부를 검사한다.

* Order 애그리거트 객체가 특정 고객의 주문인지 확인하는 스펙 예제

```java
public interface Specification<T> {
    public boolean isSatisfiedBy(T agg);
}

public class OrdererSpec implements Specification<Order> {
    private String ordererId;
    public OrdererSpec(String ordererId) {
        this.overerId = ordererId;
    }

    @Override
    public boolean isSatisfiedBy(Order agg) {
        return agg.getOrdererId().getMemberId().getId().equals(ordererId);
    }
}
```

isSatisfiedBy() 메서드의 agg 파라미터는 검사 대상이 되는 애그리거트 객체이다. 해당 메서드는 검사 객체가 조건을 충족하면 true를 리턴한다.

리포지터리는 스펙을 전달받아 애그리거트를 걸러내는 용도로 사용한다. 만약 리포지터리가 메모리에 모든 애그리거트를 보관하고 있다면 다음과 같이 스펙을 사용할 수 있다.

```java
public class MemoryOrderRepository implements OrderRepository {
    public List<Order> findAll(Specification spec) {
        List<Order> allOrders = findAll();
        return allOrders.stream().filter(order -> spec.isSatisfiedBy(order)).collect(toList());
    }
}
```

특정 조건을 충족하는 애그리거트를 찾으려면 원하는 스펙을 생성해서 리포지터리에 전달해 주기만 하면 된다.

```java
Specification<Order> ordererSpec = new OrdererSpec("madvirus");
List<Order> orders = orderRepository.findAll(ordererSpec);
```

## 스펙 조합
스펙은 조합을 통해 더 복잡한 스펙도 만들 수 있다.

* AND 조합 예제

```java
public class AndSpec<T> implements Specification<T> {
    private List<Specification<T>> specs;

    public AndSpecification(Specification<T> ... specs) {
        this.specs = Arrays.asList(specs);
    }

    public boolean isSatisfiedBy(T agg) {
        for (Specification<T> spec : specs) {
            if (!spec.isSatisfiedBy(agg))   return false;
        }

        /*if (specs.stream().allMatch(spec -> spec.isSatisfiedBy(agg)))   return true;
        else    return false;*/
    }
}
```

AndSpec을 이용하면 아래와 같이 여러 스펙을 하나의 스펙으로 만들어 리포지터리에 전달할 수 있다.

```java
Specification<Order> ordererSpec = new OrdererSpec("madVirus");
Specification<Order> orderDateSpec = new OrderDateSpec(fromDate, toDate);
AndSpec<T> spec = new AndSpec(ordererSpec, orderDateSpec);
List<Order> orders = orderRepository.findAll(spec);
```

OrSpec도 비슷하게 구현할 수 있다.

# JPA를 위한 스펙 구현
하지만 위와 같이 모든 애그리거트를 조회한 후 스펙으로 걸러내는 방식으로 조회하게 되면 실행 속도에 문제가 있을 수 있다. 애그리거트가 10만 개인 경우 10만 개 데이터를 DB에서 메모리로 로딩한 뒤에 다시 10만 개 객체를 루프 돌면서 스펙을 검사하게 되는데, 이는 시스템 성능을 느리게 하는 원인이 된다.

실제 구현에서는 쿼리의 where 절에 조건을 붙여 필요한 데이터를 걸러야 한다. 이는 즉, 스펙 구현도 메모리에서 걸러내는 방식이 아닌 쿼리의 where를 사용하는 방식으로 변경해야 함을 의미한다.

JPA에서는 다양한 검색 조건을 조합하기 위해 CriteriaBuilder, Predicate를 사용하므로 이를 스펙에 대입하여 구현할 수 있다. 
## JPA 스펙 구현

* JPA를 사용하는 리포지터리를 위한 스펙 인터페이스

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

public interface Specification<T> {
    Predicate toPredicate(Root<T> root, CriteriaBuilder cb);
}
```

* 스펙 구현 예제

```java
import com.myshop.common.jpaspec.Specification;
import com.myshop.member.domain.MemberId_;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

public class OrdererSpec implements Specification<Order> {
    private String ordererId;

    public OrdererSpec(String ordererId) {
        this.ordererId = ordererId;
    }

    @Override
    public Predicate toPredicate(Root<Order> root, CriteriaBuilder cb) {
        return cb.equal(root.get(Order_.orderer)
                            .get(Ordrer_.memberId).get(MemberId_.id), ordereId);
    }
}

Specification<Order> ordererSpec = new OrdererSpec("madVirus");
List<Order> orders = orderRepository.findAll(ordererSpec);
```

toPredicate()메서드는 Order의 orderer.memberId.id 프로퍼티가 생성자로 전달받은 ordererId와 같은지 비교하는 Predicate를 생성해서 리턴한다.

application 서비스는 원하는 스펙을 생성하고 리포지터리에 전달해서 필요한 애그리거트를 검색하면 된다.

또한, Specification 구현 클래스를 개별적으로 만들지 않고 별도 클래스에 스펙 생성 기능을 모아도 된다.

* 스펙 생성 기능을 별도 클래스로 생성한 예제

```java
import com.myshop.common.jpaspec.Specification;
import com.myshop.member.domain.MemberId_;

import java.util.Date;

public class OrderSpecs {
    public static Specification<Order> orderer(String ordererId) {
        return (root, cb) -> cb.equal(
            root.get(Order_.orderer).get(Orderer_.memberId).get(MemberId_.id), ordererId
        );
    }

    public static Specification<Order> between(Date from, Date to) {
        return (root, cb) -> cb.between(root.get(Order_.orderDate), from, to);
    }
}

Specification<Order> betweenSpec = OrderSpecs.between(fromTime, toTime);
```

## AND/OR 스펙 조합을 위한 구현

* AND를 위한 JPA 스펙 예제

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.Arrays;
import java.util.List;

public class AndSpecification<T> implements Specification<T> {
    private List<Specification<T>> specs;

    public AndSpecification(Specification<T> ... specs) {
        this.specs = Arrays.asList(specs);
    }
    public Predicate toPredicate(Root<T> root, CriteriaBuilder cb) {
        Pridicate[] predicates = specs.stream()
                                      .map(spec -> spec.toPredicate(root, cb))
                                      .toArray(size -> new Predicate[size]);
        return cb.and(predicates);    
    }
}
```

* OR를 위한 JPA 스펙

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.Arrays;
import java.util.List;

public class OrSpecification<T> implements Specification<T> {
    private List<Specification<T>> specs;

    public OrSpecification(Specification<T>... specs) {
        this.specs = Arrays.asList(specs);
    }

    @Override
    public Predicate toPredicate(Root<T> root, CriteriaBuilder cb) {
        Predicate[] predicates = specs.stream()
                                      .map(spec -> spec.toPredicate(root, cb))
                                      .toArray(Predicate[]::new);
        return cb.or(predicates);
    }
}
```

AndSpecification과 OrSpecification의 toPredicate() 메서드는 생성자로 전달받은 Specification 목록을 Predicate 목록으로 바꾸고 CriteriaBuilder의 and()와 or()를 사용해서 새로운 Predicate를 생성한다.

두 And와 Or 스펙 생성을 쉽게 하기 위해 아래와 같은 클래스를 구현할 수 도 있다.

* AND/OR 스펙을 생성해주는 팩토리 클래스

```java
public class Specs {
    public static <T> Specification<T> and(Specification<T> ... specs) {
        return new AndSpecification<>(specs);
    }
    public static <T> Specification<T> or(Specification<T> ... specs) {
        return new OrSpecification<>(specs);
    }
}
```

* 팩토리 클래스 구현 예제

```java
Specification<Order> specs = Specs.and(
    OrderSpecs.orderer("madvirus"), OrderSpecs.between(fromTime, toTime)
);
```
## 스펙을 사용하는 JPA 리포지터리 구현
이제 구현한 스펙을 리포지터리에 구현해본다. 먼저 리포지터리 인터페이스에서는 스펙을 사용하는 메서드를 제공해야 한다. 그리고 이를 상속받은 JPA 리포지터리는 아래와 같이 구현한다.

* 스펙을 활용한 JPA 리포지터리 구현

```java
public interface OrderRepository {
    public List<Order> findAll(Specification<Order> spec);
    //...
}

@Repository
public class JpaOrderRepository implements OrderRepository {
    //...
    @Override
    public List<Order> findAll(Specification<Order> spec) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> criteriaQuery = cb.createQuery(Order.class);
        Root<Order> root = criteriaQuery.from(Order.class); //  검색 조건 대상이 되는 루트(Order 클래스) 생성
        Predicate predicate = spec.toPredicate(root, cb);   //  파라미터로 전달받은 스펙을 이용해서 Predicate 생성
        criteriaQuery.where(predicate);                     //  쿼리의 조건으로 생성한 Predicate를 전달
        criteriaQuery.orderBy(
            cb.desc(root.get(Order_.number).get(OrderNo_.number))
        );
        TypedQuery<Order> query = entityManager.createQuery(criteriaQuery);
        return query.getResultList();
    }
}
```

> 도메인 모델은 구현 기술(ex. Criteria, JPA...)에 의존하지 않아야 한다. 하지만 JPA 용 Specification 인터페이스는 toPredicate() 메서드가 JPA의 Root와 CriteriaBuilder에 의존하는 모양을 띄게된다.<br>
> 그렇다면, 도메인이 Specification 구현 기술에 중립적인 형태로 의존하지 않도록 개선해야 할까? 필자는 `아니오`라고 대답하고 있다.<br>
> 왜냐하면, 리포지터리 구현 기술에 의존하지 않기 위해 스펙 구현시 추상화해야 하는데 이는 추상화하는데 드는 노력에 비해 얻는 이점이 적다. `리포지터리 구현 기술을 바꿀 정도의 변화는 드물기 때문`이다.

# 정렬 구현
JPA의 Criteria#orderBy()를 이용해서 정렬 순서를 지정할 수 있다.

정렬 순서를 지정하는 가장 쉬운 방법은 아래와 같이 JPQL에서 사용한 쿼리의 문자열을 이용하는 것이다.

```java
TypedQuery<Order> query = entityManager.createQuery(
        "select o from Order o " +
        "where o.orderer.memberId.id = :ordererId " + 
        "order by o.number.number desc",
    Order.class);

List<Order> orders = orderRepository.findAll(someSpec, "number.number desc");
```

JPA 리포지터리 구현 클래스는 문자열을 파싱해서 JPA Criteria의 Order로 변환하거나 JPQL의 order by 절로 변환하면 된다.

* 문자열로 전달받은 정렬 값을 Order로 변환하는 코드 구현 예제

```java
@Repository
public class JpaOrderRepository implements OrderRepository {
    //...

    @Override
    public List<Order> findAll(Specification<Order> spec, String ... orders) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> criteriaQuery = cb.createQuery(Order.class);
        Root<Order> root = criteriaQuery.from(Order.class);
        Predicate predicate = spec.toPredicate(root, cb);
        criteriaQuery.where(predicate);
        if (orders.length > 0) {
            criteriaQuery.orderBy(JpaQueryUtils.toJpaOrders(root, cb, orders));
        }
        TypedQuery<Order> query = entityManager.createQuery(criteriaQuery);
        return query.getResultList();
    }
}

public class JpaQueryUtils {
    public static <T> List<Order> toJpaOrders(Root<T> root, CriteriaBuilder cb, String ... orders) {
        String[] orderClause = orderStr.split(" ");
        boolean ascending = true;
        if (orderClause.length == 2 && orderClause[1].equalsIgnoreCase("desc")) {
            ascending = false;
        }

        String[] paths = orderClause[0].split("\\.");
        Path<Object> path = root.get(paths[0]);
        for (int i = 1; i < paths.length; i++) {
            path = path.get(paths[i]);
        }
        return ascending ? cb.asc(path) : cb.desc(path);
    }
}
```

# 페이징과 개수 구하기 구현
JPA 쿼리는 페이징 처리를 위해 `setFirstResult()`와 `setMaxResult()` 메서드를 제공하고 있는데 이 두 메서드를 이용해서 페이징을 구현할 수 있다.

* 페이징 처리 구현 예제

```java
@Override
public List<Order> findByOrdererId(String ordererId, int startRow, int fetchSize) {
    TypedQuery<Order> query = entityManager.createQuery(
        "select o from Order o " +
        "where o.orderer.memberId.id = :ordererId " +
        "order by o.number.number desc"), Order.class);
    )

    query.setParameter("ordererId", ordererId);
    query.setFirstResult(startRow); //  읽어올 첫 번째 행 번호를 지정
    query.setMaxResults(fetchSize); //  읽어올 행 개수를 지정
    return query.getResultList();
}

List<Order> orders = findByOrdererId("madvirus", 45, 15);
```

> 위에서 알아본 페이징 처리는 스프링 데이터 JPA라는 모듈을 활용하면 쉽게 구현할 수 있다.

# 조회 전용 기능 구현
리포지터리는 애그리거트의 저장소를 표현하는 것으로서 다음 용도로 리포지터리를 사용하는 것은 적합하지 않다.

* 여러 애그리거트를 조합해서 한 화면에 보여주는 데이터 제공
* 각종 통계 데이터 제공

첫 번째 기능을 애그리거트에서 제공하려고 시도하게 되면 JPA의 지연 로딩과 즉시 로딩 설정, 연관 매핑 설정으로 구현이 어려울 수 있다. 게다가 애그리거트 간에 직접 연관을 맺으면 ID로 참조할 때의 장점을 활용할 수 없게 된다.<br>
두 번째 통계 데이터는 다양한 테이블을 조인하거나 DBMS 전용 기능을 사용해야 구할 수 있는데, 이는 JPQL이나 Criteria로 처리하기 어렵다.

애초에 위와 같은 기능은 조회 전용 쿼리로 처리해야 한다. JPA와 하이버네이트를 사용하면 동적 인스턴스 생성, 하이버네이트의 @Subselect 확장 기능, 네이티브 쿼리를 이용해서 조회 전용 기능을 구현할 수 있다.
## 동적 인스턴스 생성
JPA는 쿼리 결과에서 임의의 객체를 동적으로 생성할 수 있는 기능을 제공하고 있다.

* JPQL에서 동적 인스턴스를 사용한 코드 예제

```java
@Repository
public class JpaOrderViewDao implements OrderViewDao {
    //...
    @Override
    public List<OrderView> selectByOrderer(String ordererId) {
        String selectQuery =
            "select new com.myshop.order.application.dto.OrderView(o, m, p) " +
            "from Order o join o.orderLines ol, Member m, Product p " +
            "where o.orderer.memberId.id = :ordererId " +
            "and o.orderer.memberId = m.id " +
            "and index(ol) = 0 " +
            "and ol.productId = p.id " +
            "order by o.number.number desc";
        TypedQuery<OrderView> query =
            em.createQuery(selectQuery, OrderView.class);
        query.setParameter("ordererId", ordererId);
        return query.getResultList();
    }
}
```

위 코드의 select 절에는 new 키워드를 통해 생성할 인스턴스의 완전한 클래스 이름을 지정하고 괄호 안에 생성자에 인자로 전달할 값을 지정한다.(ex. OrderView)<br>
이 처럼, 조회 전용 모델을 만드는 이유는 표현 영역을 통해 사용자에게 데이터를 보여주기 위함이다. 
## 하이버네이트 @Subselect 사용
하이버네이트는 JPA 확장 기능으로 @Subselect를 제공한다. @Subselect는 쿼리 결과를 @Entity로 매핑할 수 있는 유용한 기능으로 아래와 같이 구현할 수 있다.

* @Subselect를 이용해서 @Entity를 매핑한 예제

```java
import org.hibernate.annotations.Immutable;
import org.hibernate.annotations.Subselect;
import org.hibernate.annotations.Synchronize;

import javax.persistence.*;
import java.util.Date;

@Entity
@Immutable
@Subselect("select o.order_number as number, " +
            "o.orderer_id, o.orderer_name, o.total_amounts, " +
            "o.receiver_name, o.state, o.order_date, " +
            "p.product_id, p.name as product_name " +
            "from purchase_order o inner join order_line ol " +
            "   on o.order_number = ol.order_number " +
            "   cross join product p " +
            "where ol.line_idx = 0 and ol.product_id = p.product_id"
)
@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary {
    @Id
    private String number;
    private String ordererId;
    private String ordererName;
    private int totalAmounts;
    private String receiverName;
    private String state;
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "orderDate")
    private Date orderDate;
    private String productId;
    private String ProductName;

    protected OrderSummary() {
        //...
    }

    //.. get메서드
}
```

@Immutable, @Subselect, @Synchronize는 하이버네이트 전용 어노테이션인데, 이를 활용하면 테이블이 아닌 쿼리 결과를 @Entity로 매핑할 수 있다.<br>

@Subselect는 조회(select) 쿼리를 값으로 갖는다. 하이버네이트는 이 select 쿼리의 결과를 매핑할 테이블처럼 사용한다. DBMS가 여러 테이블의 조인된 결과를 한 테이블 처럼 보여주기 위한 용도로 사용하는 뷰와 같이 @Subselect를 사용하면 쿼리 실행 결과를 매핑할 테이블처럼 사용할 수 있다.<br>
뷰를 수정할 수 없듯이, @Subselect로 조회한 @Entity 역시 수정할 수 없다.

@Subselect를 이용한 @Entity의 매핑 필드를 수정하게 되면, 하이버네이트는 변경 내역을 update칠텐데, 매핑되는 테이블이 존재하지 않으므로 에러가 발생한다. 이런 문제를 방지하기 위해 `@Immutable`을 사용하면 하이버네이트는 `해당 엔티티의 매핑 필드/프로퍼티가 변경되어도 DB에 반영하지 않고 무시`한다.

@Synchronize는 아래와 같은 경우에 활용할 수 있는 어노테이션이다.

```java
// purchase_order 테이블에서 조회
Order order = orderRepository.findById(orderNumber);
order.changeShippingInfo(newInfo);  //  상태 변경

//  변경 내역이 DB에 반영되지 않았는데 purchase_order 테이블에서 조회
List<OrderSummary> summaries = ordererSummaryRepository.findByOrdererId(userId);
```

위 코드는 Order의 상태를 변경한 뒤에 OrderSummary를 조회한다. 특별한 이유가 없으면 하이버네이트는 변경사항을 트랜잭션에 커밋하는 시점에 DB에 반영하므로, Order의 변경 내역을 아직 purchase_order 테이블에 반영하지 않은 상태에서 purchase_order 테이블을 사용하는 OrderSummary를 조회하게 된다.<br>
즉, OrderSummary에는 최신 값이 아닌 이전 값이 담기게 된다.

이런 문제를 해결하기 위해 @Synchronize를 활용한다. 하이버네이트는 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 flush를 먼저 한다.<br>
OrderSummary의 @Synchronize는 'purchase_order' 테이블을 지정하고 있으므로 OrderSummary를 로딩하기 전에 purchase_order 테이블에 변경이 발생하면 관련 내역을 먼저 flush하게 된다. 따라서 OrderSummary를 로딩하는 시점에서는 변경 내역이 반영된다.

@Subselect를 사용해도 일반 @Entity와 같기 때문에 EntityManager#find(), JPQL, Criteria를 사용해서ㅑ 조회할 수 있다는 것이 @Subselect의 장점이다. 이는 스펙을 사용할 수 있다는 것도 포함된다.
