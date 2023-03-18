## Singleton Pattern

인스턴스를 오직 한 개만 제공하는 클래스

+ **시스템 런타임, 환경 세팅에 대한 정보 등**, 인스턴스가 여러 개일 때 문제가 생길 수 있는 경우가 있다.
+ **인스턴스를 오직 한 개만 만들어 제공하여야 할 경우**에 사용한다.

*백엔드 입장에서는 시스템 관련 그리고 공통 환경 세팅 말고는 다른 예시가 많이 없는 것 같다.*

*프런트 입장으로 예시를 소개하면 [애플리케이션 내부에서 항상 한 번에 하나의 음성만 출력되어야 하며 동시에 여러 음성이 겹치면 안 되는 경우](https://injae-kim.github.io/dev/2020/08/06/singleton-pattern-usage.html)가 있다.*

```java
public class Settings {

    private static Settings instance;
    
    private Settings() { }


    public static Settings getInstance() {
        if (instance == null) {
            instance = new Settings();
        }
        return instance;
    }
}

```

위의 Settings 클래스는 가장 기본적인 싱글톤 패턴을 적용한 모습을 나타낸 것이다.

위의 기본 코드의 경우 단일 스레드 환경에서는 문제가 없지만 **멀티 스레드 환경에서는 문제가 생긴다고 한다.**



문제가 생기는 부분은 `getInstance()`라고 한다.

만약 서로 다른 스레드 1,2가 있고 스레드1이 `getInstance()` 실행을 미처 다 마무리하기 이전에 스레드2가 `getInstance()`를 실행한다면 두 실행 결과로 다른 Settings 인스턴스가 만들어진다.

즉, 위의 코드는 **스레드 세이프 하지 않은 코드**인 것이다.



그렇다면 위의 코드를 스레드 세이프하게 고쳐보자.

```java
// #1 synchronized 키워드 사용
public class Settings {

    private static Settings instance;
    
    private Settings() { }


    public static synchronized Settings getInstance() {
        if (instance == null) {
            instance = new Settings();
        }
        return instance;
    }
}

// #2 eager initialization 
public class Settings {

    private static Settings INSTANCE = new Settings();
    
    private Settings() { }


    public static synchronized Settings getInstance() {
        return INSTANCE;
    }
}

// #3 double checked locking
public class Settings {

    private static volatile Settings instance;
    
    private Settings() { }

    public static Settings getInstance() {
        if (instance == null) {
          synchronized (Settings.class) {
            if (instance == null) {
              instance = new Settings();
            }
          }
        }
        return instance;
    }
}

// #4 static inner class
public class Settings {
    
    private Settings() { }
  	
  	private static class SettingsHolder() {
      private static final Settings INSTANCE = new Settings();
    }

    public static synchronized Settings getInstance() {
        return SettingsHolder.INSTANCE;
    }
}
```

### #1 synchronized

#1의 경우 `synchronized` 덕분에 서로 다른 스레드에서 접근하더라도 하나의 스레드만 접근할 수 있다.

하지만 이는 **성능에 문제가 생길 수 있다는 단점이 있다.**



### #2 eager initialization

#2의 경우 eager initialization을 하여 Settings가 만들어질 때 Settings 객체를 만든 것을 확인할 수 있다.

이는 **Settings 객체를 만드는 비용이 많이 들지 않다면 충분히 고려할 만한 방법이다.**



### #3 double checked locking

#3의 경우 double checked locking으로 말 그대로 두 번 확인한다.

이번에도 스레드 1,2가 동시에 `getInstance()`를 실행시킨다고 생각해보자.

먼저 스레드 1이 `getInstance()`를 실행시키고 도중 스레드 2가 `getInstance()`를 실행시키면 스레드 1이 `synchronized (Settings.class)`을 실행 중이기에 스레드 2는 대기하게 된다.

지금까지만 보면 #1과 별다를 게 없어 보인다.

하지만 #1의 경우 `getInstance()`를 실행하는 **모든 경우**에 `synchronized`가 걸리게 되고 #3의 경우 **`instance`가 null인 경우**에만 `synchronized`가 걸린다는 차이가 있다.



### #4 inner static class

#4는 inner static class를 만드는 방법이다.

이는 **Initialization-on-demand holder** 개념을 이용한 것으로 JVM의 static initializer에 의해 초기화되고 메모리로 올라간다.

최초로 ClassLoaderd에 의해 load 될 때 load Class 방법을 통해 올라가게 된다.

이때 내부로 `synchronized`가 실행되기에 스레드 세이프 할 수 있다.



---

### [Initialization-on-demand holder](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom)

**Initialization-on-demand holder란 lazy-loaded singleton이라 할 수 있다.**

*이러한 특징이 Factory Method 패턴과 다른 점이다. **Factory Method 패턴은 외부에서 객체를 요구하면 매번 생성**해서 전달한다.*

이는 우수한 성능으로 정적 필드의 안전하고 동시성이 높은 지연 lazy intialization을 가능하게 한다.

```java
public class Something {
    private Something() {}

    private static class LazyHolder {
        static final Something INSTANCE = new Something();
    }

    public static Something getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

Something 클래스의 초기화는 초기화할 정적 변수가 없으므로 초기화는 간단하게 완료된다.

inner static class인 LazyHolder는 JVM이 LazyHolder를 실행할 때까지 초기화되지 않는다.

inner static class LazyHolder는 정적 메서드 `getInstance()`가 **Something 클래스에서 호출될 때만 실행**되며, 이런 일이 처음 발생할 때 **JVM은 LazyHolder 클래스를 로드하고 초기화한다.**

 LazyHolder 클래스가 초기화되면 외부 클래스 Something의 private 생성자를 실행하여 정적 변수 INSTANCE가 초기화한다.

초기화 단계는 정적 변수 INSTANCE를 순차적으로 쓰기 때문에 이후의 모든 동시 호출은 추가적인 동기화 오버헤드 없이 올바르게 초기화된 동일한 INSTANCE를 반환한다.

---



### 자바와 스프링에서 싱글톤 패턴의 차이

스프링에서는 싱글톤 패턴이 아닌 **싱글톤 스코프를 제공**하는 것이다.

이는 **스프링 컨테이너네**에서 **빈의 인스턴스를 오직 하나만 생성**해서 재사용하는 것을 말한다.

