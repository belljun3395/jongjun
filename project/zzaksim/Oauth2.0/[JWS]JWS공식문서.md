## [JWS] JWS 공식문서

### Abstract

JWS(JSON Web Signature)는 **JSON 기반 데이터 구조를 사용하여 디지털 서명 또는 MAC(Message Authentication Code)로 보호되는 콘텐츠를 나타낸다.** 



### Introduction

JSON Web Signature(JWS)는 JSON 기반 데이터 구조를 사용하여 디지털 서명 또는 메시지 인증 코드(MAC)로 보호되는 콘텐츠를 나타낸다.

JWS 암호화 메커니즘은 임의의 옥텟 시퀀스에 대한 무결성 보호를 제공한다.

JWS에 대해 밀접하게 관련된 두 가지 직렬화가 정의된다.  

**JWS Compact Serialization**은 HTTP Authorization 헤더 및 URI 쿼리 매개 변수와 같은 공간 제약 환경을 위해 고안된 짧은 URL-safe한 형식이다.  

**JWS JSON Serialization**는 JWS를 JSON 개체로 나타내며 여러 서명 및/또는 MAC를 동일한 콘텐츠에 적용할 수 있다.  

둘 다 암호화 기반을 공유한다.



### Terminology

#### JSON Web Signature (JWS)

 디지털 Signature 또는 MAC 처리된 메시지를 나타내는 데이터 구조다.

#### JOSE Header

사용된 암호화 작업 및 매개 변수를 설명하는 매개 변수를 포함하는 JSON 개체다.  

JOSE(JSON 오브젝트 서명 및 암호화) 헤더는 일련의 Header 매개변수로 구성된다.

#### JWS Payload

메세지다.

페이로드는 임의의 옥텟 시퀀스를 포함할 수 있습니다.

#### JWS Signature

JWS Proteced Header 및 JWS Payload를 커버하는 디지털 Signature 또는 MAC

#### JWS Protected Header

JWS Signature 디지털 Signature 또는 MAC 작업으로 보호되는 무결성인 헤더 매개 변수를 포함하는 JSON 개체다. 

JWS Compact Serialization의 경우 **전체 JOSE Header로 구성된다.**  

JWS JSON Serialization의 경우 **JOSE Header의 구성 요소 중 하나다.**

*즉, JWS Compact Serialization의 Header는 보호되어야하고 JWS JSON Serialization는 그렇지 않아도 괜찮다.*

#### Base64url Encoding

URL 및 파일 이름 안전 문자 집합을 사용하는 Base64 인코딩으로, 모든 후행 '=' 문자가 생략되고(섹션 3.2에서 허용됨) 줄 바꿈, 공백 또는 기타 추가 문자가 포함되지 않는다.

빈 옥텟 시퀀스의 base64url 인코딩은 빈 문자열이다. 

#### JWS Signing Input

디지털 Signature 또는 MAC 계산에 대한 입력이다.  

이 값은 `JWS Protected Header || '.' || JWS Payload` 이다.

#### JWS Compact Serialization

JWS를 **URL-safe한 압축 문자열로 표현한 것이다.**

#### JWS JSON Serialization

JWS를 **JSON 개체로 표현한다.**  

JWS Compact Serialization과 달리 JWS JSON Serialization은 여러 개의 디지털 Signature 및/또는 MAC를 동일한 컨텐츠에 적용할 수 있다. 

이 표현은 **압축성이나 URL 안전성에 최적화되지 않았다.**

#### Unsecured JWS

무결성 보호 기능을 제공하지 않는 JWS다.

보안되지 않은 JWS는 "alg" 값이 "none"이다.

#### StringOrURI

임의 문자열 값을 사용할 수 있지만 ":" 문자를 포함하는 모든 값은 URI여야 한다는 추가 요구 사항이 있는 JSON 문자열 값이다.

문자열 또는URI 값은 변환이나 표준화가 적용되지 않은 대소문자를 구분하는 문자열로 비교된다. 



### JSON Web Signature (JWS) Overview

JWS는 JSON 데이터 구조와 base64url 인코딩을 사용하여 디지털 Signature 또는 MACed 콘텐츠를 나타낸다.  

