## 스프링 AOP

### AOP 개념

AOP를 잘 이해하려면 개념과 용어부터 잘 이해하는 것이 중요하다.

아래는 AOP를 이해하는데 필요한 개념과 용어를 간단히 정리한 것이다.



#### 조인 포인트

애플리케이션 실행 과정 내에 있는 명확한 지점이다.

AOP를 사용해 추가 로직을 삽입할 수 있는 **애플리케이션의 특정 지점**이다.



#### 어드바이스

특정 조인포인트에서 **실행되는 코드**로 애플리케이션 클래스 메서드로 정의된다.



#### 포인트컷

언제 어드바이스를 실행할지를 정의할 때 사용하는 **조인포인트 모음**이다.



#### 애스팩트

클래스에 캡슐화된 어드바이스와 포인트컷의 조합이다.

이는 애플리케이션에서 포함되어야 하는 로직과 로직이 어디서 실행되어야 하는지를 정의한다.



#### 위빙

애플리케이션 코드의 적절한 위치에서 **애스팩트를 삽입하는 과정**이다.



### AOP 종류

#### 정적 AOP

정적 AOP에서 위빙 프로세스는 애플리케이션 빌드 프로세스에서 이루어진다.

#### 동적 AOP

동적 위빙의 경우 구현체에 따라 다르다.

스프링에서 사용하는 방식은 어드바이스가 적용된 모든 객체에 대한 **프록시를 생성**해 필요에 따라 **어드바이스를 호출**할 수 있도록 하는 방식이다.



#### AOP 예시

```java
public class Agent {
  public void speak() {
    System.out.print("bond");
  }
}
```

```java
public class AgentDecorator implements MethodInterceptor {
  public Object invoke(MethodInvocation invocation) throws Throwable {
		System.out.print("James ");
    
    Object retVal = invocation.proceed();
    
    System.out.println("!");
    return retVal;
  }
}
```

MethodInterceptor 인터페이스는 메서드 호출 조인포인트에 적용할 어라운드 어드바이스를 구현할 때 사용하는 표준 AOP 얼라이언스 인터페이스이다.

`System.out.print("James ")` 그리고 `System.out.println("!")` 가 `invocation.proceed()` 전후로 실행하려는 것을 위의 코드를 통해 확인할 수 있다.



위와 같이 클래스만 작성한다고 AOP가 자동으로 동작하지는 않는다.

위의 Agent는 AOP가 적용될 메서드를 정의한 것이고 AgentDecorator은 **어드바이스**를 정의한 것이다.

그렇다면 이제 이를 기반으로 위빙을 해주면 AOP가 동작한다.



이번 예시의 구현 방식은 동적 AOP이다.

```java
ProxyFactory pf = new ProxyFactory();
pf.setTarget( new Agent());
pf.addAdvice(new AgentDecorator());

Agent proxy = (Agent) pf.getProxy();

proxy.speak();
```

우선 ProxyFactory를 통해 프록시 객체를 생성하였다.

이후 `pf.setTarget()`을 통해 프록시 객체가 감쌀 타겟을 설정하고

 `pf.addAdvice()`를 통해 어드바이스를 추가해주었다.

그리고 `pf.getProxy()`를 통해 프록시 객체를 가지고와 `proxy.speak()` 같이 실행해 주면 "James bond!" 처럼 앞에는 "James" 그리고 뒤에는 "!"가 추가된 결과를 얻을 수 있다.



하지만 위의 예제를 보면서 "음... MethodInterceptor는 어떻게 조인포인트를 설정하는 거지?" 하는 생각이 들 수 있다.

그래서 이를 확인해 보기 위해 아래처럼 코드를 수정하였다.

```java 
public class Agent {
  public void speak() {
    System.out.print("bond");
  }

  public void speakLouder() {
    System.out.print("BOND");
  }
}
```

```java
ProxyFactory pf = new ProxyFactory();
pf.setTarget( new Agent());
pf.addAdvice(new AgentDecorator());

Agent proxy = (Agent) pf.getProxy();

proxy.speak();
proxy.speakLouder();
```

