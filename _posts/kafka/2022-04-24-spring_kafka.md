---
date: 2022-04-24 15:36:40
layout: post
title: spring kafka 사용법
subtitle: spring kafka 라이브러리 사용 방식
description: spring에서 kafka를 사용하는 방식에 대해
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
# Spring에서 사용하는 kafka

> 스프링 카프카 라이브러리는 어드민, 컨슈머, 프로듀서, 스트림즈 기능을 제공한다.

## Spring kafka producer

> 스프링 프로듀서는 kafkaTemplate 클래스를 사용하여 데이터를 전송한다.

카프카 템플릿을 사용하는 방법은 크게 두가지로, 스프링 카프카에서 제공하는 기본 카프카 템플릿을 사용하는 방법과 직접 사용자가 카프카 템플릿을 프로듀서 팩토리로 생성하는 방법이 있다.

### 기본 카프카 템플릿

> 기본 프로듀서 팩토리를 통해 생성된 카프카 템플릿을 그대로 사용하는 방법이다.

```java

@RequiredArgsConstructor
public class KafkaService {

    private final KafkaTemplate<String, String> kafkaTemplate;

    @Async
    public void sendMessage(String topicName, String message) {

        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topicName, message);

        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() { //  비동기 형태로 브로커로 전송된 메시지의 정상적인 적재 여부를 파악한다.

            @Override
            public void onSuccess(SendResult<String, String> result) {}

            public void onFailure(Throwable ex) {
                log.error("Unable to topic : {}, send message : {} exception : {}", topicName, message, ex.getMessage());
            }
        });
    }
}
```

#### KafkaTemplate의 send 메서드 종류

- send(String topic, K key, V data) : 메시지 키, 메시지 값을 포함하여 특정 토픽으로 전달
- send(String topic, Integer partition, K key, V data) : 메시지 키, 메시지 값이 포함된 레코드를 특정 토픽의 `특정 파티션`으로 전달
- send(String topic, Integer partition, Long timestamp, K key, V data) : 메시지 키, 메시지 값, `타임스탬프가 포함된 레코드`를 특정 토픽의 특정 파티션으로 전달
- send(ProducerRecord<K, V> record) : 프로듀서 레코드(ProducerRecord) 객체를 전송. KafkaProducer가 제공하는 send(ProducerRecord<K, V> record) 메서드와 동일

### 커스텀 카프카 템플릿

> 프로듀서 팩토리를 통해 만든 카프카 템플릿 객체를 빈으로 등록하여 사용하는 방법이다.

하나의 스프링 카프카 애플리케이션 내부에 다양한 종류의 카프카 프로듀서 인스턴스를 활용하고자 할 때 사용될 수 있는 방법이다.

예를 들어, A 클러스터로 전송하는 카프카 프로듀서와 B 클러스터로 전송하는 카프카 프로듀서를 동시에 사용하고자 할 때 커스텀 카프카 템플릿을 사용하여 2개의 카프카 템플릿을 빈으로 등록하여 사용할 수 있다.

```java
@Configuration
@RequiredArgsConstructor
public class KafkaProducerConfig {
    
    private final KafkaProperties kafkaProperties;

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaProperties.getBootstrapServers());
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.ACKS_CONFIG, "all");
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

}
```

## Spring kafka consumer

> 컨슈머는 크게 레코드/배치 타입으로 나뉘고 7개의 커밋방식을 세분화한 리스너를 구현하여 사용할 수 있다.

레코드 타입의 리스너는 단 1개의 레코드씩 처리되는 반면, 배치 리스너는 한 번에 여러개의 레코드들을 처리할 수 있다.

### 컨슈머 리스너 종류 