이러한 JSON 데이터 구조는 JSON 값 또는 구조 문자 앞 또는 뒤에 공백 및/또는 줄 바꿈을 포함할 수 있다.  

JWS는 다음과 같은 논리적 값을 나타낸다 :

+ JOSE Header
+  JWS Payload
+  JWS Signature

JWS의 경우 **JOSE 헤더 멤버**는 다음 값의 멤버들의 결합이다 :

+ JWS **Protected** Header
+  JWS **Unprotected** Header

이 문서에서는 JWS에 대한 두 가지 직렬화를 정의한다. 

즉, JWS Compact Serialization(JWS 콤팩트 직렬화)과 JWS JSON Serialization(JWS JSON 직렬화)이다.  

JSON에는 임의의 옥텟 시퀀스를 직접 나타내는 방법이 없기 때문에 **두 직렬화 모두에서 JWS Protected Header, JWS Payload 및 JWS Signature는 base64url로 인코딩된다.**



### JWS Compact Serialization Overview

JWS Compact Serialization에서는 **JWS Unprotected Header가 사용되지 않는다.** 

이 경우 JOSE 헤더와 JWS Protected는 동일하다.

JWS Compact Serialization에서 JWS는 연결로 표시된다 :

```
JWS **Protected Header**  || '.' ||
JWS Payload || '.' ||
JWS Signature
```



### JWS JSON Serialization Overview

JWS JSON Serialization에서는 JWS Protected Header와 JWS Unprotected Header 중 하나 또는 둘 다가 있어야 한다. 

경우에 따라 JOSE Header의 멤버는 JWS Protected Header의 멤버와 JWS Unprotected Header 값의 조합이다.

JWS JSON Serialization에서 JWS는 다음 네 멤버 중 일부 또는 전부를 포함하는 개체로 표시된다: 

+ JWS Header 값을 사용하여 보호됨
+  "header" JWS Unprotected Header 값 포함
+ JWS Payload 값의 "payload"
+ JWS Signature 값의 "signature"

3개의 base64url-encoded 결과 문자열과 JWS Unprotected Header 값은 JSON 개체 내의 멤버로 표시된다.  

이러한 값 중 일부를 포함하는 것은 선택 사항이다.

또한 **JWS JSON Serialization는 하나의 Signature 및/또는 MAC 값이 아닌 여러 개의 Signature 및/또는 MAC 값을 나타낼 수 있다.** 



### JOSE Header

JWS의 경우 JOSE Header를 나타내는 JSON 개체의 멤버는 JWS Protected Header 및 JWS Payload에 적용된 디지털 Signature 또는 MAC을 나타내고 선택적으로 JWS의 추가 속성을 설명한다.  

JOSE Header 내의 Header 매개 변수 이름은 고유해야 한다. 

JWS Parser는 중복된 헤더 매개 변수 이름이 있는 JWS를 거부하거나 사전적으로 마지막 중복 멤버 이름만 반환하는 JSON Parser를 사용해야 한다.



Header 매개 변수 이름에는 세 가지 클래스가 있다 : 

+ Registered Header Parameter names 
+ Public Header Parameter names
+ Private Header Parameter names



### Producing and Consuming JWSs

#### Message Signature or MAC Computation

1. JWS Payload로 사용할 콘텐츠를 만든다.
2. 인코딩된 페이로드 값 JWS 페이로드를 계산한다.
3. JOSE Header(JWS Protected Header 및/또는 JWS Unprotected Header)를 구성하는 원하는 Header 매개변수 집합이 포함된 JSON 개체를 만든다.
4. 인코딩된 헤더 값 JWS Protected Header을 계산한다. 
   JWS Proteced Header가 없는 경우이 값을 빈 문자열로 지정한다.
5. JWS Signature 입력 JWS Protected Header || '.' || JWS Payload을 통해 사용되는 특정 알고리즘에 대해 정의된 방식으로 JWS Signature을 계산한다.
   JOSE Header에 "알고리즘"(알고리즘) 헤더 매개변수가 있어야 하며, JWS Signature를 구성하는 데 사용되는 알고리즘을 정확하게 나타내는 알고리즘 값이 있어야 한다.
