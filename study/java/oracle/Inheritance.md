## Inheritance

#### Definitions

다른 클래스에서 파생된 클래스를 서브 클래스라고 한다.

서브 클래스로 파생시킨 클래스를 슈퍼 클래스라고 한다.



슈퍼 클래스가 없는 Object를 제외한 모든 클래스가 단 하나의 직접적인 슈퍼 클래스를 갖는다.

다른 명시적 슈퍼 클래스가 없다면 모든 클래스는 암시적으로 Object의 서브 클래스가 된다.



상속은 새 클래스를 만들고 싶은데 원하는 코드 중 일부가 포함된 클래스가 이미 있는 경우 기존 클래스에서 새 클래스를 파생할 수 있다.

이렇게 하면 직접 작성하지 않고도 기존 클래스의 필드와 메서드를 재사용할 수 있다.



**서브 클래스는 슈퍼클래스로부터 모든 멤버(필드, 메서드, 중첩 클래스)를 상속받는다.**

생성자는 멤버가 아니므로 하위 클래스에서 상속되지 않지만, 슈퍼클래스의 생성자는 하위 클래스에서 호출할 수 있다.



#### The Java Platform Class Hierarchy

java.lang 패키지에 정의된 Object 클래스는 사용자가 작성한 클래스를 포함하여 모든 클래스에 공통된 동작을 정의하고 구현한다.

Java 플랫폼에서는 많은 클래스가 Object에서 직접 파생되고, 다른 클래스는 이러한 클래스 중 일부에서 파생되는 등의 방식으로 클래스의 계층 구조를 형성한다.



#### What You Can Do in a Subclass

서브 클래스는 서브 클래스가 **어떤 패키지에 있든 상관없이** **부모의 모든 공개 및 보호 멤버를 상속한다.**

서브 클래스가 부모와 **같은 패키지에 있는 경우** 부모의 패키지 **비공개 멤버도 상속합니다.**

상속된 멤버를 **그대로 사용**하거나, **바꾸거나**, **숨기거나**, **새 멤버로 보완**할 수 있다.



+ 상속된 필드는 다른 필드와 마찬가지로 <u>직접 사용할 수 있다.</u>
+ 하위 클래스에서 슈퍼클래스에 있는 필드와 <u>같은 이름으로 필드를 선언하여 필드를 숨길 수 있다</u> **(권장하지 않음)**.
+ 슈퍼클래스에 없는 <u>새 필드를 서브 클래스에 선언할 수 있다.</u>
+ 상속된 메서드는 <u>그대로 직접 사용할 수 있다.</u>
+ 서브 클래스에, 슈퍼클래스에 있는 것과 동일한 시그니처를 가진 <u>새 인스턴스 메서드를 작성하여 이를 재정의할 수 있다.</u>
+ 서브 클래스에, 슈퍼클래스에 있는 것과 동일한 시그니처를 가진 <u>새로운 정적 메서드를 작성하여 숨길 수 있다.</u>
+ 슈퍼클래스에 없는 <u>새로운 메서드를 서브 클래스에 선언할 수 있다.</u>
+ 암시적으로 또는 키워드 super를 사용하여 슈퍼클래스의 생성자를 호출하는 서브 클래스 생성자를 작성할 수 있다.



#### Private Members in a Superclass

서브 클래스는 **부모 클래스의 비공개 멤버를 상속하지 않는다.**

그러나 슈퍼클래스에 **비공개 필드에 액세스하기 위한 공개 또는 보호 메서드가 있는 경우** 하위 클래스에서도 **사용할 수 있다.**

또, **중첩 클래스(Nested Class)는 둘러싼 클래스의 모든 비공개 멤버에 접근할 수 있다.**

따라서 서브 클래스에 의하여 상속된 공용 또는 보호된 중첩 클래스는 슈퍼클래스의 모든 비공개 멤버에 간접적으로 접근할 수 있다.



#### Casting Objects