<table>
  <thead>
    <tr>
      <th>타입</th>
      <th>리스너 이름</th>
      <th>특징</th>  
      <th>생성 메서드 파라미터</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="8">RECORD</td>
      <td rowspan="2">MessageListener</td>
      <td rowspan="2">1개의 레코드씩 처리</td>
      <td>onMessage(ConsumerRecord< K, V > data)</td>
    </tr>
    <tr>
      <td>onMessage(V data)</td>
    </tr>
    <tr>
      <td rowspan="2">AcknowledgingMessageListener</td>
      <td rowspan="2">1개의 레코드씩 처리되면서<br>수동 커밋 방식</td>
      <td>onMessage(ConsumerRecord< K, V > data, Acknowledgment acknowledgment)</td>
    </tr>
    <tr>
      <td>onMessage(V data, Acknowledgment acknowledgment)</td>
    </tr>
    <tr>
      <td rowspan="2">ConsumerAwareMessageListener</td>
      <td rowspan="2">1개의 레코드씩 처리되면서<br>KafkaConsumer 인스턴스에<br> 직접 접근하여<br> 컨트롤할 수 있는 방식</td>
      <td>onMessage(ConsumerRecord< K, V > data, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td>onMessage(V data, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td rowspan="2">AcknowledgingConsumerAwareMessageListener</td>
      <td rowspan="2">1개의 레코드씩 처리되면서<br>수동 커밋 방식이고<br>KafkaConsumer 인스턴스에<br> 직접 접근하여<br> 컨트롤할 수 있는 방식</td>
      <td>onMessage(ConsumerRecord< K, V > data, Acknowledgment acknowledgment, Consumer<?, ?> consumer)</td>
    </tr>
    <tr>
      <td>onMessage(V data, Acknowledgment acknowledgment, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td rowspan="8">BATCH</td>
      <td rowspan="2">BatchMessageListener</td>
      <td rowspan="2">한 번에 여러개의 레코드들을 처리</td>
      <td>onMessage(ConsumerRecords< K, V > data)</td>
    </tr>
    <tr>
      <td>onMessage(List< V > data)</td>
    </tr>
    <tr>
      <td rowspan="2">BatchAcknowledgingMessageListener</td>
      <td rowspan="2">한 번에 여러개의 레코드들을 처리하면서<br>수동 커밋 방식</td>
      <td>onMessage(ConsumerRecords< K, V > data, Acknowledgment acknowledgment)</td>
    </tr>
    <tr>
      <td>onMessage(List< V > data, Acknowledgment acknowledgment)</td>
    </tr>
    <tr>
      <td rowspan="2">BatchConsumerAwareMessageListener</td>
      <td rowspan="2">KafkaConsumer 인스턴스에<br> 직접 접근하여<br> 컨트롤할 수 있는 방식</td>
      <td>onMessage(ConsumerRecords< K, V > data, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td>onMessage(List< V > data, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td rowspan="2">BatchAcknowledgingConsumerAwareMessageListener</td>
      <td rowspan="2">수동 커밋 방식이면서<br>KafkaConsumer 인스턴스에<br> 직접 접근하여<br> 컨트롤할 수 있는 방식</td>
      <td>onMessage(ConsumerRecords< K, V > data, Acknowledgment acknowledgment, Consumer< K, V > consumer)</td>
    </tr>
    <tr>
      <td>onMessage(List< V > data, Acknowledgment acknowledgment, Consumer< K, V > consumer)</td>
    </tr>
  </tbody>
</table>

<table>
  <thead>
    <tr>
      <th>타입</th>
      <th>리스너 이름</th>
      <th>상세 설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4">RECORD</td>
      <td>MessageListener</td>
      <td>Record 인스턴스 단위로 프로세싱, 오토 커밋 또는<br> 컨슈머 컨테이너의 AckMode를 사용하는 경우</td>
    </tr>
    <tr>
      <td>AcknowledgingMessageListener</td>
      <td>Record 인스턴스 단위로 프로세싱, <br>수동 커밋을 사용하는 경우</td>
    </tr>
    <tr>
      <td>ConsumerAwareMessageListener</td>
      <td>Record 인스턴스 단위로 프로세싱, <br>컨슈머 객체를 활용하고 싶은 경우</td>
    </tr>
    <tr>
      <td>AcknowledgingConsumerAwareMessageListener</td>
      <td>Record 인스턴스 단위로 프로세싱, 수동 커밋을 사용하고 컨슈머 객체를 활용하고 싶은 경우</td>
    </tr>
    <tr>
      <td rowspan="4">BATCH</td>
      <td>BatchMessageListener</td>
      <td>Record 인스턴스 단위로 프로세싱, 오토 커밋 또는<br>컨슈머 컨테이너의 AckMode를 사용하는 경우</td>
    </tr>
    <tr>
      <td>BatchAcknowledgingMessageListener</td>
      <td>Records 인스턴스 단위로 프로세싱, 수동 커밋을 사용하는 경우</td>
    </tr>
    <tr>
      <td>BatchConsumerAwareMessageListener</td>
      <td>>Records 인스턴스 단위로 프로세싱,<br>컨슈머 객체를 활용하고 싶은 경우</td>
    </tr>
    <tr>
      <td>BatchAcknowledgingConsumerAwareMessageListener</td>
      <td>Records 인스턴스 단위로 프로세싱,<br>수동 커밋을 사용하고<br>컨슈머 객체를 활용하고 싶은 경우</td>
    </tr>
  </tbody>
</table>

스프링 카프카에서는 사용자가 활용할 수 있도록 7가지의 커밋 방식을 세분화하여 미리 로직을 구현해두었다. 스프링 카프카에서는 커밋을 `AckMode`라고 부른다.

> 스프링 카프카 컨슈머의 default AckMode는 BATCH이고 컨슈머의 enable.auto.commit 옵션은 false로 저장된다.

#### AcksMode 종류

<table>
  <thead>
    <tr>
      <th>AcksMode</th>
      <th>설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>RECORD</td>
      <td>레코드 단위로 프로세싱 이후 커밋</td>
    </tr>
    <tr>
      <td>BATCH</td>
      <td>poll() 메서드로 호출된 레코드가 모두 처리된 이후 커밋</td>
    </tr>
    <tr>
      <td>TIME</td>
      <td>특정 시간 이후에 커밋<br>이 옵션을 사용할 경우에는<br>시간 간격을 선언하는 `AckTime`옵션을 설정해야 한다.</td>
    </tr>
    <tr>
      <td>COUNT</td>
      <td>특정 개수만큼 레코드가 처리된 이후에 커밋<br>이 옵션을 사용할 경우에는 레코드 개수를 선언하는 `AckCount`옵션을 설정해야 한다.</td>
    </tr>
    <tr>
      <td>COUNT_TIME</td>
      <td>TIME, COUNT 옵션 중 맞는 조건이 하나라도 나올 경우 커밋</td>
    </tr>
    <tr>
      <td>MANUAL</td>
      <td>Acknowledgement.acknowledge() 메서드가 호출되면<br>다음번 poll() 때 커밋을 한다.<br>매번 acknowledge() 메서드를 호출하면 BATCH 옵션과 동일하게 동작한다.
      <br>이 옵션을 사용할 경우 AcknowledgingMessageListener 또는 BatchAcknowledgingMessageListener를 리스너로 사용해야 한다.
      </td>
    </tr>
    <tr>
      <td>MANUAL_IMMEDIATE</td>
      <td>Acknowledgement.acknowledge() 메서드를 호출한 즉시 커밋한다.
        <br>이 옵션을 사용할 경우 AcknowledgingMessageListener 또는 BatchAcknowledgingMessageListener를 리스너로 사용해야 한다.
      </td>
    </tr>
  </tbody>
</table>

### 리스너 생성 방식

리스너는 `기본 리스너 컨테이너를 사용하는 것`과 `컨테이너 팩토리를 사용하여 직접 리스너를 만드는 것` 두 가지 방식이 있다.

#### 기본 리스너 컨테이너

- RECORD yml 설정 예제

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:8080
    listener:
      type: RECORD
```

- 레코드 리스너 선언 예제

```java
@KafkaListener(topics = "test", groupId = "test-group-00")
public void recordListener(ConsumerRecord<String, String> record) {
    log.info(record.toString());
    // 기본적인 리스너선언 방식으로, poll()이 호출되어 가져온 레코드들을 차례대로 개별 레코드의 메시지 값을 파라미터로 받게 된다.
    // 파라미터로 컨슈머 레코드를 받기 때문에 메시지 키, 메시지 값에 대한 처리를 이 메서드 안에서 수행하면 된다.
}

@KafkaListener(topics = "test", groupId = "test-group-01")
public void singleTopicListener(String messageValue) {
    log.info(messageValue);
    // 메시지 값을 파라미터로 받는 리스너
}

@KafkaListener(topics = "test", groupId = "test-group-02", properties = {"max.poll.interval.ms:60000", "auto.offset.reset:earliest"})
public void singleTopicWithPropertiesListener(String messageValue) {
    log.info(messageValue);
    // 별도의 프로퍼티 옵션값을 선언해주고 싶을 때 사용한다.
}

@KafkaListener(topics = "test", groupId = "test-group-03", concurrency = "3")
public void concurrentTopicListener(String messageValue) {
    log.info(messageValue);
    // 2개 이상의 카프카 컨슈머 스레드를 실행하고 싶을 때 concurrency 옵션을 활용할 수 있다.
    // concurrency값 만큼 컨슈머 스레드를 생성하여 병렬처리 한다.
}

@KafkaListener(topicPartitions = {
    @TopicPartition(topic = "test01", partitions = {"0", "1"}),
    @TopicPartition(topic = "test02", partitionOffsets = @PartitionOffset(partition = "0", initialOffset = "3")),
})
public void listenSpecificPartition(ConsumerRecord<String, String> record) {
    log.info(record.toString());
    // 특정 토픽의 특정 파티션만 구독하고 때 `topicPartitions` 파라미터를 사용한다.
    // `PartitionOffset` 어노테치션을 활용하면 특정 파티션의 특정 오프셋까지 지정할 수 있다.
    // 이 경우에는 그룹 아이디에 관계없이 항상 설정한 오프셋의 데이터부터 가져온다.
}
```

- BATCH yml 설정 예제

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:8080
    listener:
      type: BATCH
```

- 배치 리스너 선언 예제

```java
@KafkaListener(topics = "test", groupId = "test-group-00")
public void batchListener(ConsumerRecords<String, String> records) {
    records.forEach(record -> log.info(record.toString()));
    // 컨슈머 레코드의 묶음(ConsumerRecords)을 파라미터로 받는다.
    // 카프카 클라이언트 라이브러리에서 poll() 메서드로 리턴받은 ConsumerRecords를 리턴받아 사용하는 방식과 같다.
}

@KafkaListener(topics = "test", groupId = "test-group-01")
public void singleTopicListener(List<String> list) {
    list.forEach(recordValue -> log.info(recordValue));
    // 메시지 값을 List형태로 받는다.
}

@KafkaListener(topics = "test", groupId = "test-group-02", concurrency = "3")
public void concurrentTopicListener(ConsumerRecords<String, String> records) {
    records.forEach(record -> log.info(record.toString()));
    // 2개 이상의 컨슈머 스레드로 배치 리스너를 운영할 경우에 concurrency 옵션을 함께 선언하여 사용하면 된다.
}
```

- BATCH 컨슈머/BATCH 커밋 리스너 yml 설정 예제

> AckMode도 사용하고 컨슈머도 사용하려면 배치 커밋 리스너(BatchAcknowledgingConsumerAwareMessageListener)를 사용하면 된다.

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:8080
    listener:
      type: BATCH
      ack-mode: MANUAL_IMMEDIATE
```

- 배치 컨슈머/배치 커밋 리스너 선언 예제

```java
@KafkaListener(topics = "test", groupId = "test-group-00")
public void commitListener(ConsumerRecords<String, String> records, Acknowledgment ack) {
    records.forEach(record -> log.info(record.toString()));
    ack.acknowledge();
    // AckMode를 MANUAL또는 MANUAL_IMMEDIATE로 사용할 경우 수동 커밋을 하기 위해 파라미터로 Acknowledgment 인스턴스를 받아야 한다.
    // acknowledge() 메서드를 호출함으로써 커밋을 수행할 수 있다.
}

@KafkaListener(topics = "test", groupId = "test-group-01")
public void consumerCommitTopicListener(ConsumerRecords<String, String> records, Consumer<String, String> consumer) {
    records.forEach(record -> log.info(record.toString()));
    consumer.commitAsync();
    // 동기/비동기 커밋을 사용하고 싶다면 컨슈머 인스턴스를 파라미터로 받아서 사용하면 된다.
    // consumer 인스턴스의 commitSync(), commitAsync() 메서드를 호출하면 사용자가 원하는 타이밍에 커밋할 수 있도록 로직을 추가할 수 있다.
    // 다만 리스너가 커밋을 하지 않도록 AckMode는 MANUAL, MANUAL_IMMEDIATE로 설정해야 한다.
}
```

#### 커스텀 리스너 컨테이너

> 커스텀 프로듀서와 마찬가지로 서로 다른 설정을 가진 2개 이상의 리스너를 구현하거나 리밸런스 리스너를 구현하기 위해 사용된다.

```java

@Configuration
public class ListenerContainerConfiguration {
    
    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> customContainerFactory() {
        
        Map<String, Object> props = new HashMap<>();
        props.put(Consumerconfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:8080");
        props.put(Consumerconfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(Consumerconfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        
        DefaultKafkaConsumerFactory cf = new DefaultKafkaConsumerFactory<>(props);
        
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();  //  리스너 컨테이너를 만들기 위해 사용
        factory.getContainerProperties().setConsumerRebalanceListener(new ConsumerAwareRebalanceListener() {
            // 리밸런스 리스너를 선언하기 위해 setConsumerRebalanceListener 메서드를 호출한다.
            
            @Override
            public void onPartitionsRevokedBeforeCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {
                // 커밋이 되기 전 리밸런스가 발생했을 때 호출되는 메서드
            }
            
            @Override
            public void onPartitionsRevokedAfterCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {
                // 커밋이 일어난 이후 리밸런스가 발생했을 때 호출되는 메서드
            }
            
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                // 리밸런싱이 끝나서 파티션 소유권이 할당되고 나면 호출되는 메서드
            }
            
            @Override
            public void onPartitionsLost(Collection<TopicPartition> partitions) {
                
            }
            
        });
        
        factory.setBatchListener(false);
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.RECORD);
        factory.setConsumerFactory(cf);
        
        return factory;
    }
    
}

```

```java
@KafkaListener(topics = "test", groupId = "test-group", containerFactory = "customContainerFactory")
public void customListener(String data) {
    log.info(data);
    // customContainerFactory 옵션을 커스텀 컨테이너 팩토리로 설정하여 사용한다.
}
```
