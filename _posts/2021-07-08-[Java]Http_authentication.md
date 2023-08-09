---
layout: post
title: "[Java] Http Authentication 이란?"
date: 2021-07-08
excerpt: "Http 인증을 알아보고 구현해 보자."
tags: [Http, Authentication, Digest, Basic]
comments: true
---
## HTTP 인증 이란?

Authentication(인증) 이란 시스템이 사용자가 시스템에 접근 가능한 허용을 가지고 있는지 없는지 결정하는 프로세스이다. HTTP는 액세스 제어와 인증을 위한 프레임워크를 제공한다. 

## HTTP 인증 종류

- BASIC : 가장 일반적인 이증 방식
- Digest
- Oauth
- Token-based

## Basic Authentication

서버가 지정된 영역에 인증이 필요하다는 것을 헤더에 담아서 다시 보낸다. 사용자는 브라우저가 연결 한 아이디와 패스워드 (username + ":" +password)와 base62 인코드를 제공한다. 암호화된 String은 Authorization 헤더에 담아서 브라우저에서 각 요청 마다 보내진다. 

상대적으로 간단한 프로토콜이고 모든 주 브라우저에서 지원한다. 그러나 이 인증을 사옹하면 몇가지 단점이 있다. 

- 사용자 Credentials 를 요청에 보낸다.
- Credentails는 플레인 텍스트이다.
- 매 요청에 Credentials을 보낸다.
- 브라우즈 세션을 종료하는거 말고는 로그아웃 방법이 없다.
- cross-site request forgery (CSRF)에 취약하다.

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0708/img1.png" />


## Digest Authentication

Digest는 암호화 되지 않은 HTTP 기본 access 인증을 대체하려고 하지만 public-key나 Kerberos 인증 같이 강한 인증 프로토콜을 대체하지는 않는다. 

Digest는 사용자의 request를 네트워크 서버에서 받은 다음에 domain controller로 전송하는 인증 방식이다. 

Domain Controller는 digest session key라는 특별한 key를 처음 요청을 받은 서버에 보낸다. 

사용자는 그리고 암호화된 Response를 서버에 보낸다. 만약 사용자의 response가 올바른 형식이라면 서버는 사용자가 접근을 하도록 허용한다. 

그러나 이 인증의 단점은 digest access authentication은 클라이언트가 서버의 아이덴티티를 증멸할 수 있는 메커니즘을 제공하지 않는다. 

## OAuth Authentication

OAuth (Open Authentication/Authorization) 는 인증을 위한 표준 프로토콜이다. 구글, 페이스북, 카카오 등에서 제공하는 Authorization Server를 통해 회원 정보를 인증하고 Access Token을 발급 받는다.  비밀번호(credentials)를 사용하지 않고 한 사이트에 저장된 리소스를 다른 사이트에서 사용하도록 허용한다. 

## Token Based Authentication

토큰 기반 인증은 인증할 때 access token을 발행한다. 이 토큰은 사용되는 에코시스템에 따라 다양한 형태로 사용될 수 있다. 클라이언트는 사용자아이디와 패스워드를 access token을 얻기 위해 입력 할 수 있다. 클라이언트가 이 토큰을 갖고 있고 매 요청마다 서버에 보낸다. 클라이언트가 보낸 매 요청이 진행되는 동안, 서버는 HTTP는 stateless이기 때문에 클라이언트가 이미 인증이 되었는지 알지 못한다. 그래서 그 토큰을 통해 클라이언트가 인증이 되었는지 확인한다. 그리고나서 요청된 자원에 접근을 제공한다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0708/img2.png" />

## Digest 인증 구현하기

### Client쪽 Http Authentication 구현

### Basic Authentication

CustomBasicAuthWebSecurityConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class CustomBasicAuthWebSecurityConfig extends WebSecurityConfigurerAdapter
 {

    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("username")
                .password(passwordEncoder().encode("password"))
                .roles("USER");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        //return new BCryptPasswordEncoder();
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http .antMatcher("/**")
                .authorizeRequests()
                .anyRequest().authenticated()
                // httpBasic authentication
                .and() .csrf().disable().httpBasic();
    }

}
```

### Digest Authentication

CustomDigestAuthWebSecurityConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.www.DigestAuthenticationEntryPoint;
import org.springframework.security.web.authentication.www.DigestAuthenticationFilter;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class CustomDigestAuthWebSecurityConfig//extends WebSecurityConfigurerAdapter
{

    @Configuration
    @Order(1)
    public class AdminSecurityConfiguration extends WebSecurityConfigurerAdapter
    {
        // Configure DigestAuthenticationEntryPoint
        private DigestAuthenticationEntryPoint getDigestEntryPoint() {
            DigestAuthenticationEntryPoint digestEntryPoint = new DigestAuthenticationEntryPoint();
            digestEntryPoint.setRealmName("admin");
            digestEntryPoint.setKey("somedigestkey");
            return digestEntryPoint;
        }

        // PasswordEncoder Bean
        @Bean
        public PasswordEncoder passwordEncoder() {
            //return new BCryptPasswordEncoder();

            return NoOpPasswordEncoder.getInstance();
        }

        //Configure authentication manager

        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            auth.inMemoryAuthentication()
                    .withUser("username")
                    .password(passwordEncoder().encode("password"))
                    .roles("USER");
        }

        //default UserDetailsService configured by  UserDetailsServiceAutoConfiguration

        @Bean
        public UserDetailsService userDetailsServiceBean() throws Exception {
            return super.userDetailsServiceBean();
        }

        // Configure  DigestAuthenticationFilter.
        // Notice the we are injecting userDetailsService and DigestAuthenticationEntryPoint

        private DigestAuthenticationFilter getDigestAuthFilter() throws Exception {
            DigestAuthenticationFilter digestFilter = new DigestAuthenticationFilter();

            digestFilter.setUserDetailsService(userDetailsServiceBean());

            digestFilter.setAuthenticationEntryPoint(getDigestEntryPoint());
            return digestFilter;
        }

        //Notice that we are configuring / admin/ * * for the SecurityFilterChain
        // added DigestAuthenticationFilter, also configured the EntryPoint authentication.

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.antMatcher("/**").
                    addFilter(getDigestAuthFilter()).exceptionHandling()
                    .authenticationEntryPoint(getDigestEntryPoint())
                    .and().csrf().disable().authorizeRequests().antMatchers("/**").hasRole("USER");
        }
    }
}
```

## Server쪽 Http 인증 요청 구현

HttpRequestFactoryAuthentication.java

```java
import com.argos_labs.rpa.la.report.api.model.AuthType;
import lombok.RequiredArgsConstructor;
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScheme;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.AuthCache;
import org.apache.http.client.CookieStore;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.client.protocol.HttpClientContext;
import org.apache.http.impl.auth.BasicScheme;
import org.apache.http.impl.auth.DigestScheme;
import org.apache.http.impl.client.BasicAuthCache;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.cookie.BasicClientCookie;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.http.protocol.HttpContext;
import org.springframework.http.HttpMethod;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;

import java.net.URI;
import java.security.SecureRandom;

@RequiredArgsConstructor
public class HttpRequestFactoryAuthentication extends HttpComponentsClientHttpRequestFactory {
    private final BasicHttpContext context;
    private final AuthCache cache;
    private final CredentialsProvider credentialsProvider;

    public HttpRequestFactoryAuthentication(){
        super();
        context = new BasicHttpContext();
        cache = new BasicAuthCache();
        credentialsProvider = new BasicCredentialsProvider();

        context.setAttribute(HttpClientContext.AUTH_CACHE, cache);

    }
    public void setCredentials(HttpHost host, String authType, String username, String password){

        //AuthScheme scheme = null;
        if(authType.equals(AuthType.BasicType.value())){
            AuthScheme scheme = new BasicScheme();
            cache.put(host, scheme);
        }else if (authType.equals(AuthType.DigestType.value())){
            DigestScheme scheme = new DigestScheme();
            scheme.overrideParamter("realm","admin");
            //String nonce = Long.toString(new SecureRandom().nextLong(), 36);
            //scheme.overrideParamter("nonce", nonce);
            cache.put(host, scheme);
        }

        UsernamePasswordCredentials credentials = new UsernamePasswordCredentials(username, password);
        credentialsProvider.setCredentials(new AuthScope(host), credentials);

        CookieStore cookieStore = new BasicCookieStore();
        BasicClientCookie cookie = new BasicClientCookie("cookie","value");
        cookie.setDomain(host.getHostName());
        cookie.setPath("/");
        cookieStore.addCookie(cookie);

        setHttpClient(HttpClients.custom().setDefaultCookieStore(cookieStore).setDefaultCredentialsProvider(credentialsProvider).build());
    }
    @Override
    protected HttpContext createHttpContext(HttpMethod httpMethod, URI uri){
        return context;
    }

    public HttpClientContext createHttpClientContext(){
        HttpClientContext context = HttpClientContext.create();
        context.setCredentialsProvider(credentialsProvider);
        context.setAuthCache(cache);
        return context;

    }
}
```

