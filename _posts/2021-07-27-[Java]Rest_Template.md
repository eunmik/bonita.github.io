---
layout: post
title: "[Java] Rest Template 사용하기"
date: 2021-07-27
categories: [Language, Java]
tags: [Java, Rest Template]
comments: true
---
Rest Template은 RESTful Web Services를 사용하는 어플리케이션을 만들기 위해 사용된다. 

exchange()메소드를 사용해서 모든 HTTP 메서드를 사용하는 웹 서비스를 사용할 수 있다. 

아래 코드는 Rest Template을 Bean으로 만들어서 자등으로 Rest Template object를 만드는 방법을 보여준다. 

```java
package com.tutorialspoint.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class DemoApplication {
   public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);
   }
   @Bean
   public RestTemplate getRestTemplate() {
      return new RestTemplate();
   }
}
```

## GET

### RestTemplate로 GET API 부르기 - exchange() method

[http://localhost:8080/products](http://localhost:8080/products) URL이 아래 JSON을 반환한다고 했을 때 RestTemplate을 사용해서 API response를 consume 해보자 

```json
[
   {
      "id": "1",
      "name": "Honey"
   },
   {
      "id": "2",
      "name": "Almond"
   }
]
```

API를 consume 하기 위해서 몇가지 포인트들이 있다. 

- Autowired the Rest Template Object.
- Use HttpHeaders to set the Request Headers.
- Use HttpEntity to wrap the request object.
- Provide the URL, HttpMethod, and Return type for Exchange() method.

```java
@RestController
public class ConsumeWebService {
   @Autowired
   RestTemplate restTemplate;

   @RequestMapping(value = "/template/products")
   public String getProductList() {
      HttpHeaders headers = new HttpHeaders();
      headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
      HttpEntity <String> entity = new HttpEntity<String>(headers);
      
      return restTemplate.exchange("
         http://localhost:8080/products", HttpMethod.GET, entity, String.class).getBody();
   }
}
```

## POST

### RestTemplate 사용해서 POST API 부르기 - exchange() method

[http://localhost:8080/products](http://localhost:8080/products) URL이 아래 response를 반환한다고 했을때 Rest Template를 사용해서 이 API Response를 consume 해보자. 

아래 코드는 Request body 이다. 

```java
{
   "id":"3",
   "name":"Ginger"
}
```

아래는 Response body 이다. 

```java
Product is created succesfully
```

API를 consume 하기 위해서 몇가지 포인트들이 있다. 

- Autowired the Rest Template Object.
- Use the HttpHeaders to set the Request Headers.
- Use the HttpEntity to wrap the request object. Here, we wrap the Product object to send it to the request body.
- Provide the URL, httpMethod, and Return type for exchange() method.

```java
@RestController
public class ConsumeWebService {
   @Autowired
   RestTemplate restTemplate;

   @RequestMapping(value = "/template/products", method = RequestMethod.POST)
   public String createProducts(@RequestBody Product product) {
      HttpHeaders headers = new HttpHeaders();
      headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
      HttpEntity<Product> entity = new HttpEntity<Product>(product,headers);
      
      return restTemplate.exchange(
         "http://localhost:8080/products", HttpMethod.POST, entity, String.class).getBody();
   }
}
```

## PUT

### RestTemplate 사용해서 PUT API 부르기 - exchange() method

[http://localhost:8080/products](http://localhost:8080/products) URL이 아래 response를 반환한다고 했을때 Rest Template를 사용해서 이 API Response를 consume 해보자. 

Request body 

```java
{
   "name":"Indian Ginger"
}
```

Response body 

```java
Product is updated successfully
```

API를 consume 하기 위해서 몇가지 포인트들이 있다. 

- Autowired the Rest Template Object.
- Use the HttpHeaders to set the Request Headers.
- Use the HttpEntity to wrap the request object. Here, we wrap the Product object to send it to the request body.
- Provide the URL, httpMethod, and Return type for exchange() method.

```java
@RestController
public class ConsumeWebService {
   @Autowired
   RestTemplate restTemplate;

   @RequestMapping(value = "/template/products/{id}", method = RequestMethod.PUT)
   public String updateProduct(@PathVariable("id") String id, @RequestBody Product product) {
      HttpHeaders headers = new HttpHeaders();
      headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
      HttpEntity<Product> entity = new HttpEntity<Product>(product,headers);
      
      return restTemplate.exchange(
         "http://localhost:8080/products/"+id, HttpMethod.PUT, entity, String.class).getBody();
   }
}
```

## DELETE

RestTemplate 사용해서 DELETE API 부르기 - exchange() method

[http://localhost:8080/products](http://localhost:8080/products)/3 URL이 아래 response를 반환한다고 했을때 Rest Template를 사용해서 이 API Response를 consume 해보자. 

아래는 Response body

```java
Product is deleted successfully
```

API를 consume 하기 위해서 몇가지 포인트들이 있다. 

- Autowired the Rest Template Object.
- Use the HttpHeaders to set the Request Headers.
- Use the HttpEntity to wrap the request object.
- Provide the URL, httpMethod, and Return type for exchange() method.

```java
@RestController
public class ConsumeWebService {
   @Autowired
   RestTemplate restTemplate;

   @RequestMapping(value = "/template/products/{id}", method = RequestMethod.DELETE)
   public String deleteProduct(@PathVariable("id") String id) {
      HttpHeaders headers = new HttpHeaders();
      headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
      HttpEntity<Product> entity = new HttpEntity<Product>(headers);
      
      return restTemplate.exchange(
         "http://localhost:8080/products/"+id, HttpMethod.DELETE, entity, String.class).getBody();
   }
}
```

출처 : [https://www.tutorialspoint.com/spring_boot/spring_boot_rest_template.htm](https://www.tutorialspoint.com/spring_boot/spring_boot_rest_template.htm)