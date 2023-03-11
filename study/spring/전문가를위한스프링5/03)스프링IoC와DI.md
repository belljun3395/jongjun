## 스프링 IoC와 DI 소개

### IOC의 종류

+ 의존성 룩업 
  + 컴포넌트 스스로 의존성의 참조를 가져온다.
  + 컨테이너에 의해 정의된 클래스와 인터페이스에 항상 의존한다.
+ 의존성 주입 
  + IoC 컨테이너가 컴포넌트에 의존성을 주입한다.
  + 협력 객체를 의존 객체에게 제공하는 IoC 컨테이너와 완전히 분리되어 자유롭다.



#### 생성자 의존성 주입

```java
public class ConstructorInjection {
  private Dependency dependency;
  
  public ConstructorInjection(Dependency dependcency){
    this.dependency = dependency;
  }
}
```

컴포넌트의 생성자를 이용해서 해당 컴포넌트가 필요로하는 의존성을 제공하는 방식

이때 의존성 주입 없이는 빈을 생성할 수 없으므로 반드시 의존성을 주입해야 한다.

컨테이너가 의존성 점검 메커니즘을 제공하는지와 상관없이 의존성에 대한 요구사항을 지정할 수 있다.

또한, 생성자 주입을 사용하면 빈 객체를 불변 객체로 사용할 수 있다.



#### 수정자 의존성 주입

```java
public class SetterInjection {
  private Dependency dependency;
  
  public void setDependency(Dependency dependency){
    this.dependency = dependency;
  }
}
```

수정자 주입을 사용할 때는 의존성 없이도 객체를 생성할 수 있으며 해당 수정자를 호출해 의존성을 나중에 제공할 수 있다.

또 장점으로 인터페이스에서 모든 의존성을 선언할 수 있다.

하지만 이는 인터페이스의 모든 구현체들이 특정한 의존성을 필요로 한다고 확실할 수 없다면, 구현 클래스 각각이 자신의 의존성을 각자 정의해야한다.

아래의 비지니스 인터페이스와 그 구현체를 살펴보자.

```java
public interface Oracle {
  String defineMeaningOfLife();
}

public class BookormOracle implements Oracle {
  private Encyclopedia encyclopedia;
  
  public void setEncyclopedia(Encyclopedia encyclopedia){
    this.encyclopedia = encyclopedia;
  }
  
  @Override
  public String defineMeaningOfLife() {
    return "Encyclopedias are a waste of money";
  }
}
```

비즈니스 인터페이스에 의존성 주입을 위한 수정자는 항상 정의하는 것은 아니지만, 구성인자를 정의하는 수정자와 접근자를 두는 것은 좋은 생각이다.

이때 **구성인자**는 상세 구성 정보를 담고 있는 것으로 다음의 특징을 지닌다.

+ 수동적이다. 직접적으로 동작을 수행하는데 사용되지 않는다.
+ 그 자체로 정보다.
+ 단순한 값이거나 단순한 값들의 컬렉션이다.

아래는 구성인자를 포함한 비즈니스 인터페이스이다.

```java
public interface NewsletterSender {
  void setSmtpServer(String smtpServer);
  String getSmtpServer();
  ...
    
  void send();
}
```



### 스프링의 의존성 주입

#### 빈(Bean)과 빈 팩터리(BeanFactory)

BeanFactory는 컴포넌트의 라이프사이클뿐만 아니라 의존성까지 관리하는 인터페이스이다.

이때 **Bean**은 <u>컨테이너가 관리하는 모든 컴포넌트를 말한다.</u>

Bean Factory 구성의 경우 프로그래밍으로 할 수도 있지만, 구성 파일을 사용해 외부에서 구성하는 방법이 더 일반적이다.



### 애플리케이션 컨텍스트 구성하기

+ XML 
  + 모든 구성을 자바 코드에서 분리해 외부에서 관리할 수 있다.
+ 애너테이션
  + 코드 내에서 DI 구성을 정의하고 확인할 수 있다.



#### 스프링 컴포넌트

스프링에게 이 빈이 다른 빈에 주입될 수 있다는 것을 알려주고, 스프링이 이 빈들을 관리하게 한다.

##### 자바구성 사용하기

