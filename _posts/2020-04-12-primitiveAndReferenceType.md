---
date: 2020-04-12 17:24:49
layout: post
title: 원시타입과 참조타입 & boxing과 unboxing
subtitle: 원시타입과 참조타입 & boxing과 unboxing
description: 원시타입(primitive type)과 참조타입(reference type or wrapper class) 그리고 & boxing과 unboxing
image: https://leejaedoo.github.io/assets/img/java.jpg
optimized_image: https://leejaedoo.github.io/assets/img/java.jpg
category: java
tags:
  - java
  - 개념
paginate: true
comments: true
---

# 기본형(원시타입, primitive type)과 포장 클래스(참조타입, wrapper class)
- 원시타입 : int, double, boolean, float, char, short, byte
- 참조타입 : 원시타입을 제외한 나머지 (Integer, Double, Character...)
## 원시타입과 참조타입의 차이
- NULL을 담을 수 있는가
    ```
        int i = null;                    // 불가능
        Integer integer = null;          // 가능
    ```
- 제네릭을 사용할 수 있는가
    ```
        List<int> i;                    // 불가능
        List<Integer> integer;          // 가능
    ```
    
### 접근 속도
- 원시타입
    - `stack 메모리`에 저장됨.
- 참조타입
    - 하나의 객체이기 때문에 stack 메모리에는 `참조값`만 있고 실제 값은 `heap 메모리`에 저장 됨. 따라서 **값이 필요 할 때 마다 언박싱 과정을 거쳐야 하기 때문에 접근 속도가 느려짐**.
### 메모리 양
차지하는 메모리 양도 참조 타입이 원시 타입보다 많은 메모리가 사용됨.

> 성능과 메모리를 고려해야 한다면 원시타입, null과 제네릭을 다뤄야 한다면 참조타입을 사용
    
# 박싱(boxing)과 언박싱(unboxing)
## 박싱
원시타입을 참조타입으로 변환시키는 것.
## 언박싱
참조타입을 원시타입으로 변환시키는 것.


# 참조
- https://siyoon210.tistory.com/139