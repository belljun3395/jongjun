## [Oauth2.0] Apple 로그인 구현 

[깃허브 바로가기](https://github.com/depromeet12th/three-days-server/tree/dep-405-apple-login-poc)



### Controller

```java
@PostMapping("/apple")
public ApiResponse<ApiResponse.SuccessBody<SaveMemberResponse>> addApple(
    @RequestBody @Valid AppleSignMemberRequest request) {
  SaveMemberUseCaseResponse member = signUseCase.execute(request);
  if (Boolean.TRUE.equals(member.getIsNew())) {
    return ApiResponseGenerator.success(
        MemberConverter.to(member), HttpStatus.CREATED, MessageCode.RESOURCE_CREATED);
  } else {
    return ApiResponseGenerator.success(MemberConverter.to(member), HttpStatus.OK);
  }
}
```

Controller를 구현하면서 들었던 팁은 Endpoint가 API의 경계선이라 생각할 수 있다는 것이다.

특히 현재 프로젝트의 구조에는 UseCase 방식이 사용되고 있기 때문에

Controller를 추가하거나 삭제하는 등 Controller에 관한 행위를 할 때는 API의 경계선이라는 생각을 하면 기준이 조금 선명해질 것이라는 팁을 얻었다.



### UseCase

#### 역할

```java
public SaveMemberUseCaseResponse execute(final AppleSignMemberRequest request) {
  if (request == null) {
    return null;
  }

  MemberInfo info = getInfo(request);

  SaveMemberUseCaseResponse member =
      getUseCase.execute(MemberQueryConverter.from(info.getId(), request));

  if (member == null) {
    return saveUseCase.execute(MemberCommandConverter.from(info, request));
  }

  return member;
}
```

`execute` 메서드를 구현하며 좋은 역할 정의가 좋은 코드를 만든다는 것을 느낀 것 같다.

위의 구현은 Apple 로그인뿐 아니라 기존 로그인 `execute` 메서드에도 동일하게 구현되어 있다.

이는 아래와 같은 명확한 역할 정의가 있었기에 가능하였던 것 같다.

1. 인증서버에서 원하는 정보 받기
2. 새로운 멤버면 가입시키고 로그인
3. 기존멤버면 로그인시키기

그리고 이러한 좋은 역할 정의가 있어야만 이전에 언급한 것처럼 물리적 분리가 되지 않을 수 있다는 생각이 든다.



#### 제어할 수 있는 것과 없는 것

```java
private MemberInfo getInfo(AppleSignMemberRequest request) {
  AppleAuthProperty property = getAppleProperty(request.getCertificationSubject());
  KeyProperties keyProperties = getKeyProperties(property);
  tokenAuthenticator.authenticateIdToken(property, keyProperties, request);
  String token = getToken(property, request.getCode());
  String certificationId = tokenResolver.extractSubByToken(token);
  return MemberInfo.builder().id(certificationId).name(request.getName()).build();
}
```

`getInfo` 메서드에서는 `keyProperties`의 위치가 가장 큰 고민이었다.

`keyProperties`를 고민하다 `getInfo`에 위치시켜 두었지만, 확신은 서지 않은 가운데

**제어할 수 없는 것에 의존하지 않기**라는 주제의 이동욱 님의 발표를 듣고 확신이 섰다. (https://www.youtube.com/watch?v=DJCmvzhFVOI)

위의 강의를 요약하면 아래와 같다.

>  제어할 수 있는 것과 없는 것을 잘 파악하여야 하고 이때 제어할 수 없는 것에 제어할 수 있는 것이 의존한다면 제어할 수 있는 것도 제어할 수 없는 것이 된다

`keyProperties`는 그 값을 내가 제어할 수 없는 것이다.

그렇기에 이를  `tokenAuthenticator.authenticateIdToken`가 이를 의존하게 하는 것보다는 외부에서 주입하는 것이 좋을 것이라는 생각을 하였고

이전에는 확신이 없었지만, 이제는 이유를 가지고 `keyProperties`를 `getInfo`에 둘 수 있게 되었다.



#### 파라미터가 많다면? 객체로!

```java
private String getToken(AppleAuthProperty property, String code) {
   String clientSecret = tokenGenerator.generateClientSecret(property);

   Map<String, String> body =
         RequestBodyGenerator.generateAppleAuthRequestBody(
               AppleAuthRequestWithCodeConverter.from(property.getServiceId(), clientSecret, code));
   try {
      return authClient
            .getAppleTokenInfo(new URI(property.getHost() + property.getUri()), body)
            .getIdToken();
   } catch (URISyntaxException e) {
      throw new ExternalIntegrationException("social.login.error");
   }
}

// RequestBodyGenerator
public static Map<String, String> generateAppleAuthRequestBody(
    AppleAuthRequestWithCodeQuery query) {
  Map<String, String> body = new HashMap<>();
  body.put(CLIENT_ID_KEY, query.getClientId());
  body.put(CLIENT_SECRET_KEY, query.getClientSecret());
  body.put(GRANT_TYPE_KEY, AUTHORIZATION_CODE_VALUE);
  body.put(CODE_KEY, query.getCode());
  return body;
}
```

메서드에 파라미터가 많다면 고민이 될 수 있다.

이때 해결책 중 하나로 파라미터를 묶어 객체를 만드는 것이다.

이렇게 되면 많은 파라미터 대신 객체만 넘겨주면 되기에 조금 더 보기 좋은 코드가 될 수 있을 것 같다.



#### validate와 verify 구분

```java
// TokenValidator
public void validateIdToken(
      AppleAuthProperty property,
      AppleSignMemberRequest request,
      IdTokenProperties idTokenProperties) {

   Date currentTime = new Date(System.currentTimeMillis());
   if (!currentTime.before(idTokenProperties.getExp())) {
      throw new JsonParsingException("token.not.valid");
   }

   if (!request.getNonce().equals(idTokenProperties.getNonce())) {
      throw new JsonParsingException("token.not.valid");
   }

   if (!property.getHost().equals(idTokenProperties.getIss())) {
      throw new JsonParsingException("token.not.valid");
   }

   if (!property.getServiceId().equals(idTokenProperties.getAud())) {
      throw new JsonParsingException("token.not.valid");
   }
}

public void validateIdTokenByKeys(KeyProperties keyProperties, String idToken) {
   if (!tokenResolver.verifyPublicKey(keyProperties, idToken)) {
      throw new JsonParsingException("token.not.valid");
   }
}
```

+ validate : 사용자 요구 검증
+ verify :  함수, 클래스, 설계 검증

코드와 함께 설명하면 validate는 **"사용자 요구 대로 검증했어? if로 알아볼꺼야!"** 같은 느낌이라 할 수 있다.

반면 verify는 **"함수 재대로 구현한건가 한번보자..!"**로 이해하면 편할 것 같다.



### Naming

네이밍은 짝심 프로젝트에 중간에 투입되면서 나도 다음에는 꼭 지켜야지 생각한 부분이다.

우선 내가 생각하는 특징적인 것은 클래스나 메서드 이름은 구체적이고 변수명은 그렇지 않은 것 같다.

```java
// #1
public SaveMemberUseCaseResponse execute(final SignMemberRequest request) {}
// #2
MemberInfo info = getInfo(request.getCertificationSubject(), request.getSocialToken());
```

#1의 경우 SignMemberRequest라는 구체적인 클래스 명을 가지고 있고 request라는 그렇지 않은 변수명을 가지고 있다.

#2도 마찬가지다.

이때 변수명을 구체적이지 않게 할 때는 그 전에 생각해볼 지점이 있다.

"그 변수명이 문맥/상황만으로도 충분히 이해 가능한가?"이다.

```java
private MemberInfo getInfo(AppleSignMemberRequest request) {
  ...
  String certificationId = tokenResolver.extractSubByToken(token);
  return MemberInfo.builder().id(certificationId).name(request.getName()).build();
}
```

위와 같은 상황에서 만약 certificationId과 같이 구체적인 변수명 대신 id라는 변수명을 사용하였다면 어떨까?

`tokenResolver.extractSubByToken(token)`를 통해 토큰에서 sub값을 추출하였다는 것은 알 수 있을 것 같은데

이 값이 멤버를 특정할 수 있는 특징을 가진 값이라는 것을 id만으로 표현하지 못할 것으로 생각한다.

그렇기에 "그 변수명이 문맥/상황만으로도 충분히 이해 가능한가?" 를 변수명을 작명할 때 한번 생각해보면 좋을 것 같다.



그럼 "그 변수명이 문맥/상황만으로도 충분히 이해할 수 있는 상황"을 한 번 살펴보자.

```java
// query
public SaveMemberUseCaseResponse execute(final GetMemberQuery query) {
  MemberEntity memberEntity =
      memberRepository
          .findByCertificationIdAndCertificationSubjectAndStatus(
              query.getCertificationId(), query.getCertificationSubject(), MemberStatus.REGULAR)
          .orElse(null);
  if (memberEntity == null) {
    return null;
  }
  Token token = tokenGenerator.generateToken(memberEntity.getId());
  return MemberConverter.from(memberEntity, false, token);
}

// command
public SaveMemberUseCaseResponse execute(SaveMemberCommand command) {

  MemberEntity memberEntity = memberRepository.save(MemberConverter.to(command));
  Token token = tokenGenerator.generateToken(memberEntity.getId());
  return MemberConverter.from(memberEntity, true, token);
}
```

query, command 사례가 적절할 것 같다.

위에서 query, command는 조회, 등록하는 과정에서 해당 변수명이 그 의미를 잘 나타내고 있다고 생각한다.



### 간단 회고

이렇게 로그인 구현한 것을 정리해보았는데 로그인 기능 하나 구현한 것이지만 그 과정에서 많은 것을 배울 수 있었던 것 같다.

아직 PR을 올리고 완전히 피드백 과정을 마친 것은 아니기에 이후 추가로 기록할 만한 피드백이 있으면 2로 돌아오겠다.

이렇게 Apple 로그인 마무리! ..... 마무리 맞겠지????