6. 인코딩된 Signature 값 JWS Signature을 계산한다.
7. JWS JSON Serialization를 사용 중인 경우, 수행 중인 각 디지털 서명 또는 MAC 작업에 대해 이 프로세스(3-6단계)를 반복한다.
8. 원하는 직렬화된 출력을 한다.
   이 결과의 JWS Compact Serialization은 `JWS Protected Header || '.' || JWS Payload || '. || JWS Signature` 다.



#### Message Signature or MAC Validation

**JWS Signature 값이 여러 개 있는 경우 JWS를 수락하기 위해 JWS Signature 값 중 어떠한 값을 유효성을 검사해야 하는지는 응용 프로그램 결정한다.**

경우에 따라 모든 것이 성공적으로 유효성을 검사해야 하며 그렇지 않으면 JWS가 유효하지 않은 것으로 간주된다.  

다른 경우에는 특정 JWS Signature 값만 성공적으로 검증하면 된다. 

그러나 모든 경우에 하나 이상의 JWS 서명 값이 성공적으로 유효해야 한다. 

그렇지 않으면 JWS가 유효하지 않은 것으로 간주해야 한다.

1. JWS 표현을 구문 분석하여 JWS 구성 요소의 직렬화된 값을 추출한다.  
   JWS Compact Serialization을 사용하는 경우 이러한 구성 요소는 JWS Protected Header, JWS Payload 및 JWS Signature의 기본 64url 인코딩 표현이며, 
   JWS JSON Serialization을 사용하는 경우 이러한 구성 요소에는 인코딩되지 않은 JWS Unprotected Header 값도 포함된다.  
   JWS Compact Serialization을 사용하는 경우 JWS Protected Header, JWS Payload 및 JWS Signature는 base64url-encoded 값으로 표시되며 각 값은 다음 문자와 구분되어 하나의 마침표('.') 문자로 사용된다.  
2. Base64url - 줄 바꿈, 공백 또는 기타 추가 문자를 사용하지 않았다는 제한에 따라 JWS Protected Header의 인코딩된 표현을 디코딩한다.
3. 결과 옥텟 시퀀스가 완전히 유효한 JSON 개체의 UTF-8 인코딩 표현인지 확인한다. 
   JWS Protected Header를 이 JSON 개체로 한다.
4. JWS Compact Serialization을 사용하는 경우 JOSE Header를 JWS Protected Header로 한다.  
   그렇지 않으면 JWS JSON Serialization을 사용할 때 JOS 헤더가 해당 JWS Protected Header 및 JWS Unprotected Header의 멤버로 결합되도록 한다.
   JOSE Header는 모두 완전히 유효한 JSON 개체여야 한다. 
   이 단계에서 결과 JOSE Header에 중복된 Header 매개 변수 이름이 포함되어 있지 않은지 확인한다.  
   JWS JSON Serialization를 사용할 때 이 제한 사항에는 JOSE Header를 구성하는 고유한 JSON 개체 값에도 동일한 헤더 매개 변수 이름이 발생하면 안 된다는 내용이 포함된다.
5. 사용 중인 알고리즘 또는 "crit" Header Parameter 값에서 필요한 모든 필드를 구현이 이해하고 처리할 수 있는지, 그리고 해당 매개 변수의 값도 이해하고 지원하는지 확인한다.
6. Base64url - 줄 바꿈, 공백 또는 기타 추가 문자가 사용되지 않았다는 제한에 따라 JWS Payload의 인코딩된 표현을 디코딩한다.
7. Base64url - 줄 바꿈, 공백 또는 기타 추가 문자가 사용되지 않았다는 제한에 따라 JWS Signature의 인코딩된 표현을 디코딩한다.
8. 사용 중인 알고리즘에 대해 정의된 방식으로 JWS 서명 입력 `JWS Protected Header || '.' || JWS Payload || '. || JWS Signature`에 대해 JWS Signature을 검증한다. 
   이는 반드시 "알고리즘"(알고리즘) Header 매개 변수의 값으로 정확하게 표현되어야 한다. 
   유효성 검사 성공 여부를 기록한다.
