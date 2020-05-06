---
date: 2020-05-02 17:40:40
layout: post
title: 자바 8 인 액션
subtitle: 5. 스트림 활용
description: 5. 스트림 활용
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
## 필터링과 슬라이싱
프레디케이트 필터링, 고유 요소 필터링, 스트림의 일부 요소를 무시하거나 스트림을 주어진 크기로 축소하는 방법을 설명한다.
### 필터링(filter)
#### Predicate로 필터링
```java
List<Dish> vegetarianMenu = menu.stream()
                               `.filter(Dish::isVegetarian)`
                                .collect(toList());
```
