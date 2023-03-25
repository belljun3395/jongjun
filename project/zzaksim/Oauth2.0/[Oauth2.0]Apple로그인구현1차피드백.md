## [Oauth2.0] Apple 로그인 구현 1차 피드백



### 초안과 변경 사항

우선 이전 초안과 달라진 부분 부터 먼저 살펴 보면 아래와 같다.

```java
// 초안
public class AppleSignMemberRequest extends SignMemberRequest {

	private String idToken;
	private String code;
	private String nonce;
	private AppleUserInfo user;

	public String getName() {
		return this.getUser().getName();
	}

	@Deprecated
	@Override
	public String getSocialToken() {
		return getIdToken();
	}
}

// 1차 피드백 후
public class AppleSignMemberRequest extends SignMemberRequest {

	private String code;
	private String nonce;
	private AppleUserInfo user;

	public String getName() {
		return this.getUser().getName();
	}

	/** socialToken과 idToken은 동일하다. */
	public String getIdToken() {
		return super.getSocialToken();
	}
}
```

1차 피드백 후 변경된 것은 idToken 필드가 사라진 것이다.

처음 idToken 필드를 추가한 것은 최대한 Apple 로그인 API를 사용하였을 때 받는 값을 사용하고 싶었기 때문이다.

하지만 기존 SignMemberRequest에 socialToken이라는 필드가 있었고 이를 활용하는 것이 더 좋겠다는 피드백을 받았다.

이는 클라이언트에서도 큰 공수가 드는 작업이 아니기에 충분히 받아들여 줄 것 같은데.. **이는 추후 회의에 다시 한번 협의를 부탁해야겠다.**



추가된 것도 메서드도 있는데 `getIdToken()`이다.

없어도 될 것 같은 메서드이지만 `getIdToken()`을 추가해 준 것은

Apple 로그인에서는 ID 토큰을 사용하기에 기존의 `getSocialToken()`만 있다면 다음에 코드를 보는 사람이 social 토큰에 대해서 파악할 필요가 생기기 때문이다.



### 역할 분리란 이런거군요...

이번 피드백을 받으며 공부하면서 애매하다 느꼈던 **역할**에대해 정리할 수 있었던 기회였다.



#### 구현 초안 코드

```java
public SaveMemberUseCaseResponse execute(final AppleSignMemberRequest request) {
  if (request == null) {
    return null;
  }

  AppleAuthProperty property = getAppleProperty(request.getCertificationSubject());
  if (property == null) {
    return null;
  }

  validateToken(property, request);

  AppleToken newToken = getToken(property, request.getCode());

  String certificationId = tokenResolver.extractSubByToken(newToken.getIdToken());

  MemberInfo info = MemberInfo.of(certificationId, request.getName());

  SaveMemberUseCaseResponse member =
    getUseCase.execute(MemberQueryConverter.from(info.getId(), request));

  if (member == null) {
    return saveUseCase.execute(MemberCommandConverter.from(info, request));
  }

  return member;
}
```

위 코드는 Apple 로그인을 실행하는 코드이다.

위의 코드를 보고 처음 주신 피드백은 **"`execute()`의 역할에 대해서 다시 한번 생각해 보실래요?"** 였다.



사실 위의 코드를 처음 작성하였을 때는 나름 역할을 분리하였다고 생각하였다.

`validateToken` 을 통해 토큰 검증하는 private 메서드를 분리하였고

`getToken` 을 통해서 토큰 받아 오는 private 메서드도 분리하였다.

*"무엇을 더 분리해야 하는 거지?"* 라는 생각이 처음에는 들었던 것 같다.



피드백을 받고 난 이후든 생각은 내가 한 분리는 역할의 분리가 아닌  **"물리적 분리"**였던거 같다.

내가 한 private 메서드로 분리는 역할 분리라기보다는 **코드를 깔끔하게 보기 위한 물리적 분리였다.**



#### 역할 분리는 어떻게 해야 할까?

우선 **함수의 역할을 잘 정의**해야 한다.

피드백을 주시면서 제시해주신 `execute()` 의 역할은 아래와 같다.

1. 인증서버에서 원하는 정보 받기
2. 새로운 멤버면 가입시키고 로그인
3. 기존멤버면 로그인시키기

함수의 역할을 위와 같이 정의하였다면 **최대한 정의한 역할만 할 수 있도록 하여야 한다.**



기존 코드를 다시 한번 확인해 보자.

```java
public SaveMemberUseCaseResponse execute(final SignMemberRequest request) {
  if (request == null) {
    return null;
  }

  MemberInfo info = getInfo(request.getCertificationSubject(), request.getSocialToken());

  SaveMemberUseCaseResponse member =
      getUseCase.execute(MemberQueryConverter.from(info.getId(), request));

  if (member == null) {
    return saveUseCase.execute(MemberCommandConverter.from(info, request));
  }

  return member;
}
```