위와 같이 코드를 수정한 이유는 MethodInterceptor가 프록시가 타겟하고 있는 객체의 모든 매서드에 조인포인트를 설정하는지 알아보고 싶어서이다.

그 결과는 다음과 같다.

```
James bond!
James BOND
```

예상한 것처럼 MethodInterceptor는 프록시가 타겟하고 있는 객체의 모든 매서드에 조인포인터를 설정하고 있었다.



### 스프링 AOP 아키텍처

스프링 AOP의 핵심 아키텍처는 프록시를 기반으로 한다.

스프링은 선언적인 AOP 구성 메커니즘을 사용해 선언적으로 프록시를 생성한다.



그리고 스프링은 런타이 시점에 ApllicationContext의 빈에 정의된 횡단 관심사를 분석하고 **프록시 빈을 동적으로 생성**한다고 한다.

그리고 **호출자에** 대상 빈을 주입해 이를 직접 호출하게 하는 대신 **프록시 빈을 주입**한다.

이후 프록시 빈은 실행조건을 분석하고 이에 따라 **적절한 어드바이스를 위빙**한다.



그리고 스프링 AOP는 **메서드** 호출 조인포인트만 지원한다.



#### ProxyFactory 클래스

ProxyFactory 클래스는 스프링 AOP의 **위방**과 **프록시 생성** 과정을 제어한다.

프록시를 생성하기 전에 항상 어드바이스를 적용하는 객체나 대상 객체를 지정해야 한다.

프록시에 어드바이스를 추가하는 방법은 `addAdvice()`메서드로 어드바이스를 직접 추가하는 방법과 Advisor 구현체를 `addAdvice()` 메서드에 전달해 간접적으로 추가하는 방법이 있다.



#### 스프링의 어드바이스

+ Before(MethodBeforeAdvice)
  + 조인포인트 실행 전에 커스텀 처리를 수행할 수 있다.
+ After-Returning(AfterReturningAdvice)
  + 실행을 마치고 값을 반환한 후에 실행된다. 
  + 이는 메서드 호출 대상, 메서드에 전달되는 인수, 메서드의 반환값에 접근할 수 있다.(읽기 전용, 수정 불가)
  + 하지만 메서드 호출 자체는 제어할 수 없다.
+ After(AfterAdvice)
  + 메서드가 정상적으로 완료된 후에 실행된다.
+ Around(MethodInterceptor)
  + 메서드 호출 전후에 실행된다.
  + 그렇기에 메서드 호출이 진행되는 시점을 제어할 수 있다.
  + 반환값을 변경할 수 있다.
  + 또 직접 로직 구현체를 제공해 메서드 전체를 건너 뛸 수도 있다.
  + 즉, 메서드 전체 구현을 새로운 코드로 바꿀 수 있다.
+ Throws(ThrowsAdvice)
  + 호출이 예외를 던질 때만 실행된다.
+ Introduction



스프링은 위의 어드바이스를 구현할 수 있는 인터페이스를 제공한다.

각각의 어드바이스를 구현하기 위한 인터페이스는 괄호를 참고하면 된다.



### 스프링의 어드바이저와 포인트 컷

위의 예제들을 보았을 때 `ProxyFactory.addAdvice()` 메서드는 프록시에 어드바이스를 구성한다.

이 메서드는 내부적으로 `addAdvisor()` 에 작업을 위임하여 `DefaultPointcutAdvisor` 인스턴스를 생성하고 **모든 메서드**를 가리키는 포인트컷으로 어드바이저를 구성한다.

하지만 특정 메서드로 AOP를 적용할 대상을 제한해야 할 경우도 있다.



이때 포인트컷은 어드바이스 내에 적용 대상 메서드인지 검사하는 코드를 넣지 않더라도 어드바이스를 적용할 메서드를 구성할 수 있도록 도와준다.

그리고 포인트컷을 사용하면 각 메서드마다 한 번씩 검사가 수행되며 결과를 캐싱해 재사용하여 어듭사이스가 적용되지 않는 메서드를 최적화하고 이들 어드바이스를 빠르게 호출할 수 있도록 도와준다.



기본적인 포인트컷의 인터페이스는 아래와 같다.

```java
public interface Pointcut {
  ClassFilter getClassFilter();
  MethodMatcher getMethodMatcher();
}
```