@Configuration 애너테이션을 적용하며, 구성 클래스 내에는 스프링 IoC 컨테이너가 빈 인스턴스를 만들 때 직접 호출하는 @Bean 애너테이션이 적용된 메서드가 포함돼 있다.

```java
@Configuration
public class HelloWorldConfiguration {
  @Bean
  public MessageProvider provider(){
    return new HellowWorldMessageProvider();
  }
  
  @Bean
  public MessageRenderer renderer(){
    MessageRenderer renderer = new StandardOurMessageRenderer();
    renderer.setMessageProvider(provider());
    return renderer;
  }
}
```



##### 수정자 주입 사용하기

```java
@Service
public class StandarOutMessageRenderer implements MessageRenderer {
  ...
  @Override
  @Autowired
  public void setMessageProvider(MessageProvider provider) {
    this.messageProvider = provider;
  }
}
```

수정자 주입의 경우 위의 코드와 같이 setter에 @Autowired만 선언해 주면 된다.



##### 생성자 주입 사용하기

```java
@Service
public class ConfigurationMessageProvider implements MessageProvider {
  private String message;
  
  @Autowired
  puvlic ConfigurableMessageProvider(@Value("Configurable message") String message) {
    this.message = message;
  }
}
```

생성자 주입 역시 수정자 주입과 같이 생성자 메서드에 @Autowired만 선언해주면 된다.

그리고 @Value를 통해 생성자에 주입할 값을 정의할 수 있다.

만약 message 빈을 정의한다면 ConfigurableMessageProvider 클래스의 생성자에 지정된 인자의 이름과 동일하게 선언되었기 때문에, 스프링이 @Autowired 애너테이션을 감지하여 값을 생성자 메서드에 주입해준다.

이러한 경우에는 @Value를 제거하여도 괜찮다.



##### 필드 주입 사용하기

이는 클래스 맴버 변수에 Autowired 애너테이션을 적용하여 필드에 직접 주입한다.

필드 주입을 사용하면 개발자가 빈 초기 생성 시 의존성 주입에만 사용되는 코드를 작성하지 않아도 되므로 실용적이다.

```java
@Service
public class Singer {
  @Autowired
  private Inspiration inspirationBean;
  public void sing() {
    System.out.println("...." + inspirationBean.getLyric());
  }
}
```

inspirationBean 필드가 private인 것을 볼 수 있는데 이는 스프링 컨테이너가 **리플렉션**을 이용해 필요한 의존성을 주입하기에 문제되지 않는다.

// todo 리플렉션 공부하기

하지만 이런 필드 주입은 다음과 같은 단점으로 인해 권장되지 않는다.

+ 의존성을 추가하기 쉬워 단일 책임 원칙을 위반하기 쉬워진다.
+ 어떤 타입의 의존성이 실제로 필요한지, 의존성이 필수인지 여부가 명확하지 않을 수 있다.
+ final 키워드를 사용할 수 없다. 이는 오직 생성자 주입만에서만 이용할 수 있다.
+ 의존성을 수동으로 주입해주어야 하므로 테스트코드 작성하기가 어렵다.



##### 단순값 주입하기

단순값의 경우 @Value를 통해 주입할 수 있다.



##### SpEL을 사용해 값 주입하기

SpEL을 사용하면 그 결과를 **동적**으로 ApplicationContext 내에서 사용할 수 있다.

```java
@Service
public class InjectSimpleSpel {
  @Value("#{injectSimpleConfig.age + 1}")
  private int age;
}
```



##### 같은 구성 내에서 빈 주입하기

```java
public class InjectRef {
  private Oracle oracle;
  
  public void setOracle(Oracle oracle) {
    this.oracle = oracle;
  }
}
```



---

##### * static 변수에 값 주입하기

```java
public class InjectRef {
  private static Oracle oracle;
  
  public void setOracle(Oracle oracle) {
    this.oracle = oracle;
  }
}
```

static만 추가 되었지 같은 구성 내에서 빈 주입하는 방법과 동일하다.

---



##### 컬렉션 주입하기

```java
@Service
public class CollectionInjection {
  @Resource(name ="map")
  private Map<String, Object> map;
  @Resource(name ="props")
  private Properties props;
  @Resource(name ="set")
  private Set set;
  @Resource(name ="list")
  private List list;
}
```

