---
layout: post
title: "[Spring Boot] Exception 처리"
date: 2021-07-09
categories: [Language, Spring]
tags: [SpringBoot, Exception, ControllerAdvice]
comments: true
---

스프링을 활용한 서버 개발시에는 **일괄된 예외처리**로 일관성 있는 코드 스타일을 유지 할 필요가 있다. 
try/catch로 처리할 경우에는 예외처리를 반복적으로 작성하다보니 코드 구조에 혼란을 일으키기도 하고 정상동작과 오류처리 동작을 뒤섞는다. 
반복된 예외처리를 피하기 위해서 Spring 3.2부터 @ControllerAdvice 어노테이션이 추가되었다. 

## @ConctrollerAdvice란?

@Controller나 @RestController에서 발생한 예외를 한 곳에서 관리하고 관리하고 처리할 수 있게 도와주는 어노테이션이다. 
직접 try/catch를 통해 예외를 처리하면 **직접 처리한 것이 우선순위로** 작동하게 된다. 

예제 코드 

```java
@Slf4j
@Order(0)
@ControllerAdvice(annotations = RestController.class)
public class CustomExceptionHandler extends ResponseEntityExceptionHandler {

    public static final int ORDER = 0;
		
		@ResponseStatus(HttpStatus.BAD_REQUEST)
    @Override
    protected ResponseEntity<Object> handleMissingServletRequestParameter
            (MissingServletRequestParameterException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        log.error("MissingServletRequestParameterException", ex);
        ExceptionResponse response = new ExceptionResponse(ExceptionCode.INVALID_INPUT_VALUE, ex.getParameterName());
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }
    @ExceptionHandler(SQLException.class)
    @ResponseBody
    protected ResponseEntity<ExceptionResponse> handleSqlException(SQLException e){
        log.error("SQLException", e);
        ExceptionResponse response = new ExceptionResponse(ExceptionCode.SQL_EXCEPTION_VALUE, e.getSQLState());
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler(NullPointerException.class)
    @ResponseBody
    protected ResponseEntity<ExceptionResponse> handleNullPointerException(NullPointerException e){
        log.error("NullPointerException", e);
        ExceptionResponse response = new ExceptionResponse(ExceptionCode.NULL_POINTER_EXCEPTION_VALUE, e.getMessage());
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }

}
```

```java
@Slf4j
@Order(CustomExceptionHandler.ORDER + 1)
@ControllerAdvice
public class CustomGlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    protected ResponseEntity<ExceptionResponse> handleException(Exception e){
        ExceptionResponse response = new ExceptionResponse(ExceptionCode.EXCEPTION_VALUE);
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

Exception은 다른 클래스로 빼준 이유는 같은 클래스에 정의하면 우선순위가 높은지 
Exception.class에 해당하는 메소드가 실행이 되어 세부 처리된 Exception handle에 관한 메소드들은 실행이 되지 않아서 Order를 낮게 주어서 다른 클래스로 빼주었다. 

## 통일된 Json 객체

Exception Response 객체는 항상 동일한 객체를 가져야한다. 그렇지 않으면 클라이언트에서 예외처리를 항상 동일한 로직으로 처리하기가 어렵다. 

```java
@Getter
public class ExceptionResponse {

    private int status;
    private String message;
    private String data;
    private String error;

    public ExceptionResponse(ExceptionCode status){
        this.status = status.getCode();
        this.message = status.getMessage();
    }

    public ExceptionResponse(final ExceptionCode status, final String data){
        this.status = status.getCode();
        this.message = status.getMessage();
        this.data = data;
    }

    public ExceptionResponse(final ExceptionCode status, final String data, final String error){
        this.status = status.getCode();
        this.message = status.getMessage();
        this.data = data;
        this.error = error;
    }

}
```

## Excception Code 정의

에러 코드는 enum 타입으로 한곳에서 관리한다. 
에러 코드가 전체적으로 흩어져 있을 경우에는 코드, 메시지의 중복을 방지 하기 어력고 전체적으로 관리하는것이 매우 어렵다. 

```java
@Getter
public enum ExceptionCode {
    NOT_FOUND_USER(100, "Not Found User"),
    INVALID_INPUT_VALUE(200, "Invalid Input Value"),
    SQL_EXCEPTION_VALUE(300, "Sql Exception"),
    EXCEPTION_VALUE(400, "Exception"),
    NULL_POINTER_EXCEPTION_VALUE(500, "NullPointerException"),
    IO_EXCEPTION_VALUE(600, "IOException"),
    ILLEGAL_ARGUMENT_EXCEPTION_VALUE(700, "IllegalArgumentException"),
    ;

    private final int code;
    private final String message;

    ExceptionCode(final int code, final String message){
        this.code = code;
        this.message = message;
    }
}
```

SQL 관련 에러 발생시 아래 처럼 나온다. 

```json
{
    "status": 300,
    "message": "Sql Exception",
    "data": "42000",
    "error": null
}
```

참고한 블로그

[Spring Security + Exception 예외 처리(1)](https://niipoong.tistory.com/7) 
[Spring Guide - Exception 전략](https://cheese10yun.github.io/spring-guide-exception/)

---

## TroubleShooting

문제 : CustomExceptionHandler.java에 
MissingServletRequestParameterException 관한 method를 만들어 넣으면 서버 재시작할 때 아래와 같은 에러가 발생 한다. 

```prolog
Caused by: org.springframework.beans.BeanInstantiationException: 
Failed to instantiate [org.springframework.web.servlet.HandlerExceptionResolver]: 
Factory method 'handlerExceptionResolver' threw exception; 
nested exception is java.lang.IllegalStateException: 
Ambiguous @ExceptionHandler method mapped for 
[class org.springframework.web.bind.MissingServletRequestParameterException]: 
{public org.springframework.http.ResponseEntity com.argos_labs.rpa.la.api.controller.CustomExceptionHandler.handleMissingException(org.springframework.web.bind.MissingServletRequestParameterException), 

Caused by: java.lang.IllegalStateException: Ambiguous @ExceptionHandler method mapped for [class org.springframework.web.bind.MissingServletRequestParameterException]: {public org.springframework.http.ResponseEntity com.argos_labs.rpa.la.api.controller.CustomExceptionHandler.handleMissingException(org.springframework.web.bind.MissingServletRequestParameterException), public final org.springframework.http.ResponseEntity org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler.handleException(java.lang.Exception,org.springframework.web.context.request.WebRequest) throws java.lang.Exception}

```

이유는 CustomExceptionHandler가 ResponseEntityExceptionHandler를 상속 받고 있는데 
그 클래스에 이미 MissingServletRequestParameterException에 대한 내용이 있기 때문이다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0709/img1.png" />
<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0709/img2.png" />


그래서 재정의가 필요한 것이다. 

해결 : 

```java
@Override
    protected ResponseEntity<Object> handleMissingServletRequestParameter
            (MissingServletRequestParameterException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        log.error("MissingServletRequestParameterException", ex);
        ExceptionResponse response = new ExceptionResponse(ExceptionCode.INVALID_INPUT_VALUE);
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);

    }
```