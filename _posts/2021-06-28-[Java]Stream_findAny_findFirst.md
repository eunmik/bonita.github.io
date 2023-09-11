---
layout: post
title: "[Java] Stream().findAny vs findFirst() "
date: 2021-06-28
categories: [Language, Java]
tags: [Java, Stream]
comments: true
---

## Java 8 Stream API에 두가지 메소드 findAny() vs findFirst()

## Stream.findAny()

findAny()는 Stream에서 어떤 요소든 찾을 수 있도록 해준다. 순서와 상관없이 요소를 찾고자 할때 사용된다. 이 메소드는 Optional인스턴스를 리턴한다. 

```java
@Test
public void createStream_whenFindAnyResultIsPresent_thenCorrect() {
    List<String> list = Arrays.asList("A","B","C","D");

    Optional<String> result = list.stream().findAny();

    assertTrue(result.isPresent());
    assertThat(result.get(), anyOf(is("A"), is("B"), is("C"), is("D")));
}
```

non-parallel 작업에서는 대부분 첫번째 요소를 반환할 것이지만 보장되지는 않는다. 

## Stream.findFirst()

Stream에서 첫번째 요소를 찾는다. 그래서 시퀀스에서 첫번째 요소를 찾을 때 사용된다. 

발생 순서가 없을때는 아무 요소를 반환한다. 문서에 따르면 Stream은 정의된 발생순서가 있을수도 있고 없을수도 있다. 소스와 중간작업에 따라 달라진다. 

이 메소드도 Optional 인스턴스를 반환한다. 

```java
@Test
public void createStream_whenFindFirstResultIsPresent_thenCorrect() {

    List<String> list = Arrays.asList("A", "B", "C", "D");

    Optional<String> result = list.stream().findFirst();

    assertTrue(result.isPresent());
    assertThat(result.get(), is("A"));
}
```

출처 : [java-stream-findfirst-vs-findany](https://www.baeldung.com/java-stream-findfirst-vs-findany)