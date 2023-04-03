## Bean

### Bean 이란?

Spring Framework 문서에 나온 Bean의 정의는 아래와 같다.

> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. 
>
> A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.

즉, Spring IoC 컨테이너는 하나 이상의 Bean을 관리한다.

그리고 이러한 Bean은 컨테이너에 제공하는 구성 메타데이터로 생성된다.



#### 메타데이터

+ Package를 만족하는 클래스 네임 : **Bean으로 정의된 클래스의 실제 구현체**
+ Bean이 컨테이너에서 작동해야하는 방식을 나타내는 **Bean 작동 구성 요소 (범위, 라이프 사이클 콜백 등).**
+ Bean이 작업을 수행하는 데 필요한 **다른 Bean에 대한 참조**이다. 이러한 참조를 공동 작업자 또는 종속성이라고한다.
+ 새로 생성 된 객체에서 설정할 기타 구성 설정 — 풀의 크기 제한 또는 연결 풀을 관리하는 Bean에서 사용할 연결 수.이 메타데이터는 각 Bean 정의를 구성하는 속성 집합으로 변환된다.



특정 Bean을 생성하는 방법에 대한 정보가 포함된 Bean 정의 외에도, `ApplicationContext` 구현은 컨테이너 외부에서 (사용자가) 생성한 기존 객체의 등록도 허용한다.

이 작업은 `getBeanFactory()` 메서드를 통해 애플리케이션 컨텍스트의 `BeanFactory`에 액세스하여 수행되며, 

이 메서드는 `DefaultListableBeanFactory` 구현을 반환한다.



`DefaultListableBeanFactory`는 `registerSingleton(..)` 및 `registerBeanDefinition(..)` 메서드를 통해 Bean 등록을 지원한다.

그러나 일반적인 애플리케이션은 일반 Bean 정의 메타데이터를 통해 정의된 Bean으로만 작동한다.



#### DefaultListableBeanFactory를 활용한 Bean 등록

*출처 : https://www.javaprogramto.com/2019/07/spring-dynamically-register-beans.html*



##### GenericBeanDefinition을 활용한 방법

```java
DefaultListableBeanFactory context = new DefaultListableBeanFactory();

// OtherBean 클래스 생성 생략...

// OtherBean을 멤버로 가지고 있는 MyBean 생성
GenericBeanDefinition gbd2 = new GenericBeanDefinition();
gbd2.setBeanClass(MyBean.class);

MutablePropertyValues mpv = new MutablePropertyValues();
mpv.addPropertyValue("otherBean", context.getBean("other"));

gbd2.setPropertyValues(mpv);
context.registerBeanDefinition("myBean", gbd2);

MyBean bean = context.getBean(MyBean.class);
bean.doSomething();
```

우선 **GenericBeanDefinition를 생성한다.**

GenericBeanDefinition에 **Bean 클래스에 대한 정보, 프로퍼티에 대한 정보를 추가한다.** (`.setBeanClass()`, `setPropertyValues()`)

이후 DefaultListableBeanFactory에 해당 **GenericBeanDefinition를 등록한다.** (`.registerBeanDefinition()`)



##### BeanDefinitionBuilder를 활용한 방법

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

// str을 멤버로 가지고 있는 MyBean 생성
BeanDefinitionBuilder b =
          BeanDefinitionBuilder.rootBeanDefinition(MyBean.class)
                               .addPropertyValue("str", "myStringValue");
beanFactory.registerBeanDefinition("myBean", b.getBeanDefinition());

MyBean bean = beanFactory.getBean(MyBean.class);
bean.doSomething();
```



##### BeanFactoryPostProcessor을 활용하여 스프링 컨테이너의 BeanFactory에 Bean 등록 방법

```java
public class MyConfigBean implements BeanFactoryPostProcessor {  
  @Override
  public void postProcessBeanFactory (ConfigurableListableBeanFactory beanFactory) throws BeansException {      
    GenericBeanDefinition bd = new GenericBeanDefinition();
    // strPro를 멤버로 가지고 있는 MyBean 생성
    bd.setBeanClass(MyBean.class);
    bd.getPropertyValues().add("strProp", "my string property");      
    
    ((DefaultListableBeanFactory) beanFactory).registerBeanDefinition("myBeanName", bd);
  }
}

@Configuration
public class MyConfig {
  @Bean
  MyConfigBean myConfigBean () {
      return new MyConfigBean();
  }
}
```



### Naming Beans

**모든 Bean에는 하나 이상의 식별자가 있다.** 

**이때 빈의 식별자가 두 개 이상인 경우 여분의 식별자는 별칭으로 간주한다.**

이러한 **식별자는 Bean을 호스팅하는 컨테이너 내에서 고유해야 한다.** 



XML 기반 구성 메타데이터에서는 id 속성, name 속성 또는 둘 다를 사용하여 Bean 식별자를 지정한다. 

id 속성을 사용하면 정확히 하나의 id를 지정할 수 있다. 

일반적으로 이러한 이름은 영숫자('myBean', 'someService' 등)이지만 특수 문자도 포함할 수 있다. 

Bean의 다른 별칭을 도입하려는 경우 쉼표(,), 세미콜론(;) 또는 공백으로 구분하여 이름 속성에 지정할 수도 있다. 

id 속성은 xsd:string 유형으로 정의되지만, Bean id 고유성은 XML 파서가 아니라 컨테이너에 의해 적용된다.



#### 사례

id와 name을 잘 활용한 **전문가를위한스프링5**에서 소개한 사례를 살펴보자.

```java
@Configuration
public class MyConfig {

    @Bean(name = "superFoo")
    SuperFoo superFoo() {
        return new SuperFoo();
    }

    @Bean(name = "standardFoo")
    StandardFoo standardFoo() {
        return new StandardFoo();
    }
}
```

위와 같이 `Foo`의 구현체들을 Bean으로 등록하였다고 생각해보자.

각각의 구현체를 사용하던 중 더 이상 `StandardFoo`를 사용하지 않고 `SuperFoo`만 사용한다고 정책이 바뀐다면 가장 좋은 리펙터링 방법은 무엇일까?



**책에서 추천하는 방법은 `SuperFoo`의 name에 "standardFoo"를 별칭으로 추가하고 `StandardFoo`에 대한 정의는 삭제하는 것이다.**

```java
@Bean(name = {"superFoo", "standardFoo"})
SuperFoo superFoo() {
  return new SuperFoo();
}
```

이렇게 리펙터링 한다면 아래와 같이 @Qualifier를 통해 `StandardFoo`를 주입 받겠다고 선언한 것도 문제 없이 `SuperFoo` Bean을 주입할 수 있다.

```java
@Component
public class UseFoo {

    private Foo foo;

    public UseFoo(@Qualifier("standardFoo") Foo foo) {
        this.foo = foo;
    }
}
```