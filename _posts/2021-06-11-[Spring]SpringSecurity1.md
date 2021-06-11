---
layout: post
title: "[Spring] Spring Security 기본 API 및 Filter 이해 (1) "
date: 2021-06-11
excerpt: "Spring Security의 기본 이해 1탄"
tags: [Java, Spring, Spring Security]
comments: true
---
## 1. 인증 API - 프로젝트 구성 및 의존성 추가

의존성 추가 (pom.xml)

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
</dependency>

```

의존성 추가 후 자동으로 로그인 화면이 출력이 됨 (user) 

<img src ="https://eunmik.github.io/bonita/assets/img/210611-login.png" />

스프링 시큐리티의 의존성 추가 시 일어나는 일들

- 서버가 기동되면 스프링 시큐리티의 초기화 작업 및 보안 설정이 이루어진다
- 별도의 설정이나 구현을 하지 않아도 기본적인 웹 보안 기능이 현재 시스템에 연동되어 작동함
    1. 모든 요청은 인증이 되어야 자원에 접근이 가능한다.
    2. 인증 방식은 폼 로그인 방식과 httpBasic 로그인 방식을 제공한다.
    3. 기본 로그인 페이지를 제공한다.
    4. 기본 계정 한 개 제공한다 - username:user / password : 랜덤문자열 

문제점 

- 계정추가, 권한 추가, DB연동 등
- 기본적인 보안 기능 외에 시스템에서 필요로 하는 더 세부적이고 추가적인 보안기능이 필요

## 2. 사용자 정의 보안 기능 구현

<img src ="https://eunmik.github.io/bonita/assets/img/210611-img1.JPG" />

**WebSecurityConfigurerAdapter** 는 

스프링 시큐리티의 웹 보안 기능을 초기화 하고 설정하는 클래스이다. 

WebSecurityConfigurerAdapter가 HttpSecurity를 생성하고 HttpSecurity에서는 세부적인 보안 기능을 설정할 수 있는 API를 제공하고 있다. 

사용자 정의 보안 설정 클래스라는 SecurityConfig를 만들어서 WebSecurityConfigurerAdapter를 상속 받아서 HttpSecurity로 세부적이고 추가적으로 설정할 수 있도록 할 예정이다. 

org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter

이 클래스를 상속을 받아서 configure 메소드를 override하면 우리만의 보안 설정을 할 수 있음

```java
protected void configure(HttpSecurity http) throws Exception {
        this.logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");
        http.authorizeRequests((requests) -> { 
            ((AuthorizedUrl)requests.anyRequest()).authenticated(); //어떤 요청에도 인증을 요구하겠다.
        });
        http.formLogin(); //formLogin 방식과
        http.httpBasic(); //httpBasic 방식의 인증을 요구한다. 
    }
```

사용자 정의 보안 설정 

```java
@Configuration
@EnableWebSecurity //WebSecurityConfiguration, SpringWebMvcImportSelector, OAuth2ImportSelector., HttpSecurityConfiguration 클래스를 import받은 annotation
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http //인가 정책
                .authorizeRequests() //요청에대한 보안검색
                .anyRequest().authenticated(); //어떤 요청에도 인증을 받도록

        http //인증 정책
                .formLogin();
    }
}
```

사용자 정의 비밀번호 설정 (pom.xml)

```xml
spring.security.user.name=user
spring.security.user.password=user
```

## 3. 인증 API - Form 인증

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im2.JPG" />

인증처리 관련된 객체가 생성되고 저장되는 과정 

1. 사용자가 get 방식으로 form url로 자원에 접근을 시도 
2. Server는 인증된 사용자만 접근이 가능하다고 보안이 설정되어 있다
3. 현재 사용자가 인증이 되지 않은 경우에는 로그인 페이지로 리다이렉트 된다 
4. username + password를 입력하고 POST 방식으로 다시 인증을 시도 한다 
5. server에서는 시큐리티가 session id를 생성하게 되고 
6. 앞에 session에 최종 성공한 인증 결과를 담은 인증토큰(Authentication 타입의 클래스)을 생성한다 
7. Security Context(나중에 공부하게 될) 를 생성하게 되고 세션에 저장한다. 
8. 인증을 받은 이 후에 사용자가 home URL로 자원에 접근을 시도하면 Spring Security는 현재 사용자가 가진 session으로 부터 인증 토큰의 존재 여부를 판단 
9. 그 사용자는 인증 토큰으로 계속적으로 해당 자원에 접근하게 되고 
10. Spring Security는 session에 저장된 인증토큰이 있으면 사용자가 인증된 사용자라고 인증을 유지한다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im3.JPG" />

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Configuration
@EnableWebSecurity //웹보안 활성화를 위한 어노테이션
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http //인가정책
                .authorizeRequests() //요청에 대한 보안 검색을 하고
                .anyRequest().authenticated(); //어떤 인증에도 보안을 받도록 설정

        http //인증정책
                .formLogin() //formLogin방식으로 인증을 할 수 있도록 설정
                .loginPage("/loginPage")
                .defaultSuccessUrl("/") //인증에 성공했을 때 이동되는 URL
                .failureUrl("/login") //인증에 실패 했을 때
                .usernameParameter("userId") //parameter 항목 필드 명을 변경, default 는 username
                .passwordParameter("pwd") //default는 password
                .loginProcessingUrl("/login_proc") //form태그의 action url, default는 login
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        //Authentication : 인증에 성공했을 때 최종적으로 인증한 결과 까지 담은 인증객체
                        System.out.println("authentication : " + authentication.getName()); //인증 성공한 사용자 이름을 출력
                        httpServletResponse.sendRedirect("/");
                    }
                }) //login에 성공 했을 때 successHandler를 호출, AuthenticationSuccessHandler 인터페이스를 구현한 구현체를 설정하면 된다.
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        //AuthenticationException : 인증 예외의 객체를 파라미터로 넘겨 주고 있다.
                        System.out.println("exception : "+ e.getMessage());
                        httpServletResponse.sendRedirect("/login");
                    }
                })
                .permitAll() //loginPage로 접근하는 모든 사용자는 접근이 가능하도록 
        ;

    }
}
```

