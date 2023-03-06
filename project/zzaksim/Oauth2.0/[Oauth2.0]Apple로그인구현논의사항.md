## [Oauth2.0] Apple 로그인 구현 논의사항

구현에 앞서 살펴본 공식 문서와 기존 코드 파악을 통해 구현 전 논의사항을 정리해보려 한다.



우선 기존 코드는 로그인 시 어떤 토큰을 받고 있는지 우선 알아보자.

```java
public class SignMemberRequest {

	private CertificationSubject certificationSubject;
	private String socialToken;
}
```

우선 위의 SignMemberRequest를 통해 **인증 제공자(CertificationSubject)** 그리고 **토큰(socialToken)**에 대한 정보를 받고 있다.



그리고 해당 정보를 바탕으로 아래와 같이 멤버의 정보를 조회해 온다.

````java
MemberInfo info = getInfo(request.getCertificationSubject(), request.getSocialToken());
````



### 1-1. 어떤 토큰을 받아야 할까?

애플 로그인에서는 세 가지 토큰을 제공한다.

우선 Apple ID 서버를 통해 로그인하였을 때 받을 수 있는 **ID 토큰**이 있다.

그리고 이때 받은 ID 토큰과 인증 코드를 통해 받은 **refresh token, access token**이 있다.



문제는 세 가지 토큰 모두 기존의 토큰들처럼 인증 제공자에게 토큰을 통해 질의하였을 때 **멤버의 정보를 조회하지 못한다는 것이다.**

<img width="295" alt="스크린샷 2023-03-06 오후 11 27 48" src="https://user-images.githubusercontent.com/102807742/223138387-84671180-feae-423a-8c64-b838cd589542.png">

[Apple 공식 문서](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api)에서 확인할 수 있는 엔드포인트 목록이다.

+ Apple public key
+ token 생성 및 검증
+ token 취소

위에서 확인할 수 있듯 멤버의 정보를 조회하지 못한다.



그렇다면 토큰을 통해서 안에 있는 정보를 통해서 멤버 정보를 획득해야 하는데 멤버 정보를 제공하는 토큰은 ID 토큰이다.

하지만 ID 토큰을 굳이 가지고 받을 필요는 없을 것 같다.

왜냐하면 **ID 토큰의 경우 refresh token을 생성하거나 검증하는 과정에서 다시 한번 받을 수 있기 때문이다.**

그렇기에 현재 파악한 바로는 **refresh token을 프런트로부터 로그인을 위해 받는 것이 좋을 것 같다는 생각이다.**



### 1-2. client secret

Apple 로그인에서는 refresh token을 검증 및 생성하기 위해서 Apple ID 서버에서 제공하는 공개키를 통해 ID 토큰의 일부 요소를 검증하고 client secret과 함께 Apple ID 서버에 token 검증 및 생성을 요청해야 한다.

**client secret을 생성하기 위해서는 Apple Developer에서 다운로드한 private 키를 통해 서명해야 하는데 이 파일을 백단에서 관리해야 할 것 같다는 생각이 든다.**



그렇다면 client secret을 만들기 위해 프런트에서 백에 요청을 보낼 컨트롤러를 하나 구현할 필요가 생기게 된다.

만약 컨트롤러를 구현한다면 **client secret을 반환하는 것이 좋을까?**

아니면 이때 **차라리 ID 토큰까지 받아 token까지 발급하여 반환하는 것이 좋을까?**



### 2. MemberInfo 조회 및 회원가입 문제

Apple 로그인에서 멤버의 정보가 제공되는 포인트는 크게 두 곳이 있다.

우선 유저가 **우리 앱에 처음 로그인을 시도할 때**가 있고 이때 멤버에 대한 정보가 가장 많이 제공된다.

[Handle the Authorization Response](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js/configuring_your_webpage_for_sign_in_with_apple#3331292)

<img width="934" alt="스크린샷 2023-03-06 오후 11 35 23" src="https://user-images.githubusercontent.com/102807742/223140686-5080c280-79c4-4102-896f-afa849a10a35.png">

그리고 아래 IMPORTANT와 함께 아래 문구를 제공한다.

```
Apple은 사용자가 앱을 처음 인증할 때만 사용자 개체를 반환합니다. 
앱에서 서버로 이 정보의 유효성을 검사하고 유지합니다. 
후속 권한 요청에는 사용자 개체가 포함되지 않지만 모든 요청에 대한 ID 토큰에 사용자 이메일이 제공됩니다. 
자세한 내용은 Apple로 로그인하여 사용자 인증하기를 참조하세요.
```

문구에 따르면 **처음 인증할 때만 사용자에 대한 정보를 반환**하고 **이후에는 ID 토큰을 통해 사용자 정보를 제공**한다고 한다.



자연스레 두 번째 포인트인 ID 토큰을 알아보면 ID 토큰 클레임의 sub에는 사용자에 대한 고유 식별자 정보가 포함되어 있고 email에 사용자의 이메일 정보가 포함되어 있다.



기존 코드를 살펴보면 기존 코드가 기대하는 값은 아래와 같다.

```java
public class MemberInfo {
	private String id;
	private String name;
	private Properties properties;
}
```

하지만 Apple은 위의 두 가지 포인트에 id 그리고 제한적으로 name만을 제공한다.



**이렇게 되면 문제가 되는 것이 우선 기존 코드처럼 인증 제공자에서 제공하는 이름을 사용하지 못한다.**

새로운 기준을 통해 이름을 만들어주어야 하는데 **이 기준을 어떻게 하면 좋을까?**

*만약에 사용자 이름을 받아야 한다면 기존 로그인 및 회원가입으로 합쳐놓은 기존 코드를 로그인 그리고 회원가입으로 분리하는 방법도 있을 것 같은데 이는 백뿐만 아니라 프런트에도 코드 수정이 일어나야 하는 부분이기에 논의가 필요할 것 같다.*
