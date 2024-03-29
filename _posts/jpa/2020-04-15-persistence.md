---
date: 2020-04-15 23:27:40
layout: post
title: 자바 ORM 표준 JPA 프로그래밍
subtitle: 3. 영속성 관리
description: 3. 영속성 관리
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

JPA 기능은 `엔티티와 테이블을 매핑 하는 설계 부분`과 `매핑한 엔티티를 실제 사용하는 부분`으로 나뉜다.
여기서 매핑한 엔티티는 `엔티티 매니저(Entity Manager)`를 통해 저장/수정/삭제/조회된다.

## 엔티티 매니저 팩토리와 엔티티 매니저
일반적으로 하나의 데이터베이스를 사용하는 애플리케이션은 하나의 EntityManagerFactory만을 생성하게 된다.
엔티티 매니저 팩토리를 생성한 후, 필요할 때 마다 엔티티 매니저 팩토리에서 엔티티 매니저를 생성하면 된다.
```java

    // 공장을 만드는데는 비용이 많이듦.
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("");
    // 공장에서 엔티티 매니저를 생성할 때는 비용이 거의 안듦.
    EntityManager em = emf.createEntityManager();
``` 

##### 엔티티 매니저 팩토리
생성하는데 비용이 많이 들기 때문에 애플리케이션에서 하나만 생성하여 전체에 공유하도록 설계되어 있고, 여러 thread가 동시 접근해도 안전하므로 서로 다른 thread간에 공유가 가능하다.
##### 엔티티 매니저
생성하는데 비용이 거의 없지만, 여러 thread가 동시에 접근하면 동시성 문제가 발생하므로 thread간에 공유해선 안된다.

![EntityManager](../../assets/img/entitymanager.jpg)
EntityManagerFactory에서 다수의 엔티티 매니저를 생성한다. 
EntityManager들은 데이터베이스 연결이 꼭 필요한 시점까지 db 커넥션을 얻지 않는다.(ex. 트랜잭션을 시작할 때 커넥션을 획득.)
JPA 구현체들은 EntityManagerFactory를 생성할 때 커넥션풀도 생성하게 된다.

## 영속성 컨텍스트(Persistence Context)
영속성 컨텍스트란 `엔티티를 영구 저장하는 환경`이라 할 수 있다.
```java
    em.persist(member);
```
위와 같은 코드는 단순히 member 엔티티를 저장하는 것이 아닌, `엔티티 매니저를 사용해서 member 엔티티를 영속성 컨텍스트에 저장`한다고 말할 수 있다.
영속성 컨텍스트는 논리적인 개념으로 엔티티 매니저가 생성될 때 만들어진다. 
엔티티 매니저를 통해 영속서 컨텍스트에 접근/관리가 가능하다.

## 엔티티 생명주기
![LifeCycle](../../assets/img/entity_lifecycle.jpg)
### 비영속(new/transient)
#### 영속성 컨텍스트와 전혀 관계가 없는 상태
![Transient](../../assets/img/transient.jpg)
```java
    // 객체를 생성한 상태(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
```
### 영속(managed)
#### 영속성 컨텍스트에 저장된 상태
![Managed](../../assets/img/managed.jpg)
```java
    em.persist(member);
```
이처럼 영속성 컨텍스트에 저장되어 관리되는 엔티티를 영속상태라 한다. `em.persist(member);`를 통해 member엔티티는 비영속 상태에서 영속 상태가 되었다. 즉, 영속 상태란 `영속성 컨텍스트에 의해 관리되는 것`을 의미한다.
### 준영속(detached)
#### 영속성 컨텍스트에 저장되었다가 분리된 상태
```java
    // member 엔티티를 영속성 컨텍스트에서 분리함으로써 준영속 상태로 전환.
    em.detach(member);  
    //em.close(); 영속성 컨텍스트를 닫음으로써 준영속 상태로 전환.
    //em.clear(); 영속성 컨텍스트를 초기화함으로써 준영속 상태로 전환.
```
영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다. 위 코드를 통해 영속 상태였던 엔티티를 준영속 상태가 된다. 
### 삭제(removed)
#### 삭제된 상태
엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.
```java
    em.remove(member);  // 객체를 삭제한 상태
```

## 영속성 컨텍스트의 특징
### 영속성 컨텍스트와 식별자
영속성 컨텍스트는 엔티티를 `식별자 값(@Id로 테이블의 기본 키와 매핑한 값)`으로 구분한다. 즉, 영속 상태는 식별자 값이 반드시 필요하다. 식별자 값이 없으면 exception이 발생한다.
### 영속성 컨텍스트와 데이터베이스 저장
영속성 컨텍스트에 저장된 엔티티는 트랜잭션을 커밋하는 순간 데이터베이스에 반영된다. 이를 `flush`라 한다.
### 영속성 컨텍스트가 엔티티를 관리함으로써 얻는 장점
* 1차 캐시
* 동일성 보장
* 트랜잭션을 지원하는 쓰기 지연
* 변경 감지
* 지연 로딩

