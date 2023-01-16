## Media Server Architecture

![MediaServiceArchitecture](https://user-images.githubusercontent.com/102807742/211788805-488874d4-8421-4582-85b3-2df7bafed318.png)



~~위는 현시점(2022.1.11)에 설계한 Media Service Archecture이다.~~

~~위는 현시점(2022.1.16)에 설계한 Media Service Archiecture이다.~~

위는 현시점(2022.1.17)에 설계한 Media Service Archiecture이다. (kafka가 돌아왔다...!!!)

크게 Nginx를 기반으로 nginx-rtmp-module을 활용한 **Media Server**,

방송을 위한 **OBS**,

그리고 이 둘을 관리하기 위한 **Manager Server**,

많은 사람들이 몰렸을 경우 이를 분산 시켜 Media Server의 부담을 줄여줄 **CDN**,

라이브 영상을 서버에 저장하기 위한 **Origin Server**,

그리고 이들 사이에서 정보교환을 담당하는 **Kafka**,

마지막으로 CDN 많아질 경우 CDN에 대한 요청을 분산할 **Load Balancer** 로 구성되어 있다.



자세한 설명은 별도의 페이지에서 하도록 하고 간단하게 각 서버의 역할과 기술 선택의 이유를 설명하면 다음과 같다.



### Media Server

Media Server의 경우 유료 서비스를 고려하면 서비스의 선택지가 넓어진다.

유명한 AWS, Wowza 등의 서비스가 존재하고 이 서비스를 이용하면 Media Server 구현은 비교적 쉬워진다.

하지만 이번 프로젝트의 목표는 Media Server를 사용하는 것이 아닌 **구현**해보는 것이기에 위의 서비스들처럼 이미 모든 것이 구현되어 있고 사용만 하는 것은 제외하기로 하였다.

그래서 찾아본 것이 오픈소스였고 위키의 [List of RTMP software](https://en.wikipedia.org/wiki/List_of_RTMP_software)에는 다양한 오픈소스 후보 리스트가 존재하였고 그 중에서 Red5 그리고 Nginx with nginx-rtmp-module을 후보로 선택하였다.

위의 후보를 선택한 최우선 기준은 우선 우리 프로젝트의 요구사항인 **"RTMP to HLS를 지원하는가?"** 였습니다.

해당 기준을 통과하는 오픈소스는 다음과 같았습니다.

+ Nginx with nginx-rtmp-module
+ Nimble Streamer
+ Helix Universal Server
+ Red5 Media Server

모든 오픈소스를 조사하고 가능성을 확인하는 것은 현실적으로 문제가 있을 것이라 생각하였고 

현재 주력 언어인 java를 기반으로 만들어진 **Red5** 

그리고 Nginx 기반이어 관련 자료가 많은 **Nginx with nginx-rtmp-module**를 선택하여 조사와 가능성을 확인해 보았다.



Red5의 경우 오픈소스이고 이를 활용한 서비스(ant-media-server) 도 있을 정도로 검증된 오픈소스라는 생각이 들었지만 사실상 오픈소스가 아니라는 생각이 들었다.

Red5의 경우 Pro 버전이 존재하였고 모든 자료들이 Pro 버전을 기준으로 작성되었기에 관련 자료들을 찾기가 어려웠다.

따라서 기본적인 RTMP to HLS는 구현할 수 있겠지만 구현이 아닌 단순 사용에 그칠 가능성이 높다고 생각하였다.

// ant-media-server : 이 역시 오픈소스이며 media server을 위한 rest-api도 지원한다.



반면 Nginx with nginx-rtmp-module의 경우는 RTMP to HLS를 지원하고 Nginx 기반으로 관련자료들도 많기에 이번 프로젝트에서 사용하기에 적절하다는 판단을 하였다.

또 Nginx with nginx-rtmp-module의 장점은 위의 다른 오픈소스들과 다르게 Nginx기반의 오픈소스이기 때문에 Nginx를 통해 단순 영상 소스의 릴레이만 수행하는 것이 아닌 **검증 과정**도 추가할 수 있다는 장점이 있었다.

// 검증 과정에 대해서는 이후 자세히 알아볼 것이다. 

// 간단히 설명하면 방송 송출자가 유요한 주소로 방송을 송출하는지 확인하는 검증이다.



### OBS

~~OBS는 오픈소스 기반의 방송 송출 프로그램이다.~~

~~OBS를 통해 방송 송출자가 방송을 송출하고 종료한다.~~

~~OBS에는 **obsWebSocket**을 지원하는데 이를 활용하면 OBS를 원격으로 조정할 수도 있다고 한다.~~

~~실제로 이를 활용한 툴도 인터넷에서는 팔고 있다.~~

~~하지만 이번 프로젝트에서는 그 정도까지 obsWebSocket을 사용하지 않고 OBS가 <u>송출중</u>인지 그리고 <u>송출종료</u>하였는지 그 정보에 대해서만 사용할 예정이다.~~

~~이러한 기능이 필요한 이유는 이를 통해 방송이 송출중인지 다른 기능들(채팅, 알림)에게 알려아할 필요가 있고 채팅의 경우 방송 송출종료 역시 알려야할 필요가 있다.~~

~~Now는 방송국처럼 정해진 시간 동안 방송하고 종료하는 것이기에 위의 기능이 필요없다고 느낄 수 있지만 모든 것이 예상한 대로 흘러가지는 않는다고 생각한다 그렇기에 방송이 의도치 않게 종료되는 상황도 고려하여야 했고 그러한 의도치 않은 상황을 파악을 위한 기능이 위의 기능이다.~~

~~이때 단순히 obsWebSocket을 사용하는 것이 아닌 WebFlux 기반으로 사용해볼지 고민 중인데 이는 WebFlux의 장단에 대하여 조금더 공부해본 후 결정할 것이다.~~

OBS는 Kafka가 가능하다는 것이 확인되어 Nginx에서 라우팅하여 Manager Server에서 검증하는 것으로 하는 것이 좋을 것 같다.

그렇게 되면 OBS는 방송 송출 본연의 역활만하게 되는 것이다.



### Manager Server

Manager Server는 위의 Media Server와 OBS를 관리하기 위해 필요하다.

OBS와는 바로 위에서 언급한 것과 같이 obsWebSocket을 통해 OBS의 생존 여부를 관리할 것이다.

Media Server가 Nginx 기반이기에 Media Server 내에서 인증관련 처리를 할 수 있지만 그 검증에서 사용할 데이터는 Manager Server가 제공해주어야 한다.

아직 명확하게 책임 소재를 정하지는 못했는데 **목표**는 Media Server에서는 최대한 스트리밍에 관한 기능만 할 수 있도록 구현하는 것을 목표로 할 것이다.

---

#### ~~방안 1)~~

~~Manager Server을 통해 검증에 필요한 데이터를 받는다.~~

~~이 데이터를 DB에 저장하고 이를 Media Server의 Nginx와 공유하여 검증과정을 Media Server에서 진행한다.~~

#### 방안 2)

Media Server에서 필요한 검증의 경우 Manager Server로 그 정보들을 라우팅하여 Manager Server에서 검증을 진행한다.

: rtmp 설정중 on_publish라는 설정을 통해 방송을 하려고 하는 순간에 Manger Server에서 검증을 진행 할 수 있다.

---



### CDN, Origin Server, Kafka, FFmpeg

~~이 부분이 가장 애매하였던 부분이다.~~

~~사실 Media Server의 데이터를 그냥 CDN, Origin Server에 릴레이 형식으로 전달하여도 문제는 없다.~~

~~하지만 CDN이나 Origin Server가 적으면 문제가 없겠지만 CDN이 추가가 된다면? Origin Server가 추가된다면 하는 생각을 하면 이는 문제가 될 것 같다는 생각을 하였다.~~

~~그래서 이러한 문제를 해결할 수 있는 방법을 생각하였을 때 Kafka를 활용하면 어떨까? 하는 생각이 들었다.~~

~~Kafka를 활용하면 Media Server에서 Kafka의 메시지를 생산하고 이 메시지가 필요한 곳에서 메시지를 소비하는 것이다.~~

~~그렇게 된다면 Media Server은 Kafka에게만 메시지를 보내면 된다.~~

~~CDN이나 Origin Server가 추가된다면? 이때는 Media Server가 생산한 메시지를 소비하기만 하면된다.~~

~~이러한 생각이 가능한 것인가 확인하기 위해 테스트를 해본 결과 Kafka를 통해 영상 데이터가 옮겨지기는 한다.~~

~~그런데 이러한 사용의 장단을 아직 정확하게 파악하지는 못하여 조금 더 검토가 필요할 것 같다.~~

~~이와 더불어 Origin Server에서는 영상을 파일로 저장할 것인지 아님 DB에 저장할 것인지 아직 정하지 못하였다.~~

~~대용량 데이터의 경우 하둡과 카산드라를 통해 저장할 수 있다고 하는데 이 역시 조금 더 검토가 필요할 것 같다.~~

~~CDN과 Origin Server 역시 Nginx를 사용한다.~~

~~이렇게 되면 통신 및 그 결과는 다음과 같이 될 것이다.~~

~~**OBS** -> RTMP -> **Media** -> RTMP -> **Origin** ->  **File**(최고 화질)~~

~~**OBS** -> RTMP -> **Media** -> RTMP -> **CDN** ->  HLS (화질 다양화 by FFmpeg) -> **Consumer~~**

Kafka가 돌아왔다.

데모 버전으로 확인한 결과 라이브에서도 잘 작동된다.

기존에 Kafka를 포기한 이유는 https가 로컬상에서 잘 적용되지 않아 이것을 해결하지 못해 라이브도 Kafka로 영상이 옮겨지는지 확인하지 못해서였다.

하지만 가능하다는 것을 확인하였고 Kafka를 도입해보려한다.

**그렇다면 Kafka를 도입하면서 Kafka가 없을 때보다 어떤 장점이 있을지 아님 오히려 단점만 있을지 그것이 미디어 서버의 핵심이 될 것 같다.**



### Load Balancer

CDN의 갯수가 증가한다면 Load Balancer가 필요할 것이라 생각한다.

이전에는 Eureka를 활용하여 Server들을 찾고 Ribon을 활용하여 Load Balancing을 하였다.

하지만 위는 Spring boot 기반의 Server들에 해당한 것이기 때문이 이번에는 다양한 옵션을 고려해보아야 할 것 같고 이와 관련해서는 오픈소스들이 많기 때문에 이 역시 조금 더 검토 후 결정하면 될 것 같다.

: 이번에는 Load Balancer 역시 Nginx를 사용할 것 같다.

대신 Load Balancer를 통해 들어오는 요청을 log로 남겨두어 이를 가지고 ELK를 사용할 수 있는 기반을 만들어 두어야겠다.
