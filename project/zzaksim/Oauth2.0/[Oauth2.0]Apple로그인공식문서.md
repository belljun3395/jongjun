## [Oauth2.0] Apple 로그인 공식문서



### Authenticating users with Sign in with Apple

<img width="374" alt="sign-in-with-apple-2_dark@2x" src="https://user-images.githubusercontent.com/102807742/224207923-dcd37fe9-7cd9-4860-8ce9-ecc577607cf4.png">

위 사진은 Apple에서 제공하는 로그인 과정에 대한 시퀀스 다이어그램이다.



#### Authenticate the user and request information

앱 서버와 인증 세션을 초기화하고 **클라이언트 세션을 ID 토큰 그리고 nonce 값을 통해 연결**한다.

이름, 이메일 주소 등 사용자의 정보를 요청할 수 있고 사용자가 이 정보에 대한 액세스를 승인하면 승인 요청에 요청된 정보가 포함된다.



#### Retrieve the user’s information from Apple ID servers

사용자의 신원을 확인에 필요한 자격 증명이 있어야지 사용자를 조회할 수 있다.

서버는 **초기 요청**에 따라 **자격 증명**과 **사용자 정보**을 반환한다.

이때 반환되는 정보에는 **사용자 신원(user identity), 성명(full name), 인증된 이메일 주소(verified email address) 그리고 실 사용자 상태(real user status)**등이 포함될 수 있다.

성공적으로 유저를 인증한 이후에 서버는 **ID 토큰(identity token), 인증 코드(authorization code), 그리고 사용자 식별자(user identifier)**를 앱에 반환한다.

이때 **ID 토큰**은 **JWT로 아래와 같은 클레임**을 가지고 있다.

+ iss
  + Apple이 해당 토큰을 발급하였기에 iss 값은 `https://appleid.apple.com`이다.
+ sub
  + 주체(subject) 등록 클레임의 경우 ID 토큰의 주체를 식별한다.
    이 토큰은 앱용으로 이 값은 **사용자에 대한 고유 식별자(사용자 식별자)**가 된다.
+ aud
  + 대상(audience) 등록 클레임은 ID 토큰 수신자를 식별한다.
    **이 토큰은 앱용으로 값은 개발자 계정의 client_id이다.**
+ iat
+ exp
+ nonce
  + **클라이언트 세션을 ID 토큰과 연결하기 위한 문자열이다.**
    이 값은 리플레이 공격을 완화하며 인증 요청에 전달한 경우에만 존재한다.
+ nonce_supported
+ email
+ email_verified
+ is_private_email
+ real_user_status
+ transfer_sub



**인증 코드는 Apple 서버에서 토큰 클레임을 확인하고 refresh 토큰으로 교환 하는데 사용한다.**



그리고 **사용자 식별자**는 아래와 같은 특징을 지닌다.

+ 교유한 문자열로 **사용자의 기본 식별자 역할**을 한다.
+ Apple 개발자 계정과 연결된 개발팀의 모든 앱에서 **동일한 식별자를 사용한다.**
+ 사용자가 앱에서 Apple 로그인 사용을 중단한 후 다시 사용하여도 **변경되지 않는다.**
+ **일반적으로 데이터베이스에 사용자의 기본 키와 함께 저장된다.**

이메일 주소 대신 사용자 식별자를 사용하여 사용자를 식별할 수 있다.



#### Send information to app servers and verify tokens

앱이 자격 증명 및 사용자 정보를 앱 서버에 전달한다.

API는 사용자가 Apple로 로그인을 사용하여 **처음 로그인할 때 이 정보를 수집하여 앱과 공유한다.**

그리고 이후 사용자가 다른 기기에서 Apple로 로그인하는 경우 API는 사용자의 이름이나 이메일을 다시 묻지 않는다.

하지만 사용자가 Apple 로그인 사용을 중단하고 나중에 앱을 다시 연결하는 경우에는 정보를 다시 수집한다.



**Apple은 모든 후속 API 응답의 ID 토큰에 사용자의 이메일 주소를 제공하지만, 이름과 같은 사용자에 대한 정보는 포함하지 않는다.**

*즉, 사용자 이름을 사용하려면 처음 로그인할 때 사용자 이름에 대한 정보를 저장하여야 한다.*

