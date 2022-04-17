---
date: 2022-04-17 15:36:40
layout: post
title: kafka 프로듀서 컨슈머 데이터 중복, 순서 관리
subtitle: 데이터 중복, 순서처리에 대해서
description: 데이터 중복, 순서처리에 대해서
image: https://leejaedoo.github.io/assets/img/kafka.png
optimized_image: https://leejaedoo.github.io/assets/img/kafka.png
category: kafka
tags:
- kafka
- producer
- consumer
  paginate: true
  comments: true
---

# 프로듀서/컨슈머 데이터 유실이나 중복, 순서가 바뀌는 케이스 정리

## 프로듀서

### acks 옵션

<table>
  <thead>
    <tr>
      <th>설정 값</th>
      <th>전송 속도</th>
      <th>데이터 유실 가능 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>acks=1(default)</td>
      <td>중간</td>
      <td>O</td>
    </tr>
    <tr>
      <td>acks=0</td>
      <td>빠름</td>
      <td>O</td>
    </tr>
    <tr>
      <td>acks=-1(all)</td>
      <td>느림</td>
      <td>X</td>
    </tr>
  </tbody>
</table>

### enable.idempotence(멱등성 옵션)

프로듀서에서 브로커로 전송되는 데이터에 기존의 PID뿐만 아니라 `시퀀스 넘버`를 부여하여 이미 한번 브로커로 전송됐던 같은 데이터라도 다른 시퀀스 넘버로 구분함으로써 `데이터의 중복 전송을 방지`할 수 있다.

### 데이터 전송 여부 확인 방식

<table>
  <thead>
    <tr>
      <th>프로듀서 전송 결과 처리 방식</th>
      <th>특징</th>
      <th>데이터 순서 역전 가능 여부</th>  
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>동기 방식</td>
      <td>전송 결과를 기다렸다가 다음 데이터를 전송하기 때문에 처리 속도가 느리다.</td>
      <td>X</td>  
    </tr>
    <tr>
      <td>비동기 방식</td>
      <td>이전 전송 결과를 받기 전에 다음 데이터를 전송하고 이전 데이터의 전송이 실패된 경우에는 데이터 전송 순서가 역전될 수 있다.</td>
      <td>O</td>
    </tr>
  </tbody>
</table>

## 컨슈머

### 컨슈머 운영 방식

<table>
  <thead>
    <tr>
      <th>컨슈머 운영 방법</th>
      <th>특징</th>
      <th>데이터 중복이나 순서 역전 가능 여부</th>  
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>컨슈머 그룹 형태로 운영</td>
      <td>리밸런싱이 발생할 수 있기 때문에 이 과정에서 데이터 유실이나 순서 역전이 발생할 수 있음</td>
      <td>O</td>  
    </tr>
    <tr>
      <td>토픽의 특정 파티션만 구독하는 형태로 운영</td>
      <td>리밸런싱이 발생하지 않기 때문에 최소화할 수 있음</td>
      <td>△</td>
    </tr>
  </tbody>
</table>

### 오프셋 커밋 방식

<table>
  <thead>
    <tr>
      <th>오프셋 커밋 방식</th>
      <th>구현 방법</th>      
      <th>특징</th>
      <th>데이터 유실 가능 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>비명시적 커밋(자동 커밋)</td>
      <td>`enable.auto.commit=true` 와 `auto.commitl.inverval.ms`옵션을 통해 설정</td>
      <td>로그성 데이터 처리 같이 사용자가 오프셋, 파티션 관리를 하지 않아도 되는 경우 사용됨.<br>구현하기는 쉽지만 리밸런싱이나 컨슈머 강제 동료과 같은 상황이 발생했을 경우, 데이터 유실/중복이 발생할 수 있다.</td>
      <td>O</td>  
    </tr>
    <tr>
      <td rowspan="2">명시적 커밋(수동 커밋)</td>
      <td>(poll() 메서드 호출 후)commitSync() 호출</td>
      <td>메시지 처리 완료 때 까지 메시지를 가져온 것으로 간주되면 안되는 경우에 사용
        <br>이전 커밋이 실패하더라도 다음 커밋이 처리되긴 하지만 수동 커밋임으로 이러한 상황이 발생할 경우에 대해 대응하는 코드를 짜둠으로써(ex. consumerRecord.seek() or retry 설정) 처리가 가능한 커밋 방식이다(이전 커밋에 실패했을 때 해당 커밋을 재시도하기 위한 설정이 아님)
        <br>브로커에 커밋 요청 후 응답을 받을 때 까지 기다렸다가 다음 커밋을 진행하기 때문에 속도가 느려 처리량이 줄어든다.</td>
      <td>X</td>  
    </tr>
    <tr>
      <td>(poll() 메서드 호출 후)commitAsync() 호출</td>
      <td>커밋 요청 후 응답 받기 전에 비동기적으로 다음 커밋을 요청한다. 때문에 이전 커밋 요청이 실패했더라도 다음 커밋을 처리하는 과정에서 데이터의 순서 보장이 안되거나 중복 처리가 발생할 수 있다.</td>
      <td>O</td>
    </tr>
  </tbody>
</table>
