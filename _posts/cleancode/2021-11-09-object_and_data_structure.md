---
date: 2021-11-07 20:40:40
layout: post
title: 6. 객체와 자료구조
subtitle: 6. 객체와 자료구조
description: 6. 객체와 자료구조
image: https://leejaedoo.github.io/assets/img/cleancode.png
optimized_image: https://leejaedoo.github.io/assets/img/cleancode.png
categories: cleancode
tags:
- cleancode
paginate: true
comments: true
---
# 추상화

객체가 포함하는 자료를 표현할 가장 좋은 방법을 고민해야한다.

> 객체 지향 코드에서 어려운 변경은 절차적인 코드에서는 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다

새로운 자료 타입이 필요한 경우 > 객체 지향 기법이 적합
새로운 함수가 필요한 경우 > 절차적인 코드

> 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

# 디미터 법칙

객체는 조회 함수로 내부 구조를 공개하면 안된다.

## 기차 충돌

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

위와 같이 한 줄로 이어진 기차 처럼 보이는 코드를 `기차 충돌`이라 부르는데, 이는 일반적으로 조잡하다고 여겨지는 방식이기 때문에 피해야 한다.<br>
따라서 아래와 같이 바꾸는 것이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 코드대로라면 ctxt 객체는 Options를 포함하고, Options는 ScratchDir을 포함하며, ScratchDir은 AbsolutePath를 포함함을 알 수 있게 되는데 이는 `ctxt, Options, ScratchDir가 자료 구조가 아닌 객체`이기 때문에 내부 구조를 숨겨야 하는 디미터 법칙을 위반한다.<br>
반면, 자료 구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

위와 같은 코드는 디미터 법칙을 거론할 필요가 없다.

## 잡종 구조

객체와 자료 구조가 섞인 구조가 발생할 수 있다. 이런 구조는 양쪽 세상의 단점만 모아놓은 구조기 때문에 지양해야 한다. 개발자가 함수나 타입을 보호할지 공개할지 확신하지 못해 내놓은 설계일 뿐이다.

# 결론

객체는 동작(메서드)을 공개하고 자료(변수)는 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.<br>
자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.

새로운 자료 타입을 추가하는 유연성이 필요 > 객체
새로운 동작을 추가하는 유연성 > 절차적인 코드

