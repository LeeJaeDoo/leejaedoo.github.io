---
date: 2022-06-12 13:40:40
layout: post
title: multiple datasource 구축과 트랜잭션(1)
subtitle: multiple datasource와 트랜잭션 관리
description: multiple datasource 구축 과정에 대한 기록
image: https://user-images.githubusercontent.com/15065956/173235327-3c441ab7-8e51-4f5e-bcde-59fdc1b0c8d9.png
optimized_image: https://user-images.githubusercontent.com/15065956/173235327-3c441ab7-8e51-4f5e-bcde-59fdc1b0c8d9.png
category: issue
tags:
- issue
- springboot
- datasource
- jpa
- transaction
- jta
paginate: true
comments: true
---
# Multiple Datasource 구축을 통한 트랜잭션 관리
하나의 프로젝트에 여러개의 datasource를 붙일 수 있을까? 붙이면 두 datasource를 하나의 트랜잭션으로 관리할 수 있을까?

막연한 호기심이 생겨 직접 테스트해보고 기록을 남겨보고 싶은 마음에 의식에 흐름대로 적어보려한다.

## 프로젝트 구조

멀티 모듈을 기본으로 가되, 어떤 구조로 갈지에 대한 고민이 있었다.

우선 붙여볼 datasource마다 모듈을 생성하고, 두 모듈에 붙는 비즈니스 로직만을 갖는 하나의 모듈을 추가, 이렇게 총 3개의 모듈 형태로 갈지..

![domains](https://user-images.githubusercontent.com/15065956/173211856-a277407c-6c97-4012-99a9-2dcd0437d54f.png)

기본 기술 스택은 springboot + JPA + mysql을 기본으로 적용해보려 한다.

## 모듈마다 datasource 설정 방식

```groovy
dependencies {
    implementation project (':domain1')
    implementation project (':domain2')
}
```

먼저 Domain1과 Domain2 모듈 마다 각각 별도의 스키마로 설정해두었고 Domain1에는 Product 엔티티, Domain2에는 Member 엔티티를 생성하였다.

> 한 가지 알아야 할 점이 위와 같이 두 개의 모듈을 가져오는 구조일 경우 application.yml 에 아래와 같은 설정이 추가되어야 한다.
> ```yaml
> spring:
>   profiles:
>   active: local
>     include:
>       - domain1 
>       - domain2
> ```
> include를 통해 해당 모듈을 포함시켜주지 않으면 bean을 가져오지 못한다.
> 그리고 domain1, domain2에 생성되있는 application 파일명도 application-domain1.yml과 같이 application-{profile}.yml 형식을 맞춰야 한다.

그리고 아래와 같이 Sercie 모듈에서 한번에 두 스키마 내에 있는 테이블을 조회해오는 실험을 하였다.

```java
    @Transactional
    public void update() {
        Product product = productRepository.findById(1L).get();
        Member member = memberRepository.findById(1L).get();

        product.setTitle("new");
        member.setName("hi");
    }
```

그러자 아래와 같은 오류가 발생하였다.

```
Hibernate: select member0_.id as id1_1_0_, member0_.created_at as created_2_1_0_, member0_.name as name3_1_0_ from member member0_ where member0_.id=?
2022-06-12 11:49:40.270  WARN 99525 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 1146, SQLState: 42S02
2022-06-12 11:49:40.270 ERROR 99525 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : Table 'product.member' doesn't exist
2022-06-12 11:49:40.298  INFO 99525 --- [nio-8080-exec-1] o.h.e.internal.DefaultLoadEventListener  : HHH000327: Error performing load command
```

Member 엔티티에 접근조차 하지 못하는 오류가 났고 오류내용을 보니 스키마명이 Domain1 스키마내에서 member테이블을 찾지 못해 오류가 나고 있었다.

트랜잭션 설정이 첫번째 datasource와 먼저 설정되면서 이후 두번째 datasource로 설정된 member 엔티티는 그대로 첫번째 datasource 설정된 채로 접근을 하게 되면서 이런 오류가 발생한 것이라 추측하여 그렇다면 product와 member 조회 순서를 바꿔보면 어떨까 하고 아래처럼 수정 후, 다시 테스트를 해보았다.

```groovy
dependencies {
    implementation project (':domain2')
    implementation project (':domain1')
}
```

그랬더니 예상대로 이번엔 member엔티티는 제대로 조회해오는데 product엔티티는 member 스키마 내에서 product 테이블을 찾다가 에러를 뿜는다

```
Hibernate: select product0_.id as id1_2_0_, product0_.created_at as created_2_2_0_, product0_.finished_at as finished3_2_0_, product0_.started_at as started_4_2_0_, product0_.title as title5_2_0_, product0_.total_investing_amount as total_in6_2_0_ from product product0_ where product0_.id=?
2022-06-12 11:55:35.014  WARN 99615 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 1146, SQLState: 42S02
2022-06-12 11:55:35.014 ERROR 99615 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : Table 'member.product' doesn't exist
2022-06-12 11:55:35.023  INFO 99615 --- [nio-8080-exec-1] o.h.e.internal.DefaultLoadEventListener  : HHH000327: Error performing load command
```

일단 조회부터 해올수 있게 하려면 어떻게 해야 될까 고민을 하다가 그렇다면 서비스 모듈이 아닌 애초에 각 도메인 모듈 내에서 조회해오고 서비스 모듈로 리턴만 해오게 하면 될까? 싶었고 바로 실행해보았다.

```java
public class ProductCommandService {

    private final MemberDomainService memberDomainService;
    private final ProductDomainService productDomainService;
    
    public void update() {
        Product product = productDomainService.findBy(1L);
        Member member = memberDomainService.findBy(1L);
    }
}

public class MemberDomainService {

    private final MemberRepository memberRepository;

    @Transactional(readOnly = true)
    public Member findBy(long id) {
        return memberRepository.findById(id).orElseThrow();
    }
}

public class ProductDomainService {

    private final ProductRepository productRepository;

    @Transactional(readOnly = true)
    public Product findBy(long id) {
        return productRepository.findById(id).orElseThrow();
    }
}
```

하지만 같은 에러가 발생하였다. 애초에 애플리케이션이 구동될때 하나의 datasource만 트랜잭션 설정이 된 상태인듯 보였다.

먼저 생각해본 해결책은 각 domain1과 domain2 마다 별도의 transactionManager를 설정해주는 방법이었다.

### 모듈별 transactionManager 설정

* dataSource 설정

```java
@Configuration
public class Domain1DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public DataSource dataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "domain1DataSource")
    public DataSource domain1DataSource() {
        return new LazyConnectionDataSourceProxy(dataSource());
    }

}

@Configuration
public class Domain2DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.domain2.hikari")
    public DataSource domain2OriginDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "domain2DataSource")
    public DataSource domain2DataSource() {
        return new LazyConnectionDataSourceProxy(domain2OriginDataSource());
    }

}
```

* entity/transactionManager 설정

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.jd.domain",
    entityManagerFactoryRef = "domain1entityManagerFactory",
    transactionManagerRef = "domain1TransactionManager"
)
public class Domain1DataManagerConfig {

