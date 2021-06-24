---
layout: post
title: "[Jersey] JAX-RA Bean Validation "
date: 2021-06-21
excerpt: "Jersey에서 Bean Validation을 해보자."
tags: [Jersey, JAX-RS, Java, Spring, STS4, Eclipse]
comments: true
---
## JAX-RS란?

JAX-RS(Java API for RESTful Web Services)는 자바 플랫폼에서 경량화된 REST 방식의 웹 애플리케이션 구현을 지원하는 자바 API이다. 

## Jersey란?

Jersey는 Java에서 RESTful 웹 서비스를 개발하기 위한 오픈 소스 프레임워크이다. JAX-RS APIs를 지원하고 JAX-RS 레퍼런스로 구현되었다. 

## Jersey에서 JAX-RS Bean Validation을 적용하기

### 1. pom.xml에 dependency를 추가한다.  (솔직히 다 필요한지는 모르겠음..)

```java
<!--  jersey dependency -->
		<!-- jersey 1 버전 -->	
		<!-- 		<dependency>
								<groupId>com.sun.jersey</groupId>
								<artifactId>jersey-server</artifactId>
								<version>1.9</version>
								<version>2.0.1</version>
						</dependency> -->
		<!-- 		<dependency>
								<groupId>com.sun.jersey</groupId>
								<artifactId>jersey-json</artifactId>
								<version>1.9</version>
						</dependency> -->

		<!-- jersey 2 버전 -->

		<dependency>
		    <groupId>org.glassfish.jersey.core</groupId>
		    <artifactId>jersey-server</artifactId>
		    <version>2.22.2</version>
		</dependency>
		<dependency>
		    <groupId>org.glassfish.jersey.containers</groupId>
		    <artifactId>jersey-container-servlet</artifactId>
		    <version>2.22.2</version>
		</dependency>
		<!-- if you are using Jersey client specific features without the server side -->
		<dependency>
		    <groupId>org.glassfish.jersey.core</groupId>
		    <artifactId>jersey-client</artifactId>
		    <version>2.22.2</version>
		</dependency>
				
		<dependency>
			<groupId>com.sun.jersey.contribs</groupId>
			<artifactId>jersey-spring</artifactId>
			<version>1.9</version>			
			  <exclusions> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-aop</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-beans</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-context</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-core</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-jdbc</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-test</artifactId> 
                </exclusion> 
                <exclusion> 
                    <groupId>org.springframework</groupId> 
                      <artifactId>spring-tx</artifactId> 
                </exclusion> 
              </exclusions>              
		</dependency>
		<dependency>
	        <groupId>org.glassfish.jersey.media</groupId>
	        <artifactId>jersey-media-json-jackson</artifactId>
	        <version>2.18</version>
  	 	 </dependency>
  	 	 <!--  -->  
  	 	 <dependency>
		    <groupId>org.glassfish.jersey.ext</groupId>
		    <artifactId>jersey-bean-validation</artifactId>
		    <version>2.19</version>
		</dependency>		
  	 	 <dependency>
		    <groupId>javax.ws.rs</groupId>
		    <artifactId>javax.ws.rs-api</artifactId>
		    <!-- <version>2.0-m02</version> -->
		    <version>2.0.1</version>
		</dependency>
```

### 2. Resource Config를 정의한다.

```java
package org.epis.outdoor.controllers;

import org.glassfish.jersey.server.ResourceConfig;
import org.glassfish.jersey.server.ServerProperties;

public class ApplicationConfig extends ResourceConfig {

public ApplicationConfig() {
	
    packages("org.epis.outdoor");
    property(ServerProperties.BV_SEND_ERROR_IN_RESPONSE, true);
	}
}
```

### 3. Resource에 Validation Annotation을 추가한다.

```java
import javax.validation.constraints.NotNull;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class Foo {
	
	@NotNull(message = "a shouldn't be null or empty")
	String a;
	String b;
	String c;
	
	public String getA() {
		return a;
	}
	public void setA(String a) {
		this.a = a;
	}
	public String getB() {
		return b;
	}
	public void setB(String b) {
		this.b = b;
	}
	public String getC() {
		return c;
	}
	public void setC(String c) {
		this.c = c;
	}
}
```

### 4. Controller에서 @valid 어노테이션을 추가한다.

```java
import javax.validation.Valid;
import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.epis.outdoor.models.farm.Foo;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;

@Path("/test")
@Controller
public class TestController {
	
	private static Logger log = LoggerFactory.getLogger(TestController.class);
	
	@POST
	@Path("/test2")
	@Consumes(MediaType.APPLICATION_JSON)
	@Produces(MediaType.APPLICATION_JSON)
	public Response test2(@Valid Foo foo){
		log.debug("=======foo.a===========" + foo.getA());
				
		return new Response("OK");
	}

}
```

### 5. Controller를 호출할 때 a 값 없이 호출 하면 메시지가 return 된다.

<img src ="https://eunmik.github.io/bonita/assets/img/210621-img1.png" />