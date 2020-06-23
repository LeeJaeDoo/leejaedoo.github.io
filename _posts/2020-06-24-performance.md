---
date: 2020-06-24 23:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 15. 고급 주제와 성능 최적화
description: 15. 고급 주제와 성능 최적화
image: https://leejaedoo.github.io/assets/img/jpa.png
optimized_image: https://leejaedoo.github.io/assets/img/jpa.png
category: jpa
tags:
  - jpa
  - orm
  - 책 정리
paginate: true
comments: true
---
## 예외 처리
### JPA 표준 예외 정리
JPA의 표준 예외들은 javax.persistence.PersistenceException의 자식 클래스다. 그리고 이 예외 클래스는 RuntimeException의 자식이다. 따라서 JPA 예외는 모두 언체크 예외다.<br>

또한, 서비스 계층에서 데이터 접근 계층의 구현 기술에 직접 의존하는 것은 좋지 않은 설계다. 예외처리도 마찬가지다. 서비스 계층에서 JPA의 예외를 직접 사용하면 JPA에 의존하게 된다.<br>
 스프링 프레임워크는 이러한 문제를 해결하기 위해 데이터 접근 계층에 대한 예외를 추상화하여 개발자에게 제공한다.

#### JPA 표준 예외 및 스프링 변환 예외
* 트랜잭션 롤백을 표시하는 예외
    * 심각한 예외로 복구해선 안된다.
    * 예외 발생 시, 트랜잭션을 강제로 커밋해도 트랜잭션이 커밋되지 않고 javax.persistence.RollbackException 예외가 발생한다.
<table>
  <thead>
    <tr>
      <th>트랜잭션 롤백을 표시하는 예외</th>
      <th>설명</th>
      <th>스프링 변환 예외</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>javax.persistence.RollbackException</td>
      <td>예외 발생 시, 트랜잭션을 강제로 커밋해도 트랜잭션이 커밋되지 않는다.</td>
      <td>org.springframework.transaction.TransactionSystemException</td>
    </tr>
    <tr>
      <td>javax.persistence.EntityExistException</td>
      <td>EntityManager.persist(...) 호출 시 이미 같은 엔티티가 있으면 발생</td>
      <td>org.springframework.dao.DataIntegrityViolationException</td>
    </tr>
    <tr>
      <td>javax.persistence.EntityNotFoundException</td>
      <td>EntityManager.getReference(...)를 호출했는데 실제 사용 시 엔티티가 존재하지 않으면 발생.<br>refresh(...), lock(...)에서도 발생</td>
      <td>org.springframework.orm.jpa.JpaObjectRetrievalFailureException</td>
    </tr>
    <tr>
      <td>javax.persistence.OptimisticLockException</td>
      <td>낙관적 락 충돌 시 발생</td>
      <td>org.springframework.orm.jpa.JpaOptimisticLockingFailureException</td>
    </tr>
    <tr>
      <td>javax.persistence.PessimisticLockException</td>
      <td>EntityTransaction.commit() 실패 시 발생. 롤백이 표시되어 있는 트랜잭션 커밋 시에도 발생</td>
      <td>org.springframework.dao.PessimisticLockingFailureException</td>
    </tr>
    <tr>
      <td>javax.persistence.TransactionRequiredException</td>
      <td>트랜잭션이 필요할 때 트랜잭션이 없으면 발생. 트랜잭션 없이 엔티티를 변경할 때 주로 발생.</td>
      <td>org.springframework.ApiUsageException</td>
    </tr>    
  </tbody>
</table>
    
* 트랜잭션 롤백을 표시하지 않는 예외
    
 <table>
   <thead>
     <tr>
       <th>트랜잭션 롤백을 표시하지 않는 예외</th>
       <th>설명</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td>javax.persistence.NoResultException</td>
       <td>Query.getSingleResult() 호출 시 결과가 하나도 없을 때 발생</td>
       <td>org.springframework.dao.EmptyResultDataAccessException</td>
     </tr>
     <tr>
       <td>javax.persistence.NonUniqueResultException</td>
       <td>Query.getSingleResult() 호출 시 결과가 둘 이상일 때 발생</td>
       <td>org.springframework.dao.IncorrectResultSizeDataAccessException</td>
     </tr>
     <tr>
       <td>javax.persistence.LockTimeoutException</td>
       <td>비관적 락에서 시간 초과 시 발생</td>
       <td>org.springframework.dao.CannotAcquireLockException</td>
     </tr>
     <tr>
       <td>javax.persistence.QueryTimeoutException</td>
       <td>쿼리 실행 시간 초과 시 발생</td>
       <td>org.springframework.dao.QueryTimeoutException</td>
     </tr>
   </tbody>
 </table>
 
* JPA 예외를 스프링 예외로 변경 추가
 <table>
    <thead>
        <tr>
          <th>JPA 예외</th>
          <th>스프링 변환 예외</th>
        </tr>
    </thead>
     <tbody>
       <tr>
         <td>java.lang.IllegalStateException</td>
         <td>org.springframework.dao.InvalidDataAccessApiUsageException</td>
       </tr>
       <tr>
         <td>java.lang.IllegalArgumentException</td>
         <td>org.springframework.dao.InvalidDataAccessApiUsageException</td>
       </tr>
     </tbody>
 </table>
 