`getClassFilter()`를 통해 Class를 구분하고

`getMethodMatcher()`를 통해 포인트컷을 적용할 Method를 구분한다.



스프링은 4.0 버전에서 아래와 같은 PointCut 인터페이스 구현체를 제공한다.

+ AnnotationMatchingPoincut(애너테이션 매칭 포인트컷)
+ AspectJExpressionPointcut(AspectJ 포인트컷)
+ ComposablePointcut
+ ControlFlowPointcut
+ DynamicMethodMatcherPointcut(동적 포인트컷)
+ JdkRegexpMethodPointcut(정규식 포인트컷)
+ NameMatchMethodPointcut(단순 이름 매칭 포인트컷)
+ StaticMethodMatcherPointcut(정적 포인트컷)



#### DefaultPointcutAdvisor

Pointcut 구현체를 사용하려면 먼저 Advisor 인터페이스의 인스턴스나 PointcutAdvisor 인터페이스의 인스턴스를 생성해야 한다.

DefaultPointcutAdvisor는 **하나의 포인트컷**을 **하나의 어드바이스**와 연결시키는 간단한 PointcutAdvisor이다.



##### StaticMethodMatcherPointcut를 활용한 정적 포인트컷

```java 
public class SimpleStaticPointcut extends StaticMethodMatcherPointcut {
  @Override
  public boolean matches(Method method, Class<?> cls) {
    return ("speak".equals(method.getName()));
  }

  @Override
  public ClassFilter getClassFilter() {
    return cls -> (cls == Agent.class);
  }
  
}
```

위의 예제를 그대로 활용해 `speak()` 그리고 `speakLouder()` 중에 `speak()`에만 AOP를 적용할 것이라는 포인트컷을 만들었다.



그리고 이를 아래와 같이 적용시키면

```java
Agent target = new Agent();
Pointcut pc = new SimpleStaticPointcut();
Advice advice = new AgentDecorator();
Advisor advisor = new DefaultPointcutAdvisor(pc, advice);

ProxyFactory pf = new ProxyFactory();
pf.addAdvisor(advisor);
pf.setTarget(target);

Agent proxy = (Agent) pf.getProxy();

proxy.speak();
proxy.speakLouder();
```

```
James bond!
BOND
```

와 같이 `speak()`에만 AOP가 적용된 것을 확인할 수 있다.



##### DynamicMethodMatcherPointcut를 활용한 동적 포인트컷

정적 포인트 컷의 단점은 007 코드네임을 받는 것과 같은 동적인 검증을 하지 못한다는 것이 있다.

DynamicMethodMatcherPointcut을 통해 동적 포인트컷을 한번 만들어 보자.



우선 코드네임을 받기위해 Agent 클래스를 아래와 같이 수정해보자.

```java
public static class Agent {
  public void speak(String codName) {
    System.out.print("bond");
  }

  public void speakLouder() {
    System.out.print("BOND");
  }
}
```



그리고 DynamicMethodMatcherPointcut를 아래와 같이 구현해보자.

```java
public class SimpleStaticPointcut extends DynamicMethodMatcherPointcut {

  @Override
  public boolean matches(Method method, Class<?> cls) {
    return ("speak".equals(method.getName()));
  }

  @Override
  public boolean matches(Method method, Class<?> targetClass, Object... args) {
    System.out.println("CodeName : " + (String) args[0]);

    return "007".equals((String) args[0]);
  }

  @Override
  public ClassFilter getClassFilter() {
    return cls -> (cls == Agent.class);
  }

}
```

정적 포인트컷에 비해  `matches()`가  추가되었다.



이렇게 구현한 포인트컷을 다음과 같이 적용해보면

```java
Agent target = new Agent();
Pointcut pc = new SimpleStaticPointcut();
Advice advice = new AgentDecorator();
Advisor advisor = new DefaultPointcutAdvisor(pc, advice);

ProxyFactory pf = new ProxyFactory();
pf.addAdvisor(advisor);
pf.setTarget(target);

Agent proxy = (Agent) pf.getProxy();

proxy.speak("008");

proxy.speak("007");
```

```
CodeName : 008
bond
CodeName : 007
James bond!
```

