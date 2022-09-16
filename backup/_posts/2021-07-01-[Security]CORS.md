---
layout: post
title: "[Security] CORS란?"
date: 2021-07-01
excerpt: "Cross-Origin-Resource-Sharing이란? "
tags: [CORS, Security]
comments: true
---
## CORS란?

Cross-Origin Resource Sharing(CORS)은 추가적인 HTTP header를 사용해서 어플리케이션이 다른 rogin의 리소스에 접근할 수 있도록 하는 메커니즘이다. 

CORS는 브라우저가 preflight 요청을 만들고 교차 리소스를 호스팅하고있는 서버가 진짜 요청을 허용할지 확인하기 위해 보내는 메커니즘을 가지고 있다. preflight에서는 브라우저는 HTTP 메소드와 진짜 요청에 사용될 헤더를 나타는 헤더를 보낸다. 

보안 문제로, 브라우저는 스크립트로부터 발생한 Cross-Origin HTTP 요청을 제한한다. 예를들어 XMLHttpRequest와 FetchAPI는 같은 origin-policy를 따르는데 이것은 이 api를 사용하는 웹 어플리케이션들은 올바른 CORS 헤더를 포함한 다른 origin에서 요청이 아닌 이상 오직 어플리케이션이 로딩된 같은 origin에서의 리소스만 요청을 할수있다는 의미이다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0701/img1.png" />

CORS 메커니즘은 안전한 cross-rogin 요청과 브라우저와 서버 사이의 데이터 전송을 지원한다. 

## Spring Boot에서 CORS 설정

@CrossOrigin 어노테이션을 통해 해당 출처에서도 스크립트를 통해 자원을 획득할 수 있도록 허용한다.   ex. @CrossOrigin(origins="http://localhost:7070") 

```java
@Slf4j
@RestController
//@CrossOrigin(origins = "*")
@CrossOrigin(origins = "${argos.rpa.api-url}")
public class UxRobotCommandController
```




