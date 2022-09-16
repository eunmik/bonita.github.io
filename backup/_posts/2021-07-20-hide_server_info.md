---
layout: post
title: "NGINX/TOMCAT SERVER 정보 숨기기"
date: 2021-07-20
excerpt: "서버 버전 정보를 노출시키면 안된다. "
tags: [NGINX, TOMCAT]
comments: true
---
서버 버전정보를 header에 노출 시키기 되면 웹 공격에 사용될 우려가 있기 때문에 

서버 정보를 숨겨야 할 필요가 있다. 

## nginx 경우

1. /etc/nginx/nginx.conf 파일을 연다. 
2. 설청파일의 내용중 아래와 같은 내용이 주석이 되어있다면, 해제하고 없으면 추가로 작성한다.

```java
http{
	server_tokens off;
...
```

1. nginx 재시작

## Tomcat 경우 (Docker인 경우)

Dockerfile 에 tomcat 에러리포팅 끄기, 서버정보 끄기 설정을 추가한다. 

```bash
RUN sed -i "s,</Host>,<Valve className=\"org.apache.catalina.valves.ErrorReportValve\" showReport=\"false\" showServerInfo=\"false\" /></Host>,g" /usr/local/tomcat/conf/server.xml
RUN echo "tomcat.util.http.parser.HttpParser.requestTargetAllow=|{}"  >> /usr/local/tomcat/conf/catalina.properties
```

sed : Stream Editor. 파일에서 찾기, 바꾸기, 삽입, 지우기 등 다양한 기능을 수행한다. sed 명령어를 사용해서 파일을 열지않고도 수정할 수 있다. 

```bash
sed OPTIONS... [SCRIPT] [INPUTFILE...]

-i[SUFFIX], --in-place[=SUFFIX]
                 edit files in place (makes backup if SUFFIX supplied)
```

아래껀 특수문자가 URL 에 넘어와서 공격하는걸 막는 것이다. 
[https://youngclown.github.io/2017/12/TOMCAT8_5](https://youngclown.github.io/2017/12/TOMCAT8_5) 참고하면 좋을 듯