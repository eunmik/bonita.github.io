---
layout: post
title: "How to implement Social login - Step 3. Integrate with React"
date: 2022-03-31
excerpt: "How to implement Social Login into your webservice"
tags: ["API", "Login", "Oauth2", "Java"]
comments: true
---
index.js

```java
export const API_BASE_URL = 'http://localhost:8080';
export const ACCESS_TOKEN = 'accessToken';

export const OAUTH2_REDIRECT_URI = 'http://localhost:3000/oauth2/redirect'

export const GOOGLE_AUTH_URL = API_BASE_URL + '/oauth2/authorize/google?redirect_uri=' + OAUTH2_REDIRECT_URI;
export const FACEBOOK_AUTH_URL = API_BASE_URL + '/oauth2/authorize/facebook?redirect_uri=' + OAUTH2_REDIRECT_URI;
export const GITHUB_AUTH_URL = API_BASE_URL + '/oauth2/authorize/github?redirect_uri=' + OAUTH2_REDIRECT_URI;
export const NAVER_AUTH_URL = API_BASE_URL + '/oauth2/authorize/naver?redirect_uri=' + OAUTH2_REDIRECT_URI;
export const KAKAO_AUTH_URL = API_BASE_URL + '/oauth2/authorize/kakao?redirect_uri=' + OAUTH2_REDIRECT_URI;
```

Login.js

```java
class SocialLogin extends Component {
    render() {
        return (
            <div className="social-login">
                <a className="btn btn-block social-btn google" href={GOOGLE_AUTH_URL}>
                    <img src={googleLogo} alt="Google" /> Log in with Google</a>
                <a className="btn btn-block social-btn facebook" href={FACEBOOK_AUTH_URL}>
                    <img src={fbLogo} alt="Facebook" /> Log in with Facebook</a>
                <a className="btn btn-block social-btn github" href={GITHUB_AUTH_URL}>
                    <img src={githubLogo} alt="Github" /> Log in with Github</a>
                <a className="btn btn-block social-btn naver" href={NAVER_AUTH_URL}>
                    <img src={naverLogo} alt="Naver" /> Log in with Naver</a>
                <a className="btn btn-block social-btn kakao" href={KAKAO_AUTH_URL}>
                    <img src={kakaoLogo} alt="Kakao" /> Log in with Kakao</a>
            </div>
        );
    }
}
```

<aside>
🔖 reference
[spring-boot-react-oauth2-social-login-demo](https://github.com/callicoder/spring-boot-react-oauth2-social-login-demo, "spring-boot-react-oauth2-social-login-demo")

</aside>