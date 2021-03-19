---
layout: post
title: "[Java] Integer vs int 차이"
date: 2021-03-18
excerpt: "Interger와 int의 차이를 알아보자."
tags: [java, Integer]
comments: true
---

## Integer vs int의 차이

API를 만들면서 Controller에서 client에서 파라미터를 받았을 때 필수적인 파라미터가 아님에도 불구 하고 null이 들어오면 에러가 발생했다. 
```
Optional int parameter 'size' is present but cannot be translated into a null value due to being declared as a primitive type. Consider declaring it as object wrapper for the corresponding primitive type
```
파라미터 type을 int대신 Integer로 변경되니깐 해결이 되었다. 왜그런지 한번 알아보자! 

### Primitive 자료형 - Wrapper 클래스 관계 
int : 
- primitive 자료형 (long, float, double ...)
- 산술 연산이 가능하다.
- null로 초기화 할 수 없다.

Integer :
- Wrapper 클래스 (객체),
- Unboxing을 하지 않으면 산술 연산이 불가능 하지만, null 값을 처리할 수 있다.
- null 값 처리가 용이하기 때문에 SQL과 연동할 경우 처리가 용이하다.
- DB에서 자료형이 정수형이지만 null 값이 필요한 경우 VO에서 Integer를 사용할 수 있다.

## 기본 자료형 (Primitive data type) 이란?

실제 값을 갖는 자료형 으로 자바에서 여러 형태의 타입을 미리 정의하여 제공하는 것.
char, byte, short, int, long, flolat, double, boolean 등이 존재한다.


## Wrapper 클래스 란?

자바의 자료형은 크게 기본 타입과 참조 타입(class, interface) 으로 나누는데 
기본타입의 데이터를 객체로 표현해야 하는 경우가 있다. 
이럴때 기본 자료 타입을 객체로 다루기 위해서 사용하는 클래스 들을 래퍼 클래스라고 한다. 

### Wrapper Class 종류

| 기본타입(primitive type) | 래퍼클래스(wrapper class) |
|----------------------|----------------------|
| byte                 | Byte                 |
| char                 | Character            |
| int                  | Integer              |
| float                | Float                |
| double               | Double               |
| boolean              | Boolean              |
| long                 | Long                 |
| short                | Short                |



## Boxing과 UnBoxing

<img src="https://eunmik.github.io/bonita/assets/img/210318-boxing-unboxing.png">
기본 타입의 값을 포장 객체로 만드는 과정을 박싱, 반대로 포장객체에서 기본 타입의 값을 얻어내는 과정을 언박싱

```java
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = new Integer(17); // 박싱
        int n = num.intValue(); //언박싱
        System.out.println(n);
    }
}
```