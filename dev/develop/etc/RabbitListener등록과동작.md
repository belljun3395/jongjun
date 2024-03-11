# RabbitListener 등록과 동작



`RabbitListener`를 구현하기 가장 간단한 방법은 `@RabbitListener`을 public 메서드에 선언 하는 것입니다.

```java
@RabbitListener(bindings = @QueueBinding(
   value = @Queue,
   exchange = @Exchange(value = "exchange"),
   key = "key"
))
public void doSomething(Something something) {}
```

위의 방법은 간단히 `RabbitListener`을 구현 할 수 있다는 장점이 있지만 `RabbitListener`에 대한 설정과 메시지를 처리하는 코드가 한 곳에 모여있다는 점이 "어노테이션 말고 다른 방법으로 선언할 수 없을까?" 하는 생각으로 이어졌던 것 같습니다.

그래서 이번 글에서는 `RabbitListener`를 자바 설정으로 구현하며 `RabbitListener`가 등록되고 동작하는 과정을 공유해보려 합니다.



## SimpleMessageListenerContainer를 통한 등록

`RabbitListener`를 자바 설정으로 구현하는 방법은 해당 `RabbitListener` 포함한 `SimpleMessageListenerContainer` 빈을 선언하는 것입니다.

```java
@Bean
SimpleMessageListenerContainer doSomethingMessageListenerContainer(
 SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory) {
 SimpleMessageListenerContainer container = rabbitListenerContainerFactory.createListenerContainer();
 container.setQueueNames("queue1", "queue2");
 container.setMessageListener(doSomethingListener);
 container.setAcknowledgeMode(AcknowledgeMode.MANUAL); // 응답 모드 설정
 container.setConcurrentConsumers(CONCURRENT_CONSUMERS_COUNT); // 컨슈머 개수 설정
 container.setPrefetchCount(PREFETCH_COUNT); // prefetch 사이즈 설정
 return container;
}
```

위의 `doSomethingListener`를`doSomethingMessageListenerContainer`를 통해 등록하는 코드입니다.

이전의 `@RabbitListener`를 통한 등록과 다르게 `RabbitListener`에 대한 설정과 메시지를 처리하는 코드가 명확히 분리된 것을 확인할 수 있습니다.

이는 `RabbitListener`에 대한 설정이 많아질수록 더 강점을 가질 수 있을 것으로 생각합니다.



## RabbitListener 등록 후 동작 과정

위의 방법처럼 자바 구성으로 `RabbitListener`를 선언하든 어노테이션을 통해 선언하든 `RabbitListener`는 `MessageListenerContainer`에 속하여 관리됩니다.

이때 `SimpleMessageListenerContainer`는 아래와 같은 확장 관계를 맺습니다.

```java
// SimpleMessageListenerContainer
public class SimpleMessageListenerContainer extends AbstractMessageListenerContainer

// AbstractMessageListenerContainer
public abstract class AbstractMessageListenerContainer extends RabbitAccessor
 implements MessageListenerContainer, ApplicationContextAware, BeanNameAware, DisposableBean,
 ApplicationEventPublisherAware

// MessageListenerContainer
public interface MessageListenerContainer extends SmartLifecycle, InitializingBean
```

이 중 `SmartLifecycle`를 통해 `SimpleMessageListenerContainer`가 `RabbitListener`를 애플리케이션이 실행된 이후 동작하도록 합니다.

`SmartLifecycle`는 빈이 생성된 이후 `LifecycleProcessor`들에 의해 수행될 시작(start)과 정지(stop) 동작을 정의합니다.

`AbstractMessageListenerContainer`에서 구현한 start가 동작할 때 `SimpleMessageListenerContainer`의 run이 수행됩니다.

이에 `SimpleMessageListenerContainer`에 속한 `RabbitListener`는 별도의 스레드에서 브로커에서 애플리케이션으로 전송하는 메시지를 받기 위해 대기하는 mainLoop를 실행하며 동작하게 됩니다.



---

**디버깅 포인트**

- `SpringApplication#refreshContext`: prepareContext 이후 실행되며 Context를 갱신합니다.
- `AbstractApplicationContext#finishRefresh`의 `getLifecycleProcessor().onRefresh()`: LifecycleProcessor를 통해 Lifecycle를 구현한 빈을 갱신합니다.
- `DefaultLifecycleProcessor#startBeans`: Lifecycle를 구현한 빈을 갱신합니다.
  - Lifecycle의 phase에 따라 LifecycleGroup 구분합니다.
  - LifecycleGroup에 속한 Lifecycle 빈을 실행합니다.

- `AbstractMessageListenerContainer#start`: Lifecycle를 구현한 AbstractMessageListenerContainer가 실행되며 이를 통해 SimpleMessageListenerContainer의 run이 수행됩니다.
- `SimpleMessageListenerContainer#run`: mainLoop가 실행되며 RabbitListener는 브로커에서 애플리케이션으로 전달할 메시지를 대기합니다.



