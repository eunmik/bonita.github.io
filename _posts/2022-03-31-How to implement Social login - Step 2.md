---
layout: post
title: "How to implement Social login - Step 2. Spring Boot + Social Login"
date: 2022-03-31
categories: [Language, Java]
tags: ["API", "Login", "Oauth2", "Java"]
comments: true
---

Google OAuth2 Login API Flow

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2022/0331/Untitled.png">

# Spring Boot + Social Login

## Dependency Requirement

```xml
<dependency>
		<groupId>io.jsonwebtoken</groupId>
		<artifactId>jjwt-api</artifactId>
		<version>0.11.2</version>
</dependency>
<dependency>
		<groupId>io.jsonwebtoken</groupId>
		<artifactId>jjwt-impl</artifactId>
		<version>0.11.2</version>
		<scope>runtime</scope>
</dependency>
<dependency>
		<groupId>io.jsonwebtoken</groupId>
		<artifactId>jjwt-jackson</artifactId>
		<version>0.11.2</version>
		<scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-client</artifactId>
</dependency>
```

## application.yml

add registration information and provider information to application.yml

```yaml
security:
      oauth2:
        client:
          registration:
            google:
              clientId: *YOUR CLIENT ID*
              clientSecret: *YOUR CLIENT SECRET*
              redirectUri: "{baseUrl}/oauth2/callback/{registrationId}"
              scope:
                - email
                - profile
            facebook:
              clientId: *YOUR CLIENT ID*
              clientSecret: *YOUR CLIENT SECRET*
              redirectUri: "{baseUrl}/oauth2/callback/{registrationId}"
              scope:
                - email
                - public_profile
            github:
              clientId: *YOUR CLIENT ID*
              clientSecret: *YOUR CLIENT SECRET*
              redirectUri: "{baseUrl}/oauth2/callback/{registrationId}"
              scope:
                - user:email
                - read:user
            naver:
              client-id: *YOUR CLIENT ID*
              clientSecret: *YOUR CLIENT SECRET*
              client-authentication-method: post
              authorization-grant-type: authorization_code
              scope: name,email,profile_image
              client-name: Naver
              redirect-uri: "{baseUrl}/oauth2/callback/{registrationId}"
            kakao:
              client-id: *YOUR CLIENT ID*
              clientSecret: *YOUR CLIENT SECRET*
              client-authentication-method: post
              authorization-grant-type: authorization_code
              scope: profile_nickname,account_email,profile_image
              client-name: Kakao
              redirect-uri: "{baseUrl}/oauth2/callback/{registrationId}"

          provider:
            facebook:
              authorizationUri: https://www.facebook.com/v3.0/dialog/oauth
              tokenUri: https://graph.facebook.com/v3.0/oauth/access_token
              userInfoUri: https://graph.facebook.com/v3.0/me?fields=id,first_name,middle_name,last_name,name,email,verified,is_verified,picture.width(250).height(250)
            naver:
              authorization-uri: https://nid.naver.com/oauth2.0/authorize
              token-uri: https://nid.naver.com/oauth2.0/token
              user-info-uri: https://openapi.naver.com/v1/nid/me
              user-name-attribute: response
            kakao:
              authorization-uri: https://kauth.kakao.com/oauth/authorize
              token-uri: https://kauth.kakao.com/oauth/token
              user-info-uri: https://kapi.kakao.com/v2/user/me
              user-name-attribute: id
```

## Configuration

### SecurityConfig.java

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(
        securedEnabled = true,
        jsr250Enabled = true,
        prePostEnabled = true
)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

	...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
			     ...
                .oauth2Login()
                    .authorizationEndpoint()
                        .baseUri("/oauth2/authorize")
                        .authorizationRequestRepository(cookieAuthorizationRequestRepository())
                        .and()
                    .redirectionEndpoint()
                        .baseUri("/oauth2/callback/*")
                        .and()
                    .userInfoEndpoint()
                        .userService(customOAuth2UserService)
                        .and()
                    .successHandler(oAuth2AuthenticationSuccessHandler)
                    .failureHandler(oAuth2AuthenticationFailureHandler);

        // Add our custom Token based authentication filter
        http.addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```

### AuthProvider.java

```java
public enum  AuthProvider {
    local,
    facebook,
    google,
    github,
    naver,
    kakao
}
```

### GoogleOAuth2UserInfo.java

```java
public class GoogleOAuth2UserInfo extends OAuth2UserInfo {

    public GoogleOAuth2UserInfo(Map<String, Object> attributes) {
        super(attributes);
    }

    @Override
    public String getId() {
        return (String) attributes.get("sub");
    }

    @Override
    public String getName() {
        return (String) attributes.get("name");
    }

    @Override
    public String getEmail() {
        return (String) attributes.get("email");
    }

    @Override
    public String getImageUrl() {
        return (String) attributes.get("picture");
    }
}
```

### OAuth2UserInfoFactory.java

```java
public class OAuth2UserInfoFactory {

