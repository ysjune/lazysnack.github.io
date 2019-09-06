## Circuit Breaker 적용
API 통신에서 서킷 브레이커를 적용해야 한다는 얘기를 들었고, 이 역시 처음 듣는 용어라 일단 이해를 위해..

### Circuit Breaker 란?
1. 회로차단기로 불리며, 전기적 회로를 과전압 등으로부터 보호하기 위한 장치
2. 시스템에서 장애의 확산을 막기위해 (결함을 격리) 사용되는 패턴의 일종
3. Netflix 의 Hystrix가 있음

즉, 통신에 문제가 있을 때, pool 을 계속 점유하는 현상을 막기 위해, 차단기를 open 하여 격리한다고 생각하면 될 거 같다.  
뭔가 개념적으로는 이해가 되면서도 헷갈리는 개념이랄까나.. (~~이해안됐다는 소리...?~~)  
여하튼 본론으로 들어가서 이전에 적용했던 Feign 에 서킷브레이커를 적용시켜보도록 함.  
- [resilience4j-feign](https://github.com/resilience4j/resilience4j/tree/master/resilience4j-feign) 을 사용
- [resilience4j-circuitbreaker docs](https://resilience4j.readme.io/docs/circuitbreaker) - 서킷브레이커 설명이 잘 나온거 같다.

#### 1. 의존성 추가
이전과 같이 maven 을 사용하므로 pom.xml 에 추가해줌.
```
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-feign</artifactId>
            <version>0.17.0</version>
        </dependency>
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-circuitbreaker</artifactId>
            <version>0.17.0</version>
        </dependency>
```

#### 2. Config Decorating
1. (1)에서 작성한 [AAAClientConfig](https://github.com/ysjune/study/blob/master/JAVA/Feign%20%EC%82%AC%EC%9A%A9(1).md) 를 변경
```
    @Configuration
    public class AAAClientConfig {
    
    @Value("${url}")
    private String url;

    CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50) // 실패 임계치 50% 로 설정
            .ringBufferSizeInClosedState(20) // 20번 시도 
            .ringBufferSizeInHalfOpenState(10) // 10초 후 10번 시도
            .waitDurationInOpenState(Duration.ofSeconds(3L)) // Open 후 3초 기다림
            .build();

    CircuitBreaker circuitBreaker = CircuitBreaker.of("AAAClientCB", config);

    FeignDecorators decorators =
            FeignDecorators.builder().withCircuitBreaker(circuitBreaker).build();

    @Bean
    public AAAClient aAAClient() {
        return Resilience4jFeign.builder(decorators)
                .client(new OkHttpClient())
                .encoder(new JacksonEncoder())
                .decoder(new JacksonDecoder())
                .options(new Request.Options(3000, 5000))
                .retryer(Retryer.NEVER_RETRY)
                .logger(new Slf4jLogger(AAAClient.class))
                .logLevel(Logger.Level.FULL)
                .target(AAAClient.class, url);
    }
}
```
#### 3. Test 코드 작성

1. 정상적인 경우
2. 일부러 실패를 했을 때의 서킷브레이커 open close 확인

```
public class CBTest {
    @Rule
    public WireMockRule wireMockRule = new WireMockRule();
    
    private AAAClient client;
    
    @Before
    public void setUp() {
        
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50) // 실패 임계치 50% 로 설정
            .ringBufferSizeInClosedState(20) // 20번 시도 
            .ringBufferSizeInHalfOpenState(10) // 3초 후 10번 시도
            .waitDurationInOpenState(Duration.ofSeconds(3L)) // Open 후 3초 기다림
            .build();

        CircuitBreaker circuitBreaker = CircuitBreaker.of("AAAClientCB", config);

        FeignDecorators decorators =
              FeignDecorators.builder().withCircuitBreaker(circuitBreaker).build();
    
        aAAClient = Resilience4jFeign.builder(decorators)
                .client(new OkHttpClient())
                .encoder(new JacksonEncoder())
                .decoder(new JacksonDecoder())
                .options(new Request.Options(3000, 5000))
                .retryer(Retryer.NEVER_RETRY)
                .logger(new Slf4jLogger(AAAClient.class))
                .logLevel(Logger.Level.FULL)
                .target(AAAClient.class, "http://localhost:8080");
    }

    @Test
    public void 정상인경우_Test() {

        stubFor(
                get(urlPathMatching("/1"))
                        .willReturn(
                                aResponse()
                                        .withStatus(200)
                                        .withHeader("content-type", "application/json")
                                        .withBody("{}")
                        )
        );

        boolean allSuccess = IntStream.range(0, 40)
                .mapToObj(i -> Try.of(() -> client.getDetail(1)))
                .allMatch(t -> t.isSuccess());

        assertThat(allSuccess).isTrue();
    }
    
    @Test
    public void 서킷브레이커_Open_Close_Test() {

        stubFor(
                get(urlPathMatching("/71"))
                        .willReturn(
                                aResponse()
                                        .withStatus(200)
                                        .withHeader("content-type", "application/json")
                                        .withBody("{}")
                        )
        );

        stubFor(
                get(urlPathMatching("/72"))
                        .willReturn(
                                aResponse()
                                        .withStatus(500)
                                        .withHeader("content-type", "application/json")
                                        .withBody("{}")
                        )
        );

        // 에러 비율 50% 이상
        IntStream.range(0, 10).forEach(i -> Try.of(() -> client.getDetail(72)));

        // 3초간 오픈 상태에서 바로 에러
        sleep(1000);
        Try<ApiResponse> failFast1 = Try.of(() -> client.getDetail(71));
        sleep(1500);
        Try<ApiResponse> failFast2 = Try.of(() -> client.getDetail(71));

        // 3초 후 반 오픈
        sleep(1000);
        Try.of(() -> client.getDetail(71));
        Try<ApiResponse> realEx = Try.of(() -> client.getDetail(72));// 1개 실패로는 50% 비율 안 됨

        // 반 오픈 후 다시 열리지 않으면 모두 성공
        boolean allSuccess = IntStream.range(0, 40)
                .mapToObj(i -> Try.of(() -> client.getDetail(71)))
                .allMatch(t -> t.isSuccess());

        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(failFast1.getCause()).isInstanceOf(CallNotPermittedException.class);
            softly.assertThat(failFast2.getCause()).isInstanceOf(CallNotPermittedException.class);
            softly.assertThat(realEx.getCause()).isNotInstanceOf(CallNotPermittedException.class);
            softly.assertThat(allSuccess).isTrue();
        });
    }

    private void sleep(int sleepTime) {
        try {
            Thread.sleep(sleepTime);
        } catch (InterruptedException e) {
        }
    }
}
```

- SoftAssertions은 각 assertThat 이 실패해도 검증을 계속 실행하고 마지막에 결과를 한꺼번에 보여줌   
   - (마지막에 softly.assertAll() 추가)
- softly.assertAll() 을 사용하지 않을 경우에 대해선 [AssertJ](https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#JUnitSoftAssertions) 의 Collect all errors with soft assertions 부분 참조
