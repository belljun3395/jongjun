## private 생성자나 열거 타입으로 싱글턴임을 보증하라



싱글턴이라 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

---

싱글턴의 예로는 함수와 같은 **무상태 객체**나 **설계상 유일해야 하는 시스템 컴포넌트**를 들 수 있다.

그런데 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어렵다.

---



### 싱글턴을 만드는 방법

#### public static 멤버가 final 필드

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  
  public void leaveTheBuilding() {...}
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.



#### 정적 팩터리 메서드를 pulic static 멤버로 제공

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  
  public static Elvis getInstance() { return INSTANCE; }
  public void leaveTheBuilding() {...}
}
```

위 방법의 첫 번째 장점은 **API를 바꾸지 않고도 싱글턴이 아니게 변경**할 수 있다는 것이다.

두 번째 장점은 **정적 팩터리를 제네릭 싱글 팩터리**로 만들 수 있다는 점이다.

세 번째 장점은 **정적 팩터리의 메서드 참조를 공급자(supplier)로 사용**할 수 있다는 점이다.



*둘 중 하나의 방식으로 만들 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다.*

*모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다.*

*이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.*



#### 원소가 하나인 열거 타입을 선언

```java
public enum Elvis {
  INSTANCE;
  
  public void leaveTheBuilding() {...}
}
```

대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.