와 같이 007을 코드네임을 제시한 경우에만 "James" 그리고 "!"를 추가해줄 수 있다.



##### AnnotationMatchingPoincut을 활용한 어노테이션 포인트컷

어노테이션을 통해 조인포인트를 설정할 수 있다.

아래는 이를 위해 만든 어노테이션 예제이다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface AdviceRequired {
}
```



이를 아래와 같이 적용해주고

```java
public static class Agent {

  @AdviceRequired
  public void speak() {
    System.out.print("bond");
  }

  public void speakLouder() {
    System.out.println("BOND");
  }
}
```



```
Agent target = new Agent();
Pointcut pc = AnnotationMatchingPointcut.forMethodAnnotation(AdviceRequired.class);
Advice advice = new AgentDecorator();
Advisor advisor = new DefaultPointcutAdvisor(pc, advice);

ProxyFactory pf = new ProxyFactory();
pf.addAdvisor(advisor);
pf.setTarget(target);

Agent proxy = (Agent) pf.getProxy();

proxy.speak();
proxy.speakLouder();
```

와 같이 활용할 수 있다.



### 인트로덕션

인트로덕션은 AOP의 중요한 기능으로 사용해 기존 객체에 **새로운 기능을 동적으로 도입할 수 있다.**

( 예제에서는 수정을 감지하는 기능을 동적으로 구현하였다.)

스프링에서는 기존 객체에 인터페이스의 구현체를 인트로듀스할 수 있다.



스프링은 인트로덕션은 특별한 타입의 어드바이스로 취급하며, 좀 더 구체적으론 특별한 타입의 어라운드 어드바이스로 취급한다.

인트로덕션은 **클래스 레벨**에서만 적용되므로 **인트로덕션에서 포인트컷을 사용할 수는 없다.**

**인트로덕션**은 **새로운 인터페이스 구현체를 클래스에 추가**하고 **포인트컷**은 **어드바이스가 적용되는 메서드**를 정의하는 것이다.



인트로덕션은 `MethodINterceptor`와 `DynamicIntroductionAdvice` 인터페이스를 상속한 `InroductionInterceptor` 인터페이스를 구현하여 인트로덕션을 생성한다.

기본 구현체로는 `DelegatingIntroductionInterceptor`가 있다.



프록시에 인트로덕션을 추가할 때는 `InroductionAdvisor`을 사용한다.

InroductionAdvisor의 기본 구현체는 `DefaultInroductionAdvisor`이다.



인트로덕션에서 인트로덕션 어드바이스는 어드바이스가 적용된 객체의 상태를 구성하며 결과적으로 모든 어드바이스가 적용된 객체에 대해 고유한 어드바이스 인스턴스가 있어야 한다.

이를 **인스턴스별 라이프사이클**이라 한다.



**기존에 메서드 단위로만 생각하였던 AOP의 기능을 클래스 단위로 확장한 것이 인트로덕션이라 본인은 생각한다.**



아래 구현한 코드를 보면서 조금 더 인트로덕션에 대해서 알아보자.

우선 이전에 구현하였던 Agent를 아래와 같이 수정하였다.

```java
public class Agent {
  private String name;

  public Agent(String name) {
    this.name = name;
  }

  public void changeName(String name) {
    this.name = name;
    System.out.print(name);
  }

  public String getName() {
    return name;
  }
}
```



그리고 아래와 같이 인트로덕션을 구성하였다.

```java
public interface IsChangeName {
  boolean isChangeName();
}
```



```java
public class AgentIntroductionInterceptor extends DelegatingIntroductionInterceptor implements IsChangeName {

  private boolean isChangeName = false;

