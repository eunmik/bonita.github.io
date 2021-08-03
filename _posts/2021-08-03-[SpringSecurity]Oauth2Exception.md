---
layout: post
title: "[Spring Security] Oauth2 Exception 처리"
date: 2021-08-03
excerpt: "Spring Security에서 발생하는 Oauth2 Exception 처리"
tags: [Java, Oauth2, Spring Security, Exception]
comments: true
---
Spring Security에서 반환하는 에러 메시지는 아래처럼 형식이 달라서 필요한 데이터가 있을때는 코드를 작성해 줘야한다. 

```java
{
    "error": "invalid_grant",
    "error_description": "Invalid username or password"
}
```

1. Oauth2Exception을 상속받은 class를 만든다. 

    ```java
    import com.fasterxml.jackson.databind.annotation.JsonSerialize;
    import org.springframework.security.oauth2.common.exceptions.OAuth2Exception;

    @JsonSerialize(using  = CustomOauthExceptionSerializer.class)
    public class CustomOauthException extends OAuth2Exception {
        public CustomOauthException(String msg){
            super(msg);
        }
    }
    ```

    @JsonSerialize : marshalling 과정에서 custom serializer를 사용하도록 지정한다. 

    marshalling과 serialization은 차이가 있다. serialization은 객체를 byte stream으로 변환하는 것 만을 의미하는 반면 marshalling은 객체를 저장이나 전송을 위해 적당한 자료형태로 변형하는것을 의미한다. 

    Jackson 라이브러리의 StdSerializer를 상속하여 custom serializer를 만들 수 있다. 

2. JsonSerialize 어노테이션을 추가 했기 때문에 우리는 CustomOauthExceptionSerializer를 사용할 것이다. 

    ```java
    import com.fasterxml.jackson.core.JsonGenerator;
    import com.fasterxml.jackson.databind.SerializerProvider;
    import com.fasterxml.jackson.databind.ser.std.StdSerializer;
    import org.json.JSONObject;

    import java.io.IOException;
    import java.time.LocalDateTime;
    import java.util.HashMap;
    import java.util.Map;

    //Oauth2Exception을 Customizing 하기 위한 클래스
    public class CustomOauthExceptionSerializer extends StdSerializer<CustomOauthException> {
        public CustomOauthExceptionSerializer(){
            super(CustomOauthException.class);
        }

        @Override
        public void serialize(CustomOauthException value, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException{
            JSONObject json = new JSONObject(value.getMessage());
            Map<String, Integer> map = new HashMap<>();
            map.put("login_attempts", (Integer) json.get("login_attempts"));
            map.put("IP_login_attempts", (Integer) json.get("IP_login_attempts"));

            jsonGenerator.writeStartObject();
            jsonGenerator.writeObjectField("timestamp", LocalDateTime.now());
            jsonGenerator.writeNumberField("status", value.getHttpErrorCode());
            jsonGenerator.writeObjectField("error", value.getOAuth2ErrorCode());
            jsonGenerator.writeObjectField("message", json.get("message"));
            jsonGenerator.writeObjectField("data", map);
            if(value.getAdditionalInformation() != null){
                for (Map.Entry<String, String> entry : value.getAdditionalInformation().entrySet()) {
                    String key = entry.getKey();
                    String add = entry.getValue();
                    jsonGenerator.writeStringField(key, add);
                }
            }
            jsonGenerator.writeEndObject();

        }
    }
    ```

3. CustomOauthException를 Oauth2 Config에 설정한다. (WebResponseExceptionTranslator를 설정해주는 건데 따로 class를 만들어서 적용할 수도 있다.) 

    ```java
    @Configuration
    @EnableAuthorizationServer
    public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    	@Override
    	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

    		endpoints
    				.authenticationManager(authenticationManager)
    				.tokenStore(tokenStore())
    				.userDetailsService(userDetailsService())
    				//for authentication error exception
    				**.exceptionTranslator(exception -> {

    					if (exception instanceof OAuth2Exception) {
    						OAuth2Exception oAuth2Exception = (OAuth2Exception) exception;
    						return ResponseEntity
    								.status(oAuth2Exception.getHttpErrorCode())
    								.body(new CustomOauthException(oAuth2Exception.getMessage()));
    					} else {
    						throw new Exception();
    					}

    				});**

    	}
    ```

    따로 class를 만들어서 적용할 경우 

    ```java
    import org.springframework.http.ResponseEntity;
    import org.springframework.security.oauth2.common.exceptions.OAuth2Exception;
    import org.springframework.security.oauth2.provider.error.WebResponseExceptionTranslator;

    public class CustomWebResponseExceptionTranslator implements WebResponseExceptionTranslator {
        @Override
        public ResponseEntity<OAuth2Exception> translate(Exception exception) throws Exception {
            if (exception instanceof OAuth2Exception) {
                OAuth2Exception oAuth2Exception = (OAuth2Exception) exception;
                return ResponseEntity
                        .status(oAuth2Exception.getHttpErrorCode())
                        .body(new CustomOauthException(oAuth2Exception.getMessage()));
            } else {
                throw new Exception();
            }

        }
    }
    ```

    AuthorizationServerConfig.java

    ```java
    @Bean
    	public WebResponseExceptionTranslator webResponseExceptionTranslator(){
    		return new CustomWebResponseExceptionTranslator();
    	}

    	@Override
    	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

    		endpoints
    				.authenticationManager(authenticationManager)
    				.tokenStore(tokenStore())
    				.userDetailsService(userDetailsService())
    				//for authentication error exception
    				.exceptionTranslator(webResponseExceptionTranslator());
    	}
    ```

4. 적용 후 나오는 에러 메시지는 아래와 같다 

    ```java
    {
        "timestamp": "2021-07-29T11:07:42.427",
        "status": 400,
        "error": "invalid_request",
        "message": "Credentials Failed.",
        "data": {
            "IP_login_attempts": 1,
            "login_attempts": 1
        }
    }
    ```
    <img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0803/img1.png" />


    출처 : https://medium.com/@beladiyahardik7/spring-security-custom-oauth2exception-in-spring-b35a62af4d34