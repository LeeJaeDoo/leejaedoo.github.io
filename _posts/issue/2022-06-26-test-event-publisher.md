---
date: 2022-06-26 15:01:40
layout: post
title: ApplicationEventPublisher를 테스트하려면?
subtitle: ApplicationEventPublisher를 테스트하려면?
description: ApplicationEventPublisher를 테스트하려면?
image: https://user-images.githubusercontent.com/15065956/175823848-375b0e62-89f4-44e3-8b47-fb95cfde1f5e.png
optimized_image: https://user-images.githubusercontent.com/15065956/175823848-375b0e62-89f4-44e3-8b47-fb95cfde1f5e.png
category: issue
tags:
- spring
- junit
- test
- tdd
- eventPublisher
paginate: true
comments: true
---
# ApplicationEventPublisher를 test하려면?

applicationEventPublisher.publishEvent() 메소드가 호출되는 횟수를 카운팅하는 test code를 작성하려는데 아래와 같은 오류가 발생하면서 제대로 test 할 수 없는 상황에 직면했다.

```text
org.springframework.context.ApplicationEventPublisher#0 bean.publishEvent(
    <any com.jd.application.ProductCommand>
);
-> at com.jd.application.service.ProductCommandTest.test(ProductCommandTest.java:74)
Actually, there were zero interactions with this mock.
```

일반적으로 ApplicationEventPublisher를 테스트할 떄 보통 @SpyBean 혹은 @MockBean으로 주입하여 test code를 작성하면 정상적으로 test code가 동작하지 않는다.

왜냐하면 @ExtendWith(SpringExtension.class) 형태로 특정 객체의 TEST CODE를 작성할 경우,

ApplicationEventPublisher는 일반적으로 외부에서 주입된 빈이 아닌 spring context 자체에서 다루는 빈이기 때문에 mockBean을 생성할 수 없다.

따라서 아래와 같이 직접 test mock bean을 생성하여 주입받아 사용해야 한다.

```java
@ExtendWith(SpringExtension.class)
@Import(ProductCommand.class)
class ProductCommandTest {

    @SpyBean
    private ProductCommand productCommand;
    @MockBean
    private ProductRepository productRepository;
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Test
    public void test() {
        //given
        Product product = mock(Product.class);
        given(product.getName()).willReturn(null);
        given(product.getId()).willReturn("ID1");
        //when
        productCommand.save(product);
        //then
        verify(eventPublisher, times(1)).publishEvent(any(Product.class));
        verify(productRepository, times(1)).save(any);
    }

    @TestConfiguration
    static class MockitoPublisherConfiguration {

        @Bean
        @Primary
        ApplicationEventPublisher publisher() {
            return mock(ApplicationEventPublisher.class);
        }
    }
}
```


> [참고링크](https://github.com/spring-projects/spring-framework/issues/18907)