HttpAuth

```java
public Response httpAuth(PamStatus pamStatus, OpenapiEndpointInfo openapiEndpointInfo) throws IOException {
        try {
            String[] domain = openapiEndpointInfo.getHost().split("://");
            String[] hostAndPort = domain[1].split(":");
            String hostname = domain[1];
            String scheme = domain[0];
            int port = (scheme.equals("https") ? 443 : 80);
            if(hostAndPort.length>1){
                hostname = hostAndPort[0];
                port = Integer.parseInt(hostAndPort[1]);
            }

            HttpHost host = new HttpHost(hostname, port, scheme);

            RestTemplate restTemplate = new RestTemplate();
            HttpRequestFactoryAuthentication requestFactory = new HttpRequestFactoryAuthentication();
            requestFactory.setCredentials(host, openapiEndpointInfo.getAuthType(), openapiEndpointInfo.getUsername(), openapiEndpointInfo.getPassword());
            restTemplate.setRequestFactory(requestFactory);

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);

            Map<String, Object> mapData = new HashMap<>();
            mapData.put("details", pamStatus.getDetails());
            mapData.put("workId", pamStatus.getWorkId());
            mapData.put("taskId", pamStatus.getTaskId());
            mapData.put("progressStatus", pamStatus.getProgressStatus());
            mapData.put("updatedDate", (pamStatus.getUpdatedDate() != null ? pamStatus.getUpdatedDate().atZone(ZoneId.of("GMT")).format(DateTimeFormatter.ISO_DATE_TIME) : null));
            ObjectWriter ow = new ObjectMapper().writer().withDefaultPrettyPrinter();
            String json = ow.writeValueAsString(mapData);

            HttpEntity<String> entity = new HttpEntity<String>(json, headers);
            restTemplate.getMessageConverters().add(new MappingJackson2HttpMessageConverter());
            String response = restTemplate.postForObject(pamStatus.getEndPoint(), entity, String.class);

            return new Response(HttpStatus.OK.getReasonPhrase(), HttpStatus.OK, response);
        }catch(Exception e){
            e.printStackTrace();
            return new Response("ENDPOINT ERROR :"+e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR, null);
        }

    }
```


참고한 출처 : 
[spring-security-modules/spring-security-web-digest-auth](https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-digest-auth)
[RestTemplate Basic Authentication Example](https://attacomsian.com/blog/resttemplate-basic-authentication)
[preemptive-authentication-rest-template](https://github.com/kytkemo/preemptive-authentication-rest-template/blob/master/src/main/java/com/kytkemo/preemptiveauthenticationresttemplate/web/client/PreemptiveAuthenticationRestTemplate.java)
[Spring Boot Security Tutorial_ Digest Authentication](https://www.techgeeknext.com/spring-boot-security/digest_authentication_web_security)
[Spring security digest authentication example](https://javadeveloperzone.com/spring-boot/spring-security-digest-authentication-example/)