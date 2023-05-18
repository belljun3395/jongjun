## Bridge



브리지 패턴이란 구현부에서 **추상층을 분리하여 각자 독립적으로 변형할 수 있게 하는 패턴이다.**

*출처 [브리지 패턴 위키](https://ko.wikipedia.org/wiki/%EB%B8%8C%EB%A6%AC%EC%A7%80_%ED%8C%A8%ED%84%B4)*



![구조](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/Bridge_UML_class_diagram.svg/750px-Bridge_UML_class_diagram.svg.png)



위의 다이어그램을 보면 이전의 어댑터 패턴과 유사한 것을 확인할 수 있다.

*어댑터 패턴과 다른 점은 Implementor 부분으로 어댑터 패턴에서는 제어할 수 없는 서비스였다면 브리지 패턴에서는 추상화되어 있는 부분으로 확장에 열려있는 것이 특징이다.*



브리지 패턴은 **클래스 계층 구조가 너무 크게 확장되는 문제를 해결**해 준다.

브리지 패턴은 **상속에서 객체 합성으로 전환하여 이 문제를 해결**하려 시도한다.

이는 차원 중 하나를 별도의 클래스 계층 구조로 추출하여 원래 클래스들이 한 클래스 내에서 모든 상태와 행동들을 갖는 대신 새 계층구조의 객체를 참조하도록 하는 것이다.



그림으로 알아보면

![브리지 패턴 문제](https://refactoring.guru/images/patterns/diagrams/bridge/problem-ko.png?id=d6a269defe845caae2181e1568256f16)

위와 같았던 계층구조를

아래와 같이 분리하여 계층 구조가 너무 크게 확장되는 문제를 해결하려 시도한다.



![브리지 패턴이 제시하는 해결책](https://refactoring.guru/images/patterns/diagrams/bridge/solution-ko.png?id=5e4d726b4474819783501d71f7ce96f9)



코드를 통해 알아보자.

```java
public abstract class DefaultShape {

    private Color color;

    public DefaultShape(Color color) {
        this.color = color;
    }

    public String getShapeProps() {
        return "this shape has " + color.getClass().toString() + "color";
    }
}

public interface Color {

}
```

우선 위와 같이 모양과 색에 관한 인터페이스를 정의한다.



그리고 이 인터페이스들을 아래와 같이 구현한다.

```java
public class Circle extends DefaultShape {

    public Circle(Color color) {
        super(color);
    }
}

public class Rectangle extends DefaultShape {

    public Circle(Color color) {
        super(color);
    }
}

public class Red implements Color {

}

public class Blue implements Color {

}
```

이렇게 구현한 이후에는 필요에 따라 모양과 색을 조합하여 객체를 생성할 수 있다.

```java
new Circle(new Red()); // 빨간 원
new Circle(new Blue()); // 파란 원

new Rectangle(new Red()); // 빨간 사각형
new Rectangle(new Blue()); // 파란 사각형
```



이러한 브리지 패턴은 다음과 같은 경우에 적용할 수 있다고 한다.

1. **어떤 기능의 여러 변형을 가진 모놀리식 클래스를 나누고 정돈하려 할 때**

브리지 패턴을 사용하면 모놀리식 클래스를 여러 클래스 계층구조로 나눌 수 있다.

그런 다음 각 계층구조의 클래스들을 다른 계층구조들에 있는 클래스들과는 독립적으로 변경할 수 있다.

이 접근 방식은 코드의 유지관리를 단순화하고 기존 코드가 손상될 위험을 최소화한다.



2. **여러 독립 차원에서 클래스를 확장해야 할 때**

원래 클래스는 모든 작업을 자체적으로 수행하는 대신 추출된 계층구조들에 속한 객체들에 관련 작업을 위임한다.



3. 런타임에 구현을 전화할 수 있어야 할 때

브리지 패턴을 사용하면 추상화 내부 구현 객체를 바꿀 수 있으며, 그렇게 하려면 필드에 새 값을 할당하기만 하면 된다.

