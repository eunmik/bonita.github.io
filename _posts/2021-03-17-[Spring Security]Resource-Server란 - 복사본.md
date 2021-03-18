---
layout: post
title: "[Spring Security] Resource Server란"
date: 2021-03-17
tags: [spring-security, code, java]
comments: true
---

# OAUTH2 Resource Server 란?

Spring Security에서 JWT Token 설정을 하고 있다. 기본기가 많이 부족해서 설정을 하면서 여기저기 블로그/컬럼/공홈을 참고하며 하고 있는데 (정확히 내가 뭐하고 있는지 이해가 100%가 되지 않아서 슬푸다😢) 그게 문제인것 같다. Authorization Server만 적용 했을 때는 JWT Token에 아무 문제가 없었는데 Resource Server를 적용하는데 자꾸 header에 있는 JWT Token을 invalid access token이라고 return 한다. (Whyrano....!!!) 

OAUTH2, Spring Security도 이해를 돕기위해 나중에 꼭 다시 정리해 올려봐야겠다. 
오늘은 우선 Resource Server가 뭐길래 날 이렇게 힘들게 하는지 알아보기로 했다! 

## Resource Server 란?

Resource Server는 OAUTH 2.0에서 사용하는 용어이다. Resource Server는 어플리케이션이 access token을 얻고 난 후에 인증된 요청을 처리한다. 즉, 사용자에게 리소스 사용을 허락하기 전에 토큰을 증명하는 것이다. 큰 규모 같은 경우에는 여러개의 resource server를 가지고 있을 수도 있다. 예를 들어, 구글 서비스 같은 경우에는 Google Cloud Platform, Google Maps, Google Drive, Youtube, 등 20개정도의 resource server를 가지고 있다. 각 각의 resource server들은 나눠져 있지만 모드 같은 authorization server를 사용하고 있다.

작은 규모인 경우 보편적으로 오직 하나의 resource server를 가지고 있고 종종 authorization server와 같은 base에서 설치되곤 한다. 

Resource server는 access token가 있는 HTTP Authorization header를 가지고 있는 어플리케이션으로부터 요청을 받게 될 것이다. Resource server는 요청을 진행 시킬지 말지 결정을 위해 access  token을 증명해야 할 필요가 있다. token은 Autorization server의해 발행되었다.  

토큰의 증명은 아래 나열된 몇가지로 결정이 된다. 

- 설정된 autorrizaiton server에서 발행된 토큰인가?
- 토큰이 아직 유효한가?
- 사용하고자 하는 resource server인가?
- 요청된 리소스에 접근할 수 있는 인증을 토큰이 가지고 있는가?

Resource Server는 access token과 연관된 scopes을 알 필요가 있다. 서버는 만약 access token에 있는 scope list에 요청을 수행하기위해 필요한 scope이 포함되어 있지 않는다면 요청을 거부해야 한다. 

만약 access token이 요청된 resource의 접근 헌용이 되지 않거나 요청에 access token이 존재하지 않는다면 서버는 401 Response를 리턴해야하고 WWW-Authentiate 헤더를 response에 포함 시켜야 한다. 

<img src="https://eunmik.github.io/bonita/assets/img/210317-headers-401.png">

WWW-Authenticate 헤더는 최소 필요한 bearer token을 나타내는 Bearer 글자를 포함하고 있고 추가적인 정보를 가지고 있일 수도 있다. 

<img src="https://eunmik.github.io/bonita/assets/img/210317-diagram.png">

위 다이어그램에서 step 8을 보면 클라이언트 어플리케이션이 resource에 접근하기 위해 resource ㄴserver API를 호출할때, 우선 authorization server에 가서 요청에 있는 Authorization 헤더를 증명한다 그리고 나서 클라이언트에게 respone 한다. 

참고한 자료 출처 : <https://www.oauth.com/oauth2-servers/the-resource-server/>, <https://www.baeldung.com/spring-security-oauth-resource-server>

## Resource Server Config

@EnableResourceServer를 선언하여 Resource Server임을 알려주어야 한다. 
이 Config에서는 각 리소스에 대한 접근 권한을 설정해 줄 수 있다. 
auto configuration(jar dependency를 기준으로 Spring 어플리케이션을 자동으로 설정을 해주는 것) 을 사용과 jwt를 사용하기위해 library가 필요하다. 
(2.4.0 RELEASE 부터는 @EnableAutorizationServer, @EnableReosurceServer 어노테이션이 Deprecated가 되어있는데 어떻게 Migration 해야되는지 잘 모르겠다 😥) 

### pom.xml
{% highlight xml %}

<dependency>
			<groupId>org.springframework.security.oauth</groupId>
			<artifactId>spring-security-oauth2</artifactId>
			<version>2.3.8.RELEASE</version>
</dependency>
<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-jwt</artifactId>
			<version>1.1.1.RELEASE</version>
</dependency>
{% endhighlight %}

최소한의 resource server configuration은 

- dependency가 포함된다
- @EnableResourceServer 어노테이션이 포함된다.
- beare 토큰을 인증하기위한 strategy가 명시된다.

아래는 Resource Server Config의 예제이다.

### ReseourceServerConfig.java
{% highlight java %}
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenStore(tokenStore()).resourceId("SPRING_BOOT");
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {

        http
                .headers().frameOptions().disable().and()
                .authorizeRequests()
                .antMatchers("/").permitAll()
                .anyRequest().authenticated()
        ;
    }
}
{% endhighlight %}

참고 한 자료 출처 : <https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/html5/#boot-features-security-oauth2-resource-server>