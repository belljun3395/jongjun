## zuul 서버 소개

zuul 서버에서 구현한 filter는 다음과 같습니다.

| 필터             | 기능                                                         |
| ---------------- | ------------------------------------------------------------ |
| PreLogginFilter  | 요청이 들어오면 특정 주소를 향한 요청이 아닌 이상 access token을 검증하고 요청을 로깅하는 필터이다. |
| PreRoleFilter    | access token의 role을 검증하는 필터이다.                     |
| PostLogginFilter | 응답을 로깅하는 필터이다.                                    |
| SendErrorFilter  | Exception을 처리하는 필터이다.                               |



### PreLogginFilter

```java
@Override
public boolean shouldFilter() {
  RequestContext context = RequestContext.getCurrentContext();
  HttpServletRequest request = context.getRequest();
  String uri = request.getRequestURI();
  if (uri.matches("/auth/members") ||
      uri.matches("/auth/members/join") ||
      uri.matches("/auth/members/login") ||
      uri.matches("/auth/members/logout") ||
      uri.matches("/auth/members/token/renewal")
     ) {
    return true;
  }

  String accessToken = request.getHeader(AUTHORIZATION_HEADER);
  if (!tokenConsumer.validateToken(accessToken)) {
    request.setAttribute("Authorization", token.getNewToken());;
  }
  return true;
}
```

PreLogginFilter에서는 access token 검증을 한다.

거이 모든 요청에 대해서 access token을 검증하기에 shouldFilter에 이를 구현하였다.

(token을 얻기 위해 로그인을 하는데 로그인 요청에 token을 검증한다면 누구도 로그인을 하지 못할 것이다.)

이때 문제가 되는 것이 token 발행 서버와 지금 zuul 서버가 다르다는 것이다.

이는 key를 Spring Cloud Config를 사용한 config 서버를 통해 관리한다면 편하게 공유할 수 있다.

이에 관해서는 별도의 페이지를 통해 설명할 것이다.

그리고 검증에 통과하지 못한다면 다시 token을 발행 받아야한다.

이는 프턴트에 Exception을 띄우고 다시 token 발행 요청을 할 수 있지만 본인은 위와 같이(`token.getNewToken()`) 서버내에서 처리하였다.

이때 zuul 서버 내에는 tokenConsumer 밖에 존재하지 않는다.

이 이상의 일을 한다면 zuul 서버의 역활을 벗어나는 것이라 생각하여 token 갱신 요청은 아래와 같이 auth 서버에 요청하였다.

```java
@FeignClient(value = "auth", fallbackFactory = FeignRenewalTokenFallbackFactory.class)
public interface FeignRenewalToken {

  @RequestMapping(path = "/members/token/renewal")
  String getNewToken();

}
```

요청을 할때는 Feign이라는 라이브러리를 사용하였다.

이는 Circuit Braker을 지원하며 interface만으로 요청을 구현할 수 있다는 장점이 있다.



### SendErrorFilter

```java
@Override
public Object run()  {

  RequestContext context = RequestContext.getCurrentContext();
  HttpServletRequest request = context.getRequest();
  Throwable throwable = context.getThrowable();

  if (throwable instanceof ZuulException) {
    if (throwable.getCause() instanceof NotAllowedAPIExceptionCustom) {
      log.error("Not Allowed Api - " + request.getRequestURI());
      makeErrorResponse(context, request, new NotAllowedAPIExceptionCustom());
    }

    if (throwable.getCause() instanceof NotValidateTokenExceptionCustom) {
      log.error("Not Validate TokenException - " + request.getRequestURI());
      makeErrorResponse(context, request, new NotValidateTokenExceptionCustom());
    }
  }

  return null;
}
```

zuul은 context에 filter 전체에서 공유하는 것 정보를 가지고 있다.

exception에 관한 정보 역시 context 안에 있고 이를 `context.getThrowable()`를 통해 얻을 수 있다.

그렇게 얻은 exception을 위와 같이 `instanceof` 를 통해 구분하여 처리해 주었다.

