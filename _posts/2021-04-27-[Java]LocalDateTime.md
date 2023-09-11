---
layout: post
title: "[Java] LocalDateTime 이란 "
date: 2021-04-27
categories: [Language, Java]
tags: [Algorithm, BinarySearch]
comments: true
---

# LocalDate, LocalTime, LocalDateTime
Java8부터 java.time(joda.time) api가 출시 됐기 때문에 Java Version 8 이상만 가능합니다. 

## LocalDate
로컬 날짜 클래스로 날짜 정보만 필요할 때 사용, 날짜 정보만 출력된다. 

```java
LocalDate currentDate = LocalDate.now();
LocalDate targetDate = LocalDate.of(2019,11,12); 
```

## LocalTime
로컬 시간 클래스로 시간 정보만 필요할 때 사용, 시간 정보만 출력된다. 

```java
LocalTime currentTime = LocalTime.now(); 
LocalTime targetTime = LocalTime.of(12,33,35,22);
```

## LocalDateTime
날짜와 시간 정보 모두가 필요할 때 사용, 날짜와 시간정보가 출력된다. 

```java
LocalDateTime currentDateTime = LocalDateTime.now();
LocalDateTime targetDateTime = LocalDateTime.of(2019,11,12,12,32,22,3333); 
```
---

## 에러 발생
PamStatus라는 Json형식의 Entity 데이터를 HttpUrlConnection으로 다른 API에 RequestBody 파라미터로 넘겨야 되는 경우가 있었다. 
PamStatus라는 Entity안에는 LocalDateTime의 타입의 변수가 있었는데 호출을 받는 API에서 아래와 같은 에러를 남겼다. 

```java
Resolved [org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Expected array or string.; nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Expected array or string.
at [Source: (PushbackInputStream); line: 11, column: 19] (through reference chain: com.webhook.demo.WebhookParam["createdDate"])]
```

## -> 원인 : (출처 : <https://www.inflearn.com/questions/30590>)
스프링은 기본적으로 LocalDateTime을 처리할 때 ISO8601 형태를 사용하도록 되어 있습니다.(스프링이 내부에서 사용하는 Jackson라이브러리에 설정을 다 해둔 것이지요.) 그래서 오류가 발생한 것이지요.. 
대략 읽어보면 array or string을 기대했는데, 맞지 않다라는 것이지요.
그럼 어떻게 해결해야 하는가? ObjectMapper가 ISO8601 형태로 날짜 타입을 처리할 수 있게 설정을 넣어주면 됩니다.
(구글에 다음과 같이 검색하면 됩니다. jackson localdate iso 8601)

LocalDateTime형식은 json에서 아래처럼 형식이 만들어진다. 

```java
"createdDate" : {
    "nano" : 584000000,
    "year" : 2021,
    "monthValue" : 4,
    "dayOfMonth" : 26,
    "hour" : 17,
    "minute" : 25,
    "second" : 50,
    "month" : "APRIL",
    "dayOfWeek" : "MONDAY",
    "dayOfYear" : 116,
    "chronology" : {
      "id" : "ISO",
      "calendarType" : "iso8601"
    }
```

## -> 해결 : 
### Json타입을 보내는 쪽 서버에서 
application.yaml

```java
spring: 
  jackson:
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: false
```

pamStatus.java

```java
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    private LocalDateTime createdDate;
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    private LocalDateTime updatedDate;
```

### 호출 받는 서버에서

WebhookParam.java

```java
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    private LocalDateTime createdDate;

    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    private LocalDateTime updatedDate;
```

WebhookController.java

```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JSR310Module());
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
```

그러면 에러 없이 로그에 찍힌다. 🙌🙌

```java
{ "transactionId" : "newwwwfinaltest",
  "pamKey" : "B42E99EBD61E",
  "scenarioName" : "시나리오네이이임",
  "equipName" : "eunmi",
  "equipCode" : 2086,
  "isRunManager" : true,
  "isRunTray" : true,
  "runStatus" : "Ondemand",
  "details" : "최신!!!!!!!!!ヾ(•ω•`)oヾ(•ω•`)o~ ",
  "createdDate" : "2021-04-26T17:41:08.653",
  "updatedDate" : "2021-04-26T17:41:08.653",
  "connectionStatus" : null,
  "webhookUrl" : null,
  "taskId" : "thisistaskId"
}
```