# Consumer Acknowledgements



RabbitMQ와 같은 메시징 브로커를 사용하는 시스템은 기본적으로 분산되어 있습니다.

전송된 프로토콜 메서드(메시지)가 피어에 도달하거나 피어에서 성공적으로 처리된다는 보장이 없으므로 퍼블리셔와 컨슈머 모두 전달 및 처리 확인을 위한 메커니즘이 필요합니다.

이번 글에서는 [RabbitMQ에서 제공하는 문서](https://www.rabbitmq.com/docs/confirms)를 기반으로 컨슈머의 처리 확인에 대해 알아볼 예정입니다.



## (Consumer) Delivery Acknowledgements

RabbitMQ가 컨슈머에게 메시지를 전달할 때, 언제 메시지를 성공적으로 전송한 것으로 간주해야 하는지 알아야 합니다.

어떤 종류의 로직이 최적인지는 시스템에 따라 다르고 주로 애플리케이션이 결정합니다.

AMQP 0-9-1에서는 `basic.consume` 메서드를 사용하여 컨슈머를 등록하거나 `basic.get` 메서드를 사용하여 요청에 따라 메시지를 가져올 때 이루어집니다.



## Consumer Acknowledgement Modes and Data Safety Considerations

노드가 컨슈머에게 메시지를 전달할 때, 노드는 해당 메시지를 컨슈머가 처리한 것으로 간주할지(또는 최소한 수신한 것으로 간주할지) 결정해야 합니다.

여러 가지 요인(클라이언트 연결, 소비자 앱 등)으로 인해 실패할 수 있으므로 이 결정은 데이터 안전과 관련된 문제로 이어집니다.

메시징 프로토콜은 일반적으로 소비자가 자신이 연결된 노드로의 배달을 확인할 수 있는 확인 메커니즘을 제공합니다.

이 메커니즘의 사용 여부는 컨슈머가 구독(subscribe)할 때 결정됩니다.


```java
// factory에서 설정
SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);

// container에서 설정
SimpleMessageListenerContainer container =
				rabbitListenerContainerFactory.createListenerContainer();
container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
```



### AcknowledgeMode Manual

수동으로 전송된 확인은 긍정 또는 부정일 수 있으며 다음 프로토콜 방법 중 하나를 사용합니다.

- `basic.ack`는 긍정적인 승인에 사용됩니다. 이때 메시지가 완전히 삭제됩니다.
- `basic.nack` 은 부정적 승인에 사용됩니다. 이때 메시지를 다시 대기열에 넣는 것이 가능합니다.
- `basic.reject`은 부정적 승인에 사용됩니다. 이때 메시지가 완전히 삭제됩니다.


```java
// listener
@Override
	public void onMessage(Message message, @Nullable Channel channel) throws Exception {
    // ...
    // channel에서 제공하는 메서드(channel.basicAck, channel.basicNack, channel.basicReject) 중 하나의 메서드를 통해 명시적으로 응답하여야 한다.
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
	}
```



### AcknowledgeMode Auto

자동 확인 모드에서는 메시지가 전송되는 즉시 메시지가 성공적으로 전달된 것으로 간주합니다.

이 모드는 소비자가 따라잡을 수 있는 한 높은 처리량과 배달 및 소비자 처리의 안전성 저하를 맞바꾸는 방식입니다.

이 모드를 흔히 'fire-and-forget' 라고 합니다.



수동 승인 모델과 달리, 성공적으로 전달되기 전에 소비자의 TCP 연결 또는 채널이 닫히면 서버에서 보낸 메시지가 손실됩니다.

따라서 자동 메시지 승인은 안전하지 않으며 모든 워크로드에 적합하지 않은 것으로 간주해야 합니다.


```java
// listener
@Override
public void onMessage(Message message, @Nullable Channel channel) throws Exception {
  // ...
  // 명시적 응답이 없으면 basic.ack로 응답
}
```



자동 승인 모드를 사용할 때 고려해야 할 또 다른 중요한 사항은 소비자 과부하입니다.

수동 승인 모드는 일반적으로 채널에서 미처리(진행 중) 배달 수를 제한하는 제한된 채널 프리페치(prefetch)와 함께 사용됩니다.

그러나 자동 승인 모드에서는 정의상 이러한 제한이 없습니다.

따라서 소비자는 배달 속도에 압도되어 메모리에 백로그가 쌓여 힙이 부족해지거나 OS에 의해 프로세스가 종료될 수 있습니다.

따라서 자동 승인 모드는 배달을 효율적이고 일정한 속도로 처리할 수 있는 소비자에게만 권장됩니다.



## Acknowledging Multiple Deliveries at Once

매번 승인을 처리하는 방식은 네트워크 트래픽의 낭비가 될 수 있습니다.

그렇기에 RabbitMQ에서는 네트워크 트래픽을 줄이기 위한 수동 승인을 일괄 처리를 지원합니다.

이는 승인 방법의 다중(multiple) 필드를 true로 설정하면 됩니다.

`basic.reject`에는 원래 이 필드가 없었기 때문에 RabbitMQ에서 프로토콜 확장으로 `basic.nack`을 도입한 것입니다.

다중 필드를 true로 설정하면 RabbitMQ는 승인에 지정된 태그까지 모든 미결 배달 태그를 승인합니다.

승인과 관련된 다른 모든 항목과 마찬가지로 채널별로 범위가 지정됩니다.

예를 들어 채널에 승인되지 않은 배달 태그 5, 6, 7, 8이 있다고 가정할 때, 해당 채널에 배달 태그가 8로 설정되어 있고 다중 필드가 true로 설정된 승인 프레임이 도착하면 5에서 8까지의 모든 태그가 승인됩니다.

반면 다중 필드가 false로 설정된 경우 배달 5, 6, 7은 여전히 승인되지 않습니다.



- 프로젝트에서의 활용

```java
// Batch listener
@Override
public void onMessageBatch(List<Message> messages, Channel channel) {
  // 배치 처리
  // multiple ack를 통한 마지막에 한번에 ack 처리
  channel.basicAck(messages.get(messages.size() - 1).getMessageProperties().getDeliveryTag(), true);
}
```





## Consumer Acknowledgement Modes, Prefetch and Throughput

승인 모드와 QoS 프리페치 값은 소비자 처리량에 상당한 영향을 미칩니다.

일반적으로 프리페치를 늘리면 소비자에게 메시지가 전달되는 속도가 향상됩니다.

자동 확인 모드는 최상의 전송률을 제공합니다.

그러나 두 경우 모두 배달되었지만, 아직 처리되지 않은 메시지의 수도 증가하여 소비자의 램(RAM) 사용량이 증가합니다.

자동 승인 모드나 무제한 프리페치 기능이 있는 수동 승인 모드는 신중하게 사용해야 합니다.

승인하지 않고 많은 메시지를 소비하는 소비자는 연결된 노드의 메모리 소비를 증가시킵니다.

적절한 프리페치 값을 찾는 것은 시행착오의 문제이며 워크로드에 따라 달라질 수 있습니다.



일반적으로 100~300 범위의 값은 최적의 처리량을 제공하며 소비자에게 과부하를 줄 위험이 크지 않습니다.

이보다 높은 값은 종종 수익 감소의 법칙에 부딪히게 됩니다.



프리페치 값을 늘리지 않고 처리량을 늘리는 또 다른 방법은 멀티 컨슈머를 설정하는 것입니다.

이는 프로듀서의 메시지 발행이 컨슈머의 소비보다 빠를 때 혹은 메시지 하나의 처리가 오래 걸리는 경우에 좋을 수 있습니다.



추가로 프리페치 값을 1로 설정하면 공평한 분배(fair dispatching)를 만들어낼 수 있습니다.

이는 메시지를 처리하고 있다면 컨슈머에게 다시 메시지를 발행하지 말라는 뜻으로 메시지를 처리하고 있지 않은 다른 컨슈머에게 메시지를 전달하게 됩니다.

이러한 이유로 소비자 연결 지연 시간이 긴 환경에서는 처리량이 크게 감소합니다.

그렇기에 많은 애플리케이션에서는 더 높은 값이 적절하고 최적일 수 있습니다.