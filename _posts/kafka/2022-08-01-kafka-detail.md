---
date: 2022-08-01 13:36:40
layout: post
title: kafka 설계 시 고려해야할 점(1)
subtitle: 카프카 Messaging sementics
description: kafka Messaging sementics
image: https://leejaedoo.github.io/assets/img/kafka.png
optimized_image: https://leejaedoo.github.io/assets/img/kafka.png
category: kafka
tags:
- kafka
- producer
- consumer
- spring
- listener
- commit
paginate: true
comments: true
---

# 카프카 Messaging sementics
<table>
  <thead>
    <tr>
      <th>정책</th>
      <th>메시지 중복 허용</th>
      <th>메시지 유실 허용</th>
      <th>재전송 허용 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>최대 한번(At Most once)</td>
      <td>X</td>
      <td>O</td>
      <td>X</td>
    </tr>
    <tr>
      <td>적어도 한번(At Least once)</td>
      <td>O</td>
      <td>X</td>
      <td>O</td> 
    </tr>
    <tr>
      <td>정확히 한번(Exactly once)</td>
      <td>X</td>
      <td>X</td>
      <td>O</td>
    </tr>
  </tbody>
</table>



## At-Most-Once

> 실패나 타임아웃 등이 발생하면 메시지를 버릴 수 있다. 데이터가 일부 누락되더라도 영향이 없는 경우엔 대량처리 및 짧은 주기의 전송 서비스에 유용할 수 있다.

보통 자동 커밋 방식이 사용된다.

## At-Least-Once

> 메시지가 최소 1번 이상 전달되는 것을 보장한다. 실패나 타임아웃 등이 발생하면 메시지를 다시 전송하며, 이 경우엔 동일한 메시지가 중복으로 처리될 수 있다.

### 사례

#### Producer
> Producer-Broker 사이의 ack 소실

Producer는 Broker에 메시지를 전송하고 ack를 수신받는다.
만약 네트워크 상에서 ack가 소실/지연되어 수신받는데에 실패할 경우, Producer는 메시지 전송이 실패했다고 판단하여 재전송하게 된다.
즉, 동일한 메시지가 중복 전송될 수 있다.

#### Consumer
> offset 갱신 실패

Consumer가 메시지를 읽고 DB에 저장한 후에 offset을 갱신하기 전에 장애가 발생할 경우, Consumer는 재시작되었을 때 갱신되지 않은 offset을 기준으로 메시지를 읽어오게 된다.
즉, 이미 DB에 저장된 메시지를 중복으로 가져오게 된다.

ex) Spark Streaming의 Receiver 기반 모델에서 WAL을 사용하는 경우

### 해결방안
consumer는 결국 중복으로 subscribe된 메세지에 대한 처리를 실제 로직에서는 중복 방지될 수 있게, 즉 멱등성이 보장될 수 있도록 구현해야 한다.

1. 임의의 event-id를 기준으로 동일한 message에 대해서 insert는 skip
2. update 처리인 경우는 메세지 순서를 고려해야 하기 때문에 producer에서 메세지 publish시점 정보를 담은 publish_at 정보를 활용, 순서를 검증할 수 있다.
3. consumer offset commit 방식은 수동으로 설정하여 로직이 성공했을 떄 수동으로 offset commit을 하는 방식을 적용한다.(ex. commitSync(), acknowledge())
### producer 옵션 설정

#### ack

- ack=1 : Producer가 메시지 전송 후, partition leader로부터 일정시간 ack를 기다린다. 손실 가능성이 적고 적당한 전송 속도를 가지게 된다. ack 전송 직후 partition leader의 Broker가 follower들이 복사해가기 전에 다운되면 해당 메시지는 손실된다.
- ack=all : Producer가 메시지 전송 후, partition의 leader, follower 모두로부터 ack를 기다린다. 손실이 없지만 전송 속도가 느리다.

## Exactly-Once

> 메시지가 정확하게 한 번만 전달되는 것을 보장한다. 손실이나 중복 없이, 순서대로 메시지를 전송하는 것은 구현 난이도가 높고 비용이 많이 든다.

Kafka는 at-least-once 방식을 지원했으나, 0.11.0.0 이상부터는 enable.idempotence 옵션과 트랜잭션을 적용하여 exactly-once를 구현할 수 있다.

- ack=all

### 사례

#### Transaction 적용
> Producer에서 Consumer까지 연결되는 파이프라인 처리를 통한 트랜잭션 구현

Producer가 트랜잭션에서 처리한 데이터의 offset을 커밋함으로써, Consumer에 정확하게 메시지를 전달할 수 있다.

Producer side에서 트랜잭션을 적용하려면 Consumer side에서도 트랜잭션 기반으로 메시지를 읽어야 한다. 즉, Consumer 에도 트랜잭션 API를 적용해야 한다.

- Producer

```java
KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, "data");

producer.initTransactions();  // 트랜잭션 준비

producer.beginTransaction();  // 트랜잭션 시작
producer.send(record);        // 메시지 전송
producer.flush();
producer.commitTransaction(); // 트랜잭션 커밋

producer.close();
```



- Consumer

```java
configs.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);           // 명시적 오프셋 커밋
configs.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");   // 커밋된 메시지만 읽기

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);
consumer.subscribe(Arrays.asList(TOPIC_NAME));

while (true) {
  ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
  for (ConsumerRecord<String, String> record : records) {
    logger.info("{}", record.toString());
  }
  consumer.commitSync();
}
```

> 자세한 건 [exactly-once_transaction_구현](https://dhkdn9192.github.io/apache-kafka/kakfa-exactly-once-delivery/#4-producer-side%EC%9D%98-exactly-once-%EA%B5%AC%ED%98%84) 에서 확인

# 정리
exactly-once가 가장 이상적인 메시지 처리 방식이지만 난이도와 비용으로 인해 `at-least-once로 타협`하는 경우가 보편적이다. Kafka의 경우 at-least-once를 보장하며 일정 버전 이후에서만 옵션을 통해 exactly-once를 적용할 수 있다.

# Reference
[메세지 중복 처리 방안](https://camel-context.tistory.com/54)