9. JWS JSON Serialization를 사용하는 경우 표시에 포함된 각 디지털 Signature 또는 MAC 값에 대해 이 프로세스(4-8단계)를 반복한다.
10. 9단계의 유효성 검사에 성공하지 못한 경우 JWS는 유효하지 않은 것으로 간주되어야 한다.  
    그렇지 않은 경우, JWS JSON Serialization 사례에서 성공한 검증과 실패한 검증 중 어느 것이 성공했는지를 나타내는 결과를 응용 프로그램에 반환한다. 
    JWS Compact Serialization 사례에서 결과는 JWS가 성공적으로 검증되었는지 여부를 간단히 나타낼 수 있다.

마지막으로, 주어진 컨텍스트에서 사용할 수 있는 **알고리즘은 응용프로그램 결정한다.**  

**JWS를 성공적으로 검증할 수 있더라도 JWS에 사용된 알고리즘이 애플리케이션에 허용되지 않는 한 JWS는 유효하지 않은 것으로 간주해야 한다.**



### Key Identification

JWS의 수신자는 디지털 서명 또는 MAC 작업에 사용된 키를 확인할 수 있어야 한다. 

구체적으로 헤더 매개변수 "jku", "jwk", "kid", "x5u", "x5c", "x5t" 및 "x5t#S256"을 사용하여 사용된 키를 식별할 수 있다.  

**이러한 헤더 매개변수는 전달되는 정보가 신뢰 결정에 활용되려면 무결성 보호를 받아야 하지만, 신뢰 결정에 사용되는 유일한 정보가 키인 경우 이러한 매개변수는 무결성 보호를 받을 필요가 없다.** 

다른 키를 사용하는 방식으로 변경하면 유효성 검사가 실패하기 때문이다.

응용 프로그램이 사용된 키를 결정하기 위해 다른 수단이나 규약을 사용하지 않는 한, 제조업체는 사용된 키를 식별할 수 있는 충분한 정보를 헤더 매개 변수에 포함시켜야 한다. 

사용된 알고리즘에 키가 필요하고("없음"을 제외한 모든 알고리즘에 해당) 사용된 키를 확인할 수 없는 경우 Signature 또는 MAC 유효성 검사가 실패한다.



### Serializations

JWS는 JWS Compact Serialization 또는 JWS JSON Serialization의 두 가지 직렬화 중 하나를 사용한다.  

이 규격을 사용하는 응용 프로그램은 해당 응용 프로그램에 사용되는 직렬화 및 직렬화 기능을 지정해야 한다.  

예를 들어 JWS JSON Serialization만 사용하거나 단일 Signature 또는 MAC 값에 대한 JWS JSON Serialization 지원만 사용하거나 여러 Signature 및/또는 MAC 값을 지원하도록 지정할 수 있다. 

JWS 구현은 지원하도록 설계된 애플리케이션에 필요한 기능만 구현하면 된다.



#### JWS Compact Serialization

JWS Compact Serialization은 디지털 서명 또는 MACed 콘텐츠를 URL 안전한 콤팩트 문자열로 나타낸다.  

    JWS Protected Header || '.' ||
    JWS Payload || '.' ||
    JWS Signature
 JWS Compact Serialization에서는 **하나의 Signature/MAC만 지원**되며 **JWS Unprotected Header 값을 나타내는 구문은 제공되지 않는다.**



#### JWS JSON Serialization

 JWS JSON Serialization은 디지털 Signature 또는 MACed 콘텐츠를 JSON 개체로 나타낸다.  

이 표현은 압축성이나 URL 안전성에 최적화되지 않았다.

JWS JSON 직렬화를 위해 밀접하게 관련된 두 가지 구문이 정의되어 있다. 

하나는 두 개 이상의 디지털 Signature 및/또는 MAC 작업으로 콘텐츠를 보호할 수 있는 완전히 일반적인 구문과 하나는 디지털 Signature 또는 MAC 사례에 최적화된 플랫 구문이다.



#### General JWS JSON Serialization Syntax

 다음 멤버는 완전히 일반적인 JWS JSON Serialization 구문에 사용되는 최상위 JSON 개체에서 사용하도록 정의된다 :

