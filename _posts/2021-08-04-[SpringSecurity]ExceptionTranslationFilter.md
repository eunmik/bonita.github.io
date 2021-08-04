---
layout: post
title: "[Spring Security] ExceptionTranslationFilter로 에러 처리"
date: 2021-08-04
excerpt: "Spring Security에서 발생하는 AccessDeniedException, AuthenticationException 처리"
tags: [Java, Oauth2, Spring Security, Exception]
comments: true
---
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0804/img1.png" />

Debugging 걸어서 흐름을 한번 그려보았는데 아닐수도 있다....... 나도 다른 블로거들 처럼 작동 원리를 이해하고 싶다!!! 

Spring Seucurity에 OauthException을 커스터마이징 해서 배포하니깐 Custom Message는 원래 잘 나왔던 다른 AccessDeniedExcception이 Forbidden 메시지만 뱉어내기 시작했다. 

그래서 ExceptionTranslationFilter로 Customized AccessDeniedHandler를 만들어 보자! 

1. CustomAccessDeniedHandler를 생성한다. 

    ```java
    import com.fasterxml.jackson.databind.ObjectMapper;
    import lombok.extern.slf4j.Slf4j;
    import org.json.JSONObject;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.security.access.AccessDeniedException;
    import org.springframework.security.web.access.AccessDeniedHandler;
    import org.springframework.stereotype.Component;

    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.time.LocalDateTime;
    import java.util.HashMap;
    import java.util.LinkedHashMap;
    import java.util.Map;

    //AccessDeniedException을 Customizing 하기 위한 Handler
    @Slf4j
    @Component
    public class CustomAccessDeniedHandler  implements AccessDeniedHandler {
        @Autowired
        private ObjectMapper objectMapper;

        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException)
        throws IOException, ServletException {
            if(!response.isCommitted()) { //response가 이미 client에 commited가 되어있는지 확인한다. Committed된 response는 HTTP Status와 Headers를 가지고 있고 당신이 수정할 수 있다.

                JSONObject json = new JSONObject(accessDeniedException.getMessage());
                Map<String, Object> jsonMap = new HashMap<>();
                jsonMap.put("locked_time", json.get("locked_time"));

                response.setContentType("application/json;charset=UTF-8");
                Map map = new LinkedHashMap();
                map.put("timestamp", LocalDateTime.now());
                map.put("status", "403");
                map.put("error", "Access_Denied");
                map.put("message", json.get("message"));
                map.put("data", jsonMap);
                response.setContentType("application/json");
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write(objectMapper.writeValueAsString(map));
            }
        }

    }
    ```

1. AuthEntryPoint 클래스를 생성한다. 

    ```java
    import org.codehaus.jackson.map.ObjectMapper;
    import org.springframework.security.core.AuthenticationException;
    import org.springframework.security.web.AuthenticationEntryPoint;
    import org.springframework.stereotype.Component;

    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.time.LocalDateTime;
    import java.util.LinkedHashMap;
    import java.util.Map;

    //CustomAuthException Entry Point for token Check Failure (401 unauthorized)
    @Component
    public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
        @Override
        public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authenticationException) throws ServletException{
            Map map = new LinkedHashMap();
            map.put("timestamp", LocalDateTime.now());
            map.put("status", "401");
            map.put("error", "Unauthorized");
            map.put("message", authenticationException.getMessage());
            
            response.setContentType("application/json");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            try{
                ObjectMapper mapper = new ObjectMapper();
                mapper.writeValue(response.getOutputStream(), map);
            }catch (Exception e){
                throw new ServletException();
            }

        }

    }
    ```

2. WebSeucrityConfig에 설정한다. 

```java
@Configuration
@EnableWebSecurity
class SecurityConfigurer extends WebSecurityConfigurerAdapter {

		@Bean
			public AccessDeniedHandler accessDeniedHandler(){
				return new CustomAccessDeniedHandler();
			}
		
		@Bean
		public CustomExceptionTranslationFilter customExceptionTranslationFilter() {
			CustomExceptionTranslationFilter customExceptionTranslationFilter =  new CustomExceptionTranslationFilter(new CustomAuthenticationEntryPoint());
			return customExceptionTranslationFilter;
		}

		@Component
		public class CustomExceptionTranslationFilter extends ExceptionTranslationFilter {
			public CustomExceptionTranslationFilter(AuthenticationEntryPoint authenticationEntryPoint) {
				super(authenticationEntryPoint);
				this.setAccessDeniedHandler(accessDeniedHandler());

			}
		}

}
```

설정 전에는 

```java
{
    "timestamp": "2021-07-29T05:17:04.461+0000",
    "status": 403,
    "error": "Forbidden",
    "message": "Forbidden",
    "path": "/oauth/token"
}
```

설정 후에는 

```java
{
    "timestamp": "2021-07-29T14:25:37.56",
    "status": "403",
    "error": "Access_Denied",
    "message": "account_locked",
    "data": {
        "locked_time": "2021-07-29T14:51:28.811"
    }
}
```

인증과 인가 과정에서 발생하는 예외 2가지 

- AuthenticationException (인증) 401
- AccessDeniedException (인가) 403

ExceptionTranslationFilter는 

FilterChainProxy가 호출하는 도중 발생하는 예외를 처리하는 예외처리기이다. 

AccessDeniedException과 AuthenticationException을 처리하는 필터이다. 

핵심 메서드는 HandleSpringSecurityException() 

- 인증 과정 중 AuthenticationException이 발생한다면, AuthenticationEntryPoint를 실행하여 인증을 유도한다.
- 인가 과정 중 AccessDeniedException예외가 발생하면 먼저 현재 Authentication이 익명 사용자인지 확인한다. 만약 익명이라면 AuthenticationEntryPoint를 실행하여 인증을 유도하고 아니라면 AccessDeniedHandler에게 위임한다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0804/img1.png" />
ExceptionTranslationFilter.handleSpringSecurityException