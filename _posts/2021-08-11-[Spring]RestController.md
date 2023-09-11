---
layout: post
title: "[Spring] @RestController vs @Controller"
date: 2021-08-11
categories: [Language, Spring]
tags: [Spring, Controller]
comments: true
---
Spring MVC에서 @RestController는 @Controller 와 @ResponseBody 어노테이션을 합한 것 이다. Spring 5.0에서 RESTful 웹서비스 개발을 더 쉽게 하기 위해서 추가 되었다. 

웹 어플리케이션과 Rest API의 기본적인 차이는 

- 웹 어플리케이션은 주로 HTML+CSS +JavaScript의 뷰를 반환한다면
- REST API는 JSON 이나 XML 형태의 데이터를 반환한다.

@Controller와 @RestController에서도 차이는 극명하다. 

@Controller의 역할은 model object의 Map을 생성하고 view를 찾는 것이고 

@RestController는 단순이 object와 HttpResponse에 써있는 object data를 return 하는 것이다. 

아래 두개의 코드는 같은 역할을 한다. 

```java
@Controller
@ResponseBody
public class MVCController {
	....
}
```

```java
@RestController
public class RestFulController {
	....
}
```

출처 : [difference-restcontroller-controller-annotation-spring-mvc-rest](https://www.javacodegeeks.com/2017/08/difference-restcontroller-controller-annotation-spring-mvc-rest.html#:~:text=The%20%40Controller%20is%20a%20common,of%20%40Controller%20%2B%20%40ResponseBody%20)