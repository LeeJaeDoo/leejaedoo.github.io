---
date: 2022-01-15 10:40:40
layout: post
title: \@ExtensionMethod란??
subtitle: \@ExtensionMethod란??
description: \@ExtensionMethod란??
image: https://leejaedoo.github.io/assets/img/lombok.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lombok.jpeg
category: issue
tags:
- issue
- lombok
- extension
paginate: true
comments: true
---

# 개요

kotlin에서 활용해보았던 extension이란 기능을 자바에서도 활용할 수 있는 방법을 우연히 알게되어 그에 대한 기록을 한번 해보려한다.

# 내용

## extension이란?

일단 extension 기능은 거창한 상속이나 디자인 패턴을 도입하지 않고도 매우 간단하게 클래스를 확장함으로써 코드를 좀 더 깔끔하게 정리해주는 효과를 준다.

예를 들자면, 이 extension 기능을 활용함으로써 우리가 쉽게 수정할 수 없었던 외부 라이브러리 클래스에 새로운 함수를 추가하는 것이 가능하다.

그럼으로써, 마치 윈래 해당 클래스에 우리가 커스텀하게 추가한 함수가 있었던 것 처럼 사용되게 할 수 있다. 이러한 함수를 `extensions property`라고 부른다.

우선 Kotlin에서 extension 기능은 아래처럼 활용되는 기능이다.

* CollectionExtensions.kt

```kotlin
inline fun <T> Iterable<T>.sumByLong(selector: (T) -> Long): Long {
  var total = 0L
  for (element in this) {
    total += selector(element)
  }
  return total
}
```

위는 Iterable 구현체 타입 내 long element value의 총 합을 return 해주는 소스다. 기존에는 일일히 위와 같은 내용의 소스를 비즈니스로직에 구현해야 했지만

위와 같이 `CollectionExtensions`라는 별도 파일 내 위 내용을 구현해두고 아래와 같이 활용하면 좀 더 간결하고 가독성 좋은 로직을 짤 수 있다.  

* ex

```kotlin
val numbers: List<Long> = listOf(1L, 2L, 3L)

numbers.sumByLong { it }
```

위와 같이 구현만 하면 총 합이 6이라는 결과를 얻어낼 수 있다.

이러한 기능은 java에서는 활용할 수 없는 줄 알고 있었는데 lombok에서 제공하는 `@ExtensionMethod` 를 활용하면 비슷하게 이를 구현해 볼 수 있다.

먼저 `@ExtensionMethod`를 활용하기 위해서는 최소 lombok 버전이 1.18.20부터 사용 가능한듯 하다(확인 필요)

그리고 extension 기능으로  활용할 extension method를 구현할 class를 만들어본다.

* CollectionUtils.java

```java

public class CollectionUtils {
    
    public static Map<Long, Map<String, Long>> toSumNestedMap(Map<Long, Map<String, Long>> m1, Map<Long, Map<String, Long>> m2) { 
        Map<Long, Map<String, Long>> mergedMap = SerializationUtils.clone(new ConcurrentHashMap<>(m1));  //  deep copy
      
        Stream.of(mergedMap, m2)
              .flatMap(map -> map.entrySet().stream())
              .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (v1, v2) -> {
                  for (Map.Entry<String, Long> e : v2.entrySet()) {
                    v1.merge(e.getKey(), e.getValue(), Long::sum);    
                  }
                  
                  return v1;
              }, ConcurrentHashMap::new));
        
        m2.forEach(mergedMap::putIfAbsent);
        
        return mergedMap;
    }
    
}

```

위 내용은 내부의 중첩된 Map을 갖고 있는 두 개의 Map을 하나의 Map으로 합치는(value를 더해줌으로써) 로직이다. 

두 개의 중첩된 Map을 합쳐주는 로직이 필요할 때마다 위 로직을 구현하는 것이 번거로우니 일반적으로 Utils static method를 생성하여 활용할 수 있게 짜본 것이다.

이를 테스트하는 테스트코드에 `@ExtensionMethod` 적용 전과 후를 비교해볼 것이다.

### Before

```java

public class CollectionUtilsTest {
    
    @Test
    @DisplayName("중첩된 map 전체 merge 검증")
    void test() {
      Map<Long, Map<String, Long>> one = new HashMap<>();
      Map<Long, Map<String, Long>> two = new HashMap<>();

      Map<String, Long> innerMap = new HashMap<>();
      innerMap.put("A", 5L);
      innerMap.put("B", 10L);
      Map<String, Long> innerMap1 = new HashMap<>();
      innerMap1.put("A", 90L);
      innerMap1.put("B", 50L);
      Map<String, Long> innerMap2 = new HashMap<>();
      innerMap2.put("A", 900L);
      innerMap2.put("B", 500L);

      one.put(1L, innerMap);
      one.put(2L, innerMap1);
      two.put(2L, innerMap2);
      two.put(3L, innerMap2);

      Map<Long, Map<String, Long>> mergedMap = CollectionUtils.toSumNestedMap(one, two);

      assertThat(one.get(2L).get("A")).isEqualTo(innerMap1.get("A")); // shallow copy 검증
      assertThat(two.get(2L).get("A")).isEqualTo(innerMap2.get("A"));

      assertThat(mergedMap.get(1L).get("A")).isEqualTo(innerMap.get("A"));
      assertThat(mergedMap.get(1L).get("B")).isEqualTo(innerMap.get("B"));
      assertThat(mergedMap.get(2L).get("A")).isEqualTo(innerMap1.get("A") + innerMap2.get("A"));
      assertThat(mergedMap.get(2L).get("B")).isEqualTo(innerMap1.get("B") + innerMap2.get("B"));
      assertThat(mergedMap.get(3L).get("A")).isEqualTo(innerMap2.get("A"));
      assertThat(mergedMap.get(3L).get("B")).isEqualTo(innerMap2.get("B"));
    }
    
}

```

