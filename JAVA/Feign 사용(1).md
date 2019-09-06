이번 API 통신을 하면서 Feign 을 사용해봤는데, 어떻게 사용했는지 적어보고자 함.


## Feign 이란?
- Java to HTTP client binder
- Netflix 에서 개발함
- <https://github.com/OpenFeign/feign>

이라고 간단하게만,

API 통신을 이용해서 개발할 필요가 생겼는데, API 를 통하여 개발해 본 적이 없기에  

그래서 무작정 구글링 시작 
-> httpClient 를 이용해서 통신하는 것을 알게 됨 -> 관련 코드를 보면서 분석 시작 <br>
-> Spring 에서는 restTemplate 를 이용한다는 얘기를 들음 -> restTemplate 구글링 및 테스트 코드 작성 <br>
-> Feign 이라는 것을 들음 -> Feign 적용해보자 <br>
  
라는 흐름으로 진행. (무려 하루만에 진행된 흐름..)  
테스트 코드를 작성하면서 wireMock, MockMvc, Mockito 등도 사용했는데, 이는 추후에 따로..
<br>
#### 1. 의존성 추가
  - maven 을 사용하므로 pom.xml 에 의존성 추가
```
    <dependency>
          <groupId>io.github.openfeign</groupId>
          <artifactId>feign-jackson</artifactId>
          <version>10.2.3</version>
    </dependency>
    <dependency>
          <groupId>io.github.openfeign</groupId>
          <artifactId>feign-okhttp</artifactId>
          <version>10.2.3</version>
    </dependency>
    <dependency>
          <groupId>io.github.openfeign</groupId>
          <artifactId>feign-slf4j</artifactId>
          <version>10.2.3</version>
    </dependency>
```

#### 2. Feign 인터페이스 작성  
FeignClient Annotation이 있으나 깃 readme 에 있는 basic 방법을 사용.
```
public interface AAAClient {

    @RequestLine("GET /list")
    ApiResponse<Data> getList(@QueryMap SearchRequest SearchRequest);
    
    @RequestLine("GET /{boardId}")
    ApiResponse<BoardDetail> getDetail(@Param(value = "boardId") long boardId);
    
    @RequestLine("PUT /{boardId}")
    @Headers("Content-Type: application/json")
    ApiResponse<String> putBoard(@Param("boardId") long boardId, BoardDetail boardDetail);
}
```
- Annotation 사용에 대해
    1. @RequestLine - requestMapping 과 같은 개념, url 앞에 붙여주는 GET, POST, PUT, DELETE 으로 Rest 적용
    2. @Param - 넘길 파라미터 
    3. @QueryMap - Param과 같으나, param 의 파라미터가 3개 이상이 안되므로 3개이상의 경우 QueryMap 사용
        - ex) param -> list?start=1 , QueryMap -> list?start=1&end=2&limit=3
    4. @Headers - 해더를 추가해줌, 응답측에서 requestBody 를 사용할 경우 Content-Type: application/json 과 같이 추가해줘야 함.
    5. 이외에도 @Body 와 @HeaderMap 이 있음.
    
#### 3. ClientConfig 작성
2에서 만든 인터페이스에 대해 config 작성
```
@Configuration
public class AAAClientConfig {
    @Value("${url}")
    private String url;
    
    @Bean
    public AAAClient aAAClient() {
        return Feign.builder()
                .retryer(new Retryer.Default())
                .client(new OkHttpClient())
                .encoder(new JacksonEncoder())
                .decoder(new JacksonDecoder())
                .logger(new Slf4jLogger(AAAClient.class))
                .target(AAAClient.class, url);
    }
}
```

- 빌더 패턴을 사용한 것을 알 수 있음.
- 익숙한(?) httpClient 가 보임.

#### 4. Service 생성

```
@Service
public class BoardService {

    private final AAAClient aAAClient;

    public static final Logger LOGGER = LoggerFactory.getLogger(BoardService.class);

    @Autowired
    public BoardService(AAAClient aAAClient) {
        this.aAAClient = aAAClient;
    }
    
    public ApiResponse<Data> getList(SurveySearchRequest vo) {
        ApiResponse<Data> response;
        try {
            response = aAAClient.getList(vo);
        } catch (FeignException ex) {
            LOGGER.error("FeignException {}", ex);
        }
        return response;
    }
    // 생략
}
```

#### 5. Test 코드 생성

1. API 와 직접 통신할 필요는 없음
2. 해당 서비스 로직이 정상적으로 작동하는지 여부만 확인

```
public class ServiceTest {

    @Rule
    public WireMockRule wireMockRule = new WireMockRule();    
    @Rule
    public MockitoRule rule = MockitoJUnit.rule();
    private BoardServiceService service;
    
    @Before
    public void setUp() {
        AAAClient aAAClient = Feign.builder()
                .client(new OkHttpClient())
                .encoder(new JacksonEncoder())
                .decoder(new JacksonDecoder())
                .logger(new Slf4jLogger(SurveyClient.class))
                .logLevel(Logger.Level.FULL)
                .target(AAAClient.class, "http://localhost:8080");
        service = new AAASurveyService(aAAClient);
    }
    
    @Test
    public void 목록_test() {
        SearchRequest req = new SearchRequest();
        req.setStart(0);
        req.setLimit(10);
        StubMapping accept = stubFor(
                get(urlPathMatching("/list"))
                        .withQueryParam("start", equalTo(req.getStart().toString()))
                        .withQueryParam("limit", equalTo(req.getLimit().toString()))
                        .willReturn(aResponse()
                                .withStatus(200)
                                .withHeader("content-type", "application/json")
                                .withBody("{}")
                        )
        );
        ApiResponse<Data> ret = service.getList(req);
        assertEquals("200", ret.getCode());
        assertEquals("OK", ret.getMessage());
```

Feign 설정 및 적용은 이렇게 해서 끝.  
추후 컨트롤러를 통해 실제 API 와 연결되는지 테스트가 필요. (MockMvc 사용)  
