## [Oauth 2.0] 공식문서



### Introduction

#### Roles

**resourse owner** 

보호된 리소스에 대한 액세스 권한을 부여할 수 있는 엔티티다.

Ex) 유저

**resource server**

보호된 리소스를 호스팅하는 서버로, access token을 사용하여 보호된 리소스 요청을 수락하고 응답할 수 있는 서버다.

**authorization server**

resource owner을 성공적으로 인증하고 권한을 획득한 후 액세스 토큰을 발급하는 서버이다.

**이때 authorization server는 resource server와 동일한 서버이거나 분리된 엔티티일 수 있다.**

Ex) Apple, Google, Naver, ...

**client**

resource owner을 대신하여 리소스를 요청하는 애플리케이션으로 "client"라는 용어는 특정 구현 특성을 의미하지는 않는다.

Ex) web, user-agent, native application



#### Protocol Flow

```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+

                     Figure 1: Abstract Protocol Flow
```

**(A)** 

client가 resource owner에게 권한을 요청한다.

이때 권한 요청은 **resource owner에게 직접**할 수도 있고 **authorization server를 중계자**로 사용하여 요청할 수도 있다.

**(B)**

client가 **resource owner의 권한을 나타내는 자격증명 권한**을 부여 받는다.

**(C) & (D)**

client는 authorization server에 권한 부여를 제시하며 access token을 요청한다.

권한 부여의 유효성을 검사하고 유효하면 accesstoken을 발급한다.

**(E) & (F)**

client는 resource server에 access token을 제시하며 보호된 리소스를 요청한다.

resource server는 access token의 유효성을 검사하고 유효하면 요청을 처리한다.



#### Access Token

access token은 보호된 리소스에 접근하는데 사용되는 자격증명이다.

token은 resource owner에 의해 부여된 **접근 범위** 그리고 **지속 시간**을 나타내고  resource server 그리고 authorization server에서 이를 사용한다.

token은 인증 정보를 검색하는 데 사용되는 **식별자를 나타낼 수도 있고** 식별자를 나타내거나 **인증 정보를 자체적으로 포함할 수 있다.**



#### Refresh Token

refresh token은 access token을 얻는 데 사용되는 자격 증명이다.

refresh token은 authorization server에 의해 client에 발급되며 **access token이 유효하지 않거나 만료될 때 새 acess token을 얻는 데 사용된다.**

그리고 refresh token은 **authorization server에서만 사용**하게 되어 있으며 **resource server로는 전달하면 안 된다.**



```
  +--------+                                           +---------------+
  |        |--(A)------- Authorization Grant --------->|               |
  |        |                                           |               |
  |        |<-(B)----------- Access Token -------------|               |
  |        |               & Refresh Token             |               |
  |        |                                           |               |
  |        |                            +----------+   |               |
  |        |--(C)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(D)- Protected Resource --| Resource |   | Authorization |
  | Client |                            |  Server  |   |     Server    |
  |        |--(E)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(F)- Invalid Token Error -|          |   |               |
  |        |                            +----------+   |               |
  |        |                                           |               |
  |        |--(G)----------- Refresh Token ----------->|               |
  |        |                                           |               |
  |        |<-(H)----------- Access Token -------------|               |
  +--------+           & Optional Refresh Token        +---------------+

               Figure 2: Refreshing an Expired Access Token
```

위의 `Figure 2`는 authorization grant를 획득한 이후 획득한 access token이 만료되었을 때 이를 갱신하는 과정이다.



#### HTTP Redirection

HTTP Redirection 명세는 HTTP 리다이렉션을 client 혹은 authorization server가 **resource owner의 user-agent**를 **다른 대상으로 리다이렉션** 할 수 있도록 해준다.



### Client Registration

**프로토콜 시작 전에 client는 authorization server에 등록해야 한다.**

이때 client 등록은 주로 HTML 등록 폼으로 이루어지며 client와 authorization server 사이의 직접 상호작용이 필요하지 않다.



#### Client Types

**confidential**

그들의 자격증명 기밀을 유지할 수 있는 client다.

또는 다른 방법을 사용해서 client 인증 보안을 유지할 수 있는 client다.

Ex) web application

**public**

그들의 자격증명 기밀을 유지할 수 없는 client다.

또는 다른 방법을 사용해서 client 인증 보안을 유지할 수 없는 client다.

Ex) user-agent-based application, native application



#### Client Identifier

authorization server는 등록된 client를 대상으로 **client 식별자(identifier)**를 발행한다.

이는 유일한 문자열로 client에 의해 제공된 정보를 나타낸다.

**client 식별자는 비밀(secret)이 아니다.**

