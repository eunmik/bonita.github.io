---
layout: post
title: "실무에서 Test Code를 적용하면서..."
date: 2022-05-03
excerpt: "테스트 코드의 중요성을 깨닫게 된 계기"
tags: ["Junit5", "TestCode"]
comments: true
---


현재(2022년 기준) 다니고 있는 회사에 코드에는 테스크 코드가 없다. 전에 계셨던 이사님이 테스트 코드를 작성하려고 하신 흔적은 있지만 대부분의 기능에는 테스트 코드가 없다. 
사실 나도 이 회사에서 개발자로서는 처음 입사했고 중소기업 특성 상 제대로 된 사수도 없었기에 초기에는 테스트 코드가 얼마나 중요한지 자각하지 못하였다. 

그리고 테스트 서버도 없었다. 내가 입사하고 1년이 지나고 나서 생겼다. 그 전에는 Postman으로 API를 호출해서 잘 나오면 실서버에 배포하고 배포 하고 나서 문제 생기면 롤백하고 이런 식으로 프로세스가 있었다. 지금보면 정말 위험하고 고객에 제공하는 서비스인데 무책임하다고 느껴지기도 한다. 

재직하면서 인프런 강의나 다양한 유투브 채널, 클린 코드 북클럽, 스터디를 통해서 테스트 코드의 중요성을 깨닫고 현재 하는 업무는 혼자하는 작업이고 코드리뷰를 해줄 사람이 없기 때문에 더욱더 테스트 코드는 필수라고 생각이 들었다. 그래서 나는 퇴사하기 전에 회사 코드에 **테스트 코드를 적용하기로 결심 했다!** 

왜 이 생각을 하는데 1년이 넘게 걸렸을까? 조금 더 빨리 했으면 좋았을 텐데.. 싶지만 그래도 **BETTER LATE THAN NEVER!** 나의 업무를 인수인계 받으시는 분이 좀더 나은 개발환경에서 개발을 할 수 있도록 제공하고 싶다. 

SpringBoot에서 JUnit5를 사용해서 테스트코드를 작성해 보았다. 

## Unit Tests

### 파일 업로드 관련 서비스 테스트 코드

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.mock.web.MockMultipartFile;

import java.io.FileInputStream;
import java.io.IOException;

@ExtendWith(MockitoExtension.class)
class FileServerServiceTest {

    @InjectMocks
    FileServerService fileServerService;

    @Test
    @DisplayName("파일 업로드 테스트")
    void uploadFileWithData() throws IOException {
        //given
        String filePath = "D:\\tmp";
        String fileName = "Test_File";
        MockMultipartFile imageData = new MockMultipartFile("data", "filename.png", "image/png", new FileInputStream("testImg\\test.png"));

        String result = fileServerService.uploadFileWithData(filePath, fileName, imageData);
        Assertions.assertEquals("Success", result);
    }

}
```

- @ExtendWith(MockitoExtention.class) : Mockito의 Mock 객체를 사용하기 위한 Annotation
- @InjectMocks : 실제 객체를 생성하고 mock 의존성을 주입하는 Annotation. 테스트 클래스에서 테스트의 대상이 되는 인스턴스를 생성할 때 사용한다.
- MockMultipartFile : MultipartFile 인터페이스의 Mock 구현.
- Aseertions.assertEquals(expected, actual) : 기대한 값과 실제 값이 같은지 비교하는 함수

### RestTemplate관련 서비스 테스트 코드

```java
import lombok.SneakyThrows;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class RestTemplateClientTest {
    @InjectMocks
    RestTemplateClient restTemplateClient ;

    @Mock
    private TestRestTemplate testRestTemplate;

    @Mock
    ServiceConfiguration config;

    private static final String LICENSE_UNREG_SCENARIO_URL_FORMAT = "call/{userId}";

    @Test
    @SneakyThrows
    void unregisterScenario() {
        //given
        String userId = "eunmi@test.com";
        int scenarioId = 123456; 
        this.config.licenseManagerUrl = "https://localhost:2135";

        ResponseEntity<UpdateResult> responseEntity = new ResponseEntity<>(HttpStatus.OK);
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer 16bd5cb6-72bf-43e1-a4e6-bb40e6400222");
        when(testRestTemplate.exchange(this.config.licenseManagerUrl + LICENSE_UNREG_SCENARIO_URL_FORMAT, HttpMethod.DELETE, null, UpdateResult.class, userId, Integer.valueOf(scenarioId))).thenReturn(responseEntity);

        //when
        boolean actualResult = restTemplateClient.unregisterScenario(userId, scenarioId);

        //then
        Assertions.assertEquals(true, actualResult);
    }
}
```

- @Mock : 가짜 객체인 Mock 객체
- @SneakyThrows : throws 선언을 하지 않고도 checked exception을 던질 수 있도록 해주는 annotation
- when() :Mock객체를 위해 RestTemplate이 호출이 됐을 때 어떤 return값을 반환해야 하는지 설정하는 메소드

Unit Test를 작성하면서 지금까지는 발견하지 못했던 부분에서 nullPointerException이 날 수 있는 경우를 발견 했다. 서비스하면서 아직 문제가 발견되지 못했지만 테스트 코드를 통해 그 경우의 수를 발견 했기 때문에 테스트 코드의 중요성을 다시한번 깨닫게 되는 순간이 되었다. 

이번엔 통합 테스트를 살펴 보겠다

## Integration Tests

### 

```java
import com.argos_labs.rpa.stu.ServiceConfiguration;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.*;

