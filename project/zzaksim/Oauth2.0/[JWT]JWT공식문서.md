## [JWT] 공식문서

### Abstract

**JSON Web Token(JWT)은 두 당사자 간에 전송될 클레임을 나타내는 URL-safe한 컴팩트한 수단이다.**  

JWT의 Claim은 JSON Web Signature(JWS) 구조의 Payload 또는 JSON Web Encryption(JWE) 구조의 일반 텍스트로 사용되는 JSON 개체로 인코딩되므로 Claim을 디지털 Signature하거나 MAC(Message Authentication Code) 및/또는 암호화로 무결성을 보호할 수 있다.



### Introduction

**JSON Web Token(JWT)은 HTTP Authorization Header 및 URI 쿼리 매개 변수와 같은 공간 제약 환경을 위해 고안된 Claim 표현 형식이다.**

JWT는 JSON Web Signature(JWS) 구조의 Payload 또는 JSON Web Encryption(JWE) 구조의 일반 텍스트로 사용되는 JSON 개체로 전송되는 Claim을 인코딩하여 Claim을 MAC(Message Authentication Code) 및/암호화로 디지털 Signature하거나 무결성을 보호할 수 있다.

**JWT는 항상 JWS Compact Serialization 또는 JWE Compact Serialization을 사용하여 표시된다.**



### JSON Web Token (JWT) Overview

JWT는 JWS 및/또는 JWE 구조로 인코딩된 JSON 개체로 Claim 집합을 나타낸다. 

**JSON 개체는 JWT 클레임 집합이다.** 

JSON 개체는 0개 이상의 이름/값 쌍(또는 멤버)으로 구성된다. 

여기서 이름은 문자열이고 값은 임의의 JSON 값이다.  

이 JSON 객체는 JSON 값 또는 구조 문자 앞 또는 뒤에 공백 및/또는 줄 바꿈을 포함할 수 있다.



JWT 클레임 세트 내의 멤버의 이름을 Claim 키(key)이라고 한다.  

해당 값을 Claim 값(value)이라고 한다. 



JOSE Header의 내용은 JWT Claim 세트에 적용되는 암호화 작업을 설명한다.  

JOSE Header가 JWS용인 경우 JWT는 JWS로 표시되고 Claim은 JWT Claim 세트가 JWS Payload인 디지털 Signature 또는 MACed가 된다. 

JOSE Header가 JWE용인 경우 JWT는 JWE로 표시되고 Claim은 JWT Claim 세트가 JWE에 의해 암호화된 일반 텍스트로 암호화된다.  

JWT는 다른 JWE 또는 JWS 구조에 포함되어 중첩된 JWT를 생성하여 중첩된 Signature 및 암호화를 수행할 수 있다.



JWT는 마침표('.') 문자로 구분된 일련의 URL-safe로 표시된다.  

각 부품에는 base64url-encoded 값이 포함되어 있다. 

JWT의 부품 수는 JWS Compact Serialization(콤팩트 직렬화)을 사용한 결과 JWS 또는 JWE Compact Serialization(JWE 콤팩트 직렬화)을 사용한 JWE의 표현에 따라 달라진다.



### JWT Claims

JWT Claim 집합은 JWT에 의해 전달된 Claim의 구성원인 JSON 개체를 나타낸다.  

JWT Claim 집합 내의 Claim 이름은 고유해야 한다. 

JWT 파서는 중복된 Claim 이름이 있는 JWT를 거부하거나 사전적으로 마지막 중복 멤버 이름만 반환하는 JSON 파서를 사용해야 한다.



JWT 클레임 이름에는 세 가지 클래스가 있다: 

+ Registed Claim 이름 
+ Public Claim 이름 
+ Private Claim 이름



### JOSE Header

JWT 객체의 경우 JOSE Header로 표시되는 JSON 객체의 멤버는 JWT에 적용되는 암호화 작업과 선택적으로 JWT의 추가 속성을 설명한다.  

JWT가 JWS인지 JWE인지에 따라 JOSE Header 값에 해당하는 규칙이 적용된다.

