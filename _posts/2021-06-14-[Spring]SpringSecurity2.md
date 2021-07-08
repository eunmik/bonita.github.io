---
layout: post
title: "[Spring] Spring Security 기본 API 및 Filter 이해 (2) "
date: 2021-06-14
excerpt: "Spring Security의 기본 이해 2탄"
tags: [Java, Spring, Spring Security]
comments: true
---
## 8. 익명사용자 인증 필터 : AnonymousAuthenticationFilter

어떤 사용자가 인증을 받으면 session에 user객체가 저장 되고  

이 사용자가 어떤 자원에 접근하려고 하면 session에서 user객체가 존재하는지 확인한다. 

user객체가 null이면 이 사용자는 인증을 받지 않았다고 판단 해서 

그 자원에 접근하지 못하게 만들고 

null이 아니면 인증을 받았다고 판단하고 자원에 접근하도록 처리 한다. 

이렇게 인증된 사용자와 인증받지 않은 사용자를 구분하여 처리 한다. 

AnonymousAuthenticationFilter는

인증을 받지 않은 사용자는 null로 처리하는게 아니라 별도의 Anonymous 객체를 생성하여 처리한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img1.JPG" />

1. 사용자가 요청을 하면 AnonymoiusAuthenticationFilter가 요청을 받는다. 
2. 현재 요청하고 있는 사용자가 인증객체가 있는지 없는지 부터 판단 
    - 인증객체는 인증에 최종적으로 성공하게 되면 SecurityContext안에 저장이 되어있는데
    - 인증객체가 존재하게 되면 다른 Filter에서 처리하도록 넘어간다.
3. 인증객체가 존재하지 않는다면  AnonymousAuthenticationToken을 생성한다. 
4. SecurityContext 객체 안에 익명사용자용 토큰 객체를 저장한다. 
    - 인명사용자라고 할지라도 인증사용자처럼 동일하게 SecurityContext안에 인증객체를 저장

SecurityContext는 isAnonymous()를 검사를 할때 SecurityContext안에 객체를 검사해서 판단한다. 

AnonymousAuthenticationFilter는 인증사용자와 익명사용자를 구분하기 위해 사용되는 Filter이다. 

isAnonymous()가 true일 때 로그인 메뉴를 보이게 하고

 isAuthenticated()가 true일 때 로그아웃 메뉴를 보이게 하도록 구성할 수 있다

인증객체를 가지고 있는 익명 사용자이지만 실제로 인증을 받지않은 사용자이기 때문에 세션에 저장할 필요가 없다. 

## 9. 동시 세션 제어, 세션 고정 보호, 세션 정책

### 동시 세션 제어

현재 동일한 계정으로 인증을 받을 때 그 세션에 혀용 갯수가 초과 되었을 때

어떻게 그 세션을 계속적으로 초과하지 않고 캐시를 유지않고 사용할 수 있는 지 알아보자. 

Spring Security는 두가지 전략으로 동접을 제어하고 있다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img2.JPG" />

최대 세션 허용 개수를 초과할 경우 

1. 이전 사용자 세션 만료 시키는 전략  (허용갯수가 1일 때) 
    - 사용자 1이 로그인을 하면 서버에서 인증성공을 하고 사용자1의 세션이 생성
    - 서버에는 사용자 1의 세션이 생성이 되어있다.
    - 사용자 2가 동일한 계정으로 로그인을 한다. 사용자2의 세션이 생성.
    - 사용자2가 로그인을 해서 세션을 생성할 때 이전 사용자 세션을 만료 시키는 설정을 한다.
    - 사용자1의 세션은 만료, 사용자 2의 세션은 유효

2. 현재 사용자 인증 실패 시키는 전략 (허용갯수가 1일 때) 
    - 사용자 1이 로그인을 하면서 인증 성공 후 세션 생성
    - 사용자 2가 동일 한 계정으로 로그인 하면 인증에 실패함으로서 세션 생성 실패
    - 사용자 2의 로그인을 차단하는 전략

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img3.JPG" />

```java
http
.sessionManagement()
.maximumSessions(1) //동시 세션 최대 허용 개수
.maxSessionsPreventsLogin(false); // true일 경우 현재 사용자가 로그인을 못하게 만든다, default는 false
```

maxSessionPreventsLogin이 false일 경우에 

다른 계정에서 로그인하면 전 계정은 아래와 같은 에러 메시지가 발생한다. 

```
This session has been expired (possibly due to multiple concurrent logins being attempted as the same user).
```

maxSessionPreventsLogin이 true일 경우에 

다른 게정을 로그인 하면 현재 계정에서 아래와 같은 에러 메시지가 발생한다. 

```
exception : Maximum sessions of 1 for this principal exceeded
```

### 세션 고정 보호

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img4.JPG" />

세션 고정 공격

1. 공격자가 서버에 공격을 한다 .
2. 서버는 JSESSIONID를 발급한다. 
3. 공격자는 발급받은 세션 쿠키를 사용자에게 심어 놓는다. 
4. 사용자는 공격자의 세션 쿠키로 로그인을 시도 한다. 
5. 사용자는 인증에 성공 한다. 
6. 사용자와 공격자가 동시에 같은 세션 쿠키를 공유하게 되기 때문에 

    공격자는 인증 받지 않아도 인증 받은 것처럼 자원에 접근하게 된다. 