## 4. Form Login 인증 필터 : UsernamePasswordAuthenticationFilter

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im4.JPG" />

1. UserNamePasswordAuthenticationFilter - 인증처리를 담당하고 인증처리에 관련된 요청을 처리하는 Filter, 내부적으로 각각의 인증처리의 역할에 따라서 이 클래스를 활용해서 인증처리를 하게 된다. 현재 사용자의 요청 정보(URL)를 확인한다. 
2. AntPathRequestMatcher - 현재 요청정보 URL이 "/login"으로 시작되는지 검사
    - SecurityConfig에서 http.loginProcessingUrl로 값 변경 가능
    - Authentication 객체를 만들어서 이 객체 안에 사용자가 입력한 username과 password값을 인증객체에 저장해서 실제로 인증처리를 맡기는 역할
3. AuthenticationManager - Filter로부터 인증객체를 전달받고 인증처리를 하게 된다. 
    - 내부적으로 AuthenticationProvider 클래스 타입의 객체를 가지고 있다. 이 객체들 중에 하나를 선택 해서 인증처리를 위임하게 된다.
    - AuthenticationProvider - 실제적으로 인증처리를 담당하는 class, 인증 실패/성공의 결과를 return 한다. 실패 경우에 AuthenticationException을 발생 시킨다. 성공 경우에 User정보, Authorities 정보가 저장된 Authentication 객체를 만들고 객체를 return 한다.
4. Filter는 최종적인 Authentication 인증 객체를 SecurityContext에 저장한다. 
    - SecurityContext는 인증객체를 보관하는 저장소/보관소 이다.
5. SecurityContext를 Session에 저장되어서 전역적으로 사용자가 인증처리하는 과정에서 SecurityContext 안의 Authentication 객체를 참조할 수 있도록 한다. 
6. 인증에 성공한 이 후에 SuccessHandler 작업들을 수행하게 된다. 

SecurityContext가 Session에도 저장 

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im5.JPG" />

## 5. Logout 처리, LogoutFilter

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im6.JPG" />

1. 클라이언트가 logout 요청을 한다 
2. Spring Security가 그 요청을 받고 로그아웃 처리한다. 
    - 세션을 무효화
    - 인증 토큰/객체를 삭제
    - 쿠키정보 삭제
3. 로그인 페이지로 리다이렉트 한다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im7.JPG" />