### 스프링 프레임워크에 JPA 예외 변환기 적용
JPA 예외를 스프링 프레임워크가 제공하는 추상화된 예외로 변경하려면 `PersistenceExcetionTranslationPostProcessor`를 스프링 빈으로 등록하면 된다. 이 것은 @Repository 어노테이션을 사용한 곳에 예외 변환 AOP를 적용해서 JPA 예외를 스프링 프레임워크가 추상화한 예외로 변환해준다.
 
### 트랜잭션 롤백 시 주의사항
트랜잭션을 롤백하는 것은 데이터베이스의 반영사항만 롤백하는 것이지 수정한 자바 객체까지 원상태로 복구해주지 않는다.<br>
예를 들자면, 엔티티를 조회해 수정하는 중 문제가 발생하여 롤백처리 된다면 데이터베이스의 데이터는 원래대로 복구되지만 객체는 수정된 상태로 영속성 컨텍스트에 남아 있다. 따라서 트랜잭션이 롤백된 영속성 컨텍스트를 그대로 사용하는 것은 위험하다.
 * 새로운 영속성 컨텍스트 생성
 * EntityManager.clear() 를 호출하여 영속성 컨텍스트 초기화
 
스프링 프레임워크는 이런 문제를 해결하기 위해 영속성 컨텍스트의 범위에 따라 다른 방법을 사용한다.
* 트랜잭션 당 영속성 컨텍스트 전략
    * 기본 전략인 트랜잭션 당 영속성 컨텍스트 전략은 문제가 발생하면 트랜잭션 AOP 종료 시점에 트랜잭션을 롤백하면서 영속성 컨텍스트도 함께 종료하므로 문제가 발생하지 않는다.

하지만, OSIV처럼 영속성 컨텍스트의 범위를 트랜잭션 범위보다 넓게 사용하여 여러 트랜잭션이 하나의 영속성 컨텍스트를 사용할 때 발생한다. 트랜잭션을 롤백해도 다른 트랜잭션에서 같은 영속성 컨텍스트를 공유하기 때문에 문제가 발생한다.<br>
스프링 프레임워크는 `영속성 컨텍스트의 범위가 트랜잭션 보다 넓을 경우 트랜잭션 콜백 시 영속성 컨텍스트를 초기화(EntityManager.clear())`해서 잘못된 영속성 컨텍스트 사용을 방지한다.

## 엔티티 비교
영속성 컨텍스트 내부에는 엔티티 인스턴스를 보관하기 위한 1차 캐시가 있다. 1차 캐시는 영속성 컨텍스트와 생명주기를 같이 한다.<br>
1차 캐시의 가장 큰 장점은 `애플리케이션 수준의 반복 가능한 읽기`이다. `같은 영속성 컨텍스트에서 엔티티를 조회하면 다음 코드와 같이 항상 같은 엔티티 인스턴스를 반환`한다. 단순히 동등성 비교가 아닌 주소값이 같은 인스턴스를 반환한다.
```java
Member member1 = em.find(Member.class, "1L");
Member member2 = em.find(Member.class, "1L");

assertTrue(member1 == member2); // 둘은 같은 인스턴스
```

### 영속성 컨텍스트가 같을 때 엔티티 비교
같은 트랜잭션 범위에 같은 영속성 컨텍스트에서 조회한 엔티티는 항상 같은 인스턴스이다. 아래 3가지 조건이 모두 같다.
* 동일성 : == 비교가 같다.
* 동등성 : equals() 비교가 같다.
* 데이터베이스 동등성 : @Id인 데이터베이스 식별자가 같다.

> @Transacional 안에서 @Transactional 을 사용하게 되면 안에서 밖에 트랜잭션을 그대로 이어받게되고 없으면 새로 트랜잭션이 생성된다.

> 테스트 클래스에 @Transacional 을 적용하면 테스트 클래스가 종료될 때 커밋되지 않고 롤백된다. 하지만 롤백 시, 영속성 컨텍스트는 flush되지 않는다.

### 영속성 컨텍스트가 다를 때 엔티티 비교
영속성 컨텍스트가 다르게 되면 `동일성 비교에 실패`한다.
* 동일성 : == 비교에 실패한다.
* 동등성 : equals() 비교가 만족한다. 단, equals()를 구현해야 한다.(비즈니스 키로 구현한다.)
* 데이터베이스 동등성 : @Id인 데이터베이스 식별자는 같다.

같은 영속성 컨텍스트에서는 동일성 비교가 보장된다. 따라서 OSIV 처럼 요청의 시작부터 끝까지 같은 영속성 컨텍스트를 사용할 때는 동일성 비교가 성공한다.<br>
하지만, 영속성 컨텍스트가 달라지면 동일성 비교에 실패한다. 따라서 영속성 컨텍스트가 다를 때는 데이터베이스 동등성 비교를 해야 한다.
```java
member.getId().equals(findMember.getId());  // 데이터베이스 식별자 비교
```
하지만 데이터베이스 동등성 비교는 엔티티를 영속화해야 식별자를 얻을 수 있다는 제약이 있다. 영속화하기 전에는 식별자가 null이므로 정확한 비교가 어렵다.<br>
결국 equals()를 사용한 동등성 비교를 해야 하는데 엔티티를 비교할 때는 비즈니스 키를 활용한 동등성 비교를 하는 것이 좋다.

### 정리
같은 영속성 컨텍스트의 관리를 받는 영속 상태의 엔티티끼리만 동일성 비교가 가능하다. 다른 영속성 컨텍스트에서는 비즈니스 키를 사용한 동등성 비교를 해야 한다.