사용자가 공격자가 심은 쿠키로 접속하더라도 인증 할 때마다 새로운 세션/쿠키가 생성된다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img5.JPG" />

changeSessionId() : 기본값, 세션 ID만 변경이 된다.  서블렛 3.1 이상

migrateSession() : 새로운 세션 ID가 발급된다.  이전의 속성 값들을 그대로 사용한다. 서블렛 3.1 이하 에서 기본 값

newSession() : 새로운 세션 ID가 발급 되면서 새로운 값들의 속성을 설정한다. 

```java
http
.sessionManagement()
.sessionFixation().changeSessionId(); //none : 무방비 상태, changeSessionId() : default
```

### 세션 정책

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img6.JPG" />

JWT 가 Stateless 방식 이다. 

## 10. 세션 제어 필터 : SessionManagementFilter, ConcurrentSessionFilter

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img7.JPG" />

앞의 배웠던 4가지의 핵심적인 기능을 SessionManagementFilter가 하고 있다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img8.JPG" />

SessionManagementFilter와 마찬가지로 ConcurrentSessionFilter도 동시적 세션 제어를 하고 있다.

이 필터는 매 요청마다 현재 상요자의 세션 만료 여부를 체크 한다. 

세션이 만료되었을 경우 즉시 만료 처리 시켜 버린다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img9.JPG" />

1. 사용자가 로그인을 시도 한다. 
2. 다른 사용자가 동일한 계정으로 세션을 생성한 상태이다. 
3. 최대 세션 허용 개수가 1개일 경우에 세션 허용 개수가 초과가 되었다. 
4. 이천 사용자 세션을 만료하다. 
5. 이전 사용자가 서버에 요청을 할때 마다 ConcurrentSessionFilter가 사용자의 세션 만료 여부를 체크 한다. 
6. 확인할 때 SessionManagementFilter에서 이전 사용자 세션을 만료 했던 그 설정을 참조한다. 
7. 즉시 그 사용자 세션을 만료하고 오류 페이지를 응답한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img10.JPG" />

조건 : 최대 세션 허용 개수가 1

1. 사용자 1이 로그인을 하면 가장 먼저 ConcurrentSessionControl AuthenticationStrategy를 호출한다. 
    - 동시적 세션 제어를 처리하는 클래스
    - 인증을 시도하고 있는 사용자가 가지고 있는 계정에 생성된 세션이 count가 얼마인지를 확인한다.
    - 현재는 사용자 1이 처음으로 세션 생성을 요청했기 때문에 count는 0이다.
2. 그 다음에 ChangedSessionId AuthenticationStrategy를 호출한다. 
    - 세션 고정 보호를 처리하는 클래스
    - 사용자 1의 새롭게 session을 생성하고 새로운 세션 쿠키를 발급한다.
3. 그 다음엔 RegisterSession AuthenticationStrategy를 호출한다.
    - 사용자의 세션을 등록하고 지정하는 역할을 하는 클래스
    - 이 클래스가 처리가 되야 사용자 1의 세션 정보가 등록이 된다.
    - 그러고 세션 count가 1 이 된다.
4. 사용자 2가 동일한 계정으로 인증을 시도 한다. 
5. ConcurrentSessionControl을 호출하고 동일한 계정으로 생성된 세션이 있고 세션의 count가 1이다. 
6. 2가지 전략으로 처리 한다.
    - 인증 실패 전략인 경우 : SessionAuthenticationException, 인증 예외를 발생하고 인증 실패를 한다. ChangedSessionId, RegisterSession 클래스를 호출안하고 바로 인증 실패를 리턴한다.
    - 세션 만료 전략인 경우 : 현재 사용자는 세션 인증에 성공하고 이전 사용자의 세션을 만료 시킨다.
7. 인증에 성공한 전략 다음에 사용자1 과 동일하게 changedSessionId, 세션 고종 보호 클래스를 호출하고 
8. RegisterSession 클래스를 호출하여 세션 정보를 등록한다. 그래서 총 세션 count는 두개가 되고 현재 메모리에는 세션 count가 2개가 올라간 상태이다. 
9. 이 때, 사용자 1이 /home이라는 자원에 접근을 요청한다. 
10. ConcurrentSessionFilter 에서 사용자 1의 만료 여부를 확인하다. 
11. 사용자 2가 사용자1의 세션 만료를 true라고 설정한 부분을 가져 온다. 
12. 사용자 1의 세션의 만료는 true이기 때문에 logout 처리하고 오류 페이지를 응답한다. 

## 11. 권한 설정과 표현식

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img11.JPG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img12.JPG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img13.JPG" />

Role User 권한을 가지고 있는 사용자가 anonymous() 자원에 접근이 가능한가? NO 

SpEL : Spring Expression Language 스프링 표현식

