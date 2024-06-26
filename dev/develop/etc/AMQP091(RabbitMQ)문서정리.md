# AMQP 0-9-1(RabbitMQ) 문서 정리



RabbitMQ에서 제공하는 [AMQP 문서](https://www.rabbitmq.com/tutorials/amqp-concepts)를 바탕으로 프로젝트를 진행하며 도움이 되었던 내용을 정리합니다.

아래 내용은 `문서 번역 | 추가 메모` 로 구성되어 있습니다.



## What is AMQP 0-9-1?

AMQP 0-9-1(고급 메시지 큐 프로토콜)은 호환되는 클라이언트 애플리케이션이 **호환되는 메시징 미들웨어 브로커와 통신**할 수 있도록 하는 **메시징 프로토콜**입니다.



메시징 브로커는 퍼블리셔(메시지를 게시하는 애플리케이션, 생산자라고도 함, publisher)로부터 메시지를 수신하여 컨슈머(메시지를 처리하는 애플리케이션, consumer)에게 라우팅 합니다.

네트워크 프로토콜이기 때문에 퍼블리셔, 컨슈머, 브로커가 모두 다른 컴퓨터에 있을 수 있습니다.



---

![img](https://blog.kakaocdn.net/dn/dWB6kd/btsGAzBJddr/avwtgts26aikVc4LLnVtB1/img.png)

위와 같이 퍼블리셔, 컨슈머, 브로커가 모두 다른 컴퓨터에서 동작할 수 있기에 서버의 역할 분리가 가능합니다.



### 서버 분리 예시

특정 사이트에서는 사용자가 회원가입을 하면 축하 이메일 발송하는 기능이 있고 이때 발생하는 비용은 아래와 같다고 가정해 봅시다.

- 회원 가입 비용: 30
- 이메일 전송 비용: 70



이 과정을 분리하지 않고 한 번에 처리하면 아래와 같이 서버를 구성할 수 있습니다.

![img](https://blog.kakaocdn.net/dn/cyTquo/btsGBuTUGbP/fwmiqLElUcVdftYo48RE8k/img.png)      

추가로 이때 회원 서버가 동작하는 컴퓨터가 한 번에 처리할 수 있는 비용을 300이라 가정하면 위의 구성에서는 3개의 회원 가입 요청을 처리할 수 있습니다.



요청에 대한 처리 성능을 개선하기 위해 요청을 처리하는 데 필요한 과정을 다시 생각해 봅시다.

- 회원 가입: 해당 과정이 요청 즉시 수행되어야 합니다.
- 이메일 전송: 해당 과정이 요청 이후 수행되어도 괜찮습니다.

요청 즉시 수행되어야 하는 회원 가입 과정의 비용보다 요청 이후 수행되어도 괜찮은 이메일 전송의 비용이 더 큽니다.

이는 회원 서버의 스펙을 증가시키거나 개수를 늘리더라도 요청 처리량을 늘리는 데 한계가 있음을 의미합니다.



이를 개선하는 방법으로 이메일 전송과 같은 요청 이후 수행되어도 괜찮은 기능을 분리하는 방법이 있습니다.

회원 가입 요청을 회원 가입 과정, 이메일 전송 과정으로 분리하면 아래와 같이 서버를 분리할 수 있습니다.

![img](https://blog.kakaocdn.net/dn/bAEl8d/btsGCZrHDrx/BbxwK2uBeNDh3nwjt0Sy2k/img.png)

위와 같은 서버 구성에서 회원 서버는 비용이 많이 들던 이메일 전송 과정을 수행하지 않고 회원 가입 과정을 수행한 이후 가입 회원 정보 메시지를 발행하기만 하면 됩니다.

그렇기에 위와 같이 분리한 서버 구성에서는 분리하기 이전과 동일한 스펙의 회원 서버에서 이전의 3개보다 증가한 10개의 회원가입 요청을 처리할 수 있게 됩니다.

이때 분리된 이메일 전송 과정은 알림 서버가 브로커를 통해 전달되는 가입 회원 정보 메시지를 처리하며 수행됩니다.



이처럼 비용이 많이 들지만, 비동기적으로 처리하여도 괜찮은 기능이라면 메시징 프로토콜을 활용해 해당 기능을 분리하여 처리하면 좋을 것 같다고 생각하였습니다.





## AMQP 0-9-1 is a Programmable Protocol

AMQP 0-9-1은 브로커 관리자가 아닌 **애플리케이션 자체에서 AMQP 0-9-1 엔티티와 라우팅 체계를 주로 정의한다는 점**에서 프로그래밍이 가능한 프로토콜입니다. 

따라서 큐와 교환을 선언하고, 교환 간의 바인딩을 정의하고, 큐에 가입하는 등의 프로토콜 작업을 위한 규정이 마련되어 있습니다.

**이는 해당 프로토콜이 애플리케이션 개발자에게 많은 자유를 제공하지만, 동시에 잠재적인 충돌을 인식해야 함을 의미합니다.**



애플리케이션은 필요한 AMQP 0-9-1 엔티티를 선언하고, 필요한 라우팅 체계를 정의하며, 더 이상 사용하지 않을 경우 AMQP 0-9-1 엔티티를 삭제하도록 선택할 수 있습니다. 



## Queues

AMQP 0-9-1 모델의 큐는 다른 메시지 및 작업 큐 시스템의 큐와 매우 유사하며, 애플리케이션에서 소비되는 메시지를 저장한다. 

큐는 교환과 일부 속성을 공유하지만 몇 가지 추가 속성도 가지고 있습니다.

- Name
- Durable(브로커 재시작 후에도 큐가 유지됨)
- Exclusive(하나의 연결에서만 사용되며 해당 연결이 닫히면 큐가 삭제됨)
- Auto-delete(마지막 소비자가 구독을 취소할 때 적어도 한 명의 소비자가 있는 대기열이 삭제됨)
- Arguments(선택 사항, 메시지 TTL, 대기열 길이 제한 등과 같은 플러그인 및 브로커 별 기능에서 사용)



큐를 사용하려면 먼저 큐를 선언해야 하며 이때 큐가 아직 존재하지 않는 경우 큐는 생성됩니다.

만약 큐가 이미 존재하지만 다시 한번 큐를 선언한다면 그 속성이 기존 큐의 속성과 동일한 경우 아무런 영향을 미치지 않습니다.

하지만 그 속성이 기존 큐 속성과 같지 않다면 코드 406(PRECONDITION_FAILED)이 포함된 채널 수준 예외가 발생합니다.



---

AMQP 0-9-1과 같이 프로그래밍할 수 있는 프로토콜에서 엔티티와 라우팅 체계와 같은 정의를 어떻게 관리할 수 있는지 궁금하였는데 **코드 406**와 같은 예외를 통해 개발자가 바로 이를 확인할 수 있기에 관리가 가능할 것이라는 생각을 하였습니다.



## Queue Durability

AMQP 0-9-1에서 큐는 영구 또는 일시 큐로 선언할 수 있습니다.

영구 큐의 메타데이터는 디스크에 저장되고, 일시 큐의 메타데이터는 가능한 경우 메모리에 저장됩니다.



## Consumers

애플리케이션이 메시지를 사용할 수 없다면 대기열에 메시지를 저장하는 것은 쓸모가 없습니다.

AMQP 0-9-1 모델에서는 애플리케이션이 이를 수행할 수 있는 두 가지 방법을 제공합니다.

- **Push API**: 메시지를 전달받도록 구독, 권장되는 옵션입니다.

- **Pull API**: 이 방법은 매우 비효율적이므로 대부분의 경우 피해야 합니다.



Push API를 사용하면 애플리케이션이 특정 큐의 메시지를 소비하는 데 관심을 표시해야 합니다.

이는 애플리케이션이 Push API를 통해 메시지를 사용하려면 특정 큐에 대한 컨슈머가 되어야 함을 의미합니다.



이때 큐는 두 명 이상의 컨슈머를 보유하거나 독점 컨슈머(소비하는 동안 대기열에서 다른 모든 소비자를 제외)를 등록할 수 있습니다.

추가로 각 컨슈머는 컨슈머 태그라는 식별자를 부여받으며 이 문자열뿐인 식별자는 메시지 수신 취소를 하는 데 사용될 수 있습니다.



---

Push API의 경우 주로 사용하는 API이기 때문에 익숙하지만, Pull API의 경우 그렇지 않아 간단히 이를 비교해 보고자 합니다.



```
// Push API

// 특정 큐의 컨슈머로 등록된 애플리케이션에서는 아래와 같은 코드를 통해 브로커가 전달하는 메시지를 소비한다.
(consumerTag, delivery) -> {
	byte[] body = delivery.getBody();
	...
}
```

```
// Pull API

while (true) {
	// 컨슈머로 등록한 것이 아니기에 브로커가 메시지를 애플리케이션에게 전달하지 않는다.
	// 따라서 애플리케이션이 스스로 특정 큐에게 새로운 메시지에 대한 조회를 진행하여야 한다.
	var payload = channel.basicGet("QUEUE_NAME", true);
	if(payload == null) {
		// sleep
	} else {
		byte[] body = payload.getBody();
		...
	}
}
```

코드 참고: https://stackoverflow.com/questions/71102930/rabbitmq-pull-vs-push-consume



## Message Acknowledgements

컨슈머 애플리케이션, 즉 메시지를 수신하고 처리하는 애플리케이션은 때때로 개별 메시지를 처리하지 못하거나 서버 연결이 끊어지거나 기타 여러 가지 방식으로 실패할 수 있습니다.

또한 네트워크 문제로 인해 문제가 발생할 가능성도 있습니다.

이 경우 브로커가 언제 큐에서 메시지를 제거해야 하는지에 대한 의문이 제기됩니다.



AMQP 0-9-1는 컨슈머에게 이를 제어할 수 있는 권한을 부여합니다.

이에 두 가지 승인 모드가 존재하고 각 승인 모두에 따른 응답 시기는 아래와 같습니다.

- **자동 승인 모델**: 브로커가 `basic.deliver` 또는 `basic.get-ok` 메서드 사용하여 애플리케이션에 메시지를 보낸 후
- **명시적 승인 모델**: 애플리케이션이 `basic.ack` 메서드 사용하여 응답을 다시 보낸 후



명시적 모델을 사용하면 애플리케이션이 확인을 보낼 시기를 선택합니다.

시기는 메시지를 수신한 직후, 처리하기 전에 메시지를 데이터 저장소에 보존한 후, 또는 메시지를 완전히 처리한 후 (예: 웹 페이지를 성공적으로 가져와서 처리한 후 일부 영구 데이터 저장소에 저장)가 될 수 있습니다.

소비자가 확인을 보내지 않는다면 브로커는 다른 소비자에게 메시지를 재전송하거나, 당시 사용 가능한 소비자가 없는 경우 브로커는 재전송을 시도하기 전에 적어도 한 명의 소비자가 동일한 대기열에 등록될 때까지 기다리게 됩니다.



---

메시지의 성격에 따라 응답 모델을 다르게 설정하면 조금 더 RabbitMQ를 잘 사용할 수 있을 것 같다는 생각이 듭니다.



## Negative Acknowledgements

컨슈머 애플리케이션이 메시지를 수신하면 해당 메시지의 처리가 성공할 수도 있고 실패할 수도 있습니다. 

애플리케이션은 메시지를 거부함으로써 메시지 처리에 실패했거나 현재 처리가 불가능하다는 것을 브로커에게 알릴 수 있습니다.

메시지를 거부할 때 애플리케이션은 브로커에게 메시지를 폐기하거나 대기하도록 요청할 수 있습니다. 

대기열에 한 명의 소비자만 있는 경우 동일한 소비자의 메시지를 거부하고 다시 대기열에 추가하여 무한한 메시지 전달 루프를 만들지 않도록 주의하여야 합니다.



## Prefetching Messages

여러 컨슈머들이 대기열을 공유하는 경우, 각 소비자가 한 번에 보낼 수 있는 메시지 수를 지정할 수 있습니다. 

이는 간단한 부하 분산 기술로 사용하거나 메시지가 일괄적으로 게시되는 경향이 있는 경우 처리량을 개선하는 데 사용할 수 있습니다.