#### 엔티티 조회
영속성 컨텍스트는 내부에 영속 상태의 모든 엔티티가 저장되는 `1차 캐시` 라는 캐시를 갖고 있다.
```java
    // 비영속 상태
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");

    // 영속 상태
    em.persist(member);
```
위 코드가 실행되면 아래와 같은 상태이 영속성 컨텍스트 내부 1차 캐시에 member 엔티티가 저장된다. 이 엔티티는 데이터베이스에 저장되기 전 상태를 유지한다.
1차 캐시의 키는 식별자 값이다. 식별자 값은 데이터베이스 기본 키와 매핑되어 있고, 따라서 영속성 컨텍스트에 데이터를 저장하고 조회하는 모든 기준은 데이터베이스 기본 키 값이다.
* 엔티티 조회
```java
    Member member = em.find(Member.class, "member1");
```
* EntityManager.find() 메소드 정의
```java
    public <T> T find(Class<T> entityClass, Object primaryKey);
    // 첫 번때 파라미터는 엔티티 클래스 타입, 두 번째는 엔티티 식별자 값
```
em.find()를 호출하면 1차 캐시에서 먼저 엔티티를 찾고 없으면 데이터베이스에 조회한다.

##### 1차 캐시에서 조회
![1차캐시](../../assets/img/1stcache.jpg)
```java
    Member memer = new Member();
    member.setId("member1");
    member.setUsername("회원1");

    //1차 캐시 저장
    em.persist(member);

    //1차 캐시에서 조회
    Member member = em.find(Member.class, "member1");
```
먼저 1차 캐시에 저장 후 member 엔티티를 조회하기 때문에 데이터베이스 조회 없이 1차 캐시에서 member 엔티티가 조회된다.

##### 데이터베이스에서 조회
![데이터베이스조회](../../assets/img/db.jpg)
```java
    Member member2 = em.find(Member.class, "member2");
```
member2는 1차 캐시에 저장 처리 없이 조회를 호출했기 때문에 1차 캐시에는 member2 엔티티 정보가 존재하지 않는다.
따라서 엔티티 매니저는 1차 캐시에 해당 엔티티가 없으면 데이터베이스에서 조회해서 엔티티를 생성한 후 1차 캐시에 저장시킨 후 영속 상태의 엔티티를 반환하게 된다.

이 이후에 member1, member2를 조회하게 되면 모두 1차 캐시에 저장되어 있기 때문에 메모리에 있는 1차 캐시에서 바로 조회, 나름 성능상 이점이 있다.

##### 영속 엔티티의 동일성 보장
```java
    Member a = em.find(Member.class, "member1");
    Member b = em.find(Member.class, "member1");

    System.out.println(a == b); // 동일성 비교.
```
영속성 컨텍스트는 1차 캐시에 있는 같은 엔티티를 반환하고, 둘은 같은 인스턴스이기 때문에 참이다.

#### 엔티티 등록
```java
    EntityManager em = emf.createEntityManager();
    EntityTrasaction transaction = em.getTransaction();
    // 엔티티 매니저는 데이터 변경 시, 트랜잭션을 시작해야 한다.
    transaction.begin();    // [트랜잭션] 시작

    em.persist(memberA);
    em.persist(memberB);
    // 커밋되기 전까지 INSERT SQL을 데이터베이스에 보내지 않는다.

    transaction.commit();   // [트랜잭션] 커밋
```
![엔티티등록](../../assets/img/entity_register.jpg)
엔티티 매니저는 트랜잭션을 커밋하기 직전 까지 내부 쿼리 저장소에 INSERT SQL을 저장해 두었다가 트랜잭션이 커밋될 때 모아둔 쿼리를 데이터베이스로 보낸다. 이를 `쓰기 지연(Transactional write-behind)`라고 한다.
영속성 컨텍스트는 1차 캐시에 member 엔티티를 저장하면서 동시에 member 엔티티 정보로 등록 쿼리를 만들어 쓰기 지연 SQL 저장소에 보관한다.
![엔티티등록1](../../assets/img/entity_register1.jpg)
마지막으로 트랜잭션이 커밋되면 엔티티 메니저는 영속성 컨텍스트를 flush함으로써 영속성 컨텍스트의 변경 내용, 즉 쓰기 지연 SQL 저장소에 모인 쿼리를 데이터베이스에 보냄으로써 변경 내용을 데이터베이스에 동기화하게 된다. 동기화한 후에 실제 데이터베이스에 커밋 처리 하게 된다.