    public static OAuth2UserInfo getOAuth2UserInfo(String registrationId, Map<String, Object> attributes) {
        if(registrationId.equalsIgnoreCase(AuthProvider.google.toString())) {
            return new GoogleOAuth2UserInfo(attributes);
        } else if (registrationId.equalsIgnoreCase(AuthProvider.facebook.toString())) {
            return new FacebookOAuth2UserInfo(attributes);
        } else if (registrationId.equalsIgnoreCase(AuthProvider.github.toString())) {
            return new GithubOAuth2UserInfo(attributes);
        } else if (registrationId.equalsIgnoreCase(AuthProvider.naver.toString())) {
            return new NaverOAuth2UserInfo(attributes);
        } else if (registrationId.equalsIgnoreCase(AuthProvider.kakao.toString())) {
            return new KakaoOAuth2UserInfo(attributes);
        }

        else {
            throw new OAuth2AuthenticationProcessingException("Sorry! Login with " + registrationId + " is not supported yet.");
        }
    }
}
```

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2022/0331/Untitled%201.png">

## OAuth 2.0 Login - Advanced Configuration

`http.oauth2Login()`  provides a number of configuration options for customizing OAuth2.0 Login. 

`oauth2Login().authorizationEndpoint()` allows configuring the Authorization Endpoint.

Authorization Endpoint is used by the client to obtain authorization from the resource owner via user-agent redirection. 

`oauth2Login().tokenEndpoint()` allows configuring the Token Endpoint.

Token Endpoint is used by the client to exchange an authorization grant for an access token, typically with client authentication. 

Redirection Endpoint is used by the authorization server to return responses containing authorization credentials to the client via the resource owner user-agent. 

## Authorization Endpoint

### `AuthorizationRequestRepositiory`

`AuthorizationRequestRepository` is responsible for the persistence of the `OAuth2AuthorizationRequest` from the time toe Authorization Request is initiated to the time the Authorization Response is received (the callback). 

The default implementation of `AuthorizationRequestRepository` is `HttpSessionOAuth2AuthorizationRequestRepository`, which sotres the `OAuth2AuthorizationRequest` in the `HttpSession`. 

If you would like to provide a custom implementation of `AuthorizationRequestRepository` that stores the attributes of `OAuth2AuthorizationReuqest` in a `Cookie`, configure it as shown in the following example: 

SecurityConfig.java

```java
@Bean
    public HttpCookieOAuth2AuthorizationRequestRepository cookieAuthorizationRequestRepository() {
        return new HttpCookieOAuth2AuthorizationRequestRepository();
    }

@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
						.oauth2Login()
                 .authorizationEndpoint()
                 .baseUri("/oauth2/authorize")
                 .authorizationRequestRepository(cookieAuthorizationRequestRepository())
```

HttpCookieOAuth2AuthorizationRequestRepository.java

```java
@Component
public class HttpCookieOAuth2AuthorizationRequestRepository implements AuthorizationRequestRepository<OAuth2AuthorizationRequest> {
    public static final String OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME = "oauth2_auth_request";
    public static final String REDIRECT_URI_PARAM_COOKIE_NAME = "redirect_uri";
    private static final int cookieExpireSeconds = 180;

    @Override
    public OAuth2AuthorizationRequest loadAuthorizationRequest(HttpServletRequest request) {
        ...
    }

    @Override
    public void saveAuthorizationRequest(OAuth2AuthorizationRequest authorizationRequest, HttpServletRequest request, HttpServletResponse response) {
				...
    }

    @Override
    public OAuth2AuthorizationRequest removeAuthorizationRequest(HttpServletRequest request) {
        ...
    }

    public void removeAuthorizationRequestCookies(HttpServletRequest request, HttpServletResponse response) {
				...
    }
}
```

## Redirection Endpoint

The Redirection Endpoint is used by the Authorization Server for returning the Authorization Response (which contains the authorization credentials) to the client via the Resource Owner user-agent. 

The default Authorization Response baseUri (redirection endpoint) id /login/oauth2/code/*, which is defined in OAuth2LoginAuthenticationFilter.DEFAULT_FILTER_PROCESSES_URI.

If you would like to customize the Authorization Response baseUri, configure it as shown in the following example: 

SecurityConfig.java

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .oauth2Login()
                    .redirectionEndpoint()
                        .baseUri("/oauth2/callback/*")
```

## UserInfo Endpoint

### OAuth 2.0 UserService

`DefaultOAuth2UserService` is an implementation of an `OAuth2UserService` that supports standard OAuth 2.0 Providers.

If the default implementation does not suit your needs, you can define your own implementation of `OAuth2UserService` of standard OAuth 2.0 Provider’s. 

The following configuration demonstrates how to configure a custom `OAuth2UserService`: 

SecurityConfig.java

```java
@Autowired
    private CustomUserDetailsService customUserDetailsService;

@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
	        .oauth2Login()
                .userInfoEndpoint()
                    .userService(customOAuth2UserService)
                    .and()
```

CustomUserDetailsService.java

```java
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    UserRepository userRepository;

    @Override
    @Transactional
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {
		   ...
    }

    @Transactional
    public UserDetails loadUserById(Long id) {
			...
    }
}
```


🔖 reference
[oauth2login-advanced.html](https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/reference/html/oauth2login-advanced.html, "oauth2login")
[spring-boot-react-oauth2-social-login-demo](https://github.com/callicoder/spring-boot-react-oauth2-social-login-demo, "spring-boot-react-oauth2-social-login-demo")

