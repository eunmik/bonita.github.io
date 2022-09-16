---
layout: post
title: "[TDD] 2장 Junit and Hamcrest"
date: 2021-04-07
excerpt: "TDD 개발을 위한 기본서2"
tags: [Java, TDD]
comments: true
---
## 햄크레스트(Hamcrest) 란?

테스트 코드 내에서는 assertEquals 같은 비교 문장이 종종 사용되는데, 등호, 부등호, 비교 수준의 간략한 형태의 비교에는 한계가 있다. 또한 부등호 비교나 객체 안의 값 찾기, 문자열 안의 특정 값 찾기 등의 좀 더 다양한 판단 조건이 필요한 경우에는, 자칫 문장이 복잡해져서 판단 조건이 무엇인지 혼란스러워지기 쉽다. 이럴때 Hamcrest라는 라이브러리를 사용하면 좀 더 이해하기 쉽고 우하한 비교 문장을 만들 수 있다. 

jMock이라는 Mock 라이브러리 저자들이 참여해 만들고 있는 Matcher 라이브러리이다. 어떤 값들의 상호 일치 여부나 특정한 규칙 준수 여부 등을 판별하기 위해 만들어진 메소드나 객체를 지칭한다. 
흔히 데이터 필터링(data filtering)이나 UI 유효성 검증(validation) 등의 기능을 구현할 때 Matcher 관련 기능을 많이 사용 된다. 

> assertEquals("YoungJim", customer.getName());

> assertThat(customer.getName(), is("YoungJim"));

문장의 흐름이라는 측면에서는 아래 쪽 구분이 더 자연스럽다는 걸 알 수 있다. 

## Junit

Junit은 다음과 같은 기능을 제공한다. 

- 테스트 결과가 예상과 같은지를 판별해주는 단정문(assertions)
- 여러 테스트에서 공용으로 사용할 수 있는 테스트 픽스처(test fixture)
- 테스트 작업을 수행할 수 있게 해주는 테스트 러너(test runner)

## 테스트 픽스처

테스트를 반복적으로 수행할 수 있게 도와주고 매번 동일한 결과를 얻을 수 있게 도와주는 '**기반이 되는 상태나 환**경'을 의미한다. **일괄된 테스트 실행환경**이라고 하며, 테스트 컨텍스트(test context)라 부르기도 한다. 테스트 픽스처를 만들고, 정리하는 작업을 수행하는 메소드를 테스트 픽스처 메소드(test fixture method)라고 한다.

## AssertThat의 일반적인 사용 방법

```java
assertThat(테스트 대상, Matcher구문);
assertThat("메시지", 테스트대상, Matcher구문); 
```

<img src="https://eunmik.github.io/bonita.blog/assets/img/210407-junit.png">

## Junit5 vs Junit4

<imt src="https://repo.yona.io/files/3959">