  @Override
  public boolean isChangeName() {
    return isChangeName;
  }

  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable {
    if (!isChangeName) {
      if (invocation.getMethod().getName().equals("changeName")
          && invocation.getArguments().length == 1) {

        Object newVal = invocation.getArguments()[0]; 

        String getterName = invocation.getMethod().getName().replaceFirst("change", "get");
        Method getter = invocation.getMethod().getDeclaringClass()
          .getMethod(getterName, null);

        Object oldVal = getter.invoke(invocation.getThis(), null);

        if ((newVal == null) && (oldVal == null)) {
          isChangeName = false;
        } else if ((newVal == null) && (oldVal != null)) {
          isChangeName = true;
        } else if ((newVal != null) && (oldVal == null)) {
          isChangeName = true;
        } else {
          isChangeName = !newVal.equals(oldVal);
        }

      }
    }
    return super.invoke(invocation);
  }
}
```

우선 `DelegatingIntroductionInterceptor`을 상속하여 이 클래스가 인트로덕션임을 선언했다.

그리고 `IsChangeName` 를 구현하여 Agent가 이름을 변경하는지 확인하는 기능을 추가하였다.

만약에 Agent에 name외의 다른 codeName과 같은 요소가 더 있었다면 그것의 변경을 확인하는 인터페이스를 추가로 만들어서 기능을 추가 하면 된다.



`invoke()`에서는 이름에 변화가 있는 것인지 먼저 확인한 후 메서드를 확인한다.

이는 포인트 컷을 활용할 수 없는 인트로덕션 특성상 아래와 같은 코드로 조금은 하드 코딩하여 확인하였다.

```java
if (invocation.getMethod().getName().equals("changeName")
    && invocation.getArguments().length == 1) {
  ...
}
```



이후 `Object newVal = invocation.getArguments()[0]; ` 를 통해 변경하려는 값을 알아내고

아래와 같은 과정을 통해 변경하기 전의 값을 알아낸다.

```java
String getterName = invocation.getMethod().getName().replaceFirst("change", "get"); // changeName에서 change를 get으로 
Method getter = invocation.getMethod().getDeclaringClass().getMethod(getterName, null); // getName 메서드를 가져온다.

Object oldVal = getter.invoke(invocation.getThis(), null); // getName 메서드를 실행한다.
```

이 값들을 비교하며 변화가 있으면 `isChangeName`의 값을 수정해준다.



위와 같이 설정한 것들은 아래와 같이 활용할 수 있다.

```java
Agent target = new Agent("name1");
Advice interceptor = new AgentIntroductionInterceptor();
IntroductionAdvisor introductionAdvisor = new DefaultIntroductionAdvisor(interceptor);

ProxyFactory pf = new ProxyFactory();
pf.setTarget(target);
pf.addAdvisor(introductionAdvisor);
pf.setOptimize(true);

Agent agent = (Agent) pf.getProxy();
IsChangeName agentChecker = (IsChangeName) pf.getProxy(); // proxy 객체가 Agent를 IntroductionAdvisor로 감싼것 같다.

System.out.println("Agent Name : " + agent.getName());
System.out.println("Change Name? " + agentChecker.isChangeName());

agent.changeName("name1");
System.out.println("Agent Name : " + agent.getName());
System.out.println("Change Name? " + agentChecker.isChangeName());

agent.changeName("name2");
System.out.println("Agent Name : " + agent.getName());
System.out.println("Change Name? " + agentChecker.isChangeName());
```

```
Agent Name : name1
Change Name? false
name1Agent Name : name1
Change Name? false
name2Agent Name : name2
Change Name? true
```



### @AspectJ 방식 애너테이션 사용하기

JDK 5 이상에서 스프링 AOP를 사용할 때 @AspectJ 방식의 애너테이션을 사용해 어드바이스를 선언할 수 있다.

이때 스프링은 AspectJ의 위빙 메커니즘이 아니라 **자체 프록시 메커니즘**을 사용한다.



`@Aspect`를 통해 해당 클래스가 애스팩트 클라스라는 것을 나타낸다.

`@Pointcut("execution( ... ) && args(value) ")` 보이는 것처럼 **포인트 컷**을 설정하는 것이다.

`@AdviceType("pointcutMethodName")` **어드바이스**를 설정하는 것이다.

AdviceType으로는 위의 Before, Around와 같은 것이 있고 pointcutMethodName은 `@Poincut`이 붙은 메서드 이름을 적으면 된다.

그리고 이때 JDK 동적 프록시가 아닌 CGLIB를 생성하려면 **자바 구성파일**에 `@EnableAspectJAutoProxy(proxyTargetClass = true)` 를 설정해주면 된다.
