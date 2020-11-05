---
date: 2020-11-06 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 5. 리포지터리의 조회 기능(JPA 중심)
description: 5. 리포지터리의 조회 기능(JPA 중심)
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
category: ddd
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
# JPA를 위한 스펙 구현
## JPA 스펙 구현
## AND/OR 스펙 조합을 위한 구현
## 스펙을 사용하는 JPA 리포지터리 구현
# 정렬 구현
# 페이징과 개수 구하기 구현
# 조회 전용 기능 구현
## 동적 인스턴스 생성
## 하이버네이트 @Subselect 사용