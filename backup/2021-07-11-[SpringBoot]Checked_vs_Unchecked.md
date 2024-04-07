---
layout: post
title: "[Spring Boot] 자바 예외 구분: Checked Exception, Unchecked Exception"
date: 2021-07-11
categories: [Language, Spring]
tags: [SprintBoot, Java, Exception, Unchecked, Checked]
comments: true
---

## 예외(Exception) 란?

프로그래밍에서 예외(Exception)란 입력 값에 대한 처리가 불가능하거나, 프로그램 실행 중에 참조된 값이 잘못된 경우 등 정상적인 프로그램의 흐름을 어긋나는 것을 말한다. 그리고 자바에서 예외는 개발자가 직접 처리할 수 있기 때문에 예외 상황을 미리 예측하여 핸들링할 수 있다. 

한편, 에러(Error)는 시스템에 무엇인가 비정상적인 상황이 발생한 경우에 사용된다. 주로 자바 가상머신에서 발생시키는 것이며 에외와 반대로 이를 애플리케이션 코드에서 잡으려고 하면 안된다. (사실 잡아도 방법이 없다.) 에러의 예로는 OutOfMemoryError, ThreadDeath, StackOVerflowError 등이 있다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0711/img1.png" />
## 예외 구분

Exception은 Checked Exception과 Unchecked Exception으로 구분 할 수 있는데, 간단하게 RuntimeException을 상속하지 않는 클래스는 Checked Exception, 반대로 상속한 클래스는 Unchecked Exception으로 분류할 수 있다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0711/img2.png" />

여기서 RuntimeException은 Exception 클래스의 서브 클래스 이기 때문에 Exception의 일종이기도 하지만 자바에서는 RuntimeException과 이를 상속한 클래스를 조금 특별하게 취급한다. 명시적으로 예외 처리를 하지 않아도 되기 때문이다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0711/img3.png" />

 

### Checked Exception

- IOException
- SQLException
- DataAccessException
- ClassNotFoundException
- InvocationTargetExcception
- MalformedURLException

### UncheckedException

- NullPointerException
- ArrayIndexOutOfBaound
- IllegalArgumentException
- IllegalStateException

참고한 블로그 : [자바 예외 구분: Checked Exception, Unchecked Exception](https://madplay.github.io/post/java-checked-unchecked-exceptions)