이전에 회사에서 다른 프로젝트 소스를 보다가 http 헤더에서 보낸 정보를 controller 단에서 바로 vo 로 매핑해주는 코드를 발견했다.  
당시에 이거 꽤 유용하게 쓸 수 있겠네. 하고 지나가기만 했었는데, 이번에 기회가 생겼다.

### 목표
- 클라이언트의 IP 를 컨트롤러 단에서 바로 받아서 처리하고자 함.

#### 1. ClientInfo 생성

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ClientInfo {
   private String clientIp;
}
```

- 바인딩될 객체를 생성

#### 2. ClientResolver 어노테이션 생성

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClientResolver {
}
```

#### 3. ClientHandlerMethodArgumentResolver 생성

```java
public class ClientHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(final MethodParameter methodParameter) {
        return methodParameter.getParameterType().equals(ClientInfo.class);
    }

    @Override
    public Object resolveArgument(final MethodParameter methodParameter,
                                  final ModelAndViewContainer modelAndViewContainer,
                                  final NativeWebRequest nativeWebRequest,
                                  final WebDataBinderFactory webDataBinderFactory) throws Exception {

        ClientInfo clientInfo = new ClientInfo(clientIp(nativeWebRequest));

        return clientInfo;
    }

    private String clientIp(final NativeWebRequest request) {
        String clientIp = request.getHeader("X-Forwarded-For");
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
     
            clientIp = request.getHeader("Proxy-Client-IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
     
            clientIp = request.getHeader("WL-Proxy-Client-IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            clientIp = request.getHeader("HTTP_CLIENT_IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            clientIp = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            HttpServletRequest nativeRequest = (HttpServletRequest) request.getNativeRequest();
            clientIp = nativeRequest.getRemoteAddr();
        }
        return clientIp;
    }
}
```

- supportsParameter 는 현재 파라미터를 resolver이 지원하는지에 대한 boolean
- resolveArgument 는 바인딩할 객체

#### 4. resolver 등록
```java
@Configuration
public class WebMvcCustomConfiguration extends WebMvcConfigurerAdapter {

	...
	
    @Override
	public void addArgumentResolvers(final List<HandlerMethodArgumentResolver> argumentResolvers) {
		argumentResolvers.add(new ClientHandlerMethodArgumentResolver());
	}
}
```

- 스프링4.x 를 사용하고 있어서 `WebMvcConfigurerAdapter` 를 상속받아 사용한 것 같지만,   
spring boot 에서는 WebMvcConfigurer 을 구현해주면 되는 것 같다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private ClientHandlerMethodArgumentResolver clientHandlerMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(clientHandlerMethodArgumentResolver);
    }
}
```


#### 5. Controller 생성

```java
@Controller
public class IntroCreationController {

    private final IntroCreationService introCreationService;

    @Autowired
    public IntroCreationController(IntroCreationService introCreationService) {
        this.introCreationService = introCreationService;
    }

    @RequestMapping(value = "/api/save/intro", method = RequestMethod.POST)
    public ApiResponse createIntro(@RequestBody IntroCreationRequest request, @ClientResolver ClientInfo clientInfo) {
        ApiResponse apiResponse;
        try {
            AdminIntroData intro = introCreationService.createIntro(request, clientInfo);
            apiResponse = ApiResponse.success(intro);
        } catch (Exception e) {
            apiResponse = ApiResponse.fail(e.getMessage());
        }
        return apiResponse;
    }
}
```

- 컨트롤러 매소드의 파라미터에 커스텀 어노테이션을 사용하면 된다.