이것은 resource owner에게 노출되고 **client 인증에 혼자 사용되어서는 안 된다.(MUST NOT)**



### Protocol Endpoints

인증 프로세스는 두 개의 authorization server 앤드포인트를 사용한다.

+ Authorization endpoint

  + client가 resource owner로부터 권한을 얻기 위해 사용된다.

+ Token endopoint

  + client가 권한을 통해 access token을 얻기 위해 사용된다.

  

#### Authorization Endpoint

Authorization Endpoint는 **resource owner와 상호작용하기 위해 사용**되고 **인증 권한을 부여 받는 데 사용된다.**

Endpoint는 다른 추가적인 쿼리 파라미터가 추가되더라도 유지될  "application/x-www-form-urlencoded"를 포함하고 있다.

그리고 Endpoint는 [fragment component](https://velog.io/@roro/URL-%ED%94%84%EB%9E%98%EA%B7%B8%EB%A8%BC%ED%8A%B8-HASH)를 가지면 안 된다.("#")



Authorization Endpoint로의 요청은 사용자 인증 그리고 텍스트 자격증명(transmmission of clear-text credentials)으로 이어지기 때문에 authorization server는 TLS authorization endpoint 사용이 요구된다.

또 authorization server는 먼저 resource owner의 신원을 확인해야 하며 HTTP GET 요청을 Authorization Endpoint를 위해 지원해야만 한다. (HTTP POST를 지원하여도 괜찮다.)

Authorization Endpoint에 값 없이 전송된 매개변수는 반드시 요청에서 생략된 것처럼 처리해야 한다.

그리고 authorization server는 인식할 수 없는 요청 매개변수를 무시해야 한다.



#### Token Endpoint

Token Endpoint는 client가 그들의 **권한 혹은 refresh token을 제시하여 access toekn을 얻는 데 사용된다.**

이러한 Token Endpoint는 암시적 권한 부여 유형을 제외한 모든 권한 부여에 사용된다.

Endpoint는 다른 추가적인 쿼리 파라미터가 추가되더라도 유지될  "application/x-www-form-urlencoded"를 포함하고 있다.

그리고 Endpoint는 [fragment component](https://velog.io/@roro/URL-%ED%94%84%EB%9E%98%EA%B7%B8%EB%A8%BC%ED%8A%B8-HASH)를 가지면 안 된다.("#")



Token Endpoint로의 요청은 텍스트 자격증명(transmmission of clear-text credentials)으로 이어지기 때문에 authorization server는 TLS authorization endpoint 사용이 요구된다.

그리고 client는 HTTP POST 요청을 통해 access token 생성 요청을 보내야 한다.

Token Endpoint에 값 없이 전송된 매개변수는 반드시 요청에서 생략된 것처럼 처리해야 한다.

그리고 authorization server는 인식할 수 없는 요청 매개변수를 무시해야 한다.



### Obtaining Authorization

#### Authorization Code Grant

**code 유형은 access token과 refresh token 모두를 얻기 위해 사용된다.**

그리고 이는 confidential client에게 최적화되어 있다.

```
     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)

Note: The lines illustrating steps (A), (B), and (C) are broken into two parts as they pass through the user-agent.

                     Figure 3: Authorization Code Flow
```

이는 리다이렉션 기반 Flow이기 때문에 위의 보이는 것처럼 client는 무조건 resource owner의 user-agent와 상호 작용할 수 있어야 한다.

그리고 authorization server로부터 오는 요청을 받을 수 있어야 한다.



**(A)**

client는 resource owenr의 user-agent를 Authorization Endpoint로 보내 Flow를 시작한다.

이때 client는 **client 식별자, 요청된 범위, 로컬 상태 그리고 authorization server가 접근을 승인(혹은 거절)하면 리다이렉션 할 URI**를 포함한다.

**(B)**

authorization server는 **resource owner를 인증**하고 resource owner가 **clinet의 요청을 허용할지 거부할지 결정한다.**

**(C)**

요청을 승인한다면, 이전에 전달받은**(요청 속 혹은 client 등록 중 설정한)** 리다이렉션 URI를 사용해 user-agent로 돌아간다.

이때 **리다이렉트 URI는 authorization code 그리고 전달받은 로컬 상태를 포함한다.**

**(D)**

authorization code를 가지고 client는 authorization server의 Token Endpoint에 access token을 요청한다.

이때 client는 **authorizatio code 증명에 사용될 리다이렉트 URI**를 포함하고 있다.

**(E)**

authorization server는 **client를 인증**하고 **authorization code를 검증**하고 **리다이렉션 URI가 (C)에서 받은 URI와 일치하는지 검증한다.**