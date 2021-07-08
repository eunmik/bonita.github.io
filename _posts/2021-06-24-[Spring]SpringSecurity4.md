---
layout: post
title: "[Spring] Spring Security 주요 아키텍처 이해 (2) "
date: 2021-06-24
excerpt: "Spring Security의 기본 이해 4탄"
tags: [Java, Spring, Spring Security]
comments: true
---
## 6. 인증 흐름 이해 - Authentication Flow

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img1.JPG" />


1. Client가 로그인 인증요청을 보낸다. 
2. Form인증방식의 인증처리를 할 때 UsernamePasswordAuthenticationFilter가 작동한다. 
    - 사용자의 id+pwd를 담아서 Authentication 객체를 생성한다.
3. AuthenticationManager에 Authentication 객체를 넘기면서 인증처리를 맡긴다. 
4. AuthenticationManager는 주된 역할은 
    - 인증의 전반적인 관리
    - 실제 인증 역할을 하지 않고 적절한 AuthenticationProvider에 위임
    - Id+pwd과 일치하는지 검증에 대한 관여를 하지 않는다.
    - 내부적으로 있는 List 변수에는 한개 이상의 AuthenticationProvider가 있는데 그 중 현재 인증에 사용할 수 있는 Provider에게 인증처리를 위임한다. (DaoAuthenticationProvider가 Form인증을 처리하는 구현체이다)
5. AuthenticationProvider는 전달받은 Authentication 객체의 정보를 검증한다. 
    - loadUserByUsername에 Id(username)을 전달하면서 user 객체를 요청한다.
6. UserDetailService에 User 객체를 요청한다. 
7. DB에 User 객체를 조회하고 
    - 검증이 실패하여 예외가 발생하면 UsernamePasswordAuthenticationFilter가 예외를 받아서 처리한다. FailHandler에서 실패 관련된 처리를 한다.
    - 검증이 성공하면 UserDetails 타입으로 반환한다.
8. AuthenticationProvider는 ID를 검증에 성공했고 Authentication 객체에 있는 패스워드와 UserDetails에 있는 패스워드가 일치하는지 검증한다. 
    - 패스워드가 일치하지 않으면 BadCredentialException을 발생한다.
    - 패스워드가 일치하면 Authentication인증 객체를 만들어서 최종적으로 성공한 인증 정보(UserDetails + authorities) 를 담는다.
9. AuthenticationManager는 넘겨받은 Authentication 객체를 UsernamePasswordAuthenticationFilter에게 다시 전달한다. 
10. Filter는 SecurityContext에 전달받은 Authentication 객체를 저장한다. 

## 7. 인증 관리자 - AuthenticationManager

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img2.JPG" />

- AuthenticationManager는 Filter로 부터 인증처리를 실제로 지시받는 첫번째 클래스이다.

    Filter가 사용자가 인증처리를 요청할 때 전달한 ID,PWD를 Authentication에 저장한 다음에 AuthentifcationManager에 전달 하여 인증 처리를 지시하는 것이다. 

