## Adapter



어댑터 패턴은 클래스의 인터페이스를 **사용자가 기대하는 다른 인터페이스로 변환하는 패턴**으로, **호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.**

*출처: [어댑터 패턴 위키](https://ko.wikipedia.org/wiki/%EC%96%B4%EB%8C%91%ED%84%B0_%ED%8C%A8%ED%84%B4)*



---

***나의 언어로***

기존 코드는 수정하지 않고 **기존 코드를 포함하는 Adapter라는 사용자가 원하는 인터페이스를 구현한 클래스를 통해** 사용자가 원하는 인터페이스를 구현하는 방법을 어댑터 패턴이라 한다.

---



![어댑터 디자인 패턴의 구조 (객체 어댑터)](https://refactoring.guru/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a)

*출처: https://refactoring.guru/ko/design-patterns/adapter*



위의 다이어그램에서 사용자가 기대하는 인터페이스는 `Client Interface`이다.

그리고 수정할 수 없는 어떠한 `Service`라는 클래스는 `Adaptee`다.

이 `Adaptee`는 `Client Interface`를 구현한  `Adapter`에 멤버로 포함되어 사용자가 기대하는 동작을 수행한다.



이러한 어댑터 패턴은 다음과 같은 경우에 적용할 수 있다고 한다.

1. 어댑터 클래스는 기존 클래스를 사용하고 싶지만 그 인터페이스가 나머지 코드가 호환되지 않는 경우

Adapter 클래스가 기존 클래스와 사용자가 원하는 인터페이스 사이의 변환할 수 있다.
