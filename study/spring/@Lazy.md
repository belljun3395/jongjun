## @Lazy

### @Lazy란?

```java
@Target({TYPE,METHOD,CONSTRUCTOR,PARAMETER,FIELD})
@Retention(RUNTIME)
@Documented
public @interface Lazy
```

빈을 느리게 초기화할지 여부를 나타낸다.

@Component로 직접 또는 간접적으로 어노테이션이 달린 모든 클래스 또는 @Bean으로 주석이 달린 메서드에 사용할 수 있다.

이 어노테이션이 @Component 또는 @Bean 정의에 없으면 초기화가 진행된다. 

만약 @Lazy 어노테이션이 true로 설정된 경우, @Bean 또는 @Component는 다른 Bean이 참조하거나 동봉된 BeanFactory에서 명시적으로 검색할 때까지 초기화되지 않는다. 

만약 @Lazy 어노테이션이 존재하고 false로 설정된 경우 빈은 시작 시 싱글톤의 초기화를 수행하는 빈 팩토리에 의해 인스턴스화된다.



@Lazy가 @Configuration 클래스에 있으면 해당 @Configuration 내의 모든 @Bean 메서드가 게으르게 초기화되어야 함을 나타낸다. 

@Lazy 어노테이션이 달린 @Configuration 클래스 내의 @Bean 메서드에 @Lazy가 있고 false인 경우, 이는 'default lazy' 동작을 재정의하고 빈을 eager 초기화해야함을 나타낸다.

이 어노테이션은 구성 요소 초기화 역할 외에도 @Autowired 또는 @Inject으로 표시된 주입 지점에 배치할 수 있다.

이러한 context에서 ObjectFactory 또는 Provider를 사용하는 대신 영향을 받는 모든 종속성에 대한 lazy-resolution proxy를 생성한다. 

이러한  lazy-resolution proxy는 항상 주입된다. 

targer dependency가 없는 경우 호출 시 예외를 통해서만 알 수 있다. 

결과적으로 이러한 주입점은 선택적 종속성에 대한 비직관적 동작을 초래한다. 

더욱 정교한 느린 참조를 허용하는 프로그래밍 방식의 동등한 경우 ObjectProvider를 고려한다.



### 언제 사용할까?

빈 생성 관련 설정에 처음 접근이 일어날 때 인스턴스가 나중에 생성될 빈을 정의하려고 클래스에 적용되는 애너테이션이다.



예제를 통해 알아보자.

```java
public class Stage {

    public Singer danceSinger;
    public Singer rockSinger;

    public Stage(@Qualifier("danceSinger") Singer danceSinger) {
        this.danceSinger = danceSinger;
        danceSinger.sing();
    }

    @Autowired
    @Qualifier("rockSinger")
    public void setRockSinger(Singer rockSinger) {
        this.rockSinger = rockSinger;
    }

    public void sing() {
        rockSinger.sing();
        danceSinger.sing();
    }
}

@Configuration
public class Config {
    @Bean
    @Lazy
    public Stage stage() {
        return new Stage(danceSinger());
    }
    @Bean
    public DanceSinger danceSinger() {
        return new DanceSinger();
    }
}
```



위와 같이 설정한 경우를 살펴보자.

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        GenericApplicationContext ctx = new AnnotationConfigApplicationContext(Config.class);
    }
}
```

이 경우에는 아무런 반응이 없다.



그럼 Stage를 사용하는 경우를 살펴보자.

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        GenericApplicationContext ctx = new AnnotationConfigApplicationContext(Config.class);
        Stage stage = (Stage) ctx.getBean("stage");
    }
}
```

이 경우에는 `dance!!!!` 와 같이 반응이 있다.