```java
@Configuration
@EnableWebSecurity //WebSecurityConfiguration, SpringWebMvcImportSelector, OAuth2ImportSelector., HttpSecurityConfiguration 클래스를 import받은 annotation
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated();

        http
                .formLogin();

        http
                .logout()
                .logoutUrl("/logout") //default는 logout, Spring Security는 logout 처리를 할때 기본적으로 POST방식을 사용, GET방식으로 로그아웃을 구현하게 되면 오류를 발생
                .logoutSuccessUrl("/login")
                .addLogoutHandler(new LogoutHandler() {
                    @Override
                    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) {
                        HttpSession session = httpServletRequest.getSession();
                        session.invalidate(); //세션을 무효화
                    }
                }) //실제로 logout을 처리를 하는 logout handler가 있는데 다른 작업을 원할 때 직접 logouthandler interface를 구현할 수 있다.
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.sendRedirect("/login");
                    }
                }) //로그아웃이 성공하면 그 다음 수행할 작업, logoutSuccessUrl은 페이지만 이동하고 Handler는 다양한 더 많은 로직 구현 가능
                .deleteCookies("remember-me") //인증을 할때 서버에서 "remember-me" 쿠키를 발행한다. 로그아웃할 때 server에서 만든 쿠키를 삭제한다.
                ;

    }
}
```

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im8.JPG" />

1. LogoutFilter가 POST방식으로 요청 받아서 Logout 처리를 한다. 
2. AntPathRequestMatcher - 현재 요청정보 URL이 "/logout"으로 시작되는지 검사
3. SecurityContext로부터 인증객체를 꺼내 와서 LogoutHandler에게 전달한다 
4. LogoutFilter가 가지고 있는 몇개의 Filter중 SecurityContextLogoutHandler는 세션을 무효화, 쿠키 삭제, SecurityContext 객체를 삭제하고 인증객체도 null처리 한다. 
5. Logout이 성공적으로 처리가 되면 SimpleUrlLogoutSuccessHandler로 /login페이지를 리다이렉트 하도록 처리 한다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im9.JPG" />

## 6. Remember Me 인증

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im10.JPG" />

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im11.JPG" />

```java
@Configuration
@EnableWebSecurity //WebSecurityConfiguration, SpringWebMvcImportSelector, OAuth2ImportSelector., HttpSecurityConfiguration 클래스를 import받은 annotation
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    UserDetailsService userDetailsService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated();

        http
                .formLogin();

        http
                .logout() //logout 처리는 기본적으로 Post 방식으로 처리된다.
                .logoutUrl("/logout")
                .logoutSuccessUrl("/logout")
                .addLogoutHandler(new LogoutHandler() {
                    @Override
                    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) {
                        HttpSession session = httpServletRequest.getSession();
                        session.invalidate();

                    }
                })
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.sendRedirect("/login");
                    }
                }) //logoutSuccessUrl은 url로 이동만 하지만 handler는 더 다양하게 구현 가능
                .deleteCookies()
            .and()
                .rememberMe()
                .rememberMeParameter("remember")
                .tokenValiditySeconds(3600)
                .userDetailsService(userDetailsService); //user계정을 조회하는 클래스가 userDetailService
                ;

    }
}
```

## 7. Remember Me 인증 필터 : RememberMeAuthenticationFilter

<img src ="https://eunmik.github.io/bonita/assets/img/210611-im12.JPG" />

1. RememberMeAuthenticationFilter는 사용자 요청을 받아서 인증을 처리하는 조건이 있다. 
    - Authentication이 null일 경우

        인증을 받은 사용자는 security context에 인증객체가 존재.

        존재하지 않는 경우 :

        - 사용자 세션이 만료되었거나 끊겼을 때
        - security context가 존재하지 않기 때문에 security context안에 authentication이 없는 경우
    - Remember-me 쿠키를 가지고 온 경우

    (Authentication == null && Server로부터 remember-me 쿠키를 발급 받은 경우) 이 경우에 RememberMeAuthenticationFilter 동작한다. 

2. RememberMeServices에 구현체가 두가지가 있다. 
    - TokenBasedRememberMeServices : 메모리에 실제로 저장된 Token과 사용자가 요청할 때 가져온 쿠키 Token과 비교해서 인증처리
    - PersistentTokenBasedRememberMeServices : 영구적인 방법, DB에 발급한 Token을 저장하고 사용자가 가져온 Token 값과 DB에 저장된 값과 비교해서 인증처리
3. RememberMeService가 Token Cookie를 추출한다. 
4. 가지고 있는 Token이 Remember-me라는 Token인지 검사한다. Token이 존재하면 다음으로 진행한다. 
5. Decode Token에서 Token이 정상적으로 규칙을 지키고 있는 Token인지 판단한다. (정상유무판단) 
    - Token이 서로 일치하는 지
    - DB에 저장된 User를 조회해서 User 계정이 존재하는지
    - 새로운 Authentication (인증객체)를 생성
    - AuthenticationManager에 인증객체를 전달