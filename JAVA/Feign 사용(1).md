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

- 의존성 추가
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

- Feign 인터페이스 작성
FeignClient Annotation이 있으나 깃 readme 에 있는 basic 방법을 사용.
```
public interface AAClient {

    @RequestLine("GET /list")
    ApiResponse<Data> getList(@QueryMap SearchRequest SearchRequest);
    
    @RequestLine("GET /{boardId}")
    ApiResponse<BoardDetail> getDetail(@Param(value = "boardId") long boardId);
    
    @RequestLine("PUT /{boardId}")
    @Headers("Content-Type: application/json")
    ApiResponse<String> putBoard(@Param("boardId") long boardId, BoardDetail boardDetail);
```




