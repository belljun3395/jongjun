## Decorator



데코레이터 패턴이란 주어진 상황 및 용도에 따라 **어떤 객체에 책임을 덧붙이는 패턴**으로, 기능 확장이 필요할 때 **서브클래싱 대신 쓸 수 있는 유연한 대안**이 될 수도 있다.

*출처: [데코레이터 패턴 위키](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0_%ED%8C%A8%ED%84%B4)*

그리고 이는 래퍼(wrapper)라고 불리기도 한다.



![데코레이터 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0)

*출처: https://refactoring.guru/ko/design-patterns/decorator*



그런데 위의 다이어그램을 보면 **Adapter 패턴의 다이어그램과 유사**한 것을 확인할 수 있다.

구체적으로 말하면

+ Component : Client Interface
+ Base Decorator : Adapter
+ Concrete Component : Service

와 같이 대응된다.

둘의 차이는 `Concrete Component : Service`에서 나타나는데

**Concrete Component는 클라이언트가 원하는 인터페이스를 상속한 클래스이고 Service는 클라이언트가 원하는 인터페이스가 아니라는 것이다.**



이러한 데코레이터 패턴은 다음과 같은 경우에 적용할 수 있다고 한다.

1. 객체들을 사용하는 코드를 훼손하지 않으면서 **런타임에 추가 행동들을 객체들에 할당할 수 있어야 하는 경우**

데코레이터는 비즈니스 로직 계층을 구성하고, 각 계층에 데코레이터를 생성하고 런타임에 이 로직의 다양한 조합들로 객체를 구성할 수 있도록 한다.

이러한 모든 객체가 공통 인터페이스를 따르기 때문에 **클라이언트 코드는 해당 모든 객체를 같은 방식으로 다룰 수 있다.**



2. **상속을 사용하여 객체의 행동을 확장하는 것이 어색하거나 불가능한 경우**

클래스의 추가 확장을 방지하는 데 사용할 수 있는 final 키워드가 있다.

Final 클래스의 경우 기존 행동들을 재사용할 수 있는 유일한 방법은 데코레이터 패턴을 사용하여 클래스를 자체 래퍼로 래핑하는 것이다.

