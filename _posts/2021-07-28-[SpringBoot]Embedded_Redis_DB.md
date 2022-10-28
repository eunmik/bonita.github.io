---
layout: post
title: "[Spring Boot] Embedded Redis DB 설정"
date: 2021-07-28
excerpt: "Embedded Redis DB 사용해보기"
tags: [Java, Redis, Spring Boot]
comments: true
---
InMemory에 데이터를 저장해야 할 경우가 생겨서 InMemory DB중 하나인 Redis를 사용해 보았다. 

Redis는 RDBMS가 아니라  Key-Value 구조 데이터인 NoSQL이다. 

Redis같은 InMemory DB 설명은 나중에! 

스프링부트에서 Redis를 사용하려면 

1. pom.xml에 라이브러리를 추가한다. 

    ```java
    <!-- Redis : inmemory DB -->
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
    		<dependency>
    			<groupId>it.ozimov</groupId>
    			<artifactId>embedded-redis</artifactId>
    			<version>0.7.2</version>
    			<scope>compile</scope>
    		</dependency>
    ```

2. application.yaml 파일에 redis db 정보 설정한다.  embedded db 니깐 host는 127.0.0.1

    ```java
    ##redis
    spring:
      redis:
        lettuce:
          pool:
            max-active: 10
            max-idle: 10
        port: 6379
        host: 127.0.0.1
      main:
        allow-bean-definition-overriding: true
    ```

1. Redis 설정을 위한 자바 클래스를 만든다. 

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
    import org.springframework.data.redis.core.RedisTemplate;
    import org.springframework.data.redis.core.StringRedisTemplate;
    import org.springframework.data.redis.repository.configuration.EnableRedisRepositories;
    import org.springframework.data.redis.serializer.StringRedisSerializer;

    @Configuration
    @EnableRedisRepositories
    public class RedisConfig {

    		// Thread-safe factory of Redis connections.
        @Bean
        public RedisConnectionFactory redisConnectionFactory() {
    				//Connection factory creating Lettuce-based(Redis client) connections. 
    				//This factory creates a new LettuceConnection on each call to getConnection(). 
            LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory();
            return lettuceConnectionFactory;
        }
    		
    		//Helper class that simplifies Redis data access code.
        @Bean
        public RedisTemplate<String, Object> redisTemplate() {
            RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
            redisTemplate.setConnectionFactory(redisConnectionFactory());
            redisTemplate.setKeySerializer(new StringRedisSerializer());
            redisTemplate.setValueSerializer(new StringRedisSerializer());
            return redisTemplate;
        }
    		
    		//String-focused extension of RedisTemplate. 
        @Bean
        public StringRedisTemplate stringRedisTemplate() {
            StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
            stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
            stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
            stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
            return stringRedisTemplate;
        }

    }
    ```

    ```java
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Configuration;
    import redis.embedded.RedisServer;

    import javax.annotation.PostConstruct;
    import javax.annotation.PreDestroy;
    import java.io.IOException;

    @Slf4j
    @Configuration
    public class RedisServerConfig {
        private RedisServer redisServer;

        public RedisServerConfig(@Value("${spring.redis.port}") int redisPort) {

            redisServer = new RedisServer(redisPort);
        }

        @PostConstruct
        public void startRedis() throws IOException {
             redisServer.start();
        }

        @PreDestroy
        public void stopRedis() {

            redisServer.stop();
        }
    }
    ```

2. Redis DB를 사용하기 위해서는 Entity에 @RedisHash 어노테이션이 필요하다. 

    ```java
    import lombok.Data;
    import org.springframework.data.redis.core.RedisHash;
    import org.springframework.data.redis.core.index.Indexed;

    import javax.persistence.*;
    import java.io.Serializable;
    import java.time.LocalDateTime;

    @Data
    @Entity
    @Table(name="userinfo_login")
    @RedisHash("UserLogin")
    public class UserLogin implements Serializable {
        private static final long serialVersionUID = -587852912349881733L;

        @Id
        @Indexed
        private String id; //userId
        private int loginAttempts;
        private LocalDateTime lockedTime;
        private Boolean isLocked = false;
        private String failReason;

        public UserLogin(){

        }

        public UserLogin(String userId, int loginAttempts, LocalDateTime lockedTime, Boolean isLocked, String failReason){

            this.id = userId;
            this.loginAttempts = loginAttempts;
            this.lockedTime = lockedTime;
            this.isLocked = isLocked;
            this.failReason = failReason;

        }

    }
    ```

    - @Id 어노테이션 붙은 변수의 이름이 Id가 아니면 @Id가없다고 에러가 난다.
    - @Indexed 어노테이션을 붙이지 않으면 검색할 때 찾지 못한다.
    - @RedisHash는 Redis Hash에 저장할때 구분하기 위한 마크 같은 것이다.

1. Repository 생성한다.  CrudRepository는 Redis 전용 Repositorys는 아니다. 

    ```java
    @Repository
    public interface UserLoginRepository  extends CrudRepository<UserLogin, String> {
    }
    ```

2. Redis DB에 데이터가 잘 들어오는지 확인 하기 위해서는 cmd 창에서 아래 명령어를 친다. 

    ```java
    > telnet localhost 6379
    > monitor 

    ```

    그러면 데이터가 생성되거나 검색될 때마다 로그를 확인할 수 있다. 

    <img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0728/img1.png" />