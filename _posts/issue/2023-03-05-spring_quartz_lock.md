---
date: 2023-03-05 12:36:40
layout: post
title: spring quartz 잡 실패 처리
subtitle: spring quartz 잡 실패 처리
description: spring quartz에서 잡 trigger 실행 중 실패되었을 때 동작 방식
image: https://user-images.githubusercontent.com/15065956/216301590-944e7367-80eb-4e39-9409-70a8fa42daa5.png
optimized_image: https://user-images.githubusercontent.com/15065956/216301590-944e7367-80eb-4e39-9409-70a8fa42daa5.png
category: quartz
tags:
- quartz
- spring
- scheduler
- concurrent
paginate: true
comments: true
---

## job 실행 중 Lock이 걸리는 경우

```log
org.quartz.impl.jdbcjobstore.LockException: Failure obtaining db row lock: Lock wait timeout exceeded; try restarting transaction
```

spring quartz job을 실행하여 돌리는 도중 위와 같은 에러 로그를 만날 수 있었다.

처음엔 이것저것 테스트하면서 job이 동시에 실행되는 도중 발생한 동시성 관련 에러로 추측했으나 아니었다.

원인은 job trigger 실패 처리 구현 로직에 문제가 있었다.

## trigger 로직 구현

[Quartz Job 실행 flow](https://leejaedoo.github.io/spring_quartz/#job-%EB%8F%99%EC%9E%91-flow-%EC%88%9C%EC%84%9C)

위 링크를 보면 quartz job을 실행되는 순서를 알 수 있다.

내가 문제가 되었던 로직은 job의 trigger 로직 구현 위치였다.

기존에는 trigger 로직을 TriggerListener 구현체에서 triggerComplete 메서드에 로직을 구현했었다. 이게 문제였다.

triggerComplete 메서드 실행 도중 exception 발생 시 해당 job에 문제가 발생했음을 quartz가 제대로 인식을 하지 못하고 계속 실행 중이라고 인식하면서 위와 같은 Lock wait time out exceeded라는 에러 로그를 만나게 된 것이다.

## JobExecutionException

quartz job이 exception이 발생했음을 인지 할 수 있으려면 listener 구현 로직 안에서 발생되는 exception으로는 인지할 수 없었다.

`JobExecutionException`을 사용해야 했다.

Quartz Job에서 실행 도중 예외가 발생했을 때, 해당 Job의 실행을 중지하고 예외를 처리할 수 있는 기능을 제공하기 때문에 해당 exception을 활용해야 Quartz Job trigger 로직 도중 실패를 인지시킴으로써 위와 같은 Lock wait time out exceeded 이슈를 해결할 수 있는 것이다.

JobExecutionException은 listener에서는 던질 수 없고 Job.execute() 에서만 던질 수 있기 때문에 나는 기존에 TriggerListener.triggerComplete에 구현했던 로직을 Job 구현체로 옮겨야 했다.

* ex

```java

public class WorkflowJob implements Job {

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        try {
            //...
        } catch (Exception e) {
            context.setResult();
            JobExecutionException ex = new JobExecutionException(e);
            ex.setUnscheduleAllTriggers(true);
            
            throw ex;
        }
    } 
}

```

## trigger 상태

위 처럼 exception을 날리고 `ex.setUnscheduleAllTriggers(true);` 설정 까지 해주면 기존 job에 설정된 trigger 상태를 모두 COMPLETED 처리로 바꿔줌으로써 해당 잡에 더 이상 실행될 여지가 있는 trigger는 없어지게 된다.

다만 quartz job 자체가 처리 도중에 exception이 발생됨으로써 fail 처리 된 경우 이를 retry 처리할 수 있는 기능을 제공하기 때문에 job 자체가 삭제되진 않고 quartz job 내 정보는 남아있게 된다.

따라서 위와 같이 수정 후, job 로직 수행 도중 exception이 발생하여 fail 처리 한 이후 개발자가 retry 될 수 있게 추가 개발을 하거나 아니면 위와 같이 처리 후 별도로 `unscheduleJob()`를 실행함으로써 job 자체를 제거해주는 게 좋다.
