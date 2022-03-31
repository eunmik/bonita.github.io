---
layout: post
title: "How to implement Social login - Step 1. Register Oauth Service (Google, Github, Facebook, Kakao, Naver)"
date: 2022-03-30
excerpt: "How to implement Social Login into your webservice"
tags: ["API", "Login", "Oauth2", "Java"]
comments: true
---
# Register Oauth Service

To implement social login into my web service, it is required to register oauth service on each social platform.

## Google

[https://console.cloud.google.com/apis](https://console.cloud.google.com/apis)

1. Create OAuth client ID and add ../auth/userinfo.email and ../auth/userinfo.profile to scopes 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled.png">

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%201.png">

1. Create OAuth client ID credntials and save it for later 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%202.png">

## Naver - Korean Social Platform

[https://developers.naver.com/main/](https://developers.naver.com/main/)

1. Register Application

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%203.png">

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%204.png">

## Kakao - Korean Social Platform

[https://developers.kakao.com/](https://developers.kakao.com/)

1. Register application 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%205.png">

1. Configure 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%206.png">

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%207.png">

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%208.png">

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%209.png">

1. Active Kakao Login

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%2010.png">

1. Generate Client Secret and save it for later

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0330/Untitled%2011.png">


🔖 reference : 
[https://deeplify.dev/back-end/spring/oauth2-social-login](https://deeplify.dev/back-end/spring/oauth2-social-login)
[https://antdev.tistory.com/70](https://antdev.tistory.com/70)
[spring-boot-react-oauth2-social-login-demo](https://github.com/callicoder/spring-boot-react-oauth2-social-login-demo)