우리는 객체가 인스턴스화된 클래스의 데이터 유형이라는 것을 안다.

**캐스팅**은 상속 및 구현에서 허용하는 객체 중 **한 유형의 객체를 다른 유형 대신 사용하는 것**을 말한다.

이러한 캐스팅에는 2가지 종류가 있다.

```java
// #1 암시적 캐스팅
Object obj = new MountainBike();

// #2 명시적 캐스팅
MountainBike myBike = (MountainBike) obj;
```

명시적 캐스팅의 경우 컴파일러가 `obj`가 `MountainBike`라고 안전하게 가정할 수 있도록 객체에 `MountainBike`가 할당되었는지 **런타임 검사**를 삽입한다.

그렇기에 런타임 객체가 `MountainBike`가 아닌 경우 예외를 발생시킨다.

명시적 캐스팅을 조금 더 안전하게 하기 위해서는 아래와 같이 추가적인 검사를 거친 이후에 캐스팅하는 것도 좋다.

```java
if (obj instanceof MountainBike) {
    MountainBike myBike = (MountainBike) obj;
}
```



---

추가로 캐스팅이 된다고 해서 객체에 변화가 생기는 것은 아니다.

다만 우리가 **활용할 수 있는 방법에 변화**가 생긴다.

```java
public class A {
    public String a = "this is A";

    public A() {
    }

    public void show() {
        System.out.println(this.a + " by show()");
    }
}

public class B extends A {
    public String b = "this is B";

    public B() {
    }

    public void show() {
        System.out.println(this.b + " by show()");
    }
}
```

B 인스턴스를 생성하였을 때 우리가 사용할 수 있는 것 다음과 같다.

1. 멤버 변수 b
2. A 클래스의 `show()`를 오버라이드한 `show()`



그렇다면 B 인스턴스를 A로 캐스팅한다면 우리가 사용할 수 있는 것은 어떻게 변할까?

1. ~~멤버 변수 b~~ 멤버변수 a **( "같은 이름으로 필드를 선언하여 필드를 숨길 수 있다"을 권장하지 않았던 이유 )**
2. A 클래스의 `show()`를 오버라이드한 `show()`

2번의 경우 A 클래스의 `show()`를 오버라이드 하며 **재정의**하였기에 **변화가 없다.**

실제로 아래와 같은 결과를 확인할 수 있다.

```java
B b = new B();
b.show();
System.out.println(b.b);
A a = b;
System.out.println("a == b : " + (a == b));
a.show();
System.out.println(a.a);
```

```
this is B by show()
this is B
a == b : true 
this is B by show()
this is A
```

---



### Multiple Inheritance of State, Implementation, and Type

클래스와 인터페이스의 중요한 차이점 중 하나는 **클래스는 필드를 가질 수 있지만 인터페이스는 필드를 가질 수 없다는 것이다.**

또한 클래스를 인스턴스화여 객체를 만들 수 있지만 인터페이스로는 객체를 만들 수 없다. (*객체란 클래스에 정의된 필드에 상태를 저장하기 때문이다.*)

자바 프로그래밍 언어가 **하나 이상의 클래스를 상속하는 것을 허용하지 않는** 한 가지 이유는 여러 클래스에서 필드를 상속하는 **다중 상속 문제를 피하기 위해서이다.**

대신 자바는 클래스가 **하나 이상의 인터페이스를 구현하는** 기능인 **다중 형식 상속을 지원한다.**

**이때 객체는 자체 클래스의 유형과 클래스가 구현하는 모든 인터페이스의 유형 등 여러 유형을 가질 수 있다.**



### Overriding and Hiding Methods

#### Instance Methods

슈퍼클래스의 인스턴스 메서드와 동일한 시그니처 및 반환 형식을 가진 **하위 클래스의 인스턴스 메서 드는 슈퍼클래스의 메서드를 재정의한다.**

서브 클래스가 메서드를 재정의할 수 있는 기능을 사용하면 클래스가 동작이 "충분히 유사한" 슈퍼클래스를 상속한 다음 필요에 따라 동작을 수정할 수 있다.