API 응답에서 사용자 정보를 받으면 즉시 로컬에 저장하여 프로세스 또는 네트워크 장애 발생 시 앱에서 다시 액세스할 수 있도록 하여야 한다.

그리고 **앱 서버는 Apple ID 서버를 통해 토큰 자격 증명의 유효성을 확인하여야 한다.**



##### 정리

위의 공식 문서를 간단히 정리해보면 이후 정리할 refresh, access 토큰을 발급하는데 기반이 되는 **ID 토큰 그리고 인증코드를 발급하는 과정**이라 정리할 수 있다.



### Verifying a user

앱에서 사용자 정보를 수신한 후에는 서버에 연결된 ID 토큰을 확인하여 토큰이 만료되었는지 확인하고 앱에서 토큰이 변조되거나 재생되지 않았는지 확인할 수 있다.

아래 사진은 Apple에서 제공하는 유저 검증 시퀀스 다이어그램이다.

<img width="441" alt="sign-in-with-apple-3_dark@2x" src="https://user-images.githubusercontent.com/102807742/224207960-4067e0df-8508-4a8e-b510-6c26cf6b48ce.png">



#### Verify the identity token

ID 토큰 검증을 위해서는 먼저 **ID 토큰과 인증 코드를 앱 서버로 전송**해야 한다.

이때 ID 토큰을 검증하기 위해 앱 서버는 아래 요소를 확인할 필요가 있다.

+ **Apple ID 서버의 공개키**를 사용하여 **JWS E256 서명을 확인한다.**
+ 인증용 nonce를 확인한다.
+ iss 필드가 `https://appleid.apple.com` 인지 확인한다.
+ aud 필드가 client_id인지 확인한다.
+ 검증 당시 시간이 exp 값 이전인지 확인한다.

---

##### Apple ID 서버의 공개키

**URL**

`GET https://appleid.apple.com/auth/keys`

**Response Code**

+ `Content-Type`: application/json
+ JWKSet
  + `keys / [JWKSet.Keys]` : JSON 웹 키 객체가 포함하는 배열이다.

---

**그리고 검증에 통과한다면 clinet secret을 생성한다.**

---

##### client secret 생성

Apple 로그인을 위해서는 JWT가 각 유효성 검사 요청을 승인해야 한다.

그리고 **client secret을 만들고 난 이후에는 Apple Developer에서 다운로드한 private 키를 통해 서명해야 한다.**

서명된 JWT를 만들기 위해서는 

1. JWT 헤더를 만든다.
2. JWT 페이로드를 만든다.
3. JWT를 서명한다.

JWT를 만들기 위해 헤더에는 다음 필드와 값을 사용한다.

+ `alg` : 토큰에 서명하기 위해 사용한다. **ES256을 사용한다.**
+ `kid` : **개발자 계정과 연결된 Apple private 키로 로그인하기 위해 생성된 10자 키 식별자이다.**



페이로드에는 Apple Rest API로 로그인 및 클라이언트 앱과 관련된 정보가 포함되어 있다.

페이로드는 다음 클레임을 사용한다.

+ `iss` : iss 클레임은 client secret을 발급한 주체를 식별한다. **client secret은 개발자 팀에 속하므로 개발자 계정과 연결된 10자리 팀 ID를 사용한다.**
+ `iat` : iat 클레임은 client secret을 생성한 시간을 Epoch 이후 초 단위로 표시한다.
+ `exp` : exp 클레임은 client secret이 만료되는 시간 또는 그 이후를 식별한다. **이 값은 서버의 현재 UNIX 시간에서 15777000(초 단위로 6개월)보다 크지 않아야 한다.**
+ `aud` : aud 클레임은 client secret의 **수신자를 식별**한다. client secret은 유효성 검사 서버로 전송되므로 `https://appleid.apple.com`를 값으로 사용하면 된다.
+ `sub` : sub 클레임은 client secret의 **주체를 식별**한다. 이는 애플리케이션을 위한 것으로 **client_id와 동일한** 값을 사용하면 된다. 

---



#### Obtain a refresh token

