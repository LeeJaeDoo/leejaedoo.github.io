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

trigger마다 발생해야할 시각 정보가 규칙적이지 않을 때 사용하기 적절한 방식이다.

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

해당 방식으로 trigger를 생성하게 되면 QRTZ_SIMPLE_TRIGGERS, QRTZ_TRIGGERS 테이블에 일회성 trigger가 아니라면 trigger 발생 개수 만큼 insert query가 날아가게 된다.

```sql
INSERT INTO QRTZ_SIMPLE_TRIGGERS (SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP, REPEAT_COUNT, REPEAT_INTERNAL, TIMES_TRIGGERED) 
VALUES (?);
INSERT INTO QRTZ_TRIGGERS (SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP, JOB_NAME, JOB_GROUP, DESCRIPTION, NEXT_FIRE_TIME, PREV_FIRE_TIME, TRIGGER_STATE, TRIGGER_TYPE, START_TIME, END_TIME, CALENDAR_NAME, MISFIRE_INSTR, JOB_DATA, PRIORITY) 
VALUES (?);
```

생성한 trigger 만큼 trigger 데이터가 쌓이기 때문에 QRTZ_SIMPLE_TRIGGERS.REPEAT_COUNT, QRTZ_SIMPLE_TRIGGERS.REPEAT_INTERVAL, QRTZ_SIMPLE_TRIGGERS.TIMES_TRIGGERED 값은 0으로 생성된다.
그리고 QRTZ_TRIGGERS.START_TIME, QRTZ_TRIGGERS.END_TIME에 각 trigger 마다 startAt, endAt 필드로 설정한 시각 정보가 생성된다.

따라서 trigger 개수 만큼 insert query를 실행함으로 인해 생성해야할 trigger 개수가 많다면 성능 이슈가 발생할 수 있다.

> trigger가 발생할 시각이 일정하다면 아래 2번 방식으로 생성하는 것이 효율적이다.

#### 2번

일정한 주기마다 반복적으로 실행해야할 trigger 에 적절한 방식이다.

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

```sql
INSERT INTO QRTZ_SIMPLE_TRIGGERS (SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP, REPEAT_COUNT, REPEAT_INTERNAL, TIMES_TRIGGERED) 
VALUES (?);
INSERT INTO QRTZ_TRIGGERS (SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP, JOB_NAME, JOB_GROUP, DESCRIPTION, NEXT_FIRE_TIME, PREV_FIRE_TIME, TRIGGER_STATE, TRIGGER_TYPE, START_TIME, END_TIME, CALENDAR_NAME, MISFIRE_INSTR, JOB_DATA, PRIORITY) 
VALUES (?);
```

QRTZ_SIMPLE_TRIGGERS.REPEAT_COUNT, QRTZ_SIMPLE_TRIGGERS.REPEAT_INTERVAL 값은 SimpleScheduleBuilder 인스턴스를 통해 생성할 때 설정한 값 그대로 생성되고, QRTZ_SIMPLE_TRIGGERS.TIMES_TRIGGERED 값은 0으로 초기값은 생성되고 이후 trigger가 실행될 때 마다 1씩 증가된다.
해당 방식은 위 query가 한번씩만 실행되기 때문에 1번 방식에서 우려될 수 있는 성능 이슈는 문제 없다.