    protected void setConfigureEntityManagerFactory(LocalContainerEntityManagerFactoryBean factory) {
        JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setJpaPropertyMap(Map.of("hibernate.hbm2ddl.auto", "none",
                                         "hibernate.dialect", "org.hibernate.dialect.MySQL5InnoDBDialect",
                                         "hibernate.show_sql", "true",
                                         "hibernate.format_sql", "true"));
        factory.afterPropertiesSet();
    }

    @Bean
    @Primary
    public EntityManagerFactory domain1entityManagerFactory(DataSource dataSource) {
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setPackagesToScan("com.jd.domain");
        factory.setPersistenceUnitName("entityManagerFactory");
        setConfigureEntityManagerFactory(factory);
        return factory.getObject();
    }

    @Bean
    @Primary
    public PlatformTransactionManager domain1TransactionManager(EntityManagerFactory domain1entityManagerFactory) {
        JpaTransactionManager tm = new JpaTransactionManager();
        tm.setEntityManagerFactory(domain1entityManagerFactory);
        return tm;
    }
}

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.jd.domain2",
    entityManagerFactoryRef = "domain2EntityManagerFactory",
    transactionManagerRef = "domain2TransactionManager"
)
public class Domain2DataManagerConfig {

    protected void setConfigureEntityManagerFactory(LocalContainerEntityManagerFactoryBean factory) {
        JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setJpaPropertyMap(Map.of("hibernate.hbm2ddl.auto", "none",
                                         "hibernate.dialect", "org.hibernate.dialect.MySQL5InnoDBDialect",
                                         "hibernate.show_sql", "true",
                                         "hibernate.format_sql", "true"));
        factory.afterPropertiesSet();
    }

    @Bean(name = "domain2EntityManagerFactory")
    public EntityManagerFactory entityManagerFactory(@Qualifier("domain2DataSource") DataSource dataSource) {
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setPackagesToScan("com.jd.domain2");
        factory.setPersistenceUnitName("domain2EntityManager");
        setConfigureEntityManagerFactory(factory);
        return factory.getObject();
    }

