---
date: 2022-11-20 10:36:40
layout: post
title: kafka 설계 시 고려해야할 점(2)
subtitle: 카프카 consumer offset reset 전략
description: 카프카 consumer offset reset 전략에 따른 commit 동작 방식
image: https://leejaedoo.github.io/assets/img/kafka.png
optimized_image: https://leejaedoo.github.io/assets/img/kafka.png
category: kafka
paginate: true
comments: true
tags:
- kafka
- consumer
- spring
- listener
- commit
- offset commit
---

# offset 처리 방식

consumer에서 kafka broker topic내 할당된 partition으로부터 record 데이터를 가져와 처리한다. 이 때, consumer는 kafka broker내 partition으로부터 데이터를 어디까지 가져갔는지에 대한 정보를 kafka broker로 commit함으로써 그 정보를 기록하는데 이로인해 특정 topic의 partition을 어떤 consumer가 몇 번째 데이터를 가져갔는지 알 수 있게 된다. 이를 `offset commit`이라고 한다.

이 때 정상적으로 offset commit이 이뤄지지 않는다면 consumer가 partition으로부터 처리할 데이터를 가져올때 중복 처리되거나 누락이 될 여지가 있을 수 있다.

> consumer는 해당 consumer가 처리 완료한 데이터의 offset정보를 broker로 전달, commit한다. 

따라서, offset commit 방식에 따라 consumer에서의 데이터 처리가 달라지기 때문에 kafka에서는 여러가지 방법으로 이를 사용자가 설정할 수 있게 해두었다.

consumer offset commit 방식은 [kafka 컨슈머 정리-컨슈머 주요 옵션](https://leejaedoo.github.io/consumer/#%EC%BB%A8%EC%8A%88%EB%A8%B8-%EC%A3%BC%EC%9A%94-%EC%98%B5%EC%85%98)와 [kafka 컨슈머 데이터 중복, 순서 관리-오프셋 커밋 방식](https://leejaedoo.github.io/producer_consumer/#%EC%98%A4%ED%94%84%EC%85%8B-%EC%BB%A4%EB%B0%8B-%EB%B0%A9%EC%8B%9D) 에도 정리해두었지만 다시 한번 간략히 정리해보려한다.

## 비명시적 offset commit(자동 커밋)

> enable.auto.commit=true(default)

kafka consumer 설정 시에 위와 같이 처리함은 consumer에서의 offset commit처리 방식을 `비명시적`으로 처리함을 의미한다.

이는, `auto.commit.interval.ms`와 함께 설정돼야하며 `auto.commit.interval.ms`에 설정된 시간 간격에 따라 `자동으로 offset commit을 수행`하겠다는 의미이다.

이렇게 세팅하게되면 자동으로 offset commit이 동작하기 때문에 코드 상에 사용자가 별도의 offset commit 코드를 처리하지 않아도 된다.

반면에 자동으로 동작하기 때문에 partition으로 부터 데이터를 가져온 후 특정 consumer가 종료되거나 rebalancing이 발생하게 된다면 데이터 유실 혹은 중복 이슈가 발생할 수 있는 설정 방법이다. 그러므로 kafka messeaging 전략을 Exactly once로 가져가는 경우에는 사용해서는 안된다.

## 명시적 offset commit(수동 커밋)

> enable.auto.commit=false

수동 커밋 방식으로 간다면 이제 사용자가 결정해야할 몇 가지 사항들이 추가된다.

### 동기/비동기 처리

이에 대한 내용도 [kafka 컨슈머 정리-오프셋 정리](https://leejaedoo.github.io/consumer/#%EC%98%A4%ED%94%84%EC%85%8B-%EC%BB%A4%EB%B0%8B)에 정리해두었다.

수동으로 처리하게 되면 말그대로 사용자가 코드 상으로 commit 처리를 해줘야 하는데 이때 commit 처리를 동기방식으로 할지 비동기 방식으로 할지 결정할 수 있다.

> 원하는 시점 위치에 commitSync/Async() 메서드 선언을 통해 commit 시점을 지정할 수 있다.

### 읽어들일 offset 처리 순서 설정

이에 대한 내용도 [kafka 컨슈머 정리-컨슈머 주요 옵션](https://leejaedoo.github.io/consumer/#%EC%BB%A8%EC%8A%88%EB%A8%B8-%EC%A3%BC%EC%9A%94-%EC%98%B5%EC%85%98)에 정리해두었다.

`auto-offset-reset` 옵션값을 설정함으로써 topic에 붙은 consumer의 offset정보가 존재하지 않을 경우(ex. consumer 배포, consumer rebalancing..) 동작방식을 설정할 수 있다.

다만, auto-offset-reset 옵션 값에 따라 consumer를 재기동 or 배포를 하였을 때의 offset commit 동작방식을 조금 더 구체적으로 적어보려한다.

#### latest

말 그대로 가장 최신 오프셋부터 읽어오기 시작한다.

다만, 무조건 최신 오프셋 부터 읽어오기 때문에 consumer가 재기동되기 전에 producing된 데이터는 유실이 발생할 수 있다.

> consumer listener parameter로 Consumer interface에서 `Consumer.seekToEnd()`를 활용하면 코드 상으로 offset reset 옵션을 latest로 가져갈 수 있다.

#### ealiest

말 그대로 가장 오래전에 넣은 오프셋부터 읽기 시작한다.

여기서 처음에 오해한건, 그럼 해당 옵션값으로 세팅된 consumer는 무조건 재기동 된 이후에는 partition 내 가장 오래된 offset부터 다시 데이터를 읽어오게 되는 건가? 그럼 멱등성이 보장된 consumer가 아닌 경우에는 치명적인 오류(데이터 중복)이 발생하는게 아닌가 하는 의문이 들었다.

하지만 실제 동작방식은 조금 달랐다. 조건이 붙었다. 이전 offset commit되지 않았다는 조건이 붙어야 한다.

즉, offset commit 실패 후 다음 순서의 offset commit이 성공했다면 적용되지 않는다.

> consumer listener parameter로 Consumer interface에서 `Consumer.seekToBeginning()`를 활용하면 코드 상으로 offset reset 옵션을 earliest로 가져갈 수 있다.

#### Consumer.seek()

특정 offset commit에 실패했을 경우 위 메서드를 활용하여 특정 offset을 다시 탐색하여 처리하도록 코드상으로 구현할 수 있다. 하지만 custom consumer listener를 통해 errorHandler처리를 하면서 exception발생으로 인한 retry 정책을 가져갈 수 있기 때문에 개인적인 생각으로는 활용될 케이스는 없을 듯 하다.

### AcksMode(커밋 동작 방식)

이에 대한 내용도 [spring kafka 사용법-AcksMode 종류](https://leejaedoo.github.io/spring_kafka/#acksmode-%EC%A2%85%EB%A5%98)에 간략히 정리해두었다.

spring에서 설정할 수 있는 offset commit 동작방식으로 7가지의 commit 방식을 세분화하여 사용자에 입맛에 맞게 세팅할 수 있다.

여기서 추가로 정리하자면 spring에서 제공하는 `AcksMode를 활용하기 위해`서는 consumer listener parameter로 Acknowledgment를 활용하여 수동 커밋 처리를 구현해야 한다.

> ack.acknowledge();