@Autowired 대신 @Resource를 이용한 이유는 @Autowired 애너테이션이 배열, 컬렉션, 맵을 해당 컬렉션의 값 타입에서 파생된 빈 타입을 가져와 처리하기 때문이다.

그렇기에 의도하지 않은 의존성이 주입되거나 ContextHolder 타입 빈이 정의되지 않아 스프링이 예외를 던질 수 있다.

그래서 컬렉션 타입을 주입할 때, 빈 이름을 지정할 수 있도록 지원하는 @Resource 애너테이션을 사용하여 빈 이름을 지정함으로써 스프링에 명시적으로 의존성을 알맞게 주입하도록 알려주어야 한다.



다시 정리하면 @Autowired는 **타입**을 이용해서 의존성을 주입하고 @Resource는 **빈 이름**을 이용해서 의존성을 주입한다.

@Resource 어노테이션의 적용 순서는 다음과 같다.

1. **name 속성에 지정한** 빈 객체를 찾는다.
2. name 속성이 없을 경우, **동일한 타입**을 갖는 빈 객체를 찾는다.

3. name 속성이 없고 동일한 타입을 갖는 빈 객체가 두 개 이상일 경우, <u>같은 이름을 가진 빈 객체</u>를 찾는다.

4. name 속성이 없고 동일한 타입을 갖는 빈 객체가 두 개 이상이고 같은 이름을 가진 빈 객체가 없는 경우 <u>@Qualifier를 이용해서 주입할 빈 객체를 찾는다.</u>



#### 메서드 주입하기

메서드 주입은 룩업 메서드 주입과 메서드 대체 방식이 제공된다.

룩업 메서드 주입은 빈이 자신의 의존성을 가져올 수 있는 또 다른 메커니즘을 제공한다.

메서드 대체는 원래 소스 코드를 변경하지 않고 임의로 메서드 구현을 변경할 수 있다.



##### 룩업 메서드 주입

싱글턴 빈이 비싱글턴 빈에 의존하는 상황과 같이 **어떤 빈이 다른 라이프 사이클을 가진 빈에 의존할 때 발생하는 문제를 극복하기 위해 추가된 것이다.**

룩업 메서드 주입을 이용한다면 별도의 스프링 인터페이스를 구현하지 않고서도 싱글턴 빈이 비싱글턴 빈 의존성을 필요로 하는 경우 선언하여, 필요할 때마다 비싱글턴 빈의 새로운 인스턴스를 얻을 수 있게 해준다.

룩업 메서드 주입은 **비싱글턴 빈의 인스턴스를 반환하는 룩업 메서드를 싱글턴 빈에 선언함으로써 동작한다.**

=> ex) 

```java
@Override
@Lookup("singer")
public abstract Singer getMySinger();
```



애플리케이션에서 싱글턴에 대한 참조를 얻을 때, 스프링이 구현해 둔 룩업 메서드를 사용해서 동적으로 생성된 서브 클래스에 대한 참조를 받는다.

일반적으로 룩업 메서드 구현 클래스는 룩업 메서드를 구현 없이 정의하기만 하므로 구현 클래스를 abstract로 선언한다.

이렇게 구현 클래스를 abstract로 선언하면 메서드 주입 구성을 잊어버렸을 때 스프링이 수정한 서브 클래스의 메서드가 아닌 텅 빈 메서드 구현에서 가져온 빈 클래스를 직접 사용해 발생하는 이상한 문제를 방지한다.



아래는 룩업 메서드 주입을 애너테이션을 통해 구현한 예제이다.

```java
@Component
@Scope("prototype") // 기본 설정은 singleton이 아닌 prototype으로 설정해줄 필요가 있다.
public class Singer {
    private String lyric = "lalalala";

    public void sing() {
        System.out.println(lyric);
    }
}
```

```java
public interface DemoBean {
    Singer getMySinger();

    void doSomething();
}

```

```java
@Component
public class StandardLookupDemoBean implements DemoBean {

    private Singer singer;
  
  	@Autowired
  	@Qualifier("singer")
    private void setSinger(Singer singer) {
        this.singer = singer;
    }
    @Override
    public Singer getMySinger() {
        return this.singer;
    }

    @Override
    public void doSomething() {
        singer.sing();
    }
}
```