    @Bean(name = "domain2TransactionManager")
    public PlatformTransactionManager transactionManager(@Qualifier("domain2EntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager tm = new JpaTransactionManager();
        tm.setEntityManagerFactory(entityManagerFactory);
        return tm;
    }
}
```

이렇게 하니 결국 성공! 하는 듯 했으나 여기서 더 나아가보자.

각각의 domain 모듈 별로 transactionManager를 관리하면 트랜잭션 단위도 해당 도메인 모듈 내에서 각각 동작시키는 방식으로만 사용된다.

> domain2에 대한 commit/rollback은 동작하지 않는다.

하지만 여기서 더 나아가 두 도메인 모듈을 하나의 트랜잭션으로 관리하려면? 아래와 같이 구현하면 될 것 같지만 @Primary 빈으로 설정된 transactionManager에 대한 트랜잭션만 동작되게 되면서 다른 도메인 모듈의 트랜잭션은 동작하지 않는다.

```java
@Transactional
public void update() {
    Product product = productDomainService.findBy(1L);
    Member member = memberDomainService.findBy(1L);

    product.setTitle("old");
    member.setName("hi");
}
```

> product 테이블에 title 컬럼에만 old가 update되고 member 테이블은 Update 쿼리가 동작하지 않는다.

그렇면 어떻게 해야 될까?

열심히 구글링을 해보았고 해결할 수 있는 열쇠가 될 법한 방법을 찾아보았다. 

## ChainedTransactionManager 사용

```java
@Configuration
public class ChainedTransactionConfig {

    @Bean("multiTransactionManager")
    public PlatformTransactionManager transactionManager(PlatformTransactionManager d1TxManager, @Qualifier("domain2TransactionManager") PlatformTransactionManager d2TxManager) {
        return new ChainedTransactionManager(d1TxManager, d2TxManager);
    }

}

@Transactional(transactionManager = "multiTransactionManager")
public void update() {
    Product product = productDomainService.findBy(1L);
    Member member = memberDomainService.findBy(1L);

    product.setTitle("new");
    member.setName("hi");
}
```

이렇게 하면 product 엔티티와 member 엔티티 모두 성공적으로 commit 된다.

하지만 rollback 테스트를 해보면 예상과 다르게 동작한다.

### ChainedTransactionManager 예외 rollback

```java
@Transactional(transactionManager = "multiTransactionManager")
public void update() {
    Product product = productDomainService.findBy(1L);
    Member member = memberDomainService.findBy(1L);

    product.setTitle("old");
    member.setName("black_consumer");

    validate(member);
}

private void validate(Member member) {
    if (member.getName().equals("black_consumer")) {
        throw new CommonException("error");
    }
}
```

여기서 일반적으로 우리가 예상할 수 있는 트랜잭션 롤백 동작 결과라면 product, member 모두 롤백이 되어야 한다. 

하지만 ChainedTransactionManager는 말그대로 복수개의 transactionManager를 연결시켜 하나로 묶어줄 뿐이지 하나의 트랜잭션으로 동작되는 것이 아니다.

즉, `원자성을 기대했으나 성립되지 않는 구현 방법`이다.

따라서 product 엔티티에 대한 트랜잭션은 정상적으로 commit이 되지만 member 엔티티에 대한 트랜잭션은 checked exception에 의해 rollback 처리가 되게되면서 product와 member 트랜잭션은 따로 놀게 된다.

## JtaTransactionManager 사용

이를 해결할 수 있는 방법은 JtaTransactionManager을 사용하는 것이다.

JtaTransactionManager은 JTA(Java Transaction Api) 자바 표준으로 XA Resource 인터페이스의 구현체들을 등록하여 글로벌 transaction 형태로 동작된다.

jta 적용은 다음 번에 작성해보려 한다..



# 참고자료

- [글로벌/로컬 트랜잭션이란](http://ldg.pe.kr/framework_reference/spring/ver1.2.2/html/transaction.html)
- [LazyConnectionDataSourceProxy이란](https://sup2is.github.io/2021/07/08/lazy-connection-datasource-proxy.html)
- [다중 DataSource 설정 방법](https://supawer0728.github.io/2018/03/22/spring-multi-transaction/)
- [JTA를 활용한 트랜잭션 처리](https://d2.naver.com/helloworld/5812258)
- [JTA + JPA 예제](https://www.4te.co.kr/894)
