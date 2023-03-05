## [Oauth2.0] 기존 코드 파악

앞서는 Oauth2.0 공식 문서를 통해 Oauth2.0에 대해서 알아보았다.

공식 문서를 통해서는 Oauth2.0가 어떤 것이고 해당 서비스를 제공하려면 어떻게 할 수 있는지 알 수 있었다.

하지만 내가 알아야 하는 것은 `zzaksim에서 Oauth2.0 어떻게 사용할 것인가?`이다.

그렇기에 이번에는 기존 코드를 파악해보려 한다.



### [Git] Three-days-server 

https://github.com/depromeet12th/three-days-server



### MemberEntity

```java
public class MemberEntity {

	private Long id;

	private String certificationId;

	private String name;

	private CertificationSubject certificationSubject;

	private MemberStatus status = MemberStatus.REGULAR;

	private String resource;

	private Boolean notificationConsent;

	private LocalDateTime createAt;

	private LocalDateTime updateAt;
}
```

MemberEntity는 위와 같이 구성된다.

코드를 처음 보면서 익숙하지 않았던 속성은 `certificationId` 그리고 `CertificationSubject` 였다

우선 위의 속성들에 대해 **어떠한 이유로 저장**하고 있는지 알아보자.



#### certificationId, certificationSubject

우선 [카카오 Rest API](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#req-user-info-response)에서 로그인 후 id라는 칼럼명으로 필수로 반환해주고 있다.

<img width="822" alt="스크린샷 2023-03-05 오후 1 17 54" src="https://user-images.githubusercontent.com/102807742/222941442-d28eb878-2760-42d2-8040-e322edb9cbce.png">

이는 **카카오에 가입된 회원의 고유 id**를 뜻한다.



zzaksim에서는 이 값을 `certificationId`라는 칼럼으로 저장하고 있다.

더불어 `CertificationSubject`를 통해 **어떤 Oauth2.0 제공자를 통해 가입**하였는지 저장한다.

만약 저장된 값이 `36 | 3689 | KAKO` 와 같다고 생각해보자.

<u>이는 우리 서버에서는 id 값이 36이고 이는 KAKO를 통해 가입한 KAKO의 3689라는 id 값을 가지고 있는 회원이라는 뜻이다.</u>



조금 더 보충 설명하자면 certificationSubject는 다양한 Oauth2.0 제공자를 사용하기 위해서 필요하다.

카카오에서 `A` 의 id 값이 애플에서 `Z`의 id 값과 동일하다고 가정해보자.

이때 만약 certificationSubject를 통해 Oauth2.0 제공자를 구분하지 않았으면 우리는 A와 Z를 구분할 방법이 없다.

그렇기에 우리는 certificationSubject라는 칼럼이 필요하다.



그리고 **certificationId 칼럼이 String 타입**인 것에도 의문을 가질 수 있다.

이 역시 카카오에서는 id 값을 `1389`와 같이 숫자로 제공하고 애플에서는 `Zd92`와 같이 랜덤하게 제공한다고 가정해보자.

만약 칼럼의 타입이 String이 아니라면 우리는 카카오와 애플에서 가입한 사용자를 분리하여 관리하여야 할 것이다.

**하지만 칼럼의 타입이 String이라면 카카오 애플의 id 값이 어떻게 제공되더라도 공통으로 관리할 수 있을 것이다.**



zzaksim에서 Member를 어떻게 저장하는지 알아보았으니 이제 본격적으로 **어떻게 로그인 과정을 처리**하는지 확인해보자.



### Controller

```java
/** 로그인 또는 회원가입 */
@PostMapping
public ApiResponse<ApiResponse.SuccessBody<SaveMemberResponse>> add(
      @RequestBody @Valid SignMemberRequest request) {
  ...
}
```

처음 코드를 보며 눈에 띄었던 것은 로그인과 회원가입을 하나로 구현하였다는 것이다.

로그인과 회원가입을 항상 분리하여 구현하였는데 필요에 따라서는 하나로 구현할 수 있다는 것을 알 수 있었다.



### Service

본격적으로 로그인 및 회원가입 과정을 살펴보면 우선 Oauth2.0 제공자에게서 멤버에 대한 정보를 조회해 온다.

```java
MemberInfo info = getInfo(request.getCertificationSubject(), request.getSocialToken());
```

이때 `request.getCertificationSubject()`를 통해 어떠한 Oauth2.0 제공자인지 확인한 이후 그곳에서 제공한 토큰( `request.getSocialToken()`)을 통해 멤버 정보를 조회한다.

*이를 통해 알 수 있었던 것은 Oauth2.0 제공자가 발급하는 토큰을 백단이 아닌 프런트단에서 조회한다는 것이다.*



그리고 조회한 정보를 바탕으로 서비스에 등록된 멤버인지 확인한다.

이때 등록된 멤버라면 memberId를 토큰의 payload에 포함한 토큰을 생성한다.

*이를 통해 토큰만을 통해 서비스에 등록된 멤버의 id를 알 수 있다.*

```java
public Token generateToken(Long memberId) {
  return Token.builder()
      .accessToken(generateAccessToken(memberId))
      .refreshToken(generateRefreshToken(memberId))
      .build();
}
```

서비스에 등록된 멤버가 아니라면 서비스에 등록하고 토큰을 발급한다.

그리고 멤버 정보와 토큰을 함께 프런트로 내려준다.



---

### Wow

zzaksim에서의 로그인 과정은 위를 통해서 잘 설명되었을 것이라 생각한다.

이제는 zzaksim에서 로그인을 처리하는 코드를 보며 **Wow**라고 생각하였던 부분을 소개하려 한다.

*현재는 깃허브가 Public이지만 혹시 몰라 자세한 구현은 생략하고 이는 [코드 링크](https://github.com/depromeet12th/three-days-server/blob/develop/app/src/main/java/com/depromeet/threedays/front/domain/usecase/member/SignMemberUseCaseFacade.java)로 대신한다.*

```java
public class SignMemberUseCaseFacade {

   private final Map<String, AuthRequestProperty> authRequestPropertyMap;
  
  ...

   public AuthRequestProperty getMemberProperty(CertificationSubject subject) {
      Collection<String> authRequestProperties = authRequestPropertyMap.keySet();
      String clientName =
            authRequestProperties.stream()
                  .filter(property -> property.contains(subject.name().toLowerCase()))
                  .findFirst()
                  .orElseThrow(NoSuchFieldError::new);
      return authRequestPropertyMap.get(clientName);
   }
}
```

필요한 부분만 발췌하여 가져왔다.

위의 코드에서 본인이 처음 이해하지 못한 부분은 `private final Map<String, AuthRequestProperty> authRequestPropertyMap;` 부분이다.

`AuthRequestProperty` 만 DI 하는 것은 익숙하였는데 Map의 형태로 DI 하는 것은 처음 보았다.

스프링 전문가5를 공부하면서 [컬랙션을 DI](https://github.com/belljun3395/jongjun/blob/main/study/spring/%EC%8A%A4%ED%94%84%EB%A7%81IoC%EC%99%80DI.md#%EC%BB%AC%EB%A0%89%EC%85%98-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B8%B0) 하는 것은 공부한바 있다.

하지만 "아 그렇구나.." 정도로 넘겼는데 위에서는 Map이지만 이를 활용하는 코드를 보고 **Wow**라는 생각밖에 들지 않았다.



위에 Map에 어떻게 authRequestPropertyMap이 자동으로 DI 되는지 조금 더 자세히 알아보자.



**AuthRequestProperty**

```java
public abstract class AuthRequestProperty {

	protected String host;
	protected String uri;
	protected String unlink;

	public abstract String getAdminKey();
}

```

AuthRequestProperty는 추상 클래스다.

이렇게 추상 클래스로 선언한 이유는 Oauth2.0는 **프로토콜**로 Oauth2.0 제공자들은 **공통된 형식으로 서비스를 제공**하기 때문이다.

그리고 이렇게 추상 클래스를 선언하여 이를 상속하여 KAKAO, APPLE과 같은 Oauth2.0 제공자들 구현한다면 authRequestPropertyMap처럼 AuthRequestProperty로 이들을 묶을 수 있다.



```java
@Getter
@Component
public class KakaoAuthRequestProperty extends AuthRequestProperty {

	@Value("${kakao.adminKey}")
	String adminKey;
  
  	public KakaoAuthRequestProperty(
			@Value("${kakao.host}") String host,
			@Value("${kakao.user.uri}") String uri,
			@Value("${kakao.unlink.uri}") String unlink) {
		super(host, uri, unlink);
	}
}
```

이는 KAKAO를 구현한 것인데 AuthRequestProperty를 상속하고 @Component로 선언된 것을 확인할 수 있다.

@Component로 선언됨으로써 이는 kakaoauthrequestproperty라는 이름으로 빈으로 등록된다.



이렇게 등록된 빈은 `private final Map<String, AuthRequestProperty> authRequestPropertyMap;` 에 

String에는 빈 이름 AuthRequestProperty에는 상속하여 구현한 객체가 자동으로 DI되고 

```java
Collection<String> authRequestProperties = authRequestPropertyMap.keySet();
String clientName =
      authRequestProperties.stream()
            .filter(property -> property.contains(subject.name().toLowerCase()))
            .findFirst()
            .orElseThrow(NoSuchFieldError::new);
```

위와 같이 활용할 수 있게 된다.
