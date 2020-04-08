---
date: 2020-04-08 20:31:40
layout: post
title: Jackson 라이브러리와 ObjectMapper 객체
subtitle: Jackson과 ObjectMapper의 기능
description: Jackson과 ObjectMapper를 통해 클래스 객체를 쉽게 json 형태로 변환시킬 수 있다.
image: https://leejaedoo.github.io/assets/img/springboot.jpg
optimized_image: https://leejaedoo.github.io/assets/img/springboot.jpg
category: spring
tags:
  - springboot
  - library
  - tech
  - json
paginate: true
comments: true
---
## Jackson 라이브러리
- java 객체를 json으로 변환하거나 json을 java 객체로 변환해주는 라이브러리.
- api 개발 시 요청을 받아 DTO를 거쳐 json 형태로 변환되어 응답값을 보낼 때 이 라이브러리가 사용됨.
- `@RestController` 어노테이션을 사용한 경우 요청과 응답의 객체 변환, 및 직렬화/역직렬화처리가 필요할 때 자동으로 이 라이브러리가 적용된다.
>참고 : [@RestController와 Jackson 라이브러리의 관계](https://leejaedoo.github.io/@RestControllerAndJackson.md/)
- object -> json 변환 시에는 자동으로 맡겨도 문제 없으나, 세부적인 컨트롤이 필요할 때가 있고, 그럴 때 필요한 어노테이션이 있음. 
- 더욱 세부적인 설정이 필요할 경우 **ObjectMapper 객체**를 활용하여 설정할 수 있음.

### 설정 방법 - 어노테이션
#### @JsonInclude
**Null Value 필드에 대한 제외 처리**

클래스 레벨 or 필드 레벨 모두 사용 가능
```java
    @JsonInclude(Include.NON_NULL)
    public class Member {
        ...
    }
```
```java
    public class Member {
        private int MemberNo;

        @JsonInclude(Include.NON_NULL)
        private String memberName;
    }
```
##### include 옵션(enum type)
- **Include.ALWAYS** : 기본값
- Include.NON_NULL : null 값 제외
- Include.NON_ABCENT
- Include.NON_EMPTY
    - Collection.isEmpty() == true 면 제외
    - Array.length == 0 이면 제외
    - String.length == 0 이면 제외
- Include.NON_DEFAULT
- Include.CUSTOM
- Include.USE_DEFAULTS

#### @JsonProperty
**조건에 따른 제외 처리**
```java
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
```
##### Access 옵션
- Access.AUTO
- Access.READ_ONLY
- Access.WRITE_ONLY
- Access.READ_WRITE

#### @JsonView
- 동일한 POJO 프로젝트에서 선택적으로 서로다른 프로퍼티가 조합된 json 문자열을 만들 수 있음.

### 설정방법 - 객체를 통한 선언
**ObjectMapper** 객체를 활용함.
```java
ObjectMapper jsonMapper = new ObjectMapper(); 
jsonMapper.setSerializationInclusion(Include.NON_NULL); 
ResponseDto dtoObject = new ResponseDto(); 
String jsonStr = jsonMapper.writeValueAsString(dtoObject);
```

## 참고
- [https://cchoimin.tistory.com/entry/JAVA-JSON-다루기-정리-JACKSON-ObjectMapper](https://cchoimin.tistory.com/entry/JAVA-JSON-다루기-정리-JACKSON-ObjectMapper)
- [https://yonguri.tistory.com/71](https://yonguri.tistory.com/71)
- [https://alwayspr.tistory.com/31](https://alwayspr.tistory.com/31)
- [https://github.com/TheOpenCloudEngine/uEngine-cloud-k8s/wiki/Redis-Java-Sample](https://github.com/TheOpenCloudEngine/uEngine-cloud-k8s/wiki/Redis-Java-Sample)
- [http://jeong-pro.tistory.com/170](http://jeong-pro.tistory.com/170)
- [http://dveamer.github.io/backend/SpringCacheable.html](http://dveamer.github.io/backend/SpringCacheable.html)