#### 엔티티 수정
##### 기존 SQL 수정 쿼리의 문제점
수정 쿼리를 직접 사용하게 되면 유지보수 하는데(ex. 수정 컬럼 추가) 있어서 쿼리에 직접 손을 대야하는 경우가 늘게 되고, 결국 비즈니스 로직이 SQL에 의존하게 된다.
##### 변경 감지(Dirty Cheking)
```java
    EntityManager em = emf.createEntityManager();
    EntityTrasaction transaction = em.getTransaction();
    transaction.begin();    // [트랜잭션] 시작

    // 영속 엔티티 조회
    Member memberA = em.find(Member.class, "memberA");

    // 영속 엔티티 데이터 수정
    memberA.setUsername("hi");
    memberA.setAge(10);

    //em.update(member); 이런 코드 없이 수정이 가능.

    transaction.commit();   // [트랜잭션] 커밋
```
JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회해 데이터만 변경하면 된다. 이 때 변경 감지 기술이 적용된다.
> 변경 감지(Dirty Checking)란 엔티티의 변경 사항을 데이터베이스에 자동으로 반영해주는 기능을 말한다.

![엔티티수정](../../assets/img/entity_modify.jpg)
JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는데 이를 `스냅샷`이라 한다. 그리고 flush 시점에 스냅샷과 엔티티를 비교하여 변경된 엔티티를 찾게 된다.
변경된 엔티티가 있으면 엔티티 등록 과정과 마찬가지로 수정 쿼리를 생성하여 쓰기 지연 SQL 저장소에 보낸다.
> 변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다.

이렇게 변경 감지를 통해 적용되는 update sql을 보면 변경된 부분만 적용된 update sql이 날라가는 것이 아닌 해당 엔티티의 모든 필드가 업데이트 된다. 따라서 아래와 같은 특징이 나타난다.
* 원하지 않았던 필드도 전송됨으로써 데이터베이스 전송량이 증가한다.
* 항상 해당 엔티티에 대한 모든 필드가 수정됨으로써 해당 엔티티에 대한 수정 쿼리는 항상 같게 된다. 따라서 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성하여 재사용이 가능하다.
* 데이터베이스 특성 상, 동일한 쿼리는 재사용하게 됨으로써 성능상 이점이 있다.

> 필드가 너무 많아 원하는 필드만 수정을 원하면 @DymicUpdate 어노테이션 활용을 통해 수정된 데이터만 사용되는 동적인 UPDATE SQL을 생성하게 된다.

#### 엔티티 삭제
```java
    Member memberA = em.find(Member.class, "memberA");  // 삭제 대상 엔티티 조회
    em.remove(memberA);     // 엔티티 삭제
```
엔티티 등록과 마찬가지로 즉시 삭제되는 것이 아닌, 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록 후에 flush가 호출되면 그 때 실제 데이터베이스로 해당 삭제 쿼리를 전달하게 된다.
> 영속성 컨텍스트에서는 호출 즉시 삭제된다.

## 플러시(Flush)
flush는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.
### 플러시 처리 순서
1. 변경 감지 동작 -> 수정 쿼리 생성 후 쓰기 지연 SQL 저장소에 등록.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송.

### 영속성 컨텍스트를 플러시 하는 방법
#### 직접 호출
em.flush()를 호출하여 영속성 컨텍스트를 강제로 플러시 한다. 테스트나 다른 프레임워크와 JPA를 함께 사용할 때를 제외하곤 거의 사용하지 않는다.
#### 트랜잭션 커밋 시 플러시 자동 호출
데이터베이스에 변경 내용을 SQL로 전달않고 트랜잭션만 커밋하게 되면 어떤 데이터도 데이터베이스에 반영되지 않는다. 따라서 JPA는 이런 문제를 예방하기 위해 트랜잭션을 커밋할 때 플러시를 자동으로 호출한다.
#### JPQL 쿼리 실행 시 플러시 자동 호출 
```java
   em.persist(memberA);     
   em.persist(memberB); 
   em.persist(memberC);

    // JPQL 실행
    query = em.createQuery("select m from Member m", Member.class);
    List<Member> members = query.getResultList();     
```
위 같은 경우 JPQL 실행 시 데이터베이스에 반영이 되지 않는다면 memberA, memberB, memberC 엔티티는 영속성 컨텍스트에만 존재하지 데이터베이스에는 존재하지 않기 때문에 JPQL로 회원 엔티티가 조회될 수 없게 된다. 이러한 문제 해결을 위해 JPA는 JPQL 실행 시 플러시를 자동으로 호출하게 된다.

### 플러시 모드 옵션
엔티티 매니저에 플러시 모드를 직접 설정할 수 있다.
```java
    em.setFlushMode(FlushModeType.COMMIT)   // 플러시 모드 직접 설정
```
#### FlushModeType.AUTO(기본값)
커밋이나 쿼리를 실행할 때 플러시를 자동으로 호출.
#### FlushModeType.COMMIT
커밋할 때만 플러시를 호출. 성능 최적화가 필요할 경우 설정한다.