이때 재정의하는 메서드는 재정의 대상 메서드와 이름, 매개 변수 및 유형 및 반환 유형이 동일하다.

**재정의 메서드는 재정의 메서드가 반환한 타입의 하위 타입을 반환할 수도 있다.**

이를 하위 타입을 **공변 반환 타입**이라 한다.



메서드를 재정의할 때 컴파일러에, 슈퍼클래스에서 방법을 제정의 지시하는 `@Override` 주석을 사용할 수 있다.

컴파일러가 어떤 이유로 메서드가 슈퍼클래스 중 하나에 존재하지 않는 것을 감지하면 오류가 발생한다.



#### Static Methods

서브 클래스가 슈퍼클래스의 정적 메서드와 동일한 시그니처를 가진 정적 메서드를 정의하는 경우, **서브 클래스의 메서드는 슈퍼클래스의 메서드를 숨긴다.**



정적 메서드를 숨기는 것과 인스턴스 메서드를 재정의하는 것은 차이가 있다.

1. 호출되는 재정의된 **인스턴스 메서드**는 **서브 클래스에 있는 버전입니다.**
2. 호출되는 숨겨진 **정적 메서드**는 **슈퍼클래스에서 호출되는지 아니면 서브 클래스에서 호출되는지에 따라 달라진다.**



아래 예시를 살펴 보자.

```java
public class Animal {
    public static void testClassMethod() {
        System.out.println("The static method in Animal");
    }
    public void testInstanceMethod() {
        System.out.println("The instance method in Animal");
    }
}

public class Cat extends Animal {
    public static void testClassMethod() {
        System.out.println("The static method in Cat");
    }
    public void testInstanceMethod() {
        System.out.println("The instance method in Cat");
    }

    public static void main(String[] args) {
        Cat myCat = new Cat();
        Cat.testClassMethod();
        myCat.testInstanceMethod();
        Animal myAnimal = myCat;
        Animal.testClassMethod();
        myAnimal.testInstanceMethod();
    }
}
```

```
The static method in Cat
The instance method in Cat
The static method in Animal
The instance method in Cat
```



#### Interface Methods

인터페이스의 기본 메서드(defalut)와 추상 메서드는 **인스턴스 메서드처럼 상속된다.**

그러나 클래스 또는 인터페이스의 슈퍼 타입이 동일한 시그니처를 가진 여러 기본 메서드를 제공하는 경우 Java 컴파일러는 이름 충돌을 해결하기 위해 **상속 규칙을 따른다.**

##### 상속 규칙

1. 인터페이스 기본 메서드보다 **인스턴스 메서드가 선호된다.**
   ```java
   public class Horse {
       public String identifyMyself() {
           return "I am a horse.";
       }
   }
   
   public interface Flyer {
       default public String identifyMyself() {
           return "I am able to fly.";
       }
   }
   
   public interface Mythical {
       default public String identifyMyself() {
           return "I am a mythical creature.";
       }
   }
   
   public class Pegasus extends Horse implements Flyer, Mythical {
       public static void main(String... args) {
           Pegasus myApp = new Pegasus();
           System.out.println(myApp.identifyMyself()); // "I am a horse."
       }
   
   ```