**ID 토큰을 앱서버에서 인증한 후** `client_id`, `client_secret`를 통해 **토큰을 생성하고 검증한다.**

---

##### 토큰 생성 및 검증

**URL**

`POST https://appleid.apple.com/auth/token` 

**HTTP Body**

+ `form-data` : 서버가 인증 코드 또는 refresh 토큰의 유효성을 검사하는 데 필요한 입력 매개변수 목록이다.

+ `Content-Type` : application/x-www-form-urlencoded

+ `client_id / string / required` : 앱의 식별자다. (앱 ID 또는 서비스 ID)
+ `client_secret / string / required` : **개발자가 생성한 JWT 토큰**으로, **개발자 계정과 연결된** Apple private 키를 사용하여 로그인한다. 그리고 **인증 코드 및 refresh 토큰 유효성 검사 요청에는 이 매개변수가 필요하다.**
+ `code / string` :  앱으로 전송된 인증 응답에 포함된 인증코드다. 이 코드는 일회용이며 5분 동안 유효하다. 인증 코드 유효성 검사 요청에는 이 매개 변수가 필요하다.
+ `grant_type / string / required` :  권한 부여 유형에 따라 클라이언트 앱이 유효성 검사 서버와 상호 작용하는 방식이 결정된다. 인증 코드 및 refresh 토큰 유효성 검사 요청에는 이 매개 변수가 필요하다. 인증 코드에는 `authorization_code` refresh 토큰에는 `refresh_token`을 사용한다.
+ `refresh_token / string` : 권한 부여 요청 중에 유효성 검사 서버로부터 받은 refresh_token이다. 토큰 유효성 검사를 요청하려면 refresh_token 매개변수가 필요하다.
+ `redirect_uri / string` : 앱으로 사용자를 인증할 때 인증 요청에 제공된 대상 URI다. URI는 HTTPS 프로토콜을 사용해야 하고 도메인 이름을 포함해야 하며 IP 주소나 로컬 호스트는 포함할 수 없다. **인증 코드 요청에는 이 매개변수가 필요하다.**

**Response Code**

+ `Content-Type` : application/json
+ `TokenResponse`
  + `access_token / string` : 허용된 데이터에 액세스하는데 사용되는 토큰이다.
  + `expires_in / number` : access 토큰 만료시간 이전 남은 시간, 초이다.
  + `id_token / string` : JWT로 유저 정보를 포함하고 있다.
  + `refresh_token / string` : 인증 코드의 유효성을 검사할 때 새로운 access 토큰을 다시 생성하는 데 사용되는 refresh 토큰이다. **기존 refresh 토큰의 유효성을 검사할 때는 반환되지 않는다.**
  + `token_type / string` : access 토큰의 타입이다. 늘 `bearer` 이다.

---

**검증에 성공하면 서버에서 refresh 토큰을 발급하며, 이 토큰을 사용하여 이후 access token을 얻을 수 있다.**

하루에 한 번 refresh 토큰을 확인하여 해당 디바이스에 있는 사용자의 Apple ID가 여전히 Apple 서버에서 정상 상태인지 확인하여야 한다.



---

+ ##### 회원 탈퇴

  **URL**

  `POST https://appleid.apple.com/auth/revoke`

  **HTTP Body**

  + `form-data` : 서버가 토큰을 무효화하는 데 필요한 입력 매개변수 목록이다.
  + `Content-Type`: application/x-www-form-urlencoded
  + `client_id / string / required` : 앱의 식별자다. (앱 ID 또는 서비스 ID)
  + `client_secret / string / required` : 개발자가 생성한 JWT 토큰으로, 개발자 계정과 연결된 Apple private 키로 로그인할때 사용된다. 
  + `token / string / required` : 해지하려는 사용자의 refresh 토큰 혹은 access 토큰이다. 요청이 성공하면 제공된 토큰과 연결된 사용자 세션이 해지된다.
  + `token_type_hint / string` : 해지를 위해 제출한 토큰 유형에 대한 힌트다. `refresh_token` 혹은  `access_token`을 사용한다.

---



##### 정리

ID 토큰 그리고 인증코드을 앱서버에서 받아 이를 통해 **refresh 토큰을 발급하는 방법과 이를 검증하는 방법을 소개하였다.**