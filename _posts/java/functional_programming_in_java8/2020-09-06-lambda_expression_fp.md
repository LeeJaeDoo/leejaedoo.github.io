---
date: 2020-09-09 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 1. 헬로, 람다 표현식
description: 1. 헬로, 람다 표현식
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
categories: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---
# 함수형 스타일 코드의 큰 이점
#### 변수의 명시적인 변경이나 재할당의 문제를 피할 수 있다. 
명령형 코드에서는 반복문을 통한 처리로 변수명의 명시적인 선언이 필요하지만, 항수형 버전에서는 코드 안에 명시적인 변수 변경이 없다.

> 변수에 대한 변경이 적다는 것은 즉, 코드 안에서 발생할 수 있는 오류의 확률이 더 낮아진다는 의미.

#### 한수형 버전에서는 쉽게 병렬화가 가능하다.
명령형 코드에서는 병렬화를 적용하기 위해서 고민해야 할 부분(ex. thread-safe 등)이 많지만 함수형 코드에서는 병렬화 적용이 쉽다.

#### 서술적인 코드 작성이 가능하다.
예로 스트림의 map(), filter() 함수 등을 통해 가독성 높은 코드 작성이 가능하다.

#### 더 간결하다
단 몇 줄의 코드만으로 명령형 코드와 같은 결과를 얻을 수 있다. **간결한 코드는 작성해야 할 코드의 양 자체를 줄일 수 있으며, 이해해야 하는 코드의 양도 줄고 결국 수정할 코드도 준다는 장점**이 있다.

#### 더 직관적이다.
함수형 코드는 사람이 문제를 설명하는 방식대로 코드를 작성하면 된다.

