---
date: 2023-03-02 11:36:40
layout: post
title: mysql 테이블명 대소문자 구분 설정
subtitle: mysql 테이블명 대소문자 구분 설정
description: mysql에서는 DDL 쿼리를 통해 생성한 테이블명의 대소문자 구분 여부를 설정값으로 관리하고 있다.
image: https://user-images.githubusercontent.com/15065956/222113903-fc670672-571e-4b09-82f0-05540fc2b86d.png
optimized_image: https://user-images.githubusercontent.com/15065956/222113903-fc670672-571e-4b09-82f0-05540fc2b86d.png
category: mysql
tags:
- mysql
- aws
- aurora

paginate: true
comments: true
---

### 개요

프로젝트 구현하면서 테이블 생성을 위해 DDL 생성 중, 로컬 환경에서는 테이블명을 모두 소문자로 변경하여 세팅하여 개발을 진행했었다.

그렇게 로컬 개발을 끝내고 staging 환경에서 테스트를 진행하기 위해 DDL 생성을 했더니 아래와 같은 에러 로그를 마주할 수 있었다.

```log
2023-02-15 11:44:50.221 [52164_ClusterManager] [ERROR] org.springframework.scheduling.quartz.LocalDataSourceJobStore : ClusterManager: Error managing cluster: Failure obtaining db row lock: Table '****.QRTZ_LOCKS' doesn't exist
org.quartz.impl.jdbcjobstore.LockException: Failure obtaining db row lock: Table '****.QRTZ_LOCKS' doesn't exist
	at org.quartz.impl.jdbcjobstore.StdRowLockSemaphore.executeSQL(StdRowLockSemaphore.java:184)
	at org.quartz.impl.jdbcjobstore.DBSemaphore.obtainLock(DBSemaphore.java:113)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport.doCheckin(JobStoreSupport.java:3335)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport$ClusterManager.manage(JobStoreSupport.java:3935)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport$ClusterManager.run(JobStoreSupport.java:3972)
Caused by: java.sql.SQLSyntaxErrorException: Table '****.QRTZ_LOCKS' doesn't exist
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:120)
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeInternal(ClientPreparedStatement.java:916)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeQuery(ClientPreparedStatement.java:972)
	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
	at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeQuery(HikariProxyPreparedStatement.java)
	at org.quartz.impl.jdbcjobstore.StdRowLockSemaphore.executeSQL(StdRowLockSemaphore.java:123)
	... 4 common frames omitted
```

spring quartz를 사용하면서 quartz에서 활용하는 qrtz_locks 테이블을 못찾겠다는 로그가 주기적으로 남겨지고 있었다.

처음엔 aws ec2 기반의 staging 환경 세팅에 문제가 발생하여 spring quartz가 돌 때 qrtz_locks 테이블을 찾지 못하는 게 아닐까 의심했었는데
원인은 그게 아니었다.

### 원인 발견

본래 quartz에서 제공하는 quartz DDL 쿼리에서도 테이블명을 대문자로 제공하고 있었고 위 에러 로그처럼 spring quartz에서는 대문자 테이블명을 표기하면서 찾고 있었다.

구글링을 하다보니 실마리를 찾을 수 있었다. 바로 `lower_case_table_names`라는 mysql 설정이 있었던 것이다.

> [mysql 공식 문서 참조](https://dev.mysql.com/doc/refman/8.0/en/identifier-case-sensitivity.html)

lower_case_table_name 설정에 대해 간략히 정리하면 다음과 같다.

* lower_case_table_name 설정

<table>
  <thead>
    <tr>
      <th>value</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>테이블 및 데이터베이스 이름 생성 시 대소문자를 구분하여 디스크에 저장<br>파일 이름이 대소문자를 구분하지 않는 시스템(예: Windows 또는 macOS)에서 MySQL을 실행하는 경우 이 변수를 0으로 설정하면 안됨 --lower-case-table-names=0 대소문자를 구분하지 않는 파일 시스템에서 이 변수를 0으로 설정 하고 MyISAM다른 문자를 사용하여 테이블 이름에 액세스하면 인덱스가 손상될 수 있음</td>
    </tr>
    <tr>
      <td>1</td>
      <td>대소문자를 구분하지 않고 소문자로 테이블 및 데이터베이스 이름이 생성되어 저장. <br> 이 동작은 데이터베이스 이름과 테이블 별칭에도 적용</td>
    </tr>
    <tr>
      <td>2</td>
      <td>테이블 및 데이터베이스 이름은 CREATE TABLE 또는 CREATE DATABASE 문에 지정된 문자 대소문자를 구분하여 디스크에 저장되지만 MySQL은 조회 시 소문자로 변환(이름 비교는 대소문자를 구분 X)<br> (이것은 대소문자를 구분하지 않는 파일 시스템에서만 작동.)<br>lower_case_table_names=1인 경우 InnoDB 테이블 이름 및 뷰 이름은 소문자로 저장됨</td>
    </tr>
  </tbody>
</table>

해당 설정은 my.cnf에서 수정 후 restart하면 변경이 가능하다.

로컬 환경 db에서 lower_case_table_names 설정 값이 2로 되어있었기 때문에 staging 환경에서도 해당 설정을 동일하게 2로 변경하면 해결될 것이라 생각하였다.

### 난관

staging 환경에서 db는 aws mysql8.0버전 aurora를 사용하고 있었기 때문에 설정을 변경하려고 했다. 근데 문제가 발생하였다. **aws에서는 2 를 지원하지 않는다고 한다.**

> [aws 공식 문서 참조](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/MySQL.KnownIssuesAndLimitations.html)

lower_case_table_names 란을 보면 amazon rds는 lower_case_table_names 설정 값으로 2는 지원하지 않는다고 한다.

그러면 어쩔 수 없이 대소문자 구분을 하지 않도록 lower_case_table_names 설정을 1로 변경하려고 했다. 그러나 또 다시 문제가 발생한다.

```log
The parameter value for lower_case_table_names can't be changed for parameter group ****, because it is associated with one or more MySQL 8.0 DB instances. (Service: AmazonRDS; Status Code: 400; Error Code: InvalidParameterCombination; Request ID: *****; Proxy: null)
```

이미 생성된 인스턴스에서 lower_case_table_names 설정을 바꾸려면 기존 파라미터 수정 방식이 아닌 새로운 파라미터 그룹을 생성해야 한다고 한다.

이 정도 문제면 괜찮다. 귀찮기만 할 뿐이니까. 하지만 더 나아간다.

**aws에서 8 버전은 lower_case_table_names를 0만 제공**한다고 한다.

그럼 mysql 버전은 5.7로 낮추는 수 밖에 없다. 하지만 이는 너무 비효율적이다. 그리고 개발자로써 버전을 낮추는 것은 용납이 되지 않았다.

### 해결

그럼 남은 방법은 하나! quartz 테이블 DDL 생성을 대문자로 하는 것이다!

사실 진작 이 방법이 가장 근본적이면서 빠른 해결방법이었다. 하지만 끝까지 애플리케이션 레벨이 아닌 인프라 환경에서 수정 할 수 있는 방법을 찾아봤더니 위와 같은 과정을 거치면서 몰랐던 부분을 알 수 있게 되었다.

결과적으로 지식도 얻고 깔끔히 해결도 하고 꿩먹고 알먹고가 된 결과물이었다.







