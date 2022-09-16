---
layout: post
title: "[Java] MultiPartFile 이란?"
date: 2021-08-13
excerpt: "File은 어떻게 전송해야 할까?"
tags: [Spring Security, Refresh Token]
comments: true
---

## MultiPart란?

웹 클라이언트가 요청을 보낼 때, http 프로토콜의 바디 부분에 데이터를 여러 부분으로 나눠서 보내는 것으로 파일을 한번에 여러개 전송을 하면 body 부분에 파일이 여러개의 부분으로 연결되어 전송된다. 이렇게 전송되는 것을 Multipart data라고 한다. 

보통 파일을 전송할 때 사용한다.

## MultiPartFile이란?

멀티파트파일이란, 사용자가 업로드한 File을 핸들러에서 손쉽게 다를 수 있게 도와주는 매개변수 중 하나이다. 매개변수를 사용하기 위해서는 MultipartResolver Bean이 등록되어 있어야 한다. SpringBoot에서는 자동 등록을 지원하지만, sprintMVC에서는 기본으로 등록해주지 않는다. 

Spring Boot에서 MultiPartFile은 아래처럼 사용한다. 

```java
@PostMapping(value = "/uploadFile", consumes = { "multipart/form-data" })
    public Response<String> uploadFileData(                                                            @RequestPart(value="file", required=false) MultipartFile imageData) {
        return new Response(fileServerService.uploadFile(imageData));
    }
```

- 파일 업로드 시에 content-type은 multipart/form-data; 이어야 한다.
- method는 POST이어야 한다.
- consumes 속성에 타입을 설정해줘야 한다. (안그러면 file 데이터가 null로 넘어온다)
    - 이 속성을 사용하면 사용자가 Request Body에 담는 타입을 제한할 수 있다.

## Postman에서 MultipartFile 보내기

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0813/img1.png" />