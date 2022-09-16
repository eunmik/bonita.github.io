---
layout: post
title: "[Spring] @RequestBody 어노테이션"
date: 2021-08-10
excerpt: "@RequestBody 어노테이션 활용하기"
tags: [Spring, RequestBody]
comments: true
---
@RequestBody 메소드 파라미터 어노테이션은 HTTP request body안에 있는 json value를 HttpMessageConverter를 사용해서 java ovject로 bind 한다. 

HttpMessageConverter는 Http request message를 연관된 java ovject로 변환하는 작업을 한다. 

여기에서는 요청이 있을 때 JSON을 java object로 변환한다. spring boot가 자동적으로 설정 하기 때문에 jackson이 클래스에 있으면, MappingJackson2MessageConverter가 사용된다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0810/img1.png" />

### @RequestBody를 사용한 모습

```java
@RequestMapping(value ="/", method = RequestMethod.POST)
public ResponseEntity<Car> update(@RequestBody Car car) {
	....
}
```

### Json to Java Object

@RequestMapping을 만들고 method=RequestMethod.POST로 정의한 다음에 post 요청이 들어 올때 miles의 값을 100 올려보자. 

```java
@RequestMapping(value = "/", method = RequestMethod.POST)
public ResponseEntity<Car> update(@RequestBody Car car) {

    if (car != null) {
        car.setMiles(car.getMiles() + 100);
    }

    // TODO: call persistence layer to update
    return new ResponseEntity<Car>(car, HttpStatus.OK);
}
```

request요청할 때 request body 안에는 아래 Json이 담겨져 보내진다. 

```json
{
    "color":"Blue",
    "miles":100,
    "vin":"1234"
}
```

output result는 아래 json이다. 

```json
{
    "color":"Blue",
    "miles":200,
    "vin":"1234"
}
```

### Json to arraylist

여러개의 car가 있을 때 json array를 파라미터로 맵핑할 수 있다. 

```java
@RequestMapping(value = "/cars", method = RequestMethod.POST)
public ResponseEntity<List<Car>> update(@RequestBody List<Car> cars) {

    cars.stream().forEach(c -> c.setMiles(c.getMiles() + 100));

    // TODO: call persistence layer to update
    return new ResponseEntity<List<Car>>(cars, HttpStatus.OK);
}
```

request요청할 때 request body 안에는 아래 Json이 담겨져 보내진다. 

```json
[
  {
    "color":"Blue",
    "miles":200,
    "vin":"1234"
  },
  {
    "color":"Red",
    "miles":500,
    "vin":"1235"
  }
]
```

output result는 아래 json이다. 

```json
[
  {
    "color":"Blue",
    "miles":300,
    "vin":"1234"
  },
  {
    "color":"Red",
    "miles":600,
    "vin":"1235"
  }
]
```

### 여러개 json objects를 전달할 때

여러개의 json objects를 전달하고 싶다면 전체 JSON 내용을 담고 있는 요청을 의미하는 wrapper object를 만들 필요가 있다.  Truck 예제를 만들어 보자. 

```java
public class Truck {

    private String VIN;
    private String color;
    private Integer miles;

    //...
}
```

RequestWrapper object

```java
public class RequestWrapper {

    List<Car> cars;
    Truck truck;

    //...
}
```

new @RequestMapping 

```java
@RequestMapping(value = "/carsandtrucks", method = RequestMethod.POST)
public ResponseEntity<RequestWrapper> updateWithMultipleObjects(
        @RequestBody RequestWrapper requestWrapper) {

    requestWrapper.getCars().stream()
            .forEach(c -> c.setMiles(c.getMiles() + 100));

    // TODO: call persistence layer to update

    return new ResponseEntity<RequestWrapper>(requestWrapper, HttpStatus.OK);
}
```

요청 보내기 

```json
{
  "cars":[
    {
      "color":"Blue",
      "miles":100,
      "vin":"1234"
    },
    {
      "color":"Red",
      "miles":400,
      "vin":"1235"
    }
  ],
  "truck":{
    "color":"Red",
    "miles":400,
    "vin":"1235"
  }
}
```

출처 : [https://www.leveluplunch.com/java/tutorials/014-post-json-to-spring-rest-webservice/](https://www.leveluplunch.com/java/tutorials/014-post-json-to-spring-rest-webservice/)