```java
@Component
public abstract class AbstractLookupDemoBean implements DemoBean {
    @Override
    @Lookup 
  	// @Lookup을 통해 오버라이드해야 하는 메서드 이름을 스프링에게 알려준다. 그리고 이 메서드의 경우 인수를 받지 않아야 하며, 반환 타입은 메서드가 반환하는 빈의 타입이어야 한다.
    public abstract Singer getMySinger();

    @Override
    public void doSomething() {
        getMySinger().sing();
    }
}
```



이를 확인하기 위한 @Configuration을 다음과 같이 구성하고

```java
@Configuration
@ComponentScan
public class BeanLookUpTestContext {
}
```

이와 같이 확인해 보면 

```java
public static void main(String[] args) {
  SpringApplication.run(SpringPracticeApplication.class, args);
  AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanLookUpTestContext.class);

  AbstractLookupDemoBean abstractLookupDemoBean = applicationContext.getBean("abstractLookupDemoBean", AbstractLookupDemoBean.class);
  StandardLookupDemoBean standardLookupDemoBean = applicationContext.getBean("standardLookupDemoBean", StandardLookupDemoBean.class);
  displayInfo(abstractLookupDemoBean);
  displayInfo(standardLookupDemoBean);
}

public static void displayInfo(DemoBean demoBean) {
  Singer mySinger1 = demoBean.getMySinger();
  Singer mySinger2 = demoBean.getMySinger();
  System.out.println("same? = " + (mySinger1 == mySinger2));
}
```

```java
same? = false
same? = true
```

이와 같은 원하는 결과를 얻을 수 있었다.

또 책의 성능 테스트 결과를 보면 `431ms vs 1ms`로 룩업 메서드 주입의 성능 차이가 있다는 것을 확인할 수 있다.

이러한 룩업 메서드 주입 설계 지침은 다음과 같다.

+ IoC 목적으로만 사용되는 불필요한 정의로 비즈니스 인터페이스를 오염시키지 말것
+ 룩업 메서드를 추상화 할 것



##### 메서드 대체

서드파티 라이브러리 특정 메서드의 로직을 변경하고 싶을 때 소스 코드를 서드파티에서 제공했기 때문에 변경할 수 없다.

이때 메서드 대체를 사용해 해당 메서드의 로직을 사용자의 구현체로 변경할 수 있다.

내부적으로 빈 클래스의 서브 클래스를 동적으로 생성하면 메서드를 대체할 수 있다.

**CGLIB**를 사용해 <u>원래 메서드의 호출</u>을 **MethodReplacer 인터페이스를 구현한** <u>다른 빈에 대한 호출</u>로 **리다이렉션**할 수 있다.

// CGLIB 공부

MethodReplacer 인터페이스는 reimplement()라는 메서드 하나를 가지고 있고 세 인수가 전달된다.

첫 번째 인수는 호출된 메서드를 가진 빈, 두 번째 인수는 오버라이드할 메서드를 나타내는 Method 인스턴스 마지막은 메서드에 전달된 인수의 배열이다.

reimplement 메서드는 재구현된 로직의 수행 결과를 반환해야 하며, 반환값의 타입은 대체 메서드의 반환 타입과 호환돼야 한다.

```java
public class FormatMessageReplacer implements MethodReplacer {

    @Override
    public Object reimplement(Object arg0, Method method, Object[] args)
            throws Throwable {
        if (isFormatMessageMethod(method)) {
            String msg = (String) args[0];
            return "<h2>" + msg + "</h2>";
        } else {
            throw new IllegalArgumentException("Unable to reimplement method "
                    + method.getName());
        }
    }

    private boolean isFormatMessageMethod(Method method) {
        if (method.getParameterTypes().length != 1) {
            return false;
        }
        if (!("formatMessage".equals(method.getName()))) {
            return false;
        }
        if (method.getReturnType() != String.class) {
            return false;
        }
        if (method.getParameterTypes()[0] != String.class) {
            return false;
        }
        return true;
    }
}
```

