---
date: 2020-09-15 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 4. 람다 표현식을 이용한 설계
description: 4. 람다 표현식을 이용한 설계
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
category: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---
람다 표현식을 사용해서 로직과 함수를 쉽게 분리하고 쉽게 확장할 수 있다. 또한 평범한 인터페이스를 좀 더 활용성이 뛰어나고 직관적인 인터페이스로 변화시킬 수 있다.

# 람다를 사용한 문제의 분리
클래스를 사용하는 이유는 코드를 재사용하기 위해서이다. 좋은 방법이긴 하지만 항상 옳은 것은 아니다. 고차 함수를 사용하면 클래스의 복잡한 구조 없이도 같은 목적을 달성할 수 있다.

* stream을 활용한 람다 예제

```java
public class Asset {
    public enum AssertType {BOND, STOCK};
    private final AssertType type;
    private final int value;

    public Asset(AssertType type, int value) {
        this.type = type;
        this.value = value;
    }

    public AssertType getType() {
        return type;
    }

    public int getValue() {
        return value;
    }
}

public class AssetUtil {

    final List<Asset> assets = Arrays.asList(
        new Asset(Asset.AssertType.BOND, 1000),
        new Asset(Asset.AssertType.BOND, 2000),
        new Asset(Asset.AssertType.STOCK, 3000),
        new Asset(Asset.AssertType.STOCK, 4000)
    );

    public static int totalAssetValues(final List<Asset> assets) {
        return assets.stream()
                     .mapToInt(Asset::getValue)
                     .sum();
    }
}
```
totalAssetValues() 메서드와 같이 이터레이터(stream)를 사용한 람다 표현식을 활용할 수 있다.<br>
하지만 여기서 3가지 문제가 있다.

* 이터레이션을 어떻게 활용할지
* 어떤 값에 대한 합계를 계산할지
* 그 합계를 어떻게 구할지

이렇게 뭐가 복합적으로 뒤엉켜 있는 로직은 재사용성이 떨어진다.

* 항목에 따른 분리

```java
public static int totalBondValues(final List<Asset> assets) {
    return assets.stream()
                 .mapToInt(asset -> asset.getType() == Asset.AssertType.BOND ? asset.getValue() : 0)
                 .sum();   
}

public static int totalBondValues(final List<Asset> assets) {
    return assets.stream()
                 .mapToInt(asset -> asset.getType() == Asset.AssertType.STOCK ? asset.getValue() : 0)
                 .sum();   
}
```
분리는 했지만 반복적인 코드가 보인다. 리팩토링을 통해 개선할 여지가 충분하다.

* 리팩토링

위에서 본 코드와 같이 중복되는 함수를 보통 활용되는 인터페이스나 클래스가 아닌 람다 표현식을 활용한 strategy pattern으로 해결할 수 있다.

```java
public static int totalAssetValues(final List<Asset> assets, final Predicate<Asset> assetSelector) {
    return assets.stream()
                 .filter(assetSelector)
                 .mapToInt(Asset::getValue)
                 .sum();
}
```
함수형 인터페이스를 파라미터로 갖는 하나의 메서드로 리팩토링이 가능하다.<br>
별도로 인터페이스를 직접 생성하지 않고, java.util.function.Predicate 인터페이스를 재사용하였다. 클래스나 익명 클래스도 사용하지 않고 람다 표현식을 totalAssetValue() 메서드의 리팩토링 버전에 전달한다.

* asset의 값을 계산하는 코드

```java
System.out.println(totalAssetValues(assets, asset -> true));
System.out.println(totalAssetValues(assets, asset -> asset.getType() == Asset.AssertType.BOND));
System.out.println(totalAssetValues(assets, asset -> asset.getType() == Asset.AssertType.STOCK));
```

OCP(개방-폐쇄 원칙)를 적용하여 리팩토링 하였다. 메서드의 변경 없이 원하는 항목만 변경이 가능하게 되었다.<br>
즉, totalAssetValues() 메서드 변경 없이 원하는 항목으로 설전된 Predicate 인터페이스만 주입해주면 원하는 항목에 대한 합계를 얻을 수 있다.

# 람다 표현식을 사용하여 delegate 하기

# 람다 표현식을 활용한 decorating

# 디폴트 메서드

# 람다 표현식을 활용한 인터페이스 만들기

# 예외 처리 