### After

```java

@ExtensionMethod({CollectionUtils.class})
public class CollectionUtilsTest {
    
    @Test
    @DisplayName("중첩된 map 전체 merge 검증")
    void test() {
      Map<Long, Map<String, Long>> one = new HashMap<>();
      Map<Long, Map<String, Long>> two = new HashMap<>();

      Map<String, Long> innerMap = new HashMap<>();
      innerMap.put("A", 5L);
      innerMap.put("B", 10L);
      Map<String, Long> innerMap1 = new HashMap<>();
      innerMap1.put("A", 90L);
      innerMap1.put("B", 50L);
      Map<String, Long> innerMap2 = new HashMap<>();
      innerMap2.put("A", 900L);
      innerMap2.put("B", 500L);

      one.put(1L, innerMap);
      one.put(2L, innerMap1);
      two.put(2L, innerMap2);
      two.put(3L, innerMap2);

      Map<Long, Map<String, Long>> mergedMap = one.toSumNestedMap(two);

      assertThat(one.get(2L).get("A")).isEqualTo(innerMap1.get("A")); // shallow copy 검증
      assertThat(two.get(2L).get("A")).isEqualTo(innerMap2.get("A"));

      assertThat(mergedMap.get(1L).get("A")).isEqualTo(innerMap.get("A"));
      assertThat(mergedMap.get(1L).get("B")).isEqualTo(innerMap.get("B"));
      assertThat(mergedMap.get(2L).get("A")).isEqualTo(innerMap1.get("A") + innerMap2.get("A"));
      assertThat(mergedMap.get(2L).get("B")).isEqualTo(innerMap1.get("B") + innerMap2.get("B"));
      assertThat(mergedMap.get(3L).get("A")).isEqualTo(innerMap2.get("A"));
      assertThat(mergedMap.get(3L).get("B")).isEqualTo(innerMap2.get("B"));
    }
    
}

```

`CollectionUtils.toSumNestedMap(one, two);` 형태의 코드가 `one.toSumNestedMap(two);` 로 축약될 수 있다.

무려 메소드 명에 따라서 기존 static 클래스를 활용할 때 보다 더 직관적인 코드 이해가 가능해질 수 있다. 예를 들면 아래 코드 같은 경우는 확실히 `@ExtensionMethod`를 활용하는 것이 더 가독성이 높아진다.

* ex

```java

public class DateUtils {
    
    public static boolean isSameOrBefore(LocalDate localDate, LocalDate checkDate) {
        return localDate.isBefore(checkDate) || localDate.isEqual(checkDate);
    }
    
}

```

```java

@ExtensionMethod({DateUtils.class})
public class Test {
    
    @Test
    void test() {
        
        LocalDate localDate = LocalDate.of(2022, 1, 4);
        LocalDate checkDate = LocalDate.of(2022, 1, 5);

        assertThat(localDate.isSameOrBefore(checkDate)).isEqualTo(true);
        
    }
    
}

```

기존 같으면 `DateUtils.isSameOrBefore(localDate, checkDate)`와 같은 형태로 코드를 구현했어야 하는데 그럼 날짜 비교하는 변수에 위치에 대해서 한번 더 고민해봐야 하는 번거로움이 있을 수 있다.

하지만 위와 같이 `@ExtensionMethod`를 활용하게 되면 `localDate.isSameOrBefore(checkDate)` 와 같은 형태로 extension method 명을 통해 직관적으로 파악이 가능해진다.

이러한 부분에서 매우 편리한 기능이라고 생각한다.



하지만 단점에 대해서도 생각을 해보자면 

하나의 클래스 내에 extension 적용할 클래스를 복수로 등록하게 될 경우(Ex. `@ExtensionMethod({DateUtils.class, StringUtils.class ...})`) 오히려 혼란을 야기할 수 있을 것 같고

메소드 명을 모호하게 작명했을 경우에도 마찬가지로 혼란을 줄 수 있을 것 같다.


또한, 위 예제 에서는 테스트 코드를 통해 비교 예제를 작성했는데 compile은 문제가 없으니 실제로 테스트 환경에서 동작해볼 경우 인식하지 못한다.

테스트 환경에서 @ExtensionMethod를 왜 정상적으로 인식하지 못하는 지는 좀 더 확인이 필요할 듯 하다.