위의 코드와

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="methodReplacer" class="com.example.springPractice.methodReplace.FormatMessageReplacer"/>

    <bean id="replacementTarget" class="com.example.springPractice.methodReplace.ReplacementTarget">
        <replaced-method name="formatMessage" replacer="methodReplacer">
            <arg-type>String</arg-type>
        </replaced-method>
    </bean>

    <bean id="standardTarget" class="com.example.springPractice.methodReplace.ReplacementTarget"/>
</beans>
```

xml 설정과 함께 메서드 대체를 할 수 있다.

// todo 애너테이션만 가지고 할 수 있는 방법은 아직 찾지 못하여 추후에 업데이트

이러한 메서드 대체의 경우 동일 타입의 모든 빈이 아닌 **단일 빈에 대한 특정한 메서드만 대체하려는 경우에 유용하다.**

**하지만 런타임 바이트 코드 향상에 의존하기보다는 표준 자바 메커니즘을 통해 자바 메서드를 대체하는 것이 더 바람직하다고 한다. **

// todo 이 문장 역시 아직 이해하지 못해 더 공부



#### 빈 생성 방식 이해하기

기본적으로 스프링의 모든 빈은 싱글턴이다.

즉, 스프링은 단일 인스턴스를 유지하고 관리하며, 모든 의존 객체는 동일한 인스턴스를 사용하고 ApplicationContext.getBean()에 대한 모든 호출은 동일한 인스턴스를 반환한다.



하지만 스코프를 프로토타입으로 지정하면 스프링은 애플리케이션이 빈 인스턴스를 요청할 때만다 새 인스턴스를 생성한다.



##### 시나리오에 따른 빈 생성 방식

###### 싱글턴

+ 상태가 없는 공유 객체
+ 읽기 전용 상태를 갖는 공유 객체
+ 공유 상태를 갖는 공유 객체
+ 쓰기 기능을 갖는 대용량 처리 객체

###### 비싱글턴

+  쓰기 가능한 상태를 갖는 객체
+ private 상태를 갖는 오브젝트



##### 빈스코프

+ 싱글턴 : 스프링 IoC 컨테이너당 하나의 객체만 생성
+ 프로토타입 : 애플리케이션에서 요청할 때마다 새 인스턴스 생성
+ 요청 : 웹 애플리케이션에서 사용한다. 모든 HTTP 요청이 있을 때마다 생성되고 요청이 처리되면 소멸된다.
+ 세션 : 웹 애플리케이션에서 사용한다. 모든 HTTP 세션이 시작되면 생성되고 세션이 끝나면 소멸된다.
+ 글로벌 세션 : 포틀릿 기반 웹 애플리케이션에서 사용한다. 글로벌 세션 스코프빈은 동일한 스프링 MVC 기반 포털 애플리케이션 내의 모든 포틀릿 간 공유될 수 있다.  // todo 포틀릿 공부
+ 스레드 : 스레드에 따라 빈 인스턴스가 반환된다.
+ 사용자 정의



### 빈에 자동와이어링 하기

스프링은 다음 다섯 가지 방식의 자동와이어링을 제공한다.

+ byName : 이름에 의한 자동와이어링
+ byType : 타이벵 의한 자동와이어링, 이를 이용하면 스프링은 자동으로 ApplicationContext에서 동일한 타입의 빈을 대상 빈의 각 프로터티에 연결하려 시동한다.
+ Constructor : 주입이 수정자가 아닌 생성자를 이용해 이루어진다는 점을 제외하면 타입에 의한 와이어링과 같다.
+ default : 스프링은 **Constructor** 방식과 **byType** 방식을 기본 값으로 한다



그리고 만약 스프링이 자동와이어링 해야할 빈을 알지 못한 경우 이를 해결하기 위한 방법으로는 두 가지가 있다.

+ @Primary를 적용한다.
+ 빈 이름을 지정한다.
  + @Autowired 
+ @Lazy를 클래스에 적용한다.
  + ex
    + @Autowired setFooOne(Foo fooOne)
    + @Autowired  setFooTwo(Foo foo)
      + 이는 foo들이 싱글턴이기에 두 번 사용될 수 없어 setFooTwo에서는 fooTwo가 자동으로 들어갈 수 있는 것이다.
+ @Configuration을 통해 직접 선언한다.

