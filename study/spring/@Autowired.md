## @Autowired


```java
@Target({CONSTRUCTOR,METHOD,PARAMETER,FIELD,ANNOTATION_TYPE})
@Retention(RUNTIME)
@Documented
public @interface Autowired
```

생성자, 필드, setter 메서드 또는 config 메서드가 Spring의 DI 주입 기능에 의해 autowired 되도록 한다.

이는 JSR-330 Inject 어노테이션의 대안으로, required 그리고 optional 의미를 추가한다.



### Autowired Constructors

주어진 Bean 클래스의 생성자 하나만 이 어노테이션을 true로 설정된 required() 속성으로 선언할 수 있다.

이는 Spring Bean으로 사용될 때 생성자를 autowired로 지정함을 나타낸다. 

또한 필수 속성이 true로 설정된 경우 단일 생성자만 @Autowired로 어노테이션할 수 있다. 

필수가 아닌 여러 생성자가 어노테이션을 선언하면 자동 와이어링 후보로 간주한다. 

Spring 컨테이너에서 일치하는 Bean으로 충족할 수 있는 종속성 수가 가장 많은 생성자가 선택된다. 

후보 중 어느 것도 만족할 수 없는 경우 기본 생성자(있는 경우)가 사용된다. 

마찬가지로, 클래스가 여러 생성자를 선언하지만 @Autowired로 어노테이션된 생성자가 없는 경우 기본 생성자(있는 경우)가 사용된다. 

클래스가 처음부터 하나의 생성자만 선언하는 경우, @Autowired를 달지 않더라도 항상 해당 생성자가 사용된다. 

@Autowired가 달린 생성자는 public일 필요는 없다.



### Autowired Fields

필드는 구성 메서드가 호출되기 전에 빈을 구성한 직후에 주입된다. 

이러한 구성 필드는 public일 필요가 없다.



하지만 필드 주입의 경우 여러 단점이 있어 사용을 권장하지 않는다.

1. **단일 책임 원칙을 위반할 가능성이 높아진다.**
2. **필드 주입을 사용하면 어떤 타입의 의존성이 실제로 필요한지, 의존성이 필수인지 여부가 명확하지 않을 수 있다.**
3. 필드 주입은 final 필드에서 사용할 수 없다.
4. 테스트 코드 작성 시 수동으로 의존성을 주입해주어야 해서 작성하기 어렵다.

4번의 경우 의존성을 수동으로 주입하는 방법은 리플렉션을 활용하는 방법이다. **(리플렉션에 대해서는 추후 정리 예정)**

```java
Target t = new Target();
Field f = t.getClass().getDeclaredField("private_field"); // private로 선언된 field 가져오기
f.setAccessible(true); // private 선언을 해지하고 접근
f.set(t, new Field()); // t의 f에 new Field()를 통해 의존성 주입
f.setAccessible(false); // private 선어 복구
```



### Autowired Methods

구성 메서드는 설정에 필요한 임의의 이름과 임의의 수의 인수를 가질 수 있다.
이러한 각 인수는 스프링 컨테이너에서 일치하는 빈으로 자동 배정된다. 일반적인 구성 방법의 하나로 setter 메서드가 있다.
이러한 구성 메서드는 public일 필요는 없다.



### Autowired Parameters

Spring Framework 5.0 이후 @AutoWired는 개별 메서드 또는 생성자 매개 변수에 대해 기술적으로 선언할 수 있지만 프레임워크의 대부분은 이러한 선언을 무시한다.



### Multiple Arguments and 'required' Semantics

다중 인수 생성자 또는 메서드의 경우 `required()` 특성을 모든 인수에 적용할 수 있다. 
개별 매개변수는 기본 '필수' 의미를 재정의하는 Java-8 스타일 선택사항 또는 Spring Framework 5.0 기준으로 @Nullable 또는 not-Null 매개변수 유형으로 선언될 수 있습니다.



### Autowiring Arrays, Collections, and Maps

배열, 컬렉션 또는 맵 종속성 유형의 경우 컨테이너는 **선언된 값 유형과 일치하는 모든 빈을 자동 배정한다.** 
이러한 목적을 위해 **맵 키는 해당 빈 이름으로 확인되는 String으로 선언되어야 한다.**
즉, @Autowired 애너테이션이 배열, 컬렉션, 맵을 해당 컬렉션의 값 타입에서 파생된 대상 빈 타입을 가져와 처리하는 것이다.

2가지 유형의 예제 확인해 보자.
우선 Arrays, Collections 그리고 Map에 **특정한 빈 타입이 담긴 유형**이다.

```java
@Autowired
List<Singer> singersList;
@Autowired
Map<String, Singer> singersMap;
```

위의 예제에서는 singersList에 Singer 타입의 빈이 들어간다.
singersMap에는 키로는 Singer 타입 빈의 이름, 값으로는 Singer 타입 빈이 들어간다.

조금 더 구체적으로 생각해보자.
Singer 클래스를 상속받아  CountrySinger, DanceSinger, RapSinger와 같은 클래스를 만들었다고 생각해보자.

이러한 경우에 singersList에는 위의 클래스들이
singersMap에는 아래와 같은 값이 들어가게 된다.

```
// key, value
countrySinger, CountrySinger 빈
danceSinger, DanceSinger 빈
rapSinger, RapSinger 빈
```

다음으로 정말 Arrays, Collections 그리고 Map 타입의 빈인 경우다.
*예제 출처 : https://www.baeldung.com/spring-injecting-collections*

```java
public class CollectionsBean {

    @Autowired
    private List<String> nameList;

    public void printNameList() {
        System.out.println(nameList);
    }
}

@Configuration
public class CollectionConfig {
	@Bean
	public CollectionBean getCollectionsBean() {
		return new CollectionsBean();
	}

	@Bean
	public List<String> nameList() {
		return Arrays.asList(“John”, “Adam”, “Harry”);
	}
}
```



### Not supported in BeanPostProcessor or BeanFactoryPostProcessor

실제 주입은 BeanPostProcessor를 통해 수행되므로 @Autowired를 사용하여 BeanPostProcessor 또는 BeanFactoryPostProcessor 유형에 참조를 주입할 수 없다. 
