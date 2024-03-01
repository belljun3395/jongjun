# AutoConfiguration 대신 자바 구성을 설정하며 학습하기



우리가 스프링 부트(이하 부트)를 사용할 때 `yml` 파일을 통해 편하게 관련 설정을 하곤 하는데 그러한 일이 어떻게 가능한지 생각해 보신 적 있나요?

부트는 `spring-boot-autoconfigure`에서 정의된 `AutoConfiguration` 클래스들을 통해 우리가 편리하게 관련 설정을 할 수 있도록 돕고 있습니다.

예를 들어 우리가 부트를 통해 프로젝트를 수행하며 datasource를 위해 작성하는 아래와 같은 설정은 `DataSourceAutoConfiguration` 클래스가 우리를 돕고 있습니다.

```yml
spring:
 datasource:
 jdbcUrl: ${ DATASOURCE_URL }
 username:${ DATASOURCE_USERNAME }
 password: ${ DATASOURCE_PASSWORD }
 driver-class-name: com.mysql.cj.jdbc.Driver
```



부트는 위의 예제 외에도 많은 것을 도와주고 있습니다.

하지만 저는 편리함보다 내가 설정하는 것에 대한 의미를 조금 더 잘 알고 싶었고 부트가 도와주는 것을 직접 구현해 보며 조금 더 많은 부트에 대한 제어권을 가지고 싶었습니다.



## DataSourceAutoConfiguration을 자바 구성으로 구현해 보기

아마 처음 자바 구성을 직접 구현하려면 어디서부터 시작해야할지 전혀 감이 안 잡힐 것입니다.

이번 글에서 위의 데이터소스와 관련된 `yml` 파일을 자바 구성으로 구현해 보며 어떠한 것을 학습할 수 있는지 확인해 봅시다.



우선 `spring-boot-autoconfigure`에서 데이터소스와 관련된 자바 구성이 존재하는지 확인해 봅시다.

데이터소스와 관련된 `AutoConfiguration`으로는`org.springframework.boot.autoconfigure.jdbc` 아래의`DataSourceAutoConfiguration`가 있습니다.



`DataSourceAutoConfiguration`를 보면 다양한 어노테이션이 존재하는 것을 확인할 수 있습니다.

우리는 이를 통해 부트가 우리를 대신해 어떤 일을 도와주고 있는지 알 수 있고 `DataSourceAutoConfiguration`을 사용하지 않는다면 데이터소스를 설정하기 위해 어떠한 작업을 해야 하는지 파악할 수 있습니다.

- `@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })`
- `@EnableConfigurationProperties(DataSourceProperties.class)`

`EmbeddedDatabaseType.class`는 Enum 클래스로 우리가 정의할 필요가 없지만 `DataSource.class`의 경우 인터페이스로 구현체가 필요합니다.

`DataSourceAutoConfiguration`에서는 `DataSourceProperties.class`,`@ConfigurationProperties(prefix = "spring.datasource")`를 사용하고 있기에 위의 `yml`에서도 `spring.datasource`로 시작하는 설정을 작성하였음을 알 수 있었습니다.

직접 자바 구성을 작성하면서는 동일한 `prefix`를 사용할 수도, 별도의 `prefix`를 사용할 수도 있습니다.

이는 본인의 프로젝트 상황에 따라 달라질 수 있을 것 같습니다.

저는 여러 데이터소스가 필요한 경우 별도의 `prefix`를 정의하여 사용하고 하나의 데이터 소스만 사용한다면 기본 `prefix`와 동일하게 정의하여 사용하고 있습니다.

소소하지만 이 역시 부트에게서 가져온 소소한 우리의 제어권 아닐까요?



그렇게 직접 구현한 자바 구성은 아래와 같습니다.

```java
public class JpaDataSourceConfig {
 ...
	@Bean(name = DATASOURCE_PROPERTIES)
	@ConfigurationProperties(prefix = "spring.datasource")
	public DataSourceProperties dataSourceProperties() {
	return new DataSourceProperties();
	}

	@Bean(name = DATASOURCE)
	public DataSource dataSource(ObjectProvider> dataSourceProperties) {
	DataSourceProperties properties = dataSourceProperties.getIfAvailable();
 return DataSourceBuilder.create()
	.url(properties.getUrl())
	.username(properties.getUsername())
	.password(properties.getPassword())
	.driverClassName(properties.getDriverClassName())
	.build();
	}
}
```



## 학습할 수 있는 것

우선 프로젝트에 어떠한 클래스들이 사용되고 있는지 알 수 있습니다.

위의 데이터소스를 설정하는 예시에서는 `DataSource`, `DataSourceProperties`와 같은 클래스가 필요하다는 것을 알 수 있었습니다.

그러한 클래스들을 살펴보며 어떠한 설정을 내가 할 수 있고 어떠한 구현체가 존재하는지 살펴보다 보면 조금 더 다양한 것을 시도하고 제어할 수 있지 않을까 생각합니다.



그리고 빈을 어떻게 설정할 수 있는지 공부할 수 있습니다.

`AutoConfiguration` 클래스 위에 존재하는 많은 어노테이션를 보며 공부하다 보면 단순히 `@Configuration` / `@Bean` 조합 혹은 `@Component`를 통해 빈을 등록할 때보나 보다 세세한 설정을 할 수 있습니다.



## 마치며

백지에서 `AutoConfiguration`을 자바 구성으로 대체 해보자는 생각을 할 수 있지는 않았습니다.

좋은 기회로 합류하였던 프로젝트에서 [JpaDataSourceConfig](https://github.com/depromeet12th/three-days-server/blob/develop/data/src/main/java/com/depromeet/threedays/data/JpaDataSourceConfig.java)를 자바 구성으로 작성한 코드가 존재하였습니다.

저는 해당 코드를 보며 [Gradle Plugin](https://github.com/depromeet12th/three-days-server/blob/develop/build.gradle)으로 설정되어 있던 Flyway 관련 설정을 동일하게 자바 설정으로 바꾸면 좋지 않을까 생각하였고 이를 바꾸기 위해 `JpaDataSourceConfig`를 보며 공부한 것이 시작이었습니다.

이처럼 주어진 코드에서도 어떠한 개선점이 있을지 고민해 본다면 강의나 교육 프로그램이 아니더라도 자신만의 성장 포인트를 찾을 수 있을 것으로 생각합니다.

앞으로 코드를 볼 때는 당연한 것은 없고 어떠한 것을 조금 더 편리하게 개선할 수 있을지 고민하며 코드를 본다면 우리가 반복해서 작성하던 코드나 의미 없어 보였던 코드에서도 다른 의미를 만들어낼 수 있지 않을까요?

감사합니다.



---

**AutoConfiguration 학습에 도움이 될 링크**

- [스프링 부트의 Autoconfiguration 원리 및 만들어 보기](https://donghyeon.dev/spring/2020/08/01/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%EC%9D%98-AutoConfiguration%EC%9D%98-%EC%9B%90%EB%A6%AC-%EB%B0%8F-%EB%A7%8C%EB%93%A4%EC%96%B4-%EB%B3%B4%EA%B8%B0/)
- [SpringBoot AutoConfiguration을 대하는 자세](https://tecoble.techcourse.co.kr/post/2021-10-14-springboot-autoconfiguration/)

