## API Gateway는 어떤 것으로?

### API Gateway란?

우선 API Gateway를 올바르게 이해하기 위해서는 BFF를 이해할 필요가 있다.

BFF는 API Gateway의 한 종류로 한 종류의 클라이언트에 대한 응답만 할 수 있다.

즉, 응답이 특정 클라이언트에 의존성이 생기게 된다.

이는 BFF의 경우 흩어져있는 마이크로 서비스들을 단순히 하나의 엔드포인트로 묶는 역할만을 하기 때문이다.

이에 관한 자세한 설명은 다음의 링크를 보면 더욱더 자세히 알 수 있다.

https://youtu.be/P2nM0_YptOA



하지만 API Gateway의 경우 클라이언트들이 접근하려는 데이터를 위한 것이라는 것이 다르다.

그렇기에 API Gateway는 인증과 같은 기능을 다루고 **다른 마이크로 서비스들의 자원을 클라이언트들에게 제공하기 위해 적절하게 묶는다.**

따라서 **단순히 라우팅만 한다면** 그것은 API Gateway를 구현하는 것이 아닌 **BFF**를 구현하게 되는 것이다.



더 쉽게 이해하려면 다음과 같이 이해하면 편할 것 같다.

| Spring MVC | MSA         |
| ---------- | ----------- |
| Controller | API Gateway |
| Domain     | MSA Service |



그렇기에 API Gateway에서는 각각의 서비스들의 요청들에서 생길 수 있는 오류를 파악하여 **Timeout 및 오류 전파를 막을 수 있는 조치**를 하여야 한다.

예를 들면 다음과 같이 조치할 수 있다.

---

평시에는 평균 응답시간이 20ms 이고, 요청량이 급증해서 CPU 70% memory 80% 찍으면 응답시간이 보통 100ms 로 지연이 되는데 이러면 100ms 를 타임아웃으로 설정하고 타임아웃이 전체적으로 몇회 이상 발생하면 서킷을 30초간 오픈하고 미리 설정해둔 fallback 데이터를 응답으로 내려보냅시다.  - 선배님에게 질문하면서 들었던 예시 - 

---



그렇기 때문에 위와 같은 요소들을 고민하지 않으면 API Gateway는 단일 장애지점이 될 가능성이 높아진다.

그래서 적절하게 스케일 아웃을 하거나 webflux와 같은 프레임워크를 활용하여 reactive하게 개발을 진행하여야 한다.

// todo 위의 내용 역시 선배님게 질문하면서 들었던 피드백이다. 아직 reactive한 개발이 정확히 무엇인지 모르겠어서 이에 대한 공부를 진행해야 겠다.



#### Zuul

`Zuul is an L7 application gateway that provides capabilities for dynamic routing, monitoring, resiliency, security, and more.`

Zuul의 [깃허브](https://github.com/Netflix/zuul) 에 있는 소개 문구이다.

L7 계층 게이트워이로 동적 라우팅을 지원하고, 모니터링, 탄력성, 보안 그리고 많은 것을 지원한다.

Zuul의 경우 Zuul1 그리고 Zuul2 두 가지 버전이 존재한다.

본인이 사용해본 Zuul의 경우 Zuul1으로 이하 내용은 **Zuul1**에 관한 내용이다.



##### 장/단점

##### ![1_pz6sv69la9ek6yWNTPqymQ](/Users/jongjun/Desktop/jongjun/project/smilegate/img/1_pz6sv69la9ek6yWNTPqymQ.webp)

본인이 생각하기에 Zuul의 가장 큰 장점은 위에 사진에서 볼 수 있듯 Netflix에서 제공하는 다양한 라이브러리이다.

본인의 경우 `Hystrix, Ribbon, Eureka` 를 사용하여 간단한 MSA라고 생각했던 BFF를 [구현한](https://github.com/belljun3395/msa-jongjun.git) 경험이 있다.

(지금은 BFF이지만 추후에 볼때는 **MSA**로...!!!)



단점으로는 Zuul1을 사용한 영향도 있겠지만 Zuul 자체가 개발을 이제 더이상 진행하는 것이 아니기 때문에 Spring boot 및 모든 라이브러리의 버전을 최신 버전으로 활용하지 못하였다.

추후에 계속해서 개발을 진행한다고 생각하면 이 부분도 꽤 중요한 부분일 것이라 생각한다.

왜냐하면 라이브러리가 버전을 올리는 이유는 이전 버전에서 버그가 발견되거나 기능들이 추가되기 때문이다 .

하지만 Zuul을 사용하게되면 이러한 업데이트를 반영하지 못하게 된다.

그리고 위의 라이브러리들에서 영감을 받은 Spring Cloud의 라이브러리들이 존재하기에 Netflix가 제공하는 라이브러리가 좋다고는 하지만 다른 위의 장점을 커버하지는 못한다고 생각한다.



#### Spring Cloud Gateway(SCG)

`This project provides an API Gateway built on top of the Spring Ecosystem, including: Spring 6, Spring Boot 3 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.`

역시 SCG의 [깃허브](https://github.com/spring-cloud/spring-cloud-gateway)의 소개 문구이다.

소개 문구에서도 알 수 있듯 글 작성기준(2022.01.03) 최근에 배포된 Spring Boot 3를 포함하고 있는 것을 볼 수 있다.



##### 장/단점

사실 Spring Cloud Gateway의 가장 큰 장점은 reactive한 API Gateway를 만들 수 있다는 것이다.

이는 SCG는 Tomcat이 아닌 Netty를 사용하기 때문이다.

Tocat은 `1THREAD / 1REQUEST 방식` 이고 Netty는 비동기 WAS며 `1THREAD / MANY REQUESTS 방식` 이기에 기존 방식보다 더 많은 요청을 처리할 수 있다.

이것이 장점이지만 아직 본인에게는 큰 장점이 되지 못할 수도 있다.

왜냐하면 아직 Netty를 활용해본 경험이 없기 때문이다.

물론 기본적인 라우팅 기능은 할 수 있겠지만 Netty의 특징을 활용해 API Gateway의 성능을 올릴 수 있을 지는 아직 확신할 수 없다.

즉, SCG를 사용하게 된다면 **본인이 단점**이 된다.(단점이 되지 않도록 더 열심히...ㅎㅎㅎ)



![스크린샷 2023-01-03 오후 5.42.47](/Users/jongjun/Desktop/jongjun/project/smilegate/img/스크린샷 2023-01-03 오후 5.42.47.png)

또 위의 사진에서 볼 수 있듯 최소 2025년까지 지원 계획이 잡혀있다..ㅎㅎ



#### 결정

최종적으로 Zuul과 SCG 중에 선택할 라이브러리는 SCG이다.

구현이 급한 경우이면 Zuul을 선택하겠지만 이번 프로젝트의 목적은 그것이 아니라 성장이다.

SCG로 구현한다면 물론 리스크가 있다.

하지만 이는 단순 구현에 대한 리스크가 아닌 성능 개선 및 성장에 대한 리스크이기 때문에 이 리스크는 본인이 감당하는 것이 좋을것 같다.

캠프기간동안 열심히 해야겠다...
