---
date: 2023-04-15 12:36:40
layout: post
title: spring quartz
subtitle: spring quartz의 trigger 동작 방식
description: simple trigger에 대해서
image: https://user-images.githubusercontent.com/15065956/216301590-944e7367-80eb-4e39-9409-70a8fa42daa5.png
optimized_image: https://user-images.githubusercontent.com/15065956/216301590-944e7367-80eb-4e39-9409-70a8fa42daa5.png
category: quartz
tags:
- quartz
- spring
- scheduler
paginate: true
comments: true
---

## Quartz Trigger

Quartz Scheduler는 cron Trigger와 simple Trigger 두 가지 방식으로 동작된다.
cron trigger 방식은 cron 표현식을 활용하여 calender를 기반으로 주기적으로 특정 날짜나 시각 혹은 요일마다 trigger를 발생시키게 되고
simple trigger 방식은 특정 시각마다, 혹은 특정 주기마다 반복적으로 trigger를 발생시키는 방식이다.

## Simple Trigger

simple trigger는 크게 두 가지 동작 방식을 갖는다.

1. trigger마다 발생할 시각(ex. 2023-04-15 22:40:00) 데이터를 갖는 trigger를 원하는 **발생 횟수 만큼 생성**하여 동작시키는 방법,
2. 최초 발생 시각 데이터와 반복 횟수를 갖는 한 개의 trigger만 생성하는 방식

이렇게 두 가지 방식으로 다룰 수 있다.

### trigger 구현체 정의

Trigger interface 생성을 위해 quartz는 TriggerBuilder를 제공하여 Trigger interface를 인스턴스화 할 수 있다.

```java

public class TriggerBuilder<T extends Trigger> {
    
    private TriggerKey key;                             // 해당 trigger의 고유 키 설정
    private String description;                         // 해당 trigger에 대한 설명
    private Date startTime = new Date();                // trigger 시작 시각
    private Date endTime;                               // trigger 종료 시각
    private int priority = Trigger.DEFAULT_PRIORITY;    // trigger 발생 우선 순위(1 ~ 10, default: 5) 
    private String calenderName;                         
    private JobKey jobKey;                              // 해당 trigger가 속해있는 job의 고유 키 설정
    private JobDataMap jobDataMap = new JobDataMap();   // 해당 trigger가 실행될 때 다룰 map타입의 데이터 저장소
    private ScheduleBuilder<?> scheduleBuilder = null;  // 해당 trigger의 동작 스케줄 데이터
    
}

```

### trigger 구현

#### 1번

```
TriggerBuilder.newTrigger()
              .withIdentity(/*trigger 고유 키*/)
              .jorJob(/*해당 trigger를 담을 job 키*/)
              .startAt(/*trigger 발생 시각*/)
              .endAt(/*더 이상 trigger가 발생되지 않을 시각*/)
              .usingJobData(/*해당 trigger 발생 시 다룰 데이터*/)
              .withDescription(/*trigger 설명*/)
              .withPriority(/*다른 trigger와 동시 발생 시 우선순위 설정 */)
              .build();
```

> usingJobData는 Map타입의 JobDataMap 인스턴스를 활용하여 key-value 형태로 생성한다.

#### 2번

```
TriggerBuilder.newTrigger()
              .withIdentity(/*trigger 고유 키*/)
              .jorJob(/*해당 trigger를 담을 job 키*/)
              .withSchedule(/*해당 trigger를 얼마동안의 간격으로 몇 번 반복 실행할 지 설정*/)
              .startAt(/*trigger 발생 시각*/)
              .endAt(/*repeat 수가 남았더라도 더 이상 trigger가 발생되지 않을 시각*/)
              .usingJobData(/*해당 trigger 발생 시 다룰 데이터*/)
              .withDescription(/*trigger 설명*/)
              .build();
```

> withSchedule은 SimpleScheduleBuilder 인스턴스를 활용하여 구현할 수 있다.