2. 인터페이스 구현하는 경우

   1. 다른 후보에 의해 **이미 재정의된 메서드는 무시된다.**
      이러한 상황은 슈퍼타입이 **공통 조상을 공유할 때** 발생할 수 있다.

      ````java
      public interface Animal {
          default public String identifyMyself() {
              return "I am an animal.";
          }
      }
      
      public interface EggLayer extends Animal {
          default public String identifyMyself() {
              return "I am able to lay eggs.";
          }
      }
      
      public interface FireBreather extends Animal { }
      
      public class Dragon implements EggLayer, FireBreather {
          public static void main (String... args) {
              Dragon myApp = new Dragon();
              System.out.println(myApp.identifyMyself()); // I am able to lay eggs.
          }
      }
      ````

   2. 두 개 이상의 독립적으로 정의된 기본 메서드가 충돌하거나 기본 메서드가 추상 메서드와 충돌하는 경우 Java 컴파일러는 컴파일러 오류를 생성한다.
      이때는 슈퍼타입 메서드를 **명시적으로 재정의**해야 합니다.
   
      ```java
      public interface OperateCar {
          // ...
          default public int startEngine(EncryptedKey key) {
              // Implementation
          }
      }
      
      public interface FlyCar {
          // ...
          default public int startEngine(EncryptedKey key) {
              // Implementation
          }
      }
      
      public class FlyingCar implements OperateCar, FlyCar {
          // ...
          public int startEngine(EncryptedKey key) { // 둘 모두 사용할 것이라 명시적으로 재정의
              FlyCar.super.startEngine(key);
              OperateCar.super.startEngine(key);
          }
      }
      ```
   



#### Hiding Fields

클래스 내에서 슈퍼클래스의 필드와 **이름이 같은 필드**는 유형이 다르더라도 슈퍼클래스의 필드를 숨긴다.

하위 클래스 내에서 슈퍼클래스의 필드는 **단순한 이름으로 참조할 수 없다.**

대신 `super.슈퍼클래스의필드`와 같이 참조해야 한다.

일반적으로 **필드를 숨기는 것**은 코드를 읽기 어렵게 만들므로 **권장하지 않는다.**



#### Object as a Superclass

java.lang 패키지의 Object 클래스는 클래스 계층 구조 트리의 맨 위에 있다.

모든 클래스는 Object 클래스의 직간접적인 자손이다.

사용하거나 작성하는 모든 클래스는 Object의 인스턴스 메서드를 상속한다. 

이러한 메서드를 사용할 필요는 없지만, 사용하려는 경우 클래스 고유의 코드로 메서드를 재정의해야 할 수 있다. 



이 섹션에서 설명하는 Object에서 상속되는 메서드는 다음과 같다.

- `protected Object clone() throws CloneNotSupportedException`
  이 객체의 복사본을 생성하고 반환한다.
- `public boolean equals(Object obj)`
  다른 객체가 이 객체와 "동일한" 객체인지 여부를 나타낸다.
- `protected void finalize() throws Throwable`
  가비지 컬렉션이 객체에 대한 참조가 더 이상 없다고 판단할 때 객체의 가비지 수집기에 의해 호출된다.
  수집이 객체에 대한 참조가 더 이상 없다고 판단할 때 호출된다.
- `public final Class getClass()`
  객체의 런타임 클래스를 반환한다.
- `public int hashCode()`
  객체의 해시 코드 값을 반환한다.
- `public String toString()`
  객체의 문자열 표현을 반환한다.



#### Writing Final Classes and Methods

메서드가 서브 클래스에 의해 **재정의될 수 없음**을 나타내기 위해 `final` 키워드를 사용할 수 있다.

Object 클래스는 다수의 `final` 메서드를 가지고 있다.

변경해서는 안 되는 구현이 있고 객체의 **일관된 상태**에 중요한 경우 `final` 메서드를 사용할 수 있다.

생성자에서 호출되는 메서드는 일반적으로 `final`로 선언해야 한다.

생성자가 최종 메서드가 아닌 메서드를 호출하면 서브 클래스가 해당 메서드를 재정의하여 예상치 못한 또는 바람직하지 않은 결과를 초래할 수 있다.

**전체 클래스를 최종 클래스로 선언**할 수도 있다.

`final`로 선언된 클래스는 **서브 클래스가 될 수 없다.** 



#### Abstract Methods and Classes

추상 클래스는 `abstract`로 선언된 클래스이다.

추상 클래스는 인스턴스화할 수 없지만 서브 클래스는 만들 수 있다.



추상 메서드는 아직 구현이 되지 않은 메서드이다.

