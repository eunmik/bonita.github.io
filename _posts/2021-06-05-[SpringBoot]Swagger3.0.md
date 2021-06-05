---
layout: post
title: "[SpringBoot] Swagger3.0 적용하기 "
date: 2021-06-05
excerpt: "SpringBoot에서 Swagger 3.0 적용해 보자 "
tags: [SpringBoot, Swagger], OpenAPI]
comments: true
---

## Swagger 란?

REST API를 자동으로 문서화를 도와주는 오픈 소스 프레임 워크이다. 

Product Managers, Testers 와 Developers 사이에서 공유되는 문서로서도 사용될 수 있고 
API 관련 프로세스를 자동화 하기 위해 다양한 툴에서도 사용 될 수 있다. 

공식 홈페이지 : [https://swagger.io/](https://swagger.io/)

## Swagger 2 vs 3

### Metadata의 변화

```java
"swagger":2.0 --> "openapi":"3.0.0"
```

Swagger의 모든 reference는 **openapi**로 변경 되었다. 그러므로 버전도 변경 되어야 한다. 

### 구조 변화

<img src ="https://eunmik.github.io/bonita/assets/img/210605-swaggerdiff.png" />

### EndPoint URL 정의 변화

Swagger 2.0 에서는 API endPoint URL 정의가 3개로 나눠 졌다 : host, basePath, schemas 그리고 endpoint URL은 이 세개의 컴포넌트의 값으로 이루어 졌다. 
2.0에서는 주어진 API에 대해 오직 하나의 URL만 정의할 수 있다. 

반면에 OpenAPI 3.0에서는 여러개의 URL을 정의할 수 있다. 그리고 다양한 property로 template을 정의할 수 있게 되었다. 

출처 : [OpenAPI 3.0 vs Swagger 2.0](https://medium.com/@tgtshanika/open-api-3-0-vs-swagger-2-0-94a80f121022)
## Spring Boot에 OpenAPI 3.0 적용하기

1. pom.xml에 dependency를 추가 한다. 

    ```xml
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-ui</artifactId>
        <version>1.2.19</version>
    </dependency>
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-webmvc-core</artifactId>
        <version>1.2.19</version>
    </dependency>
    ```

2. Swagger Config 파일을 만들어 준다. 

```java
import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.*;
import io.swagger.v3.oas.models.servers.Server;
import io.swagger.v3.oas.models.tags.Tag;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

import java.util.Arrays;

public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {

        return new OpenAPI()
                .info(new Info().title("Google Calendar API").version("100"))
								.addTagsItem(new Tag().name("mytag"));
    }

    @Bean
    public OpenAPI customOpenAPI(@Value("3.0") String appVersion) {
        final String securitySchemeName = "oauth2";
        return new OpenAPI()
                .addSecurityItem(new SecurityRequirement()
                        .addList(securitySchemeName))
                .servers(Arrays.asList(new Server().url("http://localhost:1991")))
                .info(new Info().title("SpringShop API").version(appVersion)
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")));
    }
}
```

3. Swagger UI 확인 하기 

http://localhost:1991/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config

<img src ="https://eunmik.github.io/bonita/assets/img/210605-swaggerui.png" />


