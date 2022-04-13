---
date: 2020-04-08 20:43:40
layout: post
title: \@RestController와 Jackson 라이브러리의 관계
subtitle: \@RestController와 Jackson 라이브러리의 관계
description: \@RestController' 선언을 통해 jackson 라이브러리가 사용된다.
image: https://leejaedoo.github.io/assets/img/springboot.jpg
optimized_image: https://leejaedoo.github.io/assets/img/springboot.jpg
categories: spring
tags:
  - springboot
  - library
  - tech
  - json
paginate: true
comments: true
---
## @RestController
- spring4부터 생긴 개념으로 기존 @Controller + @ResponseBody과 같은 개념
- @ResponseBody는 HttpMessageConvertor를 활용하여 ResponseBody DTO객체에 **자동으로** json 형태의 컨텐츠로 변환되어 반환됨.
- springboot를 jackson라이브러리가 기본으로 내장돼있기 때문에 jackson을 사용하기 위해 별도 설정할 건 없다.(@RestController 어노테이션만 잘 선언해주면 됨)

### 참고
- [https://blog.naver.com/writer0713/221373715403](https://blog.naver.com/writer0713/221373715403)
- [https://wckhg89.github.io/archivers/understanding_jackson](https://wckhg89.github.io/archivers/understanding_jackson)