```java
abstract void moveTo(double deltaX, double deltaY);
```

그리고 만약 클래스가 추상 메서드를 포함한다면 해당 클래스는 `abstract`로 선언되어야 한다.



추상 클래스가 상속되면, 서브 클래스는 일반적으로 부모 클래스의 **모든 추상 메서드에 대한 구현을 제공**해야 하며 **그렇지 않은 경우 하위 클래스도 추상적 클래스로 선언**해야 한다.

(**추상 메서드**는 **'메서드의 본체가 완성되어 있지 않은 미완성 메서드'**를 말한다. **상속**을 목적으로 사용하며 자식 클래스에서 **오버라이딩**을 **강제**한다.)



##### Abstract Classes Compared to Interfaces

추상 클래스는 인터페이스와 유사하다. 

인스턴스화할 수 없으며 구현을 포함하거나 포함하지 않고 선언된 메서드가 혼합되어 있을 수 있다. 

그러나 추상 클래스를 사용하면 `static`과 `final`은 사용할 수 없고 `public`, `protected` 그리고 `private`로 구체적인 메서드를 정의할 수 있다.

인터페이스를 사용하면 **모든 필드**가 자동으로 `public`, `static` 그리고 `final`이 되며, 선언하거나 정의하는 **모든 메서드**(기본 메서드로서)가 **공용 메서드**가 된다. 

또한 추상적이든 아니든 하나의 클래스만 확장할 수 있지만, 인터페이스를 구현할 수 있는 개수는 무제한이다.



추상 클래스와 인터페이스 중 어떤 것을 사용해야 할까요?

**추상 클래스 고려 상황**

+ 밀접하게 관련된 여러 클래스 간에 코드를 공유하려는 경우
+ 추상 클래스를 확장하는 클래스에 공통 메서드나 필드가 많거나 `public` 이외의 액세스 수정자(예: `protected` 및 `private`)가 필요할 것으로 예상되는 경우
+ `non-static` 이거나 `non-fial` 필드를 선언하려고 하는 경우
  이렇게 하면 메서드가 속한 객체의 상태에 액세스하고 수정할 수 있는 메서드를 정의할 수 있다.

**인터페이스 고려 상황**

+ 서로 관련 없는 클래스가 인터페이스를 구현할 것으로 예상되는 경우
  예를 들어, Compareable 및 Cloneable 인터페이스는 서로 관련 없는 많은 클래스에 의해 구현된다.
+ 특정 데이터 유형의 동작을 지정하고 싶지만 누가 그 동작을 구현하는지는 신경 쓰지 않으려는 경우
+ 타입의 다중 상속을 활용하고 싶은 경우



JDK에서 추상 클래스의 예로는 컬렉션 프레임워크의 일부인 AbstractMap이 있다. 

이 클래스의 서브 클래스(해시맵, 트리맵, 컨커런트해시맵 포함)는 AbstractMap이 정의하는 많은 메서드(get, put, isEmpty, containsKey, containsValue 등)를 공유한다.



JDK에서 여러 인터페이스를 구현하는 클래스의 예로는 Serializable, Cloneable 및 Map<K, V> 인터페이스를 구현하는 HashMap이 있다. 

이 인터페이스 목록을 보면 (클래스를 구현한 개발자나 회사에 관계없이) HashMap의 인스턴스가 복제가 가능하고, 직렬화 가능(바이트 스트림으로 변환할 수 있음을 의미, 직렬화 가능 객체 섹션 참조)하며, 맵의 기능을 가지고 있다는 것을 유추할 수 있다. 

또한 Map<K, V> 인터페이스는 이 인터페이스를 구현한 이전 클래스가 정의할 필요가 없는 merge 및 forEach와 같은 많은 기본 메서드로 향상된다.



많은 소프트웨어 라이브러리는 추상 클래스와 인터페이스를 모두 사용하며, HashMap 클래스는 여러 인터페이스를 구현하고 추상 클래스인 AbstractMap을 확장하기도 한다.