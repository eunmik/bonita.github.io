---
layout: post
title: "[Windows] Windows에서 프로세스 종료하기 "
date: 2021-08-09
excerpt: "Windows에서 프로세스 찾아서 종료하기"
tags: [windows, kill]
comments: true
---
Application 시작하는데 Redis에서 에러 메시지를 뱉어냈다. 

```java
Caused by: java.lang.RuntimeException: Can't start redis server. Check logs for details.
```

그 이유는 다른 redis Instance가 중복해서 떠있기 때문이다. 

Embeded Redis를 죽이기 위해선 프로세스를 찾아서 kill 해야한다. 

1. Windows에서는 netstat 명령어를 사용해서 프로세스 이름을 찾는다. 

```java
> netstat -anb | grep 6379 -B 2 -A 2
  TCP    127.0.0.1:5939         0.0.0.0:0              LISTENING
 [TeamViewer_Service.exe]
  TCP    127.0.0.1:6379         0.0.0.0:0              LISTENING
 [redis-server-2.8.19.exe]
  TCP    127.0.0.1:6942         0.0.0.0:0              LISTENING
```

1. 작업관리자에서 해당 프로세스를 끝낸다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0809/img1.png" />