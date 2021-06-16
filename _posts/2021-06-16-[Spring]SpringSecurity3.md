---
layout: post
title: "[Spring] Spring Security 주요 아키텍처 이해 (1) "
date: 2021-06-16
excerpt: "Spring Security의 기본 이해 3탄"
tags: [Java, Spring, Spring Security]
comments: true
---
## 1. 위임 필터 및 필터 빈 초기화 - DelegatingProxyChain, FilterChainProxy

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img1.JPG" />

- Servlet Filter는 서블릿 스펙에 정의가 되어있는 기술 인데 스펙의 2.3 버전부터 도입이 되었다.
- Servlet Filter의 역할은 어떤 요청이 있을 때 그 요청이 서블릿에 가기 전에 이 Filter에서 어떤 작업을 처리 하고 다시 요청을 전달 한다. 그리고 Servlet 자원에서 어떤 처리가 끝나게 되면 최종적으로 응답하기 전에 다시 Filter가 받아서 어떤 작업을 처리하고 나서 Filter가 클라이언트에 응답을 하게 된다.
- 요청에 대한 최종적인 접근 전과 후에 이 Filter가 어떤 처리를 할 수 있도록 우리가 Filter를 사용할 수 있다.
- Servlet Filter는 서블릿 컨테이너에서 생성이 되고 실행이 된다. 그렇기 때문에 이 Filter는 Spring에서 만든 Bean을 Injection해서 사용하거나 Spring에서 사용하는 기술을 이 Servlet Filter에서는 사용할 수 없다.
- 실행되는 위치가 Filter는 Servlet Container고 Spring Bean은 Spring Container이기 때문이다.

- Spring Security는 사용자가 요청한 모든 요청에 대해서 Filter 기반으로 처리하고 있다.
- Filter에서도 Spring Bean을 사용하고자 하는 요구사항이 생겼을 때 DelegatingFilterProxy는 Servlet Filter이고, 요청을 받아서 그 요청을 Spring에서 관리하는 Filter Bean에서 위임하는 역할을 한다. 이 클래스는 Servlet Filter이기 때문에 가장 먼저 요청을 받고 Spring 에게 전달한다.

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img2.JPG" />

0번 Filter가 처리 되고 Filter Chain Proxy가 호출되고 1번 Filter가 호출되고 FilterChainProxy가 호출되고.. 반복 

FilterChainProxy는 실질적으로 보안처리를 하는 시작점이고 DelegatingFilterProxy으로 부터 위임을 받고 보안 처리를 한다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img3.JPG" />

1. 사용자가 처음 요청하게 되면 Container에서 가장먼저 요청을 받게 된다. 
2. 요청에 의해서 각각의 Filter들이 처리가 되고 그 중에서 DelegatingFilterProxy가 요청을 받게 되면 요청 객체를 springSecurityFilterChain에 delegate request를 위임하게 된다. 
3. 실제로는 DelegatingFilterProxy가 등록 될 때 springSecurityFilterChain이름으로 등록한다. 
4. springSecurityFilterChain의 이름을 가진 Bean이 바로 FilterChainProxy 이다. 
5. 스프링에서는 FilterChainProxy를 Bean으로 등록할 때 springSecurityFilterChain이라는 이름으로 등록한다. 
6. 그러면 DelegatingFilterProxy가 FilterChainProxy를 (동일한 이름으로) 찾게 되는 거고 이 필터 bean에서 delegate request을 하게 된다. 
7. FilterChainProxy는 자기가 관리하는 각각의 Filter들로 보안처리를 한다. 그리고 보안처리가 다 되고 나면 최종 자원(DispatchertServlet)에 요청을 전달한다. 

## 2. 필터 초기화와 다중 보안 설정

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img4.JPG" />

- Sprint Security가 초기화 시
    - 설정에서 생성된 Filter가 이 객체안에 변수로 담기고 RequestMatcher 변수에 정보가 담긴 SecurityFilterChain 클래스가 생성이 된다.
    - 각각이 생성된 객체를 FilterChainProxy가 SecurityFilterChains라는 리스트 변수에 저장한다.
    - 결론적으로 FilterChainProxy는 SecurityFilterChains 변수에 각각의 SecurityFilterChain 객체를 가지고 있다.
- 사용자가 /admin으로 요청하게 되면 FilterChianProxy가 요청을 받아서 각각의 객체가 가지고 있는 RequestMatcher와 매칭이 되는지 확인을 하고 해당 Filter를 가지고 와서 인증/인가처리를 한다.

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img5.JPG" />

1. 사용자가 /admin 자원에 접근을 요청하고 있다. 
2. FilterChainProxy가 각각의 Filter를 보관하고 있다. 
3. 필터를 선택할 때 요청 URL과 matches(매칭이 true가 되는지)를 한다. 
4. 객체에 포함된 RequestMatcher의 정보와 요청 정보가 매치되는 필터를 가지고 와서 처리한다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img6.JPG" />

```java
@Configuration
@EnableWebSecurity //웹보안 활성화를 위한 어노테이션
@Order(0)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
		protected void configure(HttpSecurity http)  throws Exception {
        http
                .antMatcher("/admin/**")
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .httpBasic();
    }
}
@Order(1)
@Configuration
class SecurityConfig2 extends WebSecurityConfigurerAdapter{
    protected void configure(HttpSecurity http) throws Exception{
        http
                    .authorizeRequests()
                    .anyRequest().permitAll()
                    .and()
                    .formLogin();
    }
}
```

