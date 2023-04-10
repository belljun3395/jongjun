## Bean 초기화

### Bean 초기화 메서드

Bean을 생성할 때 Bean의 메서드 하나를 지정해 초기화 콜백으로 사용할 수 있다.

이는 애플리케이션이 스프링과 결합하지 않게 할 때 유용하다.

즉, 스프링 기반 애플리케이션이 이전에 만들어진 빈이나 서드파티 벤더가 제공하는 빈을 사용해야 할 때 사용할 수 있을 것이다.



간단히 예제 코드를 작성해 보면 아래와 같을 것이다.

```java
@Component
public class Foo implements InitializingBean {

    private Bar bar = null; // Bar은 스프링 기반 애플리케이션이 이전에 만들어진 빈이나 서드파티 벤더가 제공하는 빈 이다.

    @Override
    public void afterPropertiesSet() throws Exception {
        this.bar = bar 생성 코드
    }
}
```

위의 경우 `InitializingBean`을 구현하는 방법으로 초기화 콜백을 지정하였는데 방법은 다양하다.

+ @PostConstruct
+ @Bean으로 초기화 메서드 선언
  + @Bean(initMethod = "init")
+ xml 설정



이러한 방법을 사용하면 의존성을 직접 제어할 때의 제어 권한을 잃지 않으면서 IoC가 제공하는 모든 장점을 활용할 수 있다.

하지만 이 장점은 정적 초기화 메서드를 사용하려 할 때 사라진다고 한다.

정적 초기화 메서드에서는 각 빈의 상태 정보에 접근할 수 없기 때문이다.



### 초기화 메서드 해석 순서

1. 빈 생성을 위해 생성자 호출
2. 의존성 주입 (생성자 주입, 수정자 주입 등)
3. BeanFactoryAware, ApplicationContextAware 처리
4. BeanPostProcessor의 postProcessBeforeInitialization 처리
5. 빈 초기화 (@PostConstruct, @Bean(initMethod = ..), InitializingBean)
6. BeanPostProcessor의 postProcessBeforeInitialization 처리



이를 코드로 확인 하기 위해 아래와 같이 구현한 후 결과를 확인 해보자.

```java
@Component
public class Foo  implements BeanFactoryAware, ApplicationContextAware, InitializingBean {

    public Foo() {
        System.out.println("Foo.Foo");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("Foo.setBeanFactory");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("Foo.setApplicationContext");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Foo.afterPropertiesSet");
    }

    @Component
    static class FooPostProcessor implements BeanPostProcessor { // Foo에서 Foo에 대한 후처리를 할 수 없다.

        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName)
            throws BeansException {
            if (bean instanceof Foo) {
                System.out.println("Foo.postProcessBeforeInitialization");
            }
            return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
            if (bean instanceof Foo) {
                System.out.println("Foo.postProcessBeforeInitialization");
            }
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
    }
}
```

```
Foo.Foo
Foo.setBeanFactory
Foo.setApplicationContext
Foo.postProcessBeforeInitialization
Foo.afterPropertiesSet
Foo.postProcessBeforeInitialization
```



이 순서를 잘 활용하면 아래와 같이 빈을 초기화 하여 사용할 수 있다.

대표적인 예제를 알아보자.

```java
public class MessageDigestFactoryBean implements InitializingBean {

    private String alg = "MD5";
    private MessageDigest messageDigest = null;

    public void setAlg(String alg) {
        this.alg = alg;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        messageDigest = MessageDigest.getInstance(alg);
    }
}
```

1. 의존성 주입 : 
   setAlg를 통해 의존성이 주입될 수 있다. 
   위의 경우 의존성 주입이 없다면 "MD5" 알고리즘이 있다면 주입되는 알고리즘을 사용
2. afterPropertiesSet 실행 : 
   이때 afterPropertiesSet의 반환 값이 void이기 때문에 위의 경우 messageDigest 멤버를 미리 설정하고 해당 멤버에 객체를 저장



이를 확인하기 위해 아래와 같이 테스트 코드를 작성하고 그 결과를 확인해 보자.

```java
MessageDigestFactoryBean messageDigestFactoryBean
    = context.getBean("messageDigestFactoryBean", MessageDigestFactoryBean.class);
MessageDigest md5 = messageDigestFactoryBean.getMessageDigest();

messageDigestFactoryBean.setAlg("SHA1");
MessageDigest sha1 = messageDigestFactoryBean.getMessageDigest();

String INPUT = "jongjun";
byte[] md5Digest = md5.digest(INPUT.getBytes());
byte[] sha1Digest = sha1.digest(INPUT.getBytes());

System.out.println("md5Digest = " + md5Digest);
System.out.println("sha1Digest = " + sha1Digest);
```

```
md5Digest = [B@7aea704c
sha1Digest = [B@6d0290d8
```

알고리즘에 따라 그 결과가 달라진 것을 확인할 수 있다.