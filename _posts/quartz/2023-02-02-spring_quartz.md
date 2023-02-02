---
date: 2023-02-02 12:36:40
layout: post
title: spring quartz
subtitle: spring quartz에 대해서
description: spring quartz에 대해서
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

## Overview

신규 프로젝트 개발에 착수하면서 동적으로 스케줄 잡을 생성해서 관리해야 하는 기능 개발이 필요하게 되면서 알게 된 내용을 기록해두려한다.

이전 경험으로는 간단한 스케줄링 기능이라면 `@Scheduled`를 활용하거나, spring batch + jenkins를 활용하는 형태로 개발했던 경험이 있었는데
이번 프로젝트에서 요구되었던 기능은 동적으로 커스텀한 주기를 가진 일회성 job을 생성할 수 있어야 했다.
이에 적합한 것이 Quartz였다.

## Quartz란

오픈소스로 Java 스케줄링 라이브러리로, Spring과 함께 사용될 수도 있으며, Spring과 별개로 사용될 수도 있다.

Quartz의 대표적인 기능 중에 몇 가지를 간략하게 얘기해본다면 jenkins에서는 jenkins가 제공하는 admin ui를 통해 스케줄링 설정하는 것과 달리,
Quartz는 **애플리케이션 레벨에서 스케줄링 설정**이 가능하고 또한 **클러스터링 설정**도 가능하다. 
애플리케이션 레벨에서 구현하다 보니 jenkins와 달리 설정된 주기 외에 다른 시점에 수동으로 job을 실행한다거나 할 수는 없다.

## Quartz 기본 구성

- Job : 스케줄링할 실제 작업을 가지는 객체이다.
  - JobListener
- JobDetail : Job의 정보를 구성하는 객체이다.
- Trigger : Job이 언제 시행될지에 대한 정보를 구성하는 객체이다.
  - TriggerListener
- JobData : Job에서 사용할 데이터를 전달하는 역할을 하는 객체이다.
- Scheduler : JobDetail과 Trigger 정보를 이용해서 Job을 시스템에 등록하고, Trigger가 동작하면 지정된 Job을 실행시키는 역할을 하는 객체이다.
- SchedulerFactory : Scheduler 인스턴스를 생성하는 역할을 하는 객체이다.
- quartzProperties : Quartz 스케줄러를 위한 설정 값을 관리하는 객체이다.
- JobStore : 스케줄러에 등록된 Job의 정보와 실행이력이 저장되는 공간이다. 기본적으로 RAM에 저장되어 JVM 메모리공간에서 관리되지만, 원한다면 다른 RDB에서 관리할 수 있다.

위에 언급되어있지만 Job, Trigger, Schedule는 각각 Listener interface를 갖는다. Listener들은 주로 프로세스의 실행, 종료, 중단 등의 라이프사이클에 따라 로그를 출력하거나 추가 로직을 작성할 수 있는 객체인데, 굳이 설정하지 않아도 실행하는데 문제가 되지는 않는다.

### Quartz Properties

> [Quartz Configuration](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/configuration.html)

Quartz의 세부 설정 값에 대한 설명은 위 공식 문서에 자세히 나와있다. 
다만 한가지 정리하고 싶은 내용은 클러스터 모드 사용 시 각 인스턴스 마다 Id를 세팅하기 위해 `org.quartz.scheduler.instanceId`이 
사용 되는데 k8s를 사용한다거나 하는 이유로 scaleout을 빈번하게 활용하게 될 때 .yml에서 `org.quartz.scheduler.instanceId`을 고정 값으로
세팅하게 되면 Quartz가 해당 인스턴스를 인식하지 못할 수 있는 데 이 때 `AUTO`라는 value로 설정해두면 호스트 이름과 타임스탬프를 기반으로 고유의 인스턴스 ID를 자동 생성하게 된다.

> 한 가지 주의할 점은 소문자 `auto`로 설정하게 되면 동작하지 않는다. 