+ **payload**
  **BASE64URL(JWS Payload)** 값을 포함해야 한다.

+ **signatures**
  **JSON 개체의 배열이어야 한다.** 
  각 개체는 JWS Payload 및 JWS Protected Header를 통해 Signature 또는 MAC를 나타낸다.
  + **protected**
    JWS Protected Header 값이 비어 있지 않은 경우 "보호된" 멤버가 있어야 하며 BASE64URL(UTF8(JWS Protected Header)) 값을 포함해야 한다. 
    그렇지 않은 경우에는 BASE64URL이 없어야 한다.  
    이러한 헤더 매개 변수 값은 무결성 보호를 받는다.
  + **header**
    JWS Unprotected Header 값이 비어 있지 않은 경우 "Header" 멤버가 있어야 하며 JWS Unprotected Header 값을 포함해야 한다. 
    그렇지 않으면 "Header" 멤버가 없어야 한다.  
    이 값은 문자열이 아닌 인코딩되지 않은 JSON 개체로 표시된다.  
    이러한 헤더 매개 변수 값은 무결성 보호되지 않는다.
  + **signature** 
    구성원이 있어야 하며 BASE64URL(JWS 서명) 값을 포함해야 한다.
    "alg" 헤더 매개 변수 값이 전달되려면 각 서명/MAC 계산에 대해 "protected" 및 "header" 구성원 중 하나 이상이 있어야 한다.
    위에 정의된 두 JSON 개체 모두에 추가 멤버가 있을 수 있다. 



개별 서명 또는 MAC 값을 생성하거나 검증할 때 사용되는 헤더 매개 변수 값은 존재할 수 있는 두 개의 헤더 매개 변수 값 집합 

(1) 서명/MAC의 배열 요소의 "protected" 멤버에 표시되는 JWS Protected Header

(2) 서명/MAC 배열 요소의 "header" 멤버에 있는 JWS UnProtected Header

이러한 헤더 매개변수 집합의 결합은 JOSE Header로 구성된다.  

두 위치의 Header 매개 변수 이름은 분리되어야 한다.



각 JWS Signature 값은 JWS Compact Serialization과 동일한 방법으로 해당 JOSE Header 값의 매개 변수를 사용하여 계산된다.  

이것은 Signature 배열에 표시된 각 JWS 서명 값이 JWS 압축 직렬화에서 사용된 것과 일치하는 경우, 해당 Signature/MAC 계산에 대한 JWS Protecetd Header 값이 JWS 압축 직렬화에서 동일한 매개 변수에 대해 계산되었을 값과 동일하다는 바람직한 속성을 가진다.

**요약하면 일반적인 JWS JSON 직렬화를 사용하는 JWS의 구문은 다음과 같다:**

```json
{
"payload":"<payload contents>",
"signatures":[
 {"protected":"<integrity-protected header 1 contents>",
  "header":<non-integrity-protected header 1 contents>,
  "signature":"<signature 1 contents>"},
 ...
 {"protected":"<integrity-protected header N contents>",
  "header":<non-integrity-protected header N contents>,
  "signature":"<signature N contents>"}]
}
```



#### Flattened JWS JSON Serialization Syntax

 Flattened JWS JSON Serialization 구문은 일반 구문을 기반으로 하지만 단일 디지털 Signature/MAC 사례에 맞게 최적화하여 평평하게 만든다. 

 "signatures" 멤버를 제거하고 대신 "signatures" 배열에 사용하도록 정의된 멤버("protected", "header" 및 "signatures" 멤버)를 최상위 JSON 개체("payload" 멤버와 동일한 수준)에 배치하여 이를 평평하게 한다.

이 구문을 사용할 때는 "서명" 구성원이 없어야 한다. 

이 구문의 차이를 제외하고 플랫 구문을 사용하는 JWS JSON Serialization 개체는 일반 구문을 사용하는 개체와 동일하게 처리된다. 

요약하면, 플랫화된 JWS JSON Serialization를 사용하는 JWS의 구문은 다음과 같다:

````
 {
  "payload":"<payload contents>",
  "protected":"<integrity-protected header contents>",
  "header":<non-integrity-protected header contents>,
  "signature":"<signature contents>"
 }
````
