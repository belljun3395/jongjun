# 마이크로서비스 아키텍처 정의



## 시스템 작업 식별

애플리케이션 아키텍처를 정의하는 첫 단추는 시스템 작업을 정의하는 일이다.

그 출발점은 사용자 스토리와 이와 연관된 사용자 시나리오등의 애플리케이션 요건이다.



이는 2단계 프로세스로 시스템 작업을 정의할 수 있다.

1단계는 시스템 작업을 기술하기 위해 필요한 보케블러리를 제공하는 핵심 클래스로 구성된 고수준의 도메인 모델을 생성하는 것이다.

2단계는 시스템 작업 식별 후 그 동작을 도메인 모델 관점에서 기술하는 것이다.



도메인 모델은 주로 사용자 스토리의 **명사**에서 도출한다.

이벤트 스토밍이라는 기법을 사용해도 된다.



시스템 작업은 주로 **동사**에서 도출하며, 각각 하나 이상의 도메인 객체와 그들간의 관계로 기술한다.

시스템 작업은 도메인 모델을 생성, 수정, 삭제하거나 모델간 관계를 맺고 끊을 수 있다.



### 고수준 도메인 모델 생성

시스템 작업을 정의하려면 우선 고수준 애플리케이션 도메인 모델을 대략 그려 본다.

물론 최종적으로 구현할 모델보다는 훨씬 단순한 모델이다.

각 서비스 자체는 도메인 모델을 소유하기 때문에 도메인 모델이 하나뿐인 애플리케이션은 없다.

고수준 도메인 모델은 시스템 작업의 동작을 기술하는 데 필요한 보케블러리를 정의하기 때문에 이 단계에서는 매우 유용하다.

도메인 모델은 스토리에 포함된 **명사를 분석**하고 도메인 전문가와 상담하는 등 표준 기법을 활용하여 생성한다.



예를 들어 주문하기 스토리는 다음과 같이 다양한 사용자 **시나리오**로 확장시킬 수 있다.

```java
Given
소비자가 있다.
음식점이 있다.
음식점은 소비자의 주소를 제시간에 음식을 배달할 수 있다.
주문 총액이 음식점의 최소 주문량 조건에 부합한다.

When
소비자가 음식점에 음식을 주문한다.

Then
소비자 신용카드가 승인된다.
주문이 PENDNG_ACCEPTANCE 상태로 생성된다.
생성된 주문이 소비자와 연관된다.
생성된 주문이 음식점과 연관된다.
```

이 사용자 시나리오에 포함된 명사를 보면 Consumer, Order, Restaurant, CreditCard 등 다양한 클래스가 필요할 것 같다.



마찬가지로 주문 접수 스토리는 다음 시나리오로 확장할 수 있다.

```java
Given
현재 주문은 PENDING_ACCEPTANCE 상태다.
주문 배달 가능한 배달원이 있다.

When
주문을 접수한 음식점은 언제까지 음식을 준비할 수 있다고 약속한다.

Then
주문 상태가 ACCEPTED로 변경된다.
주문의 promiseByTime 값을 음식점이 준비하기로 약속한 시간으로 업데이트한다.
주문을 배달할 배달원을 배정한다.
```

시나리오를 보면 Courier, Delivery 클래스가 필요할 것 같다.

이러한 분석을 몇 차례 거듭하면 다음과 같은 핵심 클래스로 구성된 도메인 모델이 완성된다.

<img width="547" alt="마이크로서비스_아키텍처_정의_1" src="https://user-images.githubusercontent.com/102807742/210472452-75251ccf-11db-4e58-a300-bb568b7d2ffa.png">

<img width="479" alt="마이크로서비스_아키텍처_정의_2" src="https://user-images.githubusercontent.com/102807742/210472486-f3197f6c-243c-412d-827c-7d9add905e14.png">



### 시스템 작업 정의

애플리케이션이 어떤 요청을 처리할지 식별하는 단계다.

시스템 작업은 크게 다음 두 종류로 나뉜다.

- 커맨드 : 데이터 생성, 수정, 삭제
- 쿼리 : 데이터 읽기

커멘드를 식별하려면 사용자 스토리/시나리오에 포함된 **동사**를 먼저 분석한다.

예를 들어 주문하기 스토리에서는 주문 생성작업이 필요하다.

다른 스토리도 시스템 커멘드와 직접 매핑된다.

<img width="527" alt="마이크로서비스_아키텍처_정의_3" src="https://user-images.githubusercontent.com/102807742/210472503-08db5686-7c61-4ac5-80e4-0cbec4dfb1c0.png">

커맨드는 매개변수, 반환값, 동작 방식의 도메인 모델 클래스로 정의한다.

이 명세는 작업 호출 시 충족되어야 할 선행조건, 작업 호출 후 충족되어야 할 후행 조건으로 구성된다.

가령 createOrder() 시스템 작업의 명세는 다음과 같이 정의된다.

| 작업      | createOrder(소비자ID, 결제수단, 배달주소, 배달 시각, 음식점 ID, 주문품목) |
| --------- | ------------------------------------------------------------ |
| 반환값    | orderId, …                                                   |
| 선행 조건 | 소비자가 존재하고 주문을 할 수 있다. / 주문 품목은 음식점의 메뉴 항목에 들어 있다. / 배달 주소, 시각은 음식점에서 서비스 할 수 있다. |
| 후행 조건 | 소비자 신용카드는 주문 금액만큼 승인처리 되었다. / 주문 PENDING_ACCEPTANCE 상태로 생성되었다. |

선행 조건은 주문하기 시나리오 전제를, 후행 조건은 주문하기 시나리오의 결과를 나타낸다.



---

#### NOTE : 2022.12.05

우리가 프로젝트를 진행할 때 대략적인 기획이 진행되면 요구 사항 분석을 진행한다.

기존에 요구 사항 분석을 할 때는 사실 "어떻게 분석을 해야하는 거지?" 하는 생각이 있었다.

물론 사용자의 행동을 생각하며 분석을 하였지만 "그래서 이것을 내가 비지니스 로직을 작성할 때 어떻게 사용해야하지?" 하는 생각이 많았고 그래서 요구사항을 분석을 하여도 그것을 잘 활용하지 못한 것 같다.



하지만 위의 예시 그리고 주변의 피드백을 통해 이제 깨달은 바를 정리하면 다음과 같다.

1. 우선 행동 단위 요구 사항 분석을 진행한다.
   Ex) 00이 00을 할 수 있다.
2. 위의 분석을 기준으로 하여 시나리오를 작성한다.
   Ex) Given / When / Then -> 사전 조건 / 요구 사항 / 그 결과
3. 시나리오를 기준으로 클래스와 모델를 파악한다.
4. 시나리오를 기준으로 커멘드를 파악한다.
   이때 커멘드는 위의 예시와 같이 파라미터와 함께 작성하고 전후 조건도 함께 명시한다