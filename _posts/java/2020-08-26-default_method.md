---
date: 2020-08-26 18:40:40
layout: post
title: 자바 8 인 액션
subtitle: 9. 디폴트 메서드
description: 9. 디폴트 메서드
image: https://leejaedoo.github.io/assets/img/java8.png
optimized_image: https://leejaedoo.github.io/assets/img/java8.png
category: java
tags:
  - java
  - 책 정리
paginate: true
comments: true
---
전통적인 자바에서 인터페이스와 관련 메서드는 한 몸처럼 구성된다. 인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 슈퍼클래스의 구현을 상속받아야 한다.<br>
평소에는 큰 문제가 없지만 라이브러리 설계자 입장에서 인터페이스에 새로운 메서드를 추가하는 등 인터페이스를 수정하고 싶을 때는 ``
# 변화하는 API
## API 버전 1
### 사용자 구현
## API 버전 2
### 사용자가 겪는 문제
# 디폴트 메서드란 무엇인가?
# 디폴트 메서드 활용 패턴
## 선택형 메서드
## 동작 다중 상속
### 다중 상속 형식
### 기능이 중복되지 않는 최소의 인터페이스
### 인터페이스 조합
## 해석 규칙
### 알아야 할 세 가지 해결 규칙
### 디폴트 메서드를 제공하는 서브인터페이스가 이긴다
## 충돌 그리고 명시적인 문제 해결
### 충돌 해결
## 다이아몬드 문제
# 요약