import org.springframework.test.web.servlet.MockMvc;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest()
@AutoConfigureMockMvc
class AppScenarioEditorControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    ServiceConfiguration config;

    @Test
    void aliveTest() throws Exception {
        String token = getAccessToken(config.authServerUri);
        mockMvc.perform(get("/AppScenarioEditorService/aliveTest").header(HttpHeaders.AUTHORIZATION, "Bearer " + token))
                .andDo(print())
                .andExpect(status().isOk());

    }

    public RestTemplate restTemplate;

    public String getAccessToken(String configUrl) throws Exception {
        restTemplate = new RestTemplate();
        String clientId = "sv-argos-user";
        String grant_type = "password";
        String userid = "test@vivans.net";
        String password = "eed56d004b89799f66ccb1dea2a480b532ac1d1a57488173f18eb0ffa98a0a19";
        String seed = "7fddd9a6c699e617c7a726c702f0d0b0a4faa3aff59d0c06246d7770532d53d3";

        MultiValueMap<String, String> parameters = new LinkedMultiValueMap<>();
        parameters.set("grant_type", grant_type);
        parameters.set("client_id", clientId);
        parameters.set("userid", userid);
        parameters.set("password", password);
        parameters.set("seed", seed);
        parameters.set("username", userid);

        HttpHeaders headers = new HttpHeaders();
        headers.set(HttpHeaders.AUTHORIZATION, "Basic c3YtYXJnb3MtdXNlcjo2ZWRhNzc2MGEwNjA2YTZhNThhNDZmYjk5ZDJkZWQwNg==");

        HttpEntity request = new HttpEntity(parameters, headers);

        ResponseEntity<Map> result = restTemplate.postForEntity(configUrl + "oauth/token", request, Map.class);
        return (String) result.getBody().get("access_token");
    }
}
```

- @SpringBootTest : 통합테스트를 할때 사용되는 Annotation. 실제 어플리케이션을 기동하며 포트주소가 주어지고 실제 데이터베이서에 연결되는 테스트를 수행하도록 한다.
- @AutoConfigureMockMvc : MockMvc의 auto-configuration을 테스트 클래스에 적용하고 설정하기 위해 사용되는 Annotation.

## 통합테스트에서 느낌 어려웠던 점

### Authorization Token

통합 테스트를 할 때는 Spring Security가 적용이 되어있어서 Authorization이 필요했다. 이 부분을 어떻게 적용해야 할 지 많이 헤맸던 구간이다. 그렇다고 테스트할 때만 security 적용을 뺄 수 없고.. 그건 전혀 통합테스트와 맞지 않는 행동이니깐! 
그래서 RestTemplate으로 oauth2 서버를 호출해서 token을 받아서 적용해서 사용하기로 했다! 

### NullPointerException

통합테스트에서 Service 클래스에서 Mapper가 가져오는 데이터에서 자꾸 NullPointerException이 났다. 그래서 통합테스트에서도 Unit Test처럼 Mock 객체를 만들어줘야 하는건가 고민했었다.

```java---
layout: post
title: "[인프런 이력서 오픈 마이크] 2회차/회사가 원하는 사람 이해하기"
date: 2022-04-29
excerpt: "나는 회사가 원하는 역량 가지고 있는 사람일까?"
tags: ["Resume", "Inflearn"]
comments: true
---

given(pcScenarioEditorMapper.select_company_code_from_scenario_hash_key(scenario.getHashKey(), null)).willReturn(scenario);
given(pcScenarioEditorMapper.select_company_code_using_userId(authUserId)).willReturn("H0TQ9");
```

하지만 문제의 원인은! 단위 테스틑 Mock 객체를 사용함으로써 실제 데이터를 사용하는게 아니라 가짜로 정의한 데이터와 bean 객체를 사용하는데 통합테스트에서는 실제 애플리케이션이 작동되는 flow를 확인 하기 위해 실제 데이터를 사용한다. 

근데 나는 이걸 이해 못하고 통합테스트에서도 단위 테스트에서 썼던 데이터(가짜)를 그대로 가져와서 쓰다보니깐 실제 DB에 없는 데이터를 가져오려고 하다보니 nullpointerexception이 발생하였다! 

이번 기회로 확실히 단위테스트와 통합테스트를 이해할 수 있었다!!