`getInfo()`를 통해 <u>인증 서버에서 원하는 정보를 받았고

`getUseCase.execute()`를 통해 멤버 정보를 조회한 후

<u>가입정보가 있으면 로그인시키고</u> (`return member`)

<u>아니라면 회원가입 후 로그인시켰다.</u> (`saveUseCase.execute()`)

정말 위에서 정의한 역할만 하였다.



내가 작성한 코드는 무엇이 달랐을까?

이를 조금 더 쉽게 파악하기 위해 코드를 말로 풀어보자.

```
// 내가 작성한 코드
로그인 메서드 {
	로그인 메서드야 클라이언트에서 준 토큰 검증해야지
	로그인 메서드 인증 서버에서 토큰 받아와야지
	tokenResolver야 토큰에서 certificationId 알려줘
	
	멤버 정보 있나...?
	있으면 로그인! 아님 회원가입하고 로그인!
}
```

```
// 기존 코드
로그인 메서드 {
	로그인 메서드 인증 서버에서 정보 받아와야지
	
	멤버 정보 있나...?
	있으면 로그인! 아님 회원가입하고 로그인!
}
```

내가 작성한 코드는 **로그인 메서드의 역할 말고도 너무 많은 일**을 하고 있다.



#### 피드백 후 수정한 코드

```java
	public SaveMemberUseCaseResponse execute(final AppleSignMemberRequest request) {
		if (request == null) {
			return null;
		}

		AppleAuthProperty property = getAppleProperty(request.getCertificationSubject());
		if (property == null) {
			return null;
		}

		MemberInfo info = getInfo(property, request);

		SaveMemberUseCaseResponse member =
			getUseCase.execute(MemberQueryConverter.from(info.getId(), request));

		if (member == null) {
			return saveUseCase.execute(MemberCommandConverter.from(info, request));
		}

		return member;
	}
```

기존 코드와 동일해졌다!



##### 달라진 점

**`getInfo()` 의 내부 역할이 달라졌다.**

```java
public MemberInfo getInfo(CertificationSubject subject, String oAuthToken) {
  try {
    AuthRequestProperty property = getMemberProperty(subject);
    final String bearerToken = "Bearer " + oAuthToken;
    return authClient.getInfo(new URI(property.getHost() + property.getUri()), bearerToken);
  } catch (URISyntaxException e) {
    throw new ExternalIntegrationException("social.login.error");
  }
}
```

기존에는 로그인 요청에서 <u>어떤 소셜 로그인인지 파악</u>하고 (`getMemberProperty()`)

<u>멤버 정보를 받기</u> 위해 토큰과 함께 요청을 보냈다면 (`authClient.getInfo()`)



```java
private MemberInfo getInfo(AppleAuthProperty property, AppleSignMemberRequest request) {
  tokenValidator.validateIdToken(property, request);
  AppleTokenInfo tokenInfo = getToken(property, request.getCode());
  String certificationId = tokenResolver.extractSubByToken(tokenInfo.getIdToken());
  return MemberInfo.of(certificationId, request.getName());
}
```

지금은 우선 <u>tokenValidator에게  토큰을 검증을 부탁</u>하고 (`tokenValidator.validateIdToken()`)

Apple 로그인 서버에서 <u>토큰을 받아</u>오고 (`getToken()`)

<u>tokenResolver에게 certificationId를 추출을 부탁</u>한 다음 (`tokenResolver.extractSubByToken()`)

요청에 맞는 멤버 정보를 반환한다.



**그럼  `getInfo()`의 내부 역할은 변경되어도 괜찮을까?**

그렇다!!

**왜냐하면 `execute()` 는  `getInfo()`의 내부 역할이 변경되든 아니든 관심이 없다.**

`execute()`의 역할은 멤버의 정보를 반환할 수 있는 `getInfo()`를 알고 있고 `getInfo()`에게 멤버 정보를 요청하는 것이다.

`getInfo()`가 멤버 정보를 반환하기 위해 무엇을 하는지까지 `execute()`는 관심이 없는 것이다.



#### 정리

이번 피드백을 정리해보면 아래와 같을 것 같다.

1. **함수의 역할을 잘 정의하지 않는다면 private 메서드로의 분리는 단순히 물리적 분리가 될 수 있다.**
2. **모든 메서드는 역할을 가지고 있으며** 우리는 이 **역할을 수행하기 위한 내부 역할을 잘 정의해야 한다.** 
   *( 내부 역할은 메서드의 역할을 수행하기 위한 역할이라 생각하면 될 것 같다. 그렇기에 내부 역할과 역할은 같을 수도 있다 )*
3. 역할은 메서드 **내부에서 새로운 역할로 분리**될 수도 ( ex : private 메서드 ) 
   **외부로 분리**될 수도 있다 ( ex : `다른클래스.메서드` )
