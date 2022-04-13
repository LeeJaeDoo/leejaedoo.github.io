---
date: 2021-11-11 20:40:40
layout: post
title: 7. 오류 처리
subtitle: 7. 오류 처리
description: 7. 오류 처리
image: https://leejaedoo.github.io/assets/img/cleancode.png
optimized_image: https://leejaedoo.github.io/assets/img/cleancode.png
categories: cleancode
tags:
- cleancode
paginate: true
comments: true
---

깨끗한 코드는 오류 처리 코드가 좌우한다고 해도 과언이 아니다. 왜냐하면 여기저기 흩어진 오류 처리 코드로 인해 실제 코드 파악이 어려워질 수 있기 때문이다.

## 오류 코드보다 예외를 사용하라

코드가 복잡해지기 때문에 오류가 발생하면 예외를 던져라. 논리가 오류 처리 코드와 뒤섞이지 않기 때문이다.

## Try-Catch-Finally 문부터 작성하라

먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후, 테스트를 통과하게 코드를 작성하는 방법이 좋다.

## Unchecked Exception을 사용하라

이전엔 Checked Exception이 메서드 선언시에 모두 열거되며 사용되었으나 현재는 안정적인 소프트웨어를 제작하는 요소로 Checked Exception이 반드시 필요하지는 않다.<br>
Checked Exception을 사용하게 되면 최상위 함수에서 아래 함수로 내려가는 구조에서 최하위 함수를 변경해 새로운 exception은 던진다고 가정했을 때 이에 대한 exception 처리가 최상위 함수 까지 타고 올라가게 되는 의존성이 생기게 된다.

때로는 checked exception도 아주 중요한 라이브러리를 작성하는 경우에는 필요하다. 하지만 일반적인 애플리케이션은 의존성이라는 문제가 크게 작용한다.

## 호출자를 고려해 exception 클래스를 정의하라

애플리케이션에서 오류를 정의할 때 가장 중요한 것은 `오류를 잡아내는 방법`이다. catch에서 대응하는 다수의 exception 처리가 존재하면서 처리 내용이 동일할 경우 외부에서 한번 감싸준 다음에 exception 처리를 해주는 기법을 사용하면 의존성이 확 줄어든다.<br>
다른 라이버르리로 갈아타기도 수월하고 테스트 코드 짜기도 쉬우지며 코드도 깔끔해진다. 

## null을 반환하지 마라

애초에 null을 반환하지 않도록 코드를 짬으로써 null 체크 로직을 줄이는 것이 좋다.
