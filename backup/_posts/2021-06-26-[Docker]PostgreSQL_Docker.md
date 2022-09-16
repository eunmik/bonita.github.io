---
layout: post
title: "[Docker]PostgreSQL 도커 설치 "
date: 2021-06-26
excerpt: "Docker에 PostgreSQL 설치하기"
tags: [Docker, PostgreSQL]
comments: true
---

PostgreSQL을 사용해보고 싶어서 도커에 설치해 보았다. 

```bash
# docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=1q2w3e4r -d -v pgdata:/var/lib/postgresql/data postgres
Unable to find image 'postgres:latest' locally
latest: Pulling from library/postgres
b4d181a07f80: Pull complete
46ca1d02c28c: Pull complete
a756866b5565: Pull complete
36c49e539e90: Pull complete
664019fbcaff: Pull complete
727aeee9c480: Pull complete
796589e6b223: Pull complete
6664992e747d: Pull complete
0f933aa7ccec: Pull complete
99b5e5d88b32: Pull complete
a901b82e6004: Pull complete
625fd35fd0f3: Pull complete
9e37bf358a5d: Pull complete
Digest: sha256:d28e2df4582de09c99c85d74615e3f857b3bb6fb3d336eaa22430865cb9e08cb
Status: Downloaded newer image for postgres:latest
08b51b4dbf577b0f052713faff6ba691c8a17d096087bbba965a0d3e51304c0c

# docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED              STATUS              PORTS                                       NAMES
08b51b4dbf57   postgres   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
```

```bash
# docker exec -it postgres bin/bash
root@3a5d178cd753:/# psql -U postgres
psql (13.3 (Debian 13.3-1.pgdg100+1))
Type "help" for help.

postgres=# CREATE USER admin PASSWORD '1q2w3e4r' SUPERUSER;
CREATE ROLE
postgres=# CREATE DATABASE test OWNER admin;
CREATE DATABASE
postgres=# \c test admin
You are now connected to database "test" as user "admin".
```