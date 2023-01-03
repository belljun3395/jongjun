## Spring Cloud Gateway Docs를 통한 Spring Cloud Gateway 파악

우선적으로 Spring Cloud Gateway를 시작하기 위해서는 `spring-cloud-starter-gateway` 의존성을 추가할 필요가 있다.

이는 프로젝트의 Spring Cloud 버전에 따라 알맞는 버전을 선택하면 된다.



### 용어

+ Route :
  Gateway의 기본적인 단위이다.
  그리고 이는 ID, 도착 URI, predicate 그리고  filter의 집합으로 구성된다고 한다.
+ Prediate :
  Java 8의 Predicate로 입력 타입은 `ServerWebExchange`이다.
  이는 HTTP 요청에서 header 그리고 parameter과 같은 것과 매칭해준다고 한다.
+ Filter :
  구체적 팩토리로 생성되는 `GateFilter` 의 인스턴스다.
  Filter에서 목표하는 API에게 request 하기전 request와 response를 수정할 수 있다.



### 동작 방식

![spring_cloud_gateway_diagram](https://cloud.spring.io/spring-cloud-gateway/reference/html/images/spring_cloud_gateway_diagram.png)

1. 클라이언트가 Srping Cloud Gateway에 요청을 한다.
2. Gateway Handler Mapping이 요청과 라우터를 매칭 시킨후 Gateway Web Handler로 보낸다.
3. Gateway Web Handler는 요청에 맞는 filter chain으로 전달한다.  // check
4. 요청이 `pre` 필터를 모두 지난다면 **프록시 요청**이 만들어진다. 
   그리고 프록시 요청이 만들어진 이후 `post` 필터가 동작한다.

// todo 프록시 요청 공부



### Route Predicate 팩토리 그리고 Gateway Filter 팩토리 설정

Route Predicate 팩토리 설정 방식은 두 가지다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

두 방법의 차이는 인자를 입력하는 방식의 차이이다.

`- Cookie=mycookie,mycookievalue` 

`- name: Cookie
   args:
   	name: mycookie
    regexp: mycookievalue`

이는 Cookie를 정의한 두 가지 방식인데 이처럼 둘 중 선호하는 방식을 사용하면 될 것 같다.



### Route Predicate 팩토리

+ datetime

  + After
  + Before
  + Between

  위의 설정들을 통해서 라우터에 접근할 수 있는 요청을 받을 특정 시간을 지정할 수 있다.

  

+ Cookie

  + name
  + regexp

  위의 설정을 통해 특정 쿠키(by name) 그리고 그 값에 따라 요청을 받을 수 있다.

  

+ Header

  + name
  + regexp

  위의 설정을 통해 특정 헤더(by name) 그리고 그 값에 따라 요청을 받을 수 있다.

  

+ Host

  + name's pattern

  위의 설정을 통해 호스트 특정 패턴을 가지는 호스트의 요청을 받을 수 있다.

  

+ Method

​		위의 설정을 통해 특정 HTTP 메서드의 요청을 받을 수 있다.

+ Path

  + pathMatcher

  위의 설정을 통해 특정 path를 가지는 요청을 받을 수 있다.

  유레카를 통한 자동 경로 매핑외의 **수동 경로 매핑**에서 사용된다.

+ Query

  + regexp

  위의 설정을 통해 특정 query parameter를 가지는 요청을 받을 수 있다.

  

+ RemoteAddr

  위의 설정을 통해 특정 remoteAddr을 가지는 요청을 받을 수 있다.

  

+ Weight Route

  + group
  + wieght

  위의 설정을 통해 그룹별로 요청 전달 비율을 조절할 수 있다.

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
        - id: weight_high
          uri: https://weighthigh.org
          predicates:
          - Weight=group1, 8
        - id: weight_low
          uri: https://weightlow.org
          predicates:
          - Weight=group1, 2
  ```

  위의 경우 high에 80프로 low에 20프로의 요청이 전달된다.

  

#### Remote Addresses 분해 방법 수정 // todo



### Gateway Filter 팩토리

route filter은 들어오는 HTTP request 혹은 나가는 HTTP respons를 조작하였다.

route filter은 특정 라우터를 범위로 하고 Spring Cloud Gateway는 많은 내장된 GatewayFilter 팩토리를 포함한다.



#### AddRequestHeader, GatewayFilter 팩토리

AddRequestHeader, GatewayFilter 팩토리의 경우 name 그리고 value 파라미터를 가지고 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```

위의 예시를 보면 name으로 X-Request-red를 그리고 value로 blue를 가지고 이를 header로 넣어 이후 요청을 진행한다.

이때 AddRequestHeader는 URI 변수를 인지한다.

아래는 AddRequestHeader과 URI 변수를 사용한 예제이다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

만약 요청의 path가 `/red/blue` 라면 header에 `X-Request-Red : Blue-blue` 가 추가된다.



#### AddRequestParameter, GatewayFilter 팩토리

AddRequestParameter, GatewayFilter 팩토리의 경우 name 그리고 value 파라미터를 가지고 있고 이를 parameter에 담아서 이후 요청을 진행한다.

그리고 AddRequestParameter 역시 URI 변수를 인지할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```



#### AddResponseHeader, GatewayFilter 팩토리

AddResponseHeader, GatewayFilter 팩토리의 경우 name 그리고 value 파라미터를 가지고 있고 이를 header에 담아서 이후 응답을 진행한다.

이 역시 URI 변수를 인지할 수 있다.



#### DedupeResponseHeader, GatewayFilter 팩토리

DedupeResponseHeader GatewayFilter 팩토리는 name 그리고 옵션으로 strategy를 파라미터로 가진다.

name은 여러개 기입할 수 있으며 그 구분은 space로 한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

위의 예제는 name만 2개를 가진다.

위는 `Access-Control-Allow-Credentials` 그리고  `Access-Control-Allow-Origin` 의 중복을 제거해준다.

위와 같은 경우는 gateway와 요청 api 두 곳 모두 CORS 설정을 한 경우 나타날 수 있다.

옵션이라고 하였던 strategy의 경우 `RETAIN_FIRST` (default), `RETAIN_LAST`, 그리고 `RETAIN_UNIQUE`가 있다. 

// todo 의미 조사



#### Spring Cloud CircuitBreaker GatewayFilter 팩토리

Spring Cloud CircuitBreaker GatewayFilter 팩토리는 circuit breaker 속에서 Spring Cloud CricuitBreaker API를 Gateway route들을 감싸서 사용한다고 한다.

이 Spring Cloud CircuitBreaker GatewayFilter를 사용하려면 `spring-cloud-starter-circuitbreaker-reactor-resilience4j` 가 필요하다.

이때 `fallbackUri` , 즉 에러가 발생하였을 때 돌아가 URI가 필요하다.

그러한 경우 아래처럼 `fallbackUri` 가 controller 혹은 handler 내부에 있는 것이 가장 좋다.

아래는 circuit break 가 일어나면 `/fallback` 로 이동 시키고

이는 `/fallback` 에 응답하는 `ingredients-fallback` 즉 `http://localhost:9994` 로 이동하게 된다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

 

#### FallbackHeaders GatewayFilter 팩토리

FallbackHeaders 팩토리는 Spring Cloud CircuitBreaker execution exception의 자세한 내용을 `fallbackUri`로 향하는 요청에 추가해준다.

```yaml
- id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

이때 header에는 다음과 같은 정보들을 작성할 수 있다.

- `executionExceptionTypeHeaderName` (`"Execution-Exception-Type"`)
- `executionExceptionMessageHeaderName` (`"Execution-Exception-Message"`)
- `rootCauseExceptionTypeHeaderName` (`"Root-Cause-Exception-Type"`)
- `rootCauseExceptionMessageHeaderName` (`"Root-Cause-Exception-Message"`)

// todo 이에 대한 더 자세한 것은 [Spring Cloud CircuitBreaker Factory section](https://cloud.spring.io/spring-cloud-gateway/reference/html/#spring-cloud-circuitbreaker-filter-factory) 에서 확인 할 수 있다고 한다.



#### MapRequestHeader GatewayFilter 팩토리

MapRequestHeader GatewayFilter 팩토리는 `fromHeader` 그리고 `toHeader` 이라는 파라미터를 가지고 있다.

들어오는 요청에서 `toHeader` 이름을 가진 새로운 header을 만들고 `fromHeader` 이름을 가지는 header을 지운다.

만약 요청에 `fromHeader` 가 존재하지 않는다면 filter는 영향을 주지 못한다.

그리고 요청에 `toHeader` 가 이미 존재한다면 그 값은 새로운 값으로 바뀐다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```

위의 예제는 Blue 헤더의 값을 X-Request-Red 헤더를 만들면서 그 값으로 한다.



#### PrefixPath GatewayFilter 팩토리

PrefixPath GatewayFilter 팩토리는 하나의 prefix 파라미터를 가진다.

만약 prefix 파라미터로 `/mypath`를 가진다면 `/hello` 의 경우 `/mypath/hello` 가 된다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

 

#### PreserveHostHeader GatewayFilter 팩토리

PreserveHostHeader GatewayFilter 팩토리는 파라미터는 없다.

이는 HTTP client에 의해 결정된 host header 대신 원본 origin header을 이후 요청에 전달해 준다.

// todo HTTP client에 의해 결정된 host header과  원본 origin header 차이 알아보기



#### RequestRateLimiter GatewayFilter 팩토리 with Redis

RequestRateLimiter GatewayFilter 팩토리 with Redis를 구현하기 위해서는 우선적으로 `spring-boot-starter-data-redis-reactive` 가 필요하다.

또 알고리즘으로는 [Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket) 을 사용한다.

RequestRateLimiter GatewayFilter 팩토리는 현재 요청이 허용되서 진행될지를 결정하기 위해서 RateLimiter를 구현한다.

만약 진행되지 않을 때는 `HTTP 429 - Too Many Requests` 를 기본으로 반환한다.



이 필터는 옵션으로 `keyResolver` 를 파라미터로 받는다. 

그리고 다른 파라미터들은 ralte limiter을 구체화한다.



`keyResolver` 은 `KeyResolver`을 구현한 빈이다.

`#{@myKeyResolver}` 와 같은 SpEL을 통해  나타낼 수 있다.

이러한 빈은 다음과 같이 구현할 수 있다고 한다.

```java
@Configuration
public class CustomUserKeyResolver() {
    @Bean
    KeyResolver userKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId")); // 요청 파라미터에서 userId를 가져온다.
    }
}
```

// todo 코드를 대충 이해는 하겠지만 정확한 이해는 하지 못해 추가적인 공부가 필요할 것 같다.



이제 앞서 rate limter을 구체화하는 다른 파라미터들에 대해 알아보자.

+ `key-resolver` 우리가 선언한 bean이름을 주입 해 준다.
+ `redis-rate-limiter.requestedTokens` 요청시에 소모되는 토큰의 개수이다.
+ `redis-rate-limiter.burstCapacity` 버킷의 담겨져있는 최대량, 즉 유저가 1초에 할 수 있는 최대 요청 횟수를 제한하는 것이다.
+ `redis-rate-limiter.replenishRate` 초당 버킷 회복량, 즉 유저가 누락되는 요청없이 초당 얼만큼의 요청을 허용할 지를 설정하는 것이다.



// todo 아래 이해하기

---

`replenishRate`와 `burstCapacity`에 동일한 값을 설정하면 일정한 비율로만 요청을 허용할 수 있다. `burstCapacity`를 `replenishRate` 보다 높게 설정하면 일시적으로 급증하는 요청(temporary bursts)을 허용할 수 있다. 이 땐 버스트가 두 번 연속되면 요청을 드랍하게 되므로 (`HTTP 429 - Too Many Requests`), 최초 버스트 발생 후 일정 시간이 지나야 요청을 허용할 수 있다 (`replenishRate`에 따라). 다음은 `redis-rate-limiter` 설정 예시다:

아래는 `replenishRate`을 원하는 요청 수로, `requestedTokens`를 시간 범위로(초) 설정하고, `replenishRate`와 `requestedTokens`에 따라 `burstCapacity`를 산출해 `1 request/s`로 속도를 제한한다. 예를 들어 `replenishRate=1`, `requestedTokens=60`, `burstCapacity=60`을 설정하면 `1 request/min` 으로 제한 된다.

---



우선 예제를 보자.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
	          key-resolver: "#{@userKeyResolver}"	
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

위를 보면 1초당 요청 제한을 10으로 하고 버스트는 20만큼 허용한다.

그렇기에 버스트 후 1초 동안은 10개의 요청만 허용된다.

// todo 참고자료 https://dgle.dev/RateLimiter1/



#### RedirectTo GatewayFilter 팩토리

RedirectTo GatewayFilter 팩토리는 status 그리고 url을 파라미터로 가진다.

status의 경우 300번대 redirect HTTP 코드를 사용해야한다.

url의 경우 유요한 URL이어야 한다.



#### RemoveRequest/ResponseHeader GatewayFilter 팩토리

RemoveRequest/ResponseHeader GatewayFilter 팩토리의 경우 name 파라미터를 받고 그에 해당하는 header을 지운다.

그 이후 요청을 진행하고 응답을 리턴한다.



**sensitive header 종류를 삭제하려면 반드시 RemoveResponseHeader 설정을 해주어야 한다.**

이때 Route 마다 필터를 설정할 수 있지만 `spring.cloud.gateway.default-filters` 를 통해 한번에 sensitive header 삭제를 지정할 수 있다.

**이때 sensitve header의 기본 값은 `Cookie, Set-Cookie, Authorization` 이다.**



#### RemoveRequestParameter GatewayFilter 팩토리

RemoveRequestParameter GatewayFilter 팩토리는 name 파라미터를 가지고 이에 해당하는 query 파라미터를 지운후 요청을 넘긴다.



#### RewritePath GatewayFilter 팩토리

RewritePath GatewayFilter 팩토리는 regexp 그리고 replacement를 파라미터로 가진다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red/?(?<segment>.*), /$\{segment}
```

위의 예제와 같은 설정시 `/red/blue` 요청을 받으면 요청을 넘기기 전에 path를 `blue` 로 설정한다.



#### RewriteLocationResponseHeader GatewayFilter 팩토리

RewriteLocationResponseHeader GatewayFilter 팩토리는 백엔드의 구체적인 요소들(backend-specific details)을 제거하려고 응답 header의 `Location` 의 값을 수정한다.

이는 `stripVersionMode`, `locationHeaderName`, `hostValue`, 그리고 `protocolsRegex` 를 파라미터로 가진다.

// todo 각각의 파라미터 의미 알아보기

stripVersionMode는 다음과 같은 속성을 통해서 설정할 수 있는데 그 속성은 다음과 같다.

+ `NEVER_STRIP`: 원래 요청 path에 버전이 없더라도 버전을 제거하지 않는다.
+ `AS_IN_REQUEST`: 원래 요청 path에 버전이 없을 때에만 버전을 제거한다.
+ `ALWAYS_STRIP` : 원래 요청 path에 버전이 있더라도 무조건 버전을 제거한다.

hostValue 파라미터의 경우 `Location`의 `host:posrt` 부분을 치환할 때 사용한다.

파라미터를 제공하지 않는다면 요청 헤더에 있는 `Host`값을 사용한다.

`protocolsRegex` 파라미터는 프로토콜 명을 매칭할 유요한 정규식 `String`이어야 한다.

매칭되지 않으면 필터에선 아무 작업도 수행하지 않으며 기본값은 `http|https|ftp|ftps`다.



#### RewriteResponseHeader GatewayFilter 팩토리

RewriteResponseHeader GatewayFilter 팩토리는 name, regexp 그리고 replacement를 파라미터로 가진다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

위의 예제의 경우 header `X-Response-Red` 의 값이 `/42?user=ford&password=omg!what&flag=true`라면 이 값이 `/42?user=ford&password=***&flag=true`로 전달된다.



#### SaveSession GatewayFilter 팩토리

SaveSession GatewayFilter 팩토리는 요청이 전달되기전에 WebSession이 저장되는 것을 강제한다.

이는 `Spring Session` 등을 lazy 데이터 저장소에 저장하며, 요청 전달이전에 세션을 저장하여야 할때 많이 사용한다.

// todo Spring Session 공부하기

이는 Spring Security와 Spring Session을 통합하려면 특히 중요하다고 한다.



#### SecureHeaders GatewayFilter 팩토리

SecureHeaders GatewayFilter 팩토리는 [이 블로그](https://blog.appcanary.com/2017/http-security-headers.html)에서 추천하는 헤더를 응답에 추가한다고 한다.

그 목록은 다음과 같다.

- `X-Xss-Protection:1 (mode=block`)
- `Strict-Transport-Security (max-age=631138519`)
- `X-Frame-Options (DENY)`
- `X-Content-Type-Options (nosniff)`
- `Referrer-Policy (no-referrer)`
- `Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'`
- `X-Download-Options (noopen)`
- `X-Permitted-Cross-Domain-Policies (none)`

// todo 링크 확인해서 공부해보기

그리고 이는 `properties` 에서 `spring.cloud.gateway.filter.secure-headers` 를 통해 수정할 수 있다고 한다.



#### SetPath GatewayFilter 팩토리

SetPath GatewayFilter 팩토리는 path template를 파라미터로 받는다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

위의 얘시처럼 path를 세그먼트화하여 요청 path를 조작할 수 있다.



#### SetRequest/ResponseHeader GatewayFilter 팩토리

SetRequest/ResponseHeader GatewayFilter 팩토리는 name 그리고 value를 파라미터로 받는다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

GatewayFilter는 모두 헤더를 주어진 이름으로 대체한다.

그래서 요청이 `X-Request-Red:1234` 와 같이 오더라도 `X-Request-Red:Blue` 와 같이 전달되게 되고, `X-Request-Red:1234` 와 같은 응답을 받더라도 `X-Request-Red:Blue` 로 응답하게 된다.

또 `SetRequest/ResponseHeader` 는 URI 변수를 path 또는 host에서 인식할 수 있다.



####  SetStatus GatewayFilter 팩토리

 SetStatus GatewayFilter 팩토리는 status를 파라미터로 받는다.

그 값은 스프링 HttpStatus에 유효해야 한다.

그리고 프록시된 요청에서 Origin Http 상태의 값을 반환하고 싶으면 `properties`를 다음과 같이 설정하면 된다.

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```



####  StripPrefix GatewayFilter 팩토리

 StripPrefix GatewayFilter 팩토리는 parts를 파라미터로 가진다.

이는 요청을 전달하기 전에 제거할 path의 개수를 설정한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

만약 `/name/blue/red` 와 같은 path를 가진 요청이 온다면 nameservice는 `nameservice/red` 와 같이 2개를 삭제한 후 남은 값인  red를 다음 요청에 전달한다.



#### Retry GatewayFilter 팩토리

 Retry GatewayFilter 팩토리는 다음과 같은 파라미터를 지원한다.

- `retries`: 재시도해볼 횟수.
- `statuses`: 재시도해야 하는 HTTP 상태 코드들로, `org.springframework.http.HttpStatus`로 표현한다.
- `methods`: 재시도해야 하는 HTTP 메소드들로, `org.springframework.http.HttpMethod`로 표현한다.
- `series`: 재시도할 상태 코드 시리즈들로, `org.springframework.http.HttpStatus.Series`로 표현한다.
- `exceptions`: 던져지면 재시도해야 하는 예외들 리스트.
- `backoff`: 재시도에 설정하는 exponential backoff. 재시도는 `firstBackoff * (factor ^ n)`만큼의 백오프 인터벌을 두고 수행하며, 여기서 `n`은 반복 회차(iteration)를 나타낸다. `maxBackoff`를 설정하면 백오프는 최대 `maxBackoff`까지만 적용된다. `basedOnPreviousValue`가 true일 땐 백오프를 `prevBackoff * factor` 로 계산한다.

위의 파라미터들의 기본 값은 다음과 같다.

- `retries`: Three times
- `series`: 5XX series
- `methods`: GET method
- `exceptions`: `IOException` and `TimeoutException`
- `backoff`: disabled

// todo backoff 계산 다시해보기



#### RequestSize GatewayFilter 팩토리

RequestSize GatewayFilter 팩토리는 요청의 크기가 제한보다 클 경우 요청을 전달하지 않는다.

필터는 maxSize를 파라미터로 받는다.

이때 maxSize는 DataSize 타입이다.

DataUnit suffix를 지정해도 되는데 그 기본갑은 바이트(B)이다.

만약 요청 사이즈가 maxSize 보다 크다면 `413 Payload Too Large ` 가  `errorMessage`와 함께 응답된다.



#### SetRequestHost GatewayFilter 팩토리

SetRequestHost GatewayFilter 팩토리는 덥어쓸 값이 필요하다.

SetRequestHost GatewayFilter는 존재하는 host header을 특정한 값으로 대체하며 이 값을 위해 host 파라미터를 받는다.



#### Modify a Request/Response Body GatewayFilter 팩토리

ModifyRequest/Response Body 필터는 요청이 전달되기전에 요청을 수정할 수 있다.

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri)
        .build();
}
```

// todo 아직 이해할 수 없는 코드



#### Default Filters

모든 라우터에 필터를 추가하고 정의하기 위해서 다음과 같이 `properties`에서 설정하여 적용할 수 있다.

이때 default-fiters는 필터 리스트를 받을 수 있다.

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```



### Global Filters

GlobalFiter 인터페이스는 GatewayFilter 인터페이스와 동일한 시그니처를 가지고 있다.

하지만 이는 모든 라우터에 적용된다는 특징을 가지고 있다.



#### Global Filter 결합 그리고 GatewayFilter 순서

요청이 라우터와 일치할때, filtering web handler는 모든 GlobalFiter 인스턴스와 route에 등록된 모든 GatewayFilter 인스턴스를 필터 체인에 추가한다.

필터 체인에 결합한 다음엔 `org.springframework.core.Ordered` 인터페이스로 정렬하며, 순서는 `getOrder()` 메소드를 구현하면 원하는대로 설정할 수 있다.

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

// todo 사실 위의 코드가 정확히 이해가지는 않는다.



// todo 이후에도 exchange 와 ServerWebExchangeUtils이 계속해서 나오는데 이것이 무엇인지 이해하기



#### Forward Routing Filter

ForwardRoutingFilter는 exchange의 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` 속성에서 URI룰 찾는다.

만약 URL이 `forward`스킴를 가지고 있다면 (ex forward:///localendpoint), 이는 요청ß을 다루기 위해 Spring DispatcherHandler를 사용한다.

요청 URL의 path 부분이 전달될 URL의 path에 덮어써지게 된다.

수정되지 않은 original URL은 `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 속성에 추가된다.



#### LoadBalancerClient Filter 

LoadBalancerClientFilter는 exchange 속성에서 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` URI를 찾는다.

만약 URL이 `lb` 를 가지고 있는다면 그것은 스프링 클라우드 `LoadBalancerClient`를 사용해서 이름을 (이 예시에선 `myservice`) 실제 호스트와 포트로 리졸브하고, 같은 속성에 URI를 대체해 넣는다. 

원래 URL은 `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 속성에 있는 리스트에 추가한다.

이 필터는 `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` 속성값도 `lb` 와 동일한지 확인한다.

속성값이 `lb`라면 같은 과정을 적용한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```



#### ReactiveLoadBalancerClient Filter 

#### The Netty Routing Filter

#### The Netty Write Response Fiter

// todo reactive, netty 공부 후



#### The RouteToRequestUrl Filter

exchange의 `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR` 속성에 `Route`객체가 있으면 `RouteToRequestUrlFilter`가 실행된다.

요청 URI를 기반으로 새 URI를 생성하는데, 이때 URI는 Route 객체의 URI 속성을 반영하여 만든다.

새 URI는 exchange 속성의 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` 에 추가하고 URI에 `lb` 프리픽스가 있다면 이를 제거하고 이후 필터체인에서 사용할 수 있도록 `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` 에 추가한다.



#### The Websocket Routing Filter

`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`의 URL이 `ws`혹은 `wss` 스킴를 가지고 잇다면 websocket routing filter가 동작한다.

이것은 Spring WebSocket infrastructure를 통해 웹소켓 요청을 전달한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
```

// todo 전반적인 이해가 필요할 것 같다.



#### Marking An Exchange As Routed

게이트웨이에선 `ServerWebExchange`를 라우팅한 뒤에는 exchange 속성에 `gatewayAlreadyRouted`를 추가해서 해당 exchange는 “라우팅 되었음”으로 마킹한다. 일단 요청이 라우팅된 걸로 마킹되고 나면, 다른 라우팅 필터들은 요청을 다시 라우팅하지 않고 근본적으로 필터를 건너뛰게 된다. Exchange를 라우팅 완료로 마킹하거나, 이미 라우팅 됐는지를 확인할 수 있는 간편한 메소드들이 준비되어 있다.

- `ServerWebExchangeUtils.isAlreadyRouted`는 `ServerWebExchange` 객체를 받아 “라우팅이 되었는지”를 확인한다.
- `ServerWebExchangeUtils.setAlreadyRouted`는 `ServerWebExchange` 객체를 받아 “라우팅 되었음”으로 마킹한다.



### HttpHeaders Filters

HttpHeadersFilters는 요청이 전달되기 이전에 적용된다.



#### Forwarded Headers Filter

Forwarded Headers Filter는 `Forwareded` header을 만들어 요청을 전달한다.

이것은 기존 `Forwarded` header에 현재 요청의 scheme 그리고 port를 추가한다.



#### RemoveHopByHop Headers Filter

RemoveHopByHop Headers Filter는 이전 요청의 header를 제거한다.

제거하는 header 기본 값은 다음과 같다.

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade

이 목록을 수정하려면 `properties`에서 `spring.cloud.gateway.filter.remove-non-proxy-headers.headers` 를 수정하여 수정할 수 있다.



#### XForwarded Headers Filter

XForwarded Headers Filter는 다양한 `X-Forwarded-*` 를 요청을 전달하기 전에 만든다.

각 header  생성은 다음 boolean properties로 제어할 수 있다.

- `spring.cloud.gateway.x-forwarded.for-enabled`
- `spring.cloud.gateway.x-forwarded.host-enabled`
- `spring.cloud.gateway.x-forwarded.port-enabled`
- `spring.cloud.gateway.x-forwarded.proto-enabled`
- `spring.cloud.gateway.x-forwarded.prefix-enabled`



각 header에 여러 값을 추가하려면 다음 boolean properties를 제어하면 된다.

- `spring.cloud.gateway.x-forwarded.for-append`
- `spring.cloud.gateway.x-forwarded.host-append`
- `spring.cloud.gateway.x-forwarded.port-append`
- `spring.cloud.gateway.x-forwarded.proto-append`
- `spring.cloud.gateway.x-forwarded.prefix-append`



### TLS and SSL

gateway는 Spring server 설정에 따라서 HTTPS 요청을 들을 수 있다.

```yaml
server:
  ssl:
    enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12
```

만약 HTTPS 서버로 요청을 전달한다면 아래와 같이 게이트웨이가 모든 다운스트림 인증서를 신뢰하도록 설정할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
```

하지만 이렇게 모두 허용하는 것은 프로덕션엔 적합하지 않다.

대신 gateway가 신뢰할 수 있는 인증서 셋을 아래와 같이 설정할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          trustedX509Certificates:
          - cert1.pem
          - cert2.pem
```

또 아님 trust store을 사용하는 방법도 있다.



#### TLS Handshake

HTTPS 통신중 clients가 TLS handshake를 한다.

많은 timeout이 이 handshake와 연관이 있다.

이 timeout 설정도 다음과 같이 할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          handshake-timeout-millis: 10000
          close-notify-flush-timeout-millis: 3000
          close-notify-read-timeout-millis: 0
```



### Configuration

Spring Gateway 설정은 `RouteDefinitionLocatior` 인스턴스 컬렉션을 통해 구동된다.

그리고 `PropertiesRouteDefinitionLocator`은 spring boot의 `@ConfigurationProperties` 메커니즘을 통해 프로퍼티를 로드한다.



### Http timeouts configuration

Http timeouts은 모든 라우터에 대해 설정할 수 있고 특정 라우터에 대해 재정의 할 수 있다.



#### Gobal timeouts

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

위와 같이 전체 설정할 수 있다.



#### Per-route timeouts

```yaml
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```

위와 같이 라우트 별로 설정할 수 있다.



### CORS Configuration

CORS 설정은 모든 라우터에 대해 설정할 수 있고 특정 라우터에 대해 재정의 할 수 있다.



#### Global CORS Configuration

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```



#### Route CORS Configuration

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cors_route
        uri: https://example.org
        predicates:
        - Path=/service/**
        metadata:
          cors
            allowedOrigins: '*'
            allowedMethods:
              - GET
              - POST
            allowedHeaders: '*'
            maxAge: 30
```

