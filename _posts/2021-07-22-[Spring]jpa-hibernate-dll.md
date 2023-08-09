---
layout: post
title: "[Spring] jpa-hibernate-dll 옵션"
date: 2021-07-22
excerpt: "jpa-hibernate-dll 옵션 종류"
tags: [jpa, hibernate]
comments: true
---
Spring JPA에서 Hibernate를 이용하여 DDL을 생성하여 Data Table을 자동으로 생성할 수 있다. 

## DDL이란?

데이터 정의어 (Data Definition Language, DDL) 

데이터 베이스의 테이블 생성, 면경, 삭제를 담당하는 명령어 : 

CREATE, ALTER, DROP, RENAME, TRUNCATE 

## Hibernate 의 ddl-auto

Spring JPA에서 application.yaml에 JPA관련 설정 중 ddl을 자동으로 설정할 수 있는 기능이이다.

spring.jpa.hibernate.ddl-auto 

- hibernate란 jpa를 구현하여 사용하기 편리하도록 만든 구현체
- ddl-auto 옵션 :
    - none
    - update : 테이블의 내용이 변경된 경우 자동으로 ddl 실행
    - create : 프로그램 시작 시 create
    - create-drop : 프로그램 시작시 create, 종료 시 drop
    - validate : 테이블 내용이 변경되면 변경 내용을 출력하고 프로그램 종료, never update DB schema.  —> 음.. 정확히 어떨때 쓰는지 아직 모르겠다.

ddl-auto 옵션 테스트 해보기 

공통 : account table이 자동으로 생성된다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img1.png" />

none 일 때 :

test 유저를 회원가입 했다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img2.png" />

restart 후 데이터가 그대로 유지가 된다. 

update 일 때 : restart 후에도 데이터가 그대로 유지된다. 

create 일 때 : 

회원 한명을 생성 하고 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img3.png" />

shutdown 후 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img4.png" />

start 후 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img5.png" />

create-drop 일 때: 

회원 한명을 생성 하고 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img6.png" />

shutdown 후 아예 테이블이 없어졌다 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0722/img7.png" />

validate 일 떄 : 

어플리케이션 시작할 때 테이블을 자동으로 생성해주지 않는다. 

restart 후 데이터가 유지 된다.