> 플러시란 영속성 컨텍스트를 데이터베이스에 동기화 시켜주는 것이지 초기화하는 것이 아니다.

## 준영속
`영속성 컨텍스트가 관리하는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 것`을 준영속 상태라 한다. 따라서 준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.
### 준영속 상태로 만드는 3가지 방법
#### em.detach(entity); -> 준영속 상태로 전환
```java
    // 회원 엔티티 생성, 비영속 상태
    Member member = new Member();
    member.setId("memberA");
    member.setUsername("회원A");

    // 회원 엔티티 영속 상태
    em.persist(member);

    // 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
    em.detach(member);

    // 트랜잭션 커밋
    transaction.commit();
```
![준영속](../../assets/img/detach.jpg)
detach()가 호출되는 순간 1차 캐시부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보는 삭제된다. 따라서 커밋이 호출되어도 쓰기 지연 SQL 저장소에 있던 쿼리도 삭제되었기 때문에 데이터베이스에 저장되지 않는다.
#### em.clear(entity); -> 영속성 컨텍스트 초기화
```java
    // 엔티티 조회, 영속 상태
    Member member = em.find(Member.class, "memberA");

    em.clear();     // 영속성 컨텍스트 초기화

    // 준영속 상태
    member.setUsername("changeName");
```
![준영속_초기화](../../assets/img/clear.jpg)
clear()가 호출되면 영속성 컨텍스트에 있는 모든 것이 초기화된다.
#### em.close(entity); -> 영속성 컨텍스트 종료
```java
    EntityManagerFactory emf = Persistence.createEntityFactory("jpabook");
    
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();    // [트랜잭션] 시작

    Member memberA = em.find(Member.class, "memberA");
    Member memberB = em.find(Member.class, "memberB");

    transaction.commit();   // [트랜잭션] 커밋

    em.close();     // 영속성 컨텍스트 종료
```
영속성 컨텍스트가 종료됨으로써 더는 memberA, memberB가 관리되지 않는다.
> 개발자가 직접 영속 상태의 엔티티를 준영속 상태로 만드는 일은 드물다. 영속성 컨텍스트가 종료되면서 준영속 상태가 된다.

### 준영속 상태의 특징
#### 비영속 상태와 가깝다
영속성 컨텍스트가 더 이상 관리하지 않는 상태가 됨으로써 1차캐시, 쓰기 지연, 변경 감지, 지연 로딩을 포함한 기능이 동작하지 않게 된다.
#### 식별자 값을 갖는다
비영속 상태는 식별자 값이 없을 수 있지만, 준영속 상태는 한번 영속 상태였기 때문에 반드시 식별자 값을 갖고 있다.
#### 지연 로딩이 불가하다

### 병합: merge()
준영속 상태의 엔티티를 다시 영속 상태로 변경시켜 준다.
merge() 메소드는 `준영속 상태의 엔티티를 받아 새로운 영속 상태의 엔티티를 반환`시키게 된다.
![병합](../../assets/img/merge.JPG)
1. member 엔티티를 준영속에서 영속 상태로 전환한다.
2. 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하여 1차 캐시에 저장한다.
3. 조회해온 영속 엔티티에 준영속 상태였던 member 엔티티 값을 채워 넣는다.
4. 기존 준영속 상태의 member 엔티티 값으로 채운 `새롭게 생성한 영속 상태 엔티티를 반환`한다.
> 기존 준영속 상태였던 member 엔티티는 병합 후에도 준영속 상태가 유지된다.
 
#### 비영속 병합
비영속 엔티티도 영속 상태로 만들 수 있다.
병합은 준영속/비영속 상관 없이 식별자 값으로 엔티티 조회가 가능하면 불러와 병합하고 없으면 데이터베이스에서 조회한다. 만약 데이터베이스에도 없으면 새로운 엔티티를 생성하여 병합한다.
따라서 병합은 save or update 기능을 수행한다.

## 정리
* 엔티티 매니저 팩토리를 통해 엔티티 매니저를 생성하고, 엔티티 매니저가 생성되면 그 내부에 영속성 컨텍스트도 함께 만들어진다. 영속성 컨텍스트는 엔티티 매니저를 통해 접근할 수 있다.
* 영속성 컨텍스트를 통해 `1차 캐시`, `변경 감지`, `동일성 보장`, `트랜잭션을 지원하는 쓰기 지연`, `지연 로딩` 기능을 사용할 수 있다.
* 영속성 컨텍스트에 저장된 엔티티는 플러시 시점에 데이터베이스에 반영되고, 보통 플러시는 트랜잭션이 커밋되는 시점이 실행된다.
