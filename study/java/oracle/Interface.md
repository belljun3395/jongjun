## Interface



소프트웨어 엔지니어링에서는 서로 다른 프로그래머 그룹이 소프트웨어의 상호 작용 방식을 명시하는 '계약'에 동의하는 것이 중요한 여러 상황이 있다.

이때 각 그룹은 다른 그룹의 코드가 어떻게 작성되었는지에 대한 지식 없이도 자신의 코드를 작성할 수 있어야 한다.

일반적으로 인터페이스는 이러한 계약을 말한다.



### Interface in Java

Java 프로그래밍 언어에서 인터페이스는 클래스와 유사한 참조 타입으로 상수, 메서드 시그니처, 디폴트 메서드, 정적 메서드 및 중첩된 타입을 포함할 수 있다.

이때 메서드 바디는 디폴트 메서드와 정적 메서드에만 존재한다.

인터페이스는 인스턴스화할 수 없으며 클래스에 의해서만 구현되거나 다른 인터페이스로 확장될 수 있다.



### Defining an Inteface

인터페이스 선언은 수정자, 인터페이스 키워드, 인터페이스 이름, 쉼표로 구분된 상위 인터페이스 목록(있는 경우) 및 인터페이스 바디로 구성된다.

공용 액세스 지정자는 모든 패키지의 모든 클래스에서 인터페이스를 사용할 수 있음을 나타낸다.

인터페이스가 공용임을 지정하지 않으면 인터페이스와 동일한 패키지에 정의된 클래스만 인터페이스에 액세스할 수 있다.

인터페이스는 클래스 서브클래스처럼 다른 인터페이스를 확장하거나 다른 클래스를 확장할 수 있다. 

그러나 클래스는 다른 클래스를 하나만 확장할 수 있지만, 인터페이스는 인터페이스를 얼마든지 확장할 수 있다. 

인터페이스 선언에는 확장하는 모든 인터페이스의 쉼표로 구분된 목록이 포함된다.



#### Interface Body

인터페이스 바디에는 추상 메서드, 디폴트 메서드, 정적 메서드가 포함될 수 있다. 

인터페이스 내의 추상 메서드 뒤에는 세미콜론이 오지만 중괄호는 없다(추상 메서드에는 구현이 포함되지 않음). 

디폴트 메서드는 `default` 수정자로, 정적 메서드는 정적 키워드로 정의된다. 

인터페이스의 모든 추상, 디폴트 및 정적 메서드는 암시적으로 공용이므로 공용 수정자를 생략할 수 있다.

또한 인터페이스에는 상수 선언이 포함될 수 있다. 

인터페이스에 정의된 모든 상숫값은 암시적으로 공용, 정적, 최종적(final)이다. 



### Implementing an Interface

인터페이스를 구현하는 클래스를 선언하려면 클래스 선언에 구현을 포함하면 된다. 

클래스는 둘 이상의 인터페이스를 구현할 수 있으므로 구현 키워드 뒤에는 클래스가 구현하는 인터페이스의 쉼표로 구분된 목록이 이어진다. 

관례에 따라 구현 절은 extends 절(있는 경우)이 있는 경우 그 뒤에 온다.



### Using an Interface as a Type

새로운 인터페이스를 정의하는 것은 새로운 타입의 데이터를 정의하는 것이다.

데이터 타입을 사용할 수 있는 곳이면 어디에서나 인터페이스를 사용할 수 있다.

그리고 만약 유형이 인터페이스인 참조 변수를 정의하는 경우, 여기에 할당하는 모든 객체는 인터페이스를 구현하는 클래스의 인스턴스여야 한다.



### Evolving Interfaces

```java
public interface DoIt {
   void doSomething(int i, double x);
   int doSomethingElse(String s);
}
```

위의 DoIt과 같은 인터페이스를 선언하였다고 가정해 보자.

그런데 시간이 흐른 후, DoIt에 **새로운 메서드를 추가하고 싶다면 어떻게 해야 할까?**

가장 먼저 떠오르는 생각은 DoIt에 아래처럼 새로운 메서드를 추가하는 것이다.

```java
public interface DoIt {
   void doSomething(int i, double x);
   int doSomethingElse(String s);
   boolean didItWork(int i, double x, String s); // 새로운 메서드
}
```

하지만 이러한 변경으로 인해 이전 DoIt 인터페이스를 구현하는 모든 클래스는 새로 추가된 메서드 때문에 더 이상 인터페이스를 구현하지 않은 상태가 된다.

그렇기에 인터페이스의 경우 모든 용도를 예상하고 처음부터 완벽히 선언하는 것이 중요하다.



하지만 **이미 선언된 인터페이스에 메서드를 추가하는 몇 가지 방법**이 존재한다.



우선 **인터페이스를 확장하는 방법**이다.

위의 코드처럼 새로운 메서드를 DoIt에 추가하는 것이 아닌 DoIt을 확장한 DoItPlus를 만들어 그곳에 새로운 메서드를 추가하는 것이다.