## 3. 인증 개념 이해 - Authentication

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img7.JPG" />

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img8.JPG" />

1. 사용자가 username+password로 인증을 하고 입력한 정보를 전달한다. 
2. UsernamePasswordAuthenticationFilter가 정보를 받아서 username과 password를 추출한 다음
3. Authentication이라는 객체를 하나 생성한다. 
4. AuthenticationManager가 인증객체를 가지고 인증처리를 한다. 
5. 인증에 성공하게 되면 인증 전에 만든 Authentication과 동일한 Type의 Authentication 객체를 만든다. 
6. Principal에는 최종적으로 인증에 성공한 User객체를 담는다. 
7. SecurityContextHodler안에 SecurityContext안에 Authentication 객체를 저장하고 전역적으로 사용할 수 있다. 

## 4. 인증 저장소 - SecurityContextHolder, SecurityContext

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img9.JPG" />

ThreadLocal이란? Thread는 Thread마다 고유하게 할당된 저장소가 있는데 그게 ThreaLocal이다. Thread간에 공유가 되지 않고 각 Thread에게만 할당된 저장소이다.

SecurityContext의 가장 주된 역할은 Authentication객체를 저장하는 저장소이다. 

Authentication authentication = SecurityContextHolder.getContext().getAuthentication() 구문은 어떤 메소드에서도 다 참조가 가능하도록 Spring Security가 제공하고 있다. 

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img10.JPG" />

1. 사용자가 로그인을 하면 Server에서 요청을 받는다. 
2. Server에서 Thread를 생성한다. 이 Thread는 ThreadLocal이 할당된다. 
3. 이 Thread가 인증 처리를 하면 Authentication 객체가 생성된다. 
4. 인증이 실패하게 되면 SecurityContextHolder.clearContext()로 SecurityContext를 null로 초기화 한다. 
5. 인증에 성공하게 되면 최종적으로 인증에 성공한 인증객체를 담는다. 
    - SecurityContexHolder가 ThreadLocal 의 객체를 가지고 있고 ThreadLocal이 SecurityContext를 담고 있는 것이다.
6. 최종적으로는 SecurityContextHolder가 HttpSession에 SPRING_SECURITY_CONTEXT라고 저장이 된다. 

 

TestController.java

```java
@RestController
public class TestController {

    @GetMapping
    public String index(HttpSession session){

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        SecurityContext context = (SecurityContext) session.getAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY);
        Authentication authentication1 = context.getAuthentication();
        System.out.println((authentication == authentication1) ? true : false); //true이다. 

        return "home";
    }

@GetMapping("/thread")
    public String thread() {
        new Thread(
                new Runnable() {
                    @Override
                    public void run() {
                        Authentication authentication = SecurityContextHolder.getContext().getAuthentication(); //authentication = null 이다.
                    }
                }
        ).start();
        return "thread";
    }
```

securityconfig.java MODE를 변경할 수 있다. 

```java
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL)
;
```

## 5. 인증 저장소 필터 - SecurityContextPersistenceFilter

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img11.JPG" />

- SecurityContextPersistenceFilter는 익명사용자든 인증 시든 인증 후든 SecurityContext객체를 SecurityContextHolder에 저장하는 역할을 한다.
- 그런데 익명사용자나 인증 시에는 새로운 SecurityContext객체를 생성하여 저장하고 인증 이후에는 Session에서 SecurityContext 객체를 꺼내서 저장한다.
- 이 Filter는 FilterChainProxy에서 두번째에 위치하고 있다. SecurityContextHolder에 일단 저장해 놓고 그다음 필터들이 Holder에서 SecurityContext를 꺼내어서 참조할 수 있도록 두번째 위치에서 작업을 처리하고 있다.

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img12.JPG" />

1. SecurityContextPersistenceFilter는 사용자가 인증을 받았던 안받았던 상관없이 매 요청마다 그 요청을 처리한다. 
2. 내부적으로 HttpSecurityContextRepository가 있는데 이 클래스가 실제로 SecurityContext를 생성하고 조회하는 역할을 한다. 
    - 인증 전이면
        - 2-1. 새로운 SecurityContext를 생성한다. Session에서 SecurityContext를 있는지 여부를 확인하는데 인증 전에는 session에는 SecurityContext가 없다. 인증객체는 null이다.
        - 2-2. 인증을 시도하는 시점에 AuthFilter가 인증을 처리 한다.
        - 2-3. 인증이 성공되면 AuthFilter가 SecurityContextHolder안에서 SecurityContext객체 안에 인증에 성공한 결과를 저정한 Authentication객체를 저장한다.
        - 2-4. 최종적으로 Client에 응답하는 시점에 SecurityContextPersistenceFilter가 SecurityContext를 Session에 저장한다.
        - 2-5. SecurityContextHolder에서 SecurityContext를 제거 시킨다.
        - 2-6. 최종적으로 Client에게 응답한다.
    - 인증 후이면
        - 2-1. Session에서 SecurityContext 객체가 있는지 확인한다. 있다.
        - 2-2. Session에 있는 SecurityContext를 꺼내서 SecurityContextHolder에 저장한다.
        - 2-3. 다음 필터로 이동하게 된다.

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img13.JPG" />

- 인증을 받기 전에는 createSecurityContext (새로운 SecurityContext를 생성한다)
- 최초로는 인증을 받기 전이기 때문에 Authentication이 null인 상태이다.

<img src ="https://eunmik.github.io/bonita/assets/img/210616-img14.JPG" />