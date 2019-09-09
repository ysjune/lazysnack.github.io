순서가 어긋난 감이 있긴 하지만, 지난 번 API 통신을 위해 Feign 을 사용했었는데,  
테스트를 진행할 때 실제로 API 접속을 할 수는 없으니 Mock Web 서버를 만드는 기술인 WireMock 를 사용한 내용을 올리고자 한다.  

## WireMock 이란?
- HTTP-based API를 위한 시뮬레이터
- Mock 으로 만드는 웹서버
- <http://wiremock.org/>

#### 1. 의존성 추가
- Maven 을 사용하므로 pom.xml 에 의존성 추가
```
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.24.1</version>
    <scope>test</scope>
</dependency>

```

#### 2. 테스트 코드 작성
```
public class Test {

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(8089); // 작성하지 않을 경우 default 8080    
    
    private BoardServiceService service;
    ...
    
    @Test
    public void 목록_test() {
        
        StubMapping accept = stubFor(
                get(urlPathMatching("/board/([a-zA-Z0-9/-]*)")) // board/71 같은 걸 찾기 위해 정규표현식 사용
                        .willReturn(aResponse()
                                .withStatus(200)
                                .withHeader("content-type", "application/json")
                                .withBody("Hello world!")
                        )
        );
        ApiResponse<Data> ret = service.getBoard(71); // 서비스에서 API 호출 (/board/71)
        assertEquals("200", ret.getCode());
        assertEquals("OK", ret.getMessage());
        ...
    }
}
```  
  
- Junit 을 지원하기에 좀 편하게 테스트할 수 있음. (5에서도 가능할 것 같긴한데...)
- sample 은 간단하게 작성하였지만, 제공하는 기능들이 많다. (https, proxy, 상태저장 등)
- 좀 더 자세한 내용은 [WireMock](http://wiremock.org/docs/) 페이지에서 확인 가능
