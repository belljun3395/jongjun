

## MSA 환경구축을 위한 사전조사

![MSAvsMONOLITHC](/Users/jongjun/Desktop/jongjun/project/smilegate/img/MSAvsMONOLITHC.png)

위는 모놀리식 방식과 MSA 방식을 나타내는 사진이다.

각 기능이 독립적인 것을 볼 수 있다.

이는 개별 서비스가 **다른 서비스에 부정적인 영향을 주지 않으면서** 작동할 수 있음을 의미한다.

또 각 마이크로 서비스들이 서비스에 가장 적합한 언어, 버전 그리고 데이터 베이스도 제약 없이 선택할 수 있다.

이러한 장점들로 인해 MSA 방식으로 서버를 구축한다.



## MSA 환경 구축을 위해 필요한 것

+ Config Server
+ Config Client
+ Eureka Server
+ Eureka Registry Client
+ Circuit Breaker
  + Resilience4j
+ Gateway, Loadbalancing



## 설명

### Config Server/Client

중앙 집중식 서비스로 애플리케이션 구성 데이터 관리를 담당하고 애플리케이션 데이터를 마이크로 서비스와 완전히 분리하는 역활을 한다.

공통 구성 서버에서 설정한 값을 클라이언트 서버에서 읽어와 그 값을 사용하여 환경을 구성한다.

**Config-Server을 설정함으로써 공통적인 환경을 가져오거나 애플리케이션에 맞게 가져올 수 있는 것이다.**

그리고 actuator의 refresh를 통해 변경된 설정 값도 서버의 재시작 없이 적용 가능하다.



### Eureka Server/Client

Eureka Server는 Discovery Server라고도 한다.

Eureka Server는 서버가 자신의 서비스 이름과 ip주소 그리고 포트를 등록하고 서비스 이름으로 서버 목록을 조회할 수 있다.

Eureka Server가 동적으로 서버를 찾아주어 우리가 수동적으로 추가되거나 제거되는 서버를 수정할 필요가 없어진다.



### Circuit Breaker

**일정 시간** 동안, **일정 개수** 이상의 호출이 발생한 경우, **일정비율** 이상의 에러가 발생하면 -> **Circuit Open**

**일정 시간** 경과 후에 단 한개의 요청에 대해서 호출을 허용하며, 이 호출이 성공하면 -> **Circuit Close**

![feign](/Users/jongjun/Desktop/jongjun/project/smilegate/img/feign.png)

위 사진은 Circuit Breaker를 구현하기 위한 대표적인 라이브러리인 Fegin의 요청 흐름을 나타내는 것이다.

위에서 언급한 것처럼 **일정 시간** 동안, **일정 개수** 이상의 호출이 발생한 경우, **일정비율** 이상의 에러가 발생하면 가장 앞단의 Circuit이 Open 되는 것을 볼 수 있다.

Circuit의 Open 여부가 timeout에 의해 결정되기 때문에 **timeout** 시간 설정은 중요하다.

추후에 나오겠지만 loadBlancer와 함께 사용한다면 고려해야 할 것이 많아진다.

loadBalncer가 등록된 서버에 round robine 방식이든 다른 로직을 통해 요청을 보내는 시간을 고려해야한다.



### Gateway

우선 MSA에서 API Gateway는 Single Endpoint를 제공해준다.

이는 Client들이 API Gateway 주소 하나만 인지 하면 Gateway가 포함하고 있는 API를 모두 사용할 수 있다.

그리고 **Logging, Authentication, Authorization**과 같은 API 공통 로직을 구현 한다.

또 **트래픽**을 조절할 수 있다.

---

Key Word:  API Quota, Throttling  -> 추가적인 공부 필요

---

이런 Gateway를 구현하는데는 크게 Spring Cloud와 Zuul를 사용하여 구현할 수 있다.

![zuul](/Users/jongjun/Desktop/jongjun/project/smilegate/img/zuul.png)

![springCloud](/Users/jongjun/Desktop/jongjun/project/smilegate/img/springCloud.png)

위의 두 사진은 zuul과 spring cloud gateway가 어떻게 동작하는지를 나타내는 사진들이다.

둘다 비슷하게 Client에게 요청을 받아 filter를 거처 Service로 전달해주는 것을 확인할 수 있다.

1인 프로젝트에서 zuul을 써본 경험에 비추어 생각해보면 gate way에서 받는 http request를 그대로 Service에게 전달하지는 않는다는 생각을 하였다.

그 이유는 zuul 설정으로 `sensitiveHeaders` 가 존재한다.

이는 아무런 설정을 하지 않을 경우 `Cookie,Set-Cookie,Authorization` 가 기본값으로 설정되어 저 기본값들이 Service까지 전달되지 않는다.

응답에서도 마찬가지로 저 기본값들이 응답에 포함되지 않는다.



부가적인 설명은 이만하고 Zuul과 Spring Cloud Gateway중에서 우리 프로젝트에 더 맞는 것은 Spring Cloud Gateway이다.

이유는 간단하다 Zuul은 websocket이 지원되지 않는다.

그리고 이제 netflix에서 더이상 개발은 하지 않고 유지보수만 하고 있는 상태라 최신의 스프링 버전을 사용할 수 없어 다른 라이브러리를 사용하는 것에도 제약을 줄 수 있다.

따라서 우리가 사용해야할 api gateway는 Spring Cloud Gateway가 적절한 것 같다.



**Spring Cloud로 구현하는 MSA Archietecture 예시)**

![SpringCloudMSAArchietecture](/Users/jongjun/Desktop/jongjun/project/smilegate/img/SpringCloudMSAArchietecture.png)



### LoadBalancer

Ribbon은 Spring Cloud의 LoadBalancer이다.

이는 클라이언트를 사용자가 직접 사용하는 것이 아니라 Spring Cloud의 Http 통신이 필요한 요소에 내장되어 있다고 한다.

+ Zuul Api Gateway
+ RestTemplate with @LoadBalanced 
+ Spring Cloud Feign

또 등록된 서버가 잘 살아 있는지 확인하는 등 다양한 설정을 할 수 있다고 한다.



## 정리

MSA 환경을 구축하는 파트의 경우 기본적으로 다음의 것들을 구현하여야 한다.

+ Config Server
+ Eureka Server
+ Circuit Breaker
  + Resilience4j
+ Gateway
+ Loadbalancing

또 이것과 더하여 인증, 인가를 함께하는 것이 효율적일 것이라 생각한다.

![kakaLoginwithApiGateway](/Users/jongjun/Desktop/jongjun/project/smilegate/img/kakaLoginwithApiGateway.png)

위는 카카오의 Api Gateway와 함께 구축한 인증, 인가 과정이다.

위의 과정을 참고하여 인증, 인가 과정을 설계하면 좋을 것 같다.



또 이것과 더하여 다음과 같은 모니터링 도구 역시 구현해야한다.

+ zipkin
+ turbine
+ spring boot admin

아직 구체적인 방법은 모르겠지만 이러한 모니터링 도구와 로그 데이터를 잘 남겨 놓아 모니터링 도구를 통해 에러를 감지하면 알림이와 로그 데이터를 보며 이를 수정할 수 있는 환경을 구축하면 좋을 것 같다.