이 사양은 JWT가 JWS인 경우와 JWE인 경우 모두에서 Header 매개변수를 사용하도록 지정한다.



### Unsecured JWTs

JWT 콘텐츠가 JWT에 포함된 Signature 및/또는 암호화 이외의 방법으로 보호되는 사용 사례를 지원하기 위해 Signature 또는 암호화 없이 JWT를 생성할 수도 있다.  

Unsecured JWT는 JWA 사양에 정의된 대로 "alg" Header Parameter 값 "none"을 사용하고 JWS Signature 값에 빈 문자열을 사용하는 JWS이며, JWT Claim 세트가 JWS Payload로 설정된 Unsecured JWS이다.



### Creating and Validating JWTs

#### Creating a JWT

1. 원하는 Claim이 포함된 JWT Claim 세트를 만든다.
   표현에는 공백이 명시적으로 허용되며 인코딩 전에 정규화를 수행할 필요가 없다.

2. 메시지를 JWT Claim 세트의 UTF-8 표현의 옥텟으로 한다.

3. 원하는 Header 매개변수 집합을 포함하는 JOSE Header를 만든다.
   JWT는 또는 JWE 사양을 준수해야 한다.
   표현에는 공백이 명시적으로 허용되며 인코딩 전에 정규화를 수행할 필요가 없다.

4. JWT가 JWS인지 JWE인지에 따라 두 가지 경우가 있다 
      *  JWT가 JWS인 경우 메시지를 JWS 페이로드로 사용하여 JWS를 생성한다. 
      *  JWT가 JWE인 경우 메시지를 JWE의 일반 텍스트로 사용하여 JWE를 만든다. 

5. 중첩된 Signature 또는 암호화 작업을 수행할 경우, 메시지를 JWS 또는 JWE로 지정하고 해당 단계에서 작성된 새 JOSE Header에서 "Cty"(내용 유형) 값 "JWT"를 사용하여 3단계로 돌아간다.

6. 그렇지 않은 경우, 결과 JWT를 JWS 또는 JWE로 한다.



#### Validating a JWT

1. JWT에 마침표('.') 문자가 하나 이상 포함되어 있는지 확인한다.
 2. 인코딩된 JOSE Header를 첫 번째 마침표('.') 문자 앞에 있는 JWT의 일부로 한다.
 3. Base64url은 줄 바꿈, 공백 또는 기타 추가 문자를 사용하지 않았다는 제한에 따라 Encoded JOSE Header를 디코딩합니다.
 4. 결과 옥텟 시퀀스가 RFC 7159를 준수하는 완전히 유효한 JSON 개체의 UTF-8 인코딩된 표현인지 확인한다. 
    JOSE Header를 이 JSON 개체로 한다.
5. 결과 JOSE Header에 구문과 의미가 모두 이해되고 지원되는 매개 변수와 값만 포함되어 있는지, 이해되지 않을 때 무시되도록 지정된 값만 포함되어 있는지 확인한다.
6. JWT가 JWS인지 JWE인지 확인한다.
7. JWT가 JWS인지 JWE인지에 따라 두 가지 경우가 있다: 
   + JWT가 JWS인 경우 JWS의 유효성을 검사한다.  
     메시지가 JWS Payload를 디코딩하는 base64url의 결과가 되도록 한다.
   + JWT가 JWE인 경우 JWE를 검증한다.  
     메시지를 일반 텍스트로 지정한다.
8. JOSE 헤더에 "cty"(내용 유형) 값이 "JWT"인 경우, 메시지는 중첩된 Signature 또는 암호화 작업의 대상이었던 JWT이다. 
   이 경우 메시지를 JWT로 사용하여 1단계로 돌아간다.
9. 그렇지 않으면 base64url은 줄 바꿈, 공백 또는 기타 추가 문자를 사용하지 않았다는 제한에 따라 메시지를 디코딩한다.
10. 결과 옥텟 시퀀스가 RFC 7159를 준수하는 완전히 유효한 JSON 개체의 UTF-8 인코딩 표현인지 확인한다. JWT Claim 세트를 이 JSON 개체로 한다.