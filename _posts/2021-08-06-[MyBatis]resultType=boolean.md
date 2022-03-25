---
layout: post
title: "[MyBatis] resultType=boolean"
date: 2021-08-06
excerpt: "resultType이 boolean일 때"
tags: [mybatis, sql]
comments: true
---
Mybatis에서 resultType을 boolean으로 사용하고 싶을 때는 아래처럼 사용하면 된다. 

```sql
<select id="isExist" resultType="boolean">
  SELECT    IF(COUNT(*) = 1, 1, 0)
  FROM    test
  WHERE    testId = #{testId}
</select>
```