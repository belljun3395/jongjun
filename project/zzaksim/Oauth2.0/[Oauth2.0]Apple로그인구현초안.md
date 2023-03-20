## [Oauth2.0] Apple 로그인 구현 초안

[깃허브 바로가기](https://github.com/depromeet12th/three-days-server/tree/dep-405-apple-login-poc)



### ID 토큰 주요 클레임 값 의미

+ iss
  + Apple이 해당 토큰을 발급하였기에 iss 값은 `https://appleid.apple.com`이다.
+ sub
  + 주체(subject) 등록 클레임의 경우 ID 토큰의 주체를 식별한다.
    이 토큰은 앱용으로 이 값은 **사용자에 대한 고유 식별자(사용자 식별자)**가 된다.
+ aud
  + 대상(audience) 등록 클레임은 ID 토큰 수신자를 식별한다.
    **이 토큰은 앱용으로 값은 개발자 계정의 client_id이다.**
+ nonce
  + **클라이언트가 Apple ID 서버에 ID 토큰을 요청할 때 함께 전달하는 임의의 값이다.**
  + **클라이언트 세션을 ID 토큰과 연결하기 위한 문자열이다.**
    이 값은 리플레이 공격을 완화하며 인증 요청에 **전달한 경우에만 존재한다.**
    + <u>**클라이언트와 협의 필요!**</u>

### Controller

#### RequestBody

```java
public class SignMemberRequest {

	private CertificationSubject certificationSubject;
	private String socialToken;
}
```

기존에는 위와 같이 로그인을 위한 요청을 받고 있었다.

하지만 이를 그대로 사용하기에는 Apple 로그인을 위해 받아야 할 요소는 많았다.

하지만 `certificationSubject` 그리고 `socialToken` 는 Apple 로그인을 위해서도 필요한 정보이기에 아래와 같이 상속받아 구현하였다.

*정확히는 `socialToken`이 아닌 token이 필요했다.*

```java
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
```

위에서 조금 특이한 부분은 `getSocialToken()`의 @Deprecated이다.

Apple 로그인에는 socialToken이 아닌 idToken을 받는다.

하지만 `SignMemberRequest`에는 socialToken 필드가 있다.

그렇기에 `getSocialToken()`을 사용하는 경우를 대비하여야 하는 데 본인은 우선 @Deprecated를 붙여 사용하지 않을 것을 권장하고

리턴값은 null 값일 socialToken이 아닌 `getIdToken()`을 통해 idToken에 해당하는 값을 리턴하였다.



### UseCase

#### AppleAuthProperty

```java
@Getter
@Component
public class AppleAuthProperty extends AuthRequestProperty {

	@Value("${apple.key.uri}")
	private String keyURI;

	@Value("${apple.key_id}")
	private String keyId;

	@Value("${apple.client_id}")
	private String clientId;

	@Value("${apple.team_id}")
	private String teamId;

	public AppleAuthProperty(
			@Value("${apple.host}") String host,
			@Value("${apple.token.uri}") String uri,
			@Value("${apple.unlink.uri}") String unlink) {
		super(host, uri, unlink);
	}

	@Override
	public String getAdminKey() {
		return null;
	}
}
```

기존 AuthRequestProperty에는 host, uri, unlik와 같이 Oauth2.0을 위한 기본적인 프로퍼티는 가지고 있다.

하지만 Apple 로그인을 위해서는 위에서 확인할 수 있듯 이보다 많은 프로퍼티가 필요하고 이를 위해 AuthRequestProperty를 상속하여 AppleAuthProperty를 만들었다.



그렇기에 Apple 로그인에서 원하는 프로퍼티를 모두 사용하기 위해서는 AuthProperty를 반환해주는 기존 코드의 `getMemberProperty()`의 값을 AppleAuthProperty로 캐스팅하여 사용해야 한다.



이렇게 AppleAuthProperty를 만든 이후에는AppleAuthProperty와 RequestBody의 AppleSignMemberRequest의 필드를 활용하여 이후 작업을 수행해주면 된다.



1. AppleSignMemberRequest의 ID 토큰 검증
1. 검증 이후 AppleSignMemberRequest의 Code를 통한 Apple ID 서버에서의 ID 토큰 획득
1. ID 토큰에서 certificationId 획득
1. certificationId와 AppleSignMemberRequest의 `getName()`을 통해 MemberInfo 생성
1. 정보 조회(로그인) or 정보 저장(회원가입)