```java
public interface DoItPlus extends DoIt {
   boolean didItWork(int i, double x, String s);  
}
```

이렇게 확장을 하여 새로운 메서드를 추가하면 **코드 사용자는 이전 인터페이스를 계속 사용할지 아니면 새 인터페이스로 업그레이드할지 선택할 수 있게 된다.**



다른 방법으로는 **디폴트 메서드를 사용하는 방법**이 있다.

```java
public interface DoIt {
   void doSomething(int i, double x);
   int doSomethingElse(String s);
   default boolean didItWork(int i, double x, String s) {
       // Method body 
   }  
}
```

이때 디폴트 메서드의 경우 구현을 제공해야 한다.

새로운 디폴트 메서드로 추가 메서드를 코드 사용자는 추가 메서드를 수용하기 위해 클래스를 수정하거나 다시 컴파일할 필요가 없다.

***(정적 메서드를 추가하는 방법도 있지만 이는 코드 사용자가 필수 핵심 메서드가 아닌 유틸리티 메서드로 간주할 가능성이 크다.)***



### Default Methods

디폴트 메서드를 사용하면 라이브러리 인터페이스에 새로운 기능을 추가하고 해당 인터페이스의 이전 버전용으로 작성된 코드와의 바이너리 호환성을 보장할 수 있다.



```java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
}
```

위와 같은 TimeClient 인터페이스가 선언되어 있고 `ZonedDateTime getZonedDateTime(String zoneString);`와 같은 메서드가 추가되어야 한다고 생각해 보자.



이는 디폴트 메서드를 활용하여 아래와 같이 추가할 수 있다.

```java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
    
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
        
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}
```

이렇게 디폴트 메서드를 통해 메서드를 추가하면 **구현 클래스를 수정할 필요가 없다.**

**이미 디폴트 메서드가 정의되어 있을 것이다.**



#### Extending Interfaces That Contain Default Methods

디폴트 메서드가 포함된 인터페이스를 확장할 때 아래의 경우가 있을 수 있다.

1. 디폴트 메서드를 전혀 언급하지 않음 : 확장된 인터페이스가 디폴트 메서드를 상속할 수 있다.
2. 디폴트 메서드를 추상화한다.
3. 디폴트 메서드를 오버라이드하여 디폴트 메서드를 재정의한다.

이를 위의 TimeClient 인터페이스를 확장하며 하나씩 살펴보자.



##### 디폴트 메서드를 전혀 언급하지 않음

```java
public interface AnotherTimeClient extends TimeClient { }
```

AnotherTimeClient 인터페이스를 구현하는 모든 클래스는 디폴트 메서드 TimeClient.getZonedDateTime으로 지정된 구현을 갖는다.



##### 디폴트 메서드를 추상화 한다.

```java
public interface AbstractZoneTimeClient extends TimeClient {
    public ZonedDateTime getZonedDateTime(String zoneString);
}
```

하지만 위와 같이 TimeClient 인터페이스를 확장한다면 getZonedDateTime가 다른 인터페이스 메서드와 마찬가지로 메서드가 추상화 된다.

즉, AbstractZoneTimeClient 인터페이스를 구현하는 모든 클래스는 getZonedDateTime를 구현해야 하는 것이다.



##### 디폴트 메서드를 오버라이드하여 디폴트 메서드를 재정의 한다.

```java
public interface HandleInvalidTimeZoneClient extends TimeClient {
    default public ZonedDateTime getZonedDateTime(String zoneString) {
        try {
            return ZonedDateTime.of(getLocalDateTime(),ZoneId.of(zoneString)); 
        } catch (DateTimeException e) {
            System.err.println("Invalid zone ID: " + zoneString +
                "; using the default time zone instead.");
            return ZonedDateTime.of(getLocalDateTime(),ZoneId.systemDefault());
        }
    }
}
```

위와 같은 경우는 HandleInvalidTimeZoneClient 인터페이스가 TimeClient의 디폴트 메서드를 재정의한 것이다.



### Static Methods

기본 메서드 외에도 인터페이스에 정적 메서드를 정의할 수 있다. 

이렇게 하면 라이브러리에서 헬퍼 메서드를 더 쉽게 구성할 수 있으며, 인터페이스에 특정한 정적 메서드를 별도의 클래스가 아닌 동일한 인터페이스에 보관할 수 있다. 



### Summary of Interfaces

인터페이스 선언에는 메서드 서명, 디폴트 메서드, 정적 메서드 및 상수 정의가 포함될 수 있다. 

구현이 있는 메서드는 디폴트 메서드와 정적 메서드뿐이다.

인터페이스를 구현하는 클래스는 인터페이스에 선언된 모든 메서드를 구현해야 한다.

인터페이스 이름은 유형을 사용할 수 있는 모든 곳에서 사용할 수 있다.