```java
@Override
    protected void configure(HttpSecurity http) throws Exception{

        http //인증정책
                .formLogin() //
				http
                    .authorizeRequests()
                    .antMatchers("/user").hasRole("USER")
                    .antMatchers("/admin/pay").hasRole("ADMIN")
                    .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
                    .anyRequest().authenticated()
        ;

    }

    //사용자 생성을 위한 오버라이드
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //메모리 방식으로 사용자를 생성, 갯수 제한은 없음
        auth.inMemoryAuthentication().withUser("user").password("{noop}1111").roles("USER"); //noop은 암호화 하지 않은 암호화 유형
        auth.inMemoryAuthentication().withUser("sys").password("{noop}1111").roles("SYS");
        auth.inMemoryAuthentication().withUser("admin").password("{noop}1111").roles("ADMIN");
    }
```

## 12. 예외 처리 및 요청 캐시 필터 : ExceptionTranslationFilter, RequestCacheAwareFilter

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img14.JPG" />

이 예외는 과연 누가 발생 시키는가? FilterSecurityInterceptorFilter 이다. 

Spring Security가 관리하는 보안 필터 중에서 맨 마지막에 위치하고 있다. 

이 필터 앞에 위치하는 필터가 ExceptionTranslationFitler이다. 

이 필터가 실제로 사용자 요청을 받을 때 그 다음 필터로 그 요청을 전달 해 줄 때 

try-catch로 감싸서 이 FilterSecurityInterceptor를 호출하고 있다. 

그렇기 때문에 FilterSecurityInterceptor에서 생기는 인증 예외와  인가 예외는 ExceptionTranslationFilter로 throw 하고 있다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img15.JPG" />

1. 사용자가 /user 자원에 접근을 시도하고 있다. 
2. 인증을 받지 않고 바로 /user 자원에 접근하고 /user는 인증된 사용자만 접근할 수 있을 때 
3. FilterSecurityInterceptor가 요청을 받고 있고 이 사용자는 인증을 받지 않았다. → 인증을 받지 않고 자원에 접근 했기 때문에 익명 사용자가 접근을 한 것이다.  → 인가 예외가 발생한다. 
4. ExceptionTranslationFilter가 AccessDeniedException을 발생하지만 익명사용자나 Remember-me로 인증된 사용자 경우에는 AccessDeniedHandler로 가지 않고 AuthenticationException로 가게 된다. 
5. AuthenticationException안에서 2가지 역할을 처리하는 과정이 있다.
    - AuthenticationEntryPoint 인터페이스 구현체를 호출해서 구현체 안에서 /login 페이지로 리다이렉트 한다. 그 전에 Security Context를 null로 만드는 작업을 한다.
    - Login 페이지로 redirect 하기 전에 사용자가 요청한 정보를 저장한다. 이 정보는 DefaultSavedRequest 객체안에 저장이 되고 이 객체는 다시 Session에 저장이 되는 역할을HttpSessionRequestCache이 하고 있다.

1. 사용자가 ADMIN 권한이 없는 데 ADMIN 권한이 필요한 /user 자원에 접근할 때 
2. 인가 예외가 발생한다. 
3. ExceptionTranslationFilter는 인가 예외를 처리한다. 
4. AccessDeniedException은 AccessDeniedHandler를 호출 한다. 
5. AccessDeniedHandler는 denied 페이지로 이동한다. 
6. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img16.JPG" />

```java
http //인증정책
                .formLogin() //formLogin방식으로 인증을 할 수 있도록 설정
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        RequestCache requestCache = new HttpSessionRequestCache(); // 인가/인증 에외가 발생했을 때 로그인 페이지로 가기 전에 요청 정보를 저장하는 것
                        SavedRequest savedRequest = requestCache.getRequest(httpServletRequest, httpServletResponse); //이 정보에는 원래 사용자가 가려고 하는 요청정보가 저장되어 있다.
                        String redirectUrl = savedRequest.getRedirectUrl();
                        httpServletResponse.sendRedirect(redirectUrl); //인증에 성공한 다음에 세션에 저장되어있던 url로 이동하도록 처리
                    }
                }) //login에 성공 했을 때 successHandler를 호출, AuthenticationSuccessHandler 인터페이스를 구현한 구현체를 설정하면 된다.
http
                    .authorizeRequests()
                    .antMatchers("/login").permitAll()
                    .antMatchers("/user").hasRole("USER")
                    .antMatchers("/admin/pay").hasRole("ADMIN")
                    .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
                    .anyRequest().authenticated()
        ;

        http
                    .exceptionHandling()
                    .authenticationEntryPoint(new AuthenticationEntryPoint() {
                        @Override
                        public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                            httpServletResponse.sendRedirect("/login");
                        }
                    }) //인증 예외 발생 시
                    .accessDeniedHandler(new AccessDeniedHandler() {
                        @Override
                        public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
                            httpServletResponse.sendRedirect("/denied");
                        }
                    }) //인가 예외 발생시
        ;
```

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img17.JPG" />

## 13. 사이트 간 요청 위조 - CSRF, CsrtFilter

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img18.JPG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/210614-img19.JPG" />

서버가 발급한 토큰을 가지고와야지 시큐리티는 발급한 토큰을 검사를 한다. 