- AuthenticationManager는 인터페이스로 제공이되고 인터페이스로 구현한 구현체가 ProviderManager이다.  ProviderManager는 AuthenticaitonProvider들을 관리하는 클래스이다.
- AuthenticationProvider 목록 중에서 인증 처리 요건에 맞는 AuthenticationProvider를 찾아 인증처리를 위임하는게 AuthenticationManager이다. AuthenticationManager는 Provider를 찾아서 Provider에게 인증처리를 위임하는 역할을 맡고 있다.
- 사용자가 로그인 요청 → 여러가지 방식의 인증 유형 중 하나의 인증 유형으로 인증을 요청할 때 ProviderManager가 요청된 인증 유형을 처리할 수 있는 AuthenticationProvider에게 전달하여 인증처리를 위임한다.
- Oauth 인증 요청이 왔을 때 Oauth 인증 처리를 해줄 수 있는 AuthenticationProvider가 없을 때ProviderManager는 parent 속성을 찾는다.
- parent 속성도 동일하게 AuthenticationManager 타입을 저장할 수 있다.
- ProviderManager 타입의 객체를 저장할 수 있다. 이 객체에는 동일하게 AuthenticationProvider를 저장하고 있다. 부모 ProviderManager가 OauthAuthenticationProvider를 가지고 있다고 가정 하면 parent 속성에 저장되어있는 ProviderManager까지 탐색을 한다. `

## 8. 인증 처리자 - AuthenticationProvider

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img3.JPG" />

- AuthenticationProvider는 인증처리를 할 때 가장 핵심적인 역할을 하는 클래스이다.
- ID, PWD를 받아서 검증을 하고 추가적인 검증을 처리 한다.
- AuthenticationManager가 Provider에 인증처리를 맡기기 때문에 Provider에서 성공한 인증객체를 Manager에게 전달하게 된다.
- 두 개의 메소드가 제공이 된다.
    - authenticate - 인증처리에 대한 실제적인 검증을 제공
    - supports  - 현재 form 인증, remeber-me 인증 같은 인증 조건을 검사
1. authenticate는 authentication 객체를 전달 받는다. 
2. System에 저장된 정보와 일치하는지 검증을 한다. 
    - ID 검증 → UserDetailsService라는 인터페이스에서 현재 사용자의 존재 유무를 검사, 존재하면 사용자의 유저 객체를 생성해서 UserDetails 타입으로 변환해서 전달한다.
    - password 검증 → UserDetails 객체에 저장된 password와 로그인시 전달 받은 password와 비교하여 검증한다. passwordEncode 클래스를 사용해서 암호화되 저장된 pwd와 암호화된 사용자의 pwd와 비교한다.
3. 최종적으로 Authentication을 생성하고 사용자가 인증에 성공한 user객체와 사용자에 부여된 권한 정보를 저장을 하고 AuthenticationManager에 전달한다. 

### 

## 9. 인가 개념 및 필터 이해 - Authorization, FilterSecurityInterceptor

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img4.JPG" />

- 인가 처리란 인증을 받은 사용자가 자원에 접근할 수 있는 자격이 되는지 그 처리를 하는게 인가 영역이다.
- 사용자가 어떤 자원에 접근하고자 할때 사용자가 Authentication을 받았는지 판단을 하고 (Authentication 영역)
- 인증을 받은 사용자가 그 사용자가 가진 권한 들이 해당 Resource에 설정된 권한에 충분한 자격을 가지고 있는지 권한 심사를 한다.

    (Authorization 영역) 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img5.JPG" />

- 웹 계층
    - URL 요청에 따른 메뉴 혹은 화면단위의 레벨 보안
    - 사용자가 /user 경로로 자원에 접근하고자 할때 자원에 설정된 권한을 심사한다.
- 서비스 게층
    - 화면 단위가 아닌 메소드 같은 기능 단위의 레벨 보안
    - 사용자가 user()라는 메소드에 호출해서 진입하고자 할때 메소드에 설정된 권한을 심사한다.
- 도메인 계층 (Access Control List, 접근제어목록)
    - 객체 단위의 레벨 보안
    - user객체를 파일/폴더 어플리케이션에 write하고자 할때 도메인에 설정된 권한을 심사한다.

이 강좌에서는 도메인 계층에 대해서는 다루지 않는다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img6.JPG" />

- FilterSecurityInterceptor는 인가처리를 담당하는 필터
- 여러가지 보안 필터 중에서도 가장 마지막에 존재
- 맨 마지막에 필터가 최종적으로 접근하고자 하는 자원에 대해 승인/거부를 판단
- 이 필터까지 통과하게 되면 사용자가 원하는 자원에 접근이 가능하다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img7.JPG" />

1. 사용자가 어떤 자원에 접근을 하기 위해 요청한다. 
2. FilterSecurityInterceptor가 그 요청을 받아서 인증 여부를 체크 한다. 사용자가 인증객체를 가지고 있는 여부를 판단한다. 인증객체는 SecurityContext객체안에 저장하기 때문에 인증이 된 사용자라면 인증객체가 존재한다. 
3. 사용자가 /user라는 URL로 접근하면 /user에는 ROLE_USER라는 권한이 설정되어 있다면SecurityMetadataSource 클래스는 권한 정보를 가지고 와서 전달한다. 
    - /user라는 자원에 권한 정보가 존재하지 않는 경우 권한 심사를 하지 않는다.
4. AccessDecisionManager는 /user자원의 권한정보를 전달 받는다. 
    - AccessDecisionVoter가 현재 사용자가 자원에 자격이 있는지 없는지 판단
5. 심의 결과 값에 따라 AccessDeniedException을 발생하거나 접근을 허용한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img8.JPG" />

## 10. 인가 결정 심의자 - AccessDecisionManager, AccessDecisionVoter

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img9.JPG" />

- FilterSecurityInterceptor 권한을 전반적으로 처리하는 필터에서 인증정보, 요청정보, 권한정보를 AccessDecisionManager에 전달해준다.
- AccessDecisionManager는 사용자가 접근하고자 하는 자원에 허용할 것인지를 결정한다.
- 접근 결정의 세가지 유형
    - AffirmativeBased - 여러개의 Voter 클래스 중 하나라도 접근 허가로 결론을 내면 접근 허가로 판단한다. 거부의 개수와 관계없이 하나 만이라도 승인이 되면 승인이 된다.
    - ConsensusBased - 다수 표에 의해 최종 결정을 판단한다. 동수일 경우 기본은 접근허가이나 allowfEqualGrantedDeniedDecisions을 false로 설정할 경우 접근 거부로 결정 된다.
    - UnanimousBased - 모든 보터가 만장일치로 접근을 승인해야 하며 그렇지 않은 경우 접근을 거부한다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img10.JPG" />

- 각각의 Voter 마다 사용자 요청에 의해서 현재 자원에 접근할 자격이 있는지 결정을 AccessDecisionManager에게 리턴하는 클래스
- 접근 거부 → -1, 접근 보류 → 0
- 결정방식의 값을 계산해서 AccessDecisionManager가 결정한다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img11.JPG" />

## 11. 스프링 시큐리티 필터 및 아키텍처 정리

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0624/img12.JPG" />