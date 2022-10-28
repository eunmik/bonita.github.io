---
layout: post
title: "[Spring Security] Refresh Token 갱신하기"
date: 2021-08-12
excerpt: "Refresh Token이란? 갱신할수있을까?"
tags: [Spring Security, Refresh Token]
comments: true
---
## Refresh Token 이란?

보안의 이유로, access token은 짧은 시간동안만 유효하다.  유효시간이 끝나면, 사용자 어플리케이션은 refresh token으로 access token을 갱신(refresh) 할 수 있다. 

refresh token은 사용자 어플리케이션이 다시 로그인 할 필요 없이 새로운 access token을 발급 하도록 해주는 자격증명 artifact 이다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0812/img1.png" />

사용자는 refresh token이 유효한 이상 새로운 access token을 받을 수 있다. 

## Refresh Token을 사용할 때

필수 파라미터 : 

grant_type : refresh_token으로 설정 되어야 한다. 

refresh_token : 전에 사용자한테 발급된 refresh token이다. 

처음에 token을 발급 받을 때 

```json
curl --location --request POST 'http://localhost:8083/oauth/token' \
--header 'Authorization: Basic c3YtYXJnb3MtdXNlcjo2ZWRhNzc2MGEwNjA2YTZhNThhNDZmYjk5ZDJkZWQwNg==' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: JSESSIONID=7CDE43176D086328E067D5461762EB98' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'scope=read' \
--data-urlencode 'client_id=sv-argos-user' \
--data-urlencode 'auth_type=local' \
--data-urlencode 'userid=em.kong2210@gmail.com' \
--data-urlencode 'username=em.kong2210@gmail.com' \
--data-urlencode 'password=73221f4eb4f718f0dbd9e027f3f069b928cef38fd4ca7a59b0c118328f26ad2e' \
--data-urlencode 'seed=db2cd4f4d1814ebcd2c99232d9eabcb5a4b974650d0ec96b2b462a229bf7d287'
```

```json
{
    "access_token": "985ad63e-cbc8-4b52-993e-48406579a286",
    "token_type": "bearer",
    "refresh_token": "fdeaec58-790c-45d8-9ada-6dbd0b037731",
    "expires_in": 3599,
    "scope": "read"
}
```

refresh token으로 access token 갱신 했을 때 

```json
curl --location --request POST 'http://localhost:8083/oauth/token' \
--header 'Authorization: Basic c3YtYXJnb3MtdXNlcjo2ZWRhNzc2MGEwNjA2YTZhNThhNDZmYjk5ZDJkZWQwNg==' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: JSESSIONID=7CDE43176D086328E067D5461762EB98' \
--data-urlencode 'grant_type=refresh_token' \
--data-urlencode 'client_id=sv-argos-user' \
--data-urlencode 'seed=db2cd4f4d1814ebcd2c99232d9eabcb5a4b974650d0ec96b2b462a229bf7d287' \
--data-urlencode 'refresh_token=fdeaec58-790c-45d8-9ada-6dbd0b037731'
```

```json
{
    "access_token": "ca5e0a5e-9961-47fa-ad2a-fc060e93fc2f",
    "token_type": "bearer",
    "refresh_token": "fdeaec58-790c-45d8-9ada-6dbd0b037731",
    "expires_in": 3599,
    "scope": "read"
}
```

access token 갱신할 때 refresh token도 갱신하고 싶다면 

Authorization Server Config에 reuseRefreshTokens을 false로 설정해 주면 된다. 

```json
@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

		endpoints
				.allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST)
				.authenticationManager(authenticationManager)
				.tokenStore(tokenStore())
				.reuseRefreshTokens(false)
				.userDetailsService(userDetailsService())
				//for authentication error exception
				.exceptionTranslator(webResponseExceptionTranslator());

```

refresh token으로 access token을 갱신하고 

다시 같은 refresh tokne으로 호출 하면 아래처럼 refresh token이 만료된것을 확인할 수 있다.

```json
{
    "timestamp": "2021-08-11T08:07:56.977+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "Invalid refresh token: 31b42f1f-f1c7-449a-81e2-a530544097f8",
    "path": "/oauth/token"
}
```