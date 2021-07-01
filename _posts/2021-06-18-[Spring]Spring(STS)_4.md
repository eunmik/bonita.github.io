---
layout: post
title: "[Spring] Spring (STS) 4 시작하기 "
date: 2021-06-18
excerpt: "이클립스에서 스프링4 프로젝트 시작하기"
tags: [Java, Spring, STS4, Eclipse]
comments: true
---
1. Eclipse에서 Help → Eclipse MarketPlace  → STS4 설치 하기 

    <img src ="https://eunmik.github.io/bonita/assets/img/210618-img1.png" />

2. Tomcat 서버를 설정한다. 

    2-1. Servers 화면에서 Click을 한 다음에 

    <img src ="https://eunmik.github.io/bonita/assets/img/210618-img2.png" />

    2-2. Apache → Tomcat 8.5 선택 

    2-3. Browse로 Tomcat이 설치된 디렉토리를 선택

3. 프로젝트 생성하기

    3-1. File → New → Spring Legacy Project 

    Spring Legacy Project가 없으면 Market Places 에서 STS3을 설치한다. 

    <img src ="https://eunmik.github.io/bonita/assets/img/210618-img3.png" />

    3-2.  Spring MVC Project 

    <img src ="https://eunmik.github.io/bonita/assets/img/210618-img4.png" />

4. pom.xml에서 버전을 확인 한다. 

    ```xml
    <properties>
    		<java-version>1.7</java-version>
    		<org.springframework-version>4.0.0.RELEASE</org.springframework-version>
    		<org.aspectj-version>1.6.10</org.aspectj-version>
    		<org.slf4j-version>1.6.6</org.slf4j-version>
    	</properties>
    ```

5. 서버에서 기동이 되는지 확인한다. Run As → Run on Server
6. JDBC Tests를 위해 Oracle Driver를 설치한다. 

    ```xml
    <!-- JDBC -->
    		<dependency>
    		    <groupId>com.oracle.database.jdbc</groupId>
    		    <artifactId>ojdbc6</artifactId>
    		    <version>11.2.0.4</version>
    		</dependency>
    ```

7. [JDBCTests.java](http://jdbctests.java) 

    ```java
    package org.epis.outdoor;

    import static org.junit.Assert.fail;

    import java.sql.Connection;
    import java.sql.DriverManager;

    import org.junit.Test;

    import lombok.extern.log4j.Log4j;

    @Log4j
    public class JDBCTests {
    	static {
    		try {
    			Class.forName("oracle.jdbc.driver.OracleDriver");
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    	}
    	
    	@Test
    	public void testConnection() {
    		try(
    			Connection con = DriverManager.getConnection("jdbc:oracle:thin:@175.203.84.105:51521:orcl", "smartfarm", "smartfarm")){
    			log.info(con);
    		}catch(Exception e) {
    			fail(e.getMessage());
    		}
    	}

    }
    ```

    7-1. Run As → JUnit Test 

    7-2. Output 

    ```java
    INFO : org.epis.outdoor.JDBCTests - oracle.jdbc.driver.T4CConnection@5cc96746
    ```