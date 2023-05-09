## FactoryMethod



팩토리 메서드 패턴은 **부모 클래스**에 **알려지지 않은 구체 클래스를 생성하는 패턴**이며, **자식 클래스**가 **어떤 객체를 생성할지를 결정**하도록 하는 패턴이기도 하다.

부모 클래스 코드에 구체 클래스 이름을 감추기 위한 방법으로도 사용한다.

*출처 : [팩토리 메서드 패턴 위키](https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4)*

---

***나의 언어로***

팩토리 메서드 패턴은 **부모 클래스(Creator/Facotry)에서는 알려지지 않은 객체(추상 객체, Product)를 생성**하고 **해야 할 일을 정의**한다.

이를 상속, 구현한 **자식 클래스에서 구체적으로 어떤 객체를 생성할지 결정**하도록 한다.

*Creator은 Factory라고 생각해도 괜찮을 것 같다.*



**팩토리 메서드 패턴에서 부모 클래스는 알려지지 않은 객체를 만들 수 있고 그 객체가 가지고 있는 구체적인 객체의 특성을 가지고 어떠한 일을 할 수 있는지에 관심이 있다고 생각한다.**

---



![structure-2x](https://refactoring.guru/images/patterns/diagrams/factory-method/structure-2x.png)

*출처 : https://refactoring.guru/ko/design-patterns/factory-method*



위의 다이어그램에 설명을 적용하면 부모는 Creator 그리고 자식은 Product가 된다.

다이어그램을 보면 Creator은 Product 인터페이스에 의존하며 구체적으로 자식 클래스가 어떠한지 알지 못한다.



그리고 팩토리 메서드 페턴에서 Creator 부분을 사용하는 코드를 **클라이언트 코드**라고 한다.

이때, 이 클라이언트 코드는 Product의 다양한 자식 클래스들에서 실제로 반환되는 여러 클래스 간의 차이를 알지 못하고 이들을 단지 이들을 Product로 간주한다.

즉, 클라이언트 코드는 객체가 Product이고 `doStuff`를 할 수 있다는 것만 알고 있는 것이다.



이제는 코드를 통해 확인해 보자.

```java
// Creator, Factory
public interface Creator {

    default void someOperation() {
        doSomething();
        Product product = createProduct();
        product.doStuff();
    }

    private void doSomething() {}

    Product createProduct();
}

// Product
public interface Product {

    void doStuff();
}
```

우선 인터페이스부터 살펴보면 위와 같다.

위의 Creator 인터페이스는 `someOperation`이라는 default 메서드를 가지고 있다.

클라이언트 코드는 이 `someOperation`을 통해 팩터리 클래스에서 만든 객체를 사용할 수 있을 것이다.



그럼 `someOperation`을 조금 더 살펴보자.

`someOperation` 내부에서 `creatProduct` 메서드를 사용하여 Product 객체를 생성하는 것을 확인할 수 있다.

그리고 이 product의 구체적인 클래스는 알지 못하지만 Product 인터페이스의 `doStuff`를 활용해 `someOperation`을 수행하고 있는 것을 확인할 수 있다.



위의 인터페이스를 구체화하는 코드를 살펴보기 전에 정리하면 아래와 같을 것이다.

1. **Product를 만드는 Creator이 존재한다.**
2. Creator은 Product가 무엇인지 **구체적으로 알지는 못**하지만 **공통적으로 정의된 동작(`doStuff`)를 통해 특정 동작(`someOperation`)을 수행할 수 있다.**



그럼 위의 인터페이스를 구현한 코드를 확인해 보자.

```java
// ConcreteProductA
public class ConcreteProductA implements Product{

    @Override
    public void doStuff() {}
}

// ConcreteCreatorA, ProductAFactory
public class ConcreteCreatorA implements Creator {

    @Override
    public Product createProduct() {
        return new ConcreteProductA();
    }
}
```

우선 Product를 구현한 ConcreteProductA를 구현하였다.

그리고 이 ConcreateProductA를 생성할 ConcreteCreatorA를 생성하였다.

ConcreateCreatorA는 `createProduct`를 재정의하여 ConcreateProductA를 생성하는 것을 확인할 수 있다.



*그리고 ConcreateCreatorA가 Product 타입의 **멤버로 가지고 있지 않은 것에 주목**하자.*

***Creator, Factory 단어 뜻에 맞게 만들어 주는 것이지 소유하고 있지 않다.***



이러한 팩토리 메서드 패턴은 **다음과 같은 경우에 적용**할 수 있다고 한다.

1. 함께 작동해야 하는 객체들의 정확한 유형들과 의존관계를 모르는 경우

**팩토리 메서드는 제품 생성 코드를 실제로 사용하는 코드와 분리**한다.(`createProduct`)

그렇기에 어떤 제품을 만들 것인지 구체적으로 알지 못하더라도 코드를 작성할 수 있다.



2. 사용자가 **내부 컴포넌트(Product)를 확장하는 방법을 제공**하고 싶은 경우

팩토리 메서드 패턴은 Creator 인터페이스와 Product 인터페이스를 제공한다.

Product 인터페이스를 활용하여 원하는 구체 클래스(ProductA)를 만들고 Creator 인터페이스를 통해 구체 클래스(CreatorA)를 만들어 Product 구체 클래스를 생성하여 반환하는 메서드를 재정의하면 된다.



3. 기존 객체들을 매번 재구축하는 대신 이를 재사용하여 시스템 리소스를 절약하고 싶은 경우

위의 예제를 보면 Creator에 default 메서드로 `someOperation`이라는 메서드가 정의되어 있었다.

**`someOperation`과 같은 메서드에 Product를 활용하여서 할 수 있는 공통 작업을 정의한다면 매번 재구축하는 수고를 덜 수 있다.**