#### 클러스터 모드

quartz를 클러스터 모드로 활용하려면 스케줄러에 등록된 Job의 정보와 실행이력이 저장되는 공간으로 메모리가 아닌 DB를 사용해야 한다. 즉, DB 환경이 세팅돼야 한다.
그러기 위해서 quartz 관련 테이블 스키마를 생성해야 하는데 이는 spring-boot-starter-quartz가 로드 된 상태에서 `tables_mysql_innodb.sql`을 찾아서 생성할 수 있다.
추가로 `org.quartz.jobStore.isClustered= true`로 세팅해야 한다.

### Quartz 초기 설정 Bean 세팅

```java
@Configuration
@RequiredArgsConstructor
public class QuartzConfig {
    
    private final MyJobListener jobListener;
    private final MyTriggerListener myTriggerListener;
    
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean(DataSource dataSource, QuartzProperties quartzProperties, PlatformTransactionManager transactionManager, ApplicationContext applicationContext) {
        QuartzJobFactory quartzJobFactory = new QuartzJobFactory();
        quartzJobFactory.setApplicationContext(applicationContext);
        SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
        Properties properties = new Properties();
        properties.putAll(quartzProperties.getProperties());
        schedulerFactoryBean.setJobFactory(quartzJobFactory);
        schedulerFactoryBean.setApplicationContext(applicationContext);
        schedulerFactoryBean.setDataSource(dataSource);
        schedulerFactoryBean.setQuartzProperties(properties);
        schedulerFactoryBean.setTransactionManager(transactionManager);
        schedulerFactoryBean.setOverwriteExistingJobs(true);
        schedulerFactoryBean.setGlobalJobListeners(jobListener);
        schedulerFactoryBean.setGlobalTriggerListeners(myTriggerListener);
        
        return schedulerFactoryBean;
    }
}
```

위 예시 코드를 보면 SchedulerFactoryBean에서 DataSource, QuartzProperties, PlatformTransactionManager, ApplicationContext 4가지 객체를 주입받고 있는데 각각 의미가 있다.

- DataSource : quartz에서 스케줄러에 등록된 Job의 정보와 실행이력을 저장하기 위한 공간으로 크게 메모리와 DB를 선택할 수 있다. DB로 선택했을 때 기존 애플리케이션에 세팅돼있는 DB 저장소 정보를 quartz에서도 동일하게 활용하기 위해 세팅돼있는 DataSource를 주입한다.
- QuartzProperties : application.yml에 세팅된 quartz config 값을 가져온다.
- PlatformTransactionManager : DataSource와 마찬가지로 기존에 애플리케이션에서 관리하는 transactionManager를 동일하게 활용함으로써 서비스 로직과 같은 트랜잭션 내에서 처리하기 위해 주입한다.
- ApplicationContext : quartz job을 동적으로 생성하게 됐을 경우 applicationContext에 미리 올라가 있는 다른 서비스 로직이 담긴 bean들을 Job에서 주입하여 사용할 수 없다. 이를 해결하기 위해 나는 `QuartzJobFactory`을 생성하여 동적으로 생성된 job에서도 bean을 주입할 수 있게 활용하기 위한 목적으로 주입한다.

```java
@Configuration
public class QuartzJobFactory extends AdaptableJobFactory implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    @Override
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
        Object jobInstance = super.createJobInstance(bundle);
        autowireBean(jobInstance);
        return jobInstance;
    }
    private void autowireBean(Object jobInstance) {
        AutowireCapableBeanFactory autowireCapableBeanFactory = applicationContext.getAutowireCapableBeanFactory();
        autowireCapableBeanFactory.autowireBean(jobInstance);
    }
}
```

### job 생성

위에서 quartz 관련 세팅이 끝났으면 아래와 같이 job을 생성할 수 있다.

```java
@Component
@RequiredArgsConstructor
public class ScheduleHandler {

  private final Scheduler scheduler;

  public void createJob(long id, String jobDescription, String jobGroupName, Set<TriggerParams> triggers) {
    try {
      JobDetail jobDetail = buildJobDetail(id, jobDescription, jobGroupName);
      scheduler.scheduleJob(jobDetail, TriggerParams.toDataMapping(jobDetail, triggers, jobGroupName), true);
      log.info("create success {} {} {} job", expId, jobDescription, jobGroupName);
      getExecutingJobInfo(jobGroupName);
    } catch (Exception e) {
      throw new JobException("job 생성 실패", e);
    }
  }

  public JobDetail buildJobDetail(long id, String description, String jobGroupName) {
    JobDataMap jobDataMap = new JobDataMap();       //  Job 생성 시 들고 있어야할 정보를 Map 타입 형태로 함께 저장한다.(Trigger 생성 시에도 동일하게 활용가능하다)
    jobDataMap.put("ID", id);

    return JobBuilder.newJob(WorkflowJob.class)
                     .withIdentity(StringUtils.joinWith("-", "ID", id), jobGroupName)
                     .withDescription(description)
                     .usingJobData(jobDataMap)
                     .build();
  }
}
```

Scheduler 객체를 통해 Job의 정보를 구성하는 JobDetail 객체와 Job이 언제 시행될지에 대한 정보를 구성하는 Trigger 객체를 인자로 활용하여 job을 생성한다.

### Quartz 구현체

위에서 생성한 job이 실행이 될 때 함께 실행되어야할 비즈니스 로직이 필요할 경우 아래와 같이 listener 구현체를 활용할 수 있다.

* TriggerListener

```java
public interface TriggerListener {

    String getName();

    void triggerFired(Trigger trigger, JobExecutionContext context);

    boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

    void triggerMisfired(Trigger trigger);

    void triggerComplete(Trigger trigger, JobExecutionContext context, int triggerInstructionCode);
}
```

* JobListener

```java
public interface JobListener {

    String getName();

    void jobToBeExecuted(JobExecutionContext context);

    void jobExecutionVetoed(JobExecutionContext context);

    void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException);

}
```

```java
public interface Job {
    void execute(JobExecutionContext context) throws JobExecutionException;
}
```

위 Listener를 구현하는 구현체를 생성하여 비즈니스 로직을 구현함으로써 job이 동작하게 된다.

#### job 동작 flow 순서

1. TriggerListener.triggerFired : job trigger 발생
2. TriggerListener.vetoJobExecution : 해당 job 계속 진행 or 중지 여부 결정
3. JobListener.jobToBeExecuted : job 실행 시작
4. Job.execute : job 실행
5. JobListener.jobWasExecuted : job 실행 완료
6. TriggerListener.triggerComplete : trigger 실행 완료

job과 trigger는 1 : N 관계를 갖는다. 즉 하나의 job 안에 여러 개의 trigger를 설정하여 생성한 trigger가 갖고 있는 시작/종료 시점 마다 별도의 로직을 실행 시키는 구조가 가능하다.

## 마무리

quartz도 batch + jenkins 처럼 cron tab 주기를 설정하여 특정 주기마다 job을 실행시키게 설정은 가능하지만 
내가 필요했던 기능은 동적으로 사용자가 원하는 시각에 job이 일회성으로 동작되고 나면 삭제될 수 있는 기능이었다.

이를 구현하기에 quartz는 적합했다.

quartz는 어떤 DB를 활용할지 부터 인스턴스 클러스터링 설정 여부까지 모든 설정을 애플리케이션 레벨에서 세팅 할 수 있다.

때문에 별도의 admin ui가 필요하지 않는 스케줄러 기능이 필요하다면 굳이 jenkins 세팅 없이 quartz만 활용해도 충분하다.

아니면 admin ui가 필요한데 jenkin ui는 맘에 들지 않는 경우에 직접 admin ui를 구현하여 quartz 애플리케이션과 연동하는 형태로
서비스 운영도 가능하다.
