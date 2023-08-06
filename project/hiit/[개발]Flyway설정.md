# Flyway 설정

<br>

[관련 코드 바로가기](https://github.com/belljun3395/hiit-server/blob/main/src/main/java/com/hiit/api/infra/flyway/config/EntityFlywayConfig.java)



<br>



flyway는 데이터베이스의 형상 관리를 목적으로 사용하는 툴입니다.

그리고 데이터베이스 스키마를 코드로 옮기는 기능을 제공합니다.

<br>



아직 형상 관리에 대한 필요성은 느낀 경험은 없지만 스키마를 코드로 옮기는 기능을 제공하는 것은 큰 매력으로 다가왔습니다.

Flyway에 대한 학습은 아래 링크로 대체하고 이번 프로젝트에 Flyway 관련 설정을 어떻게 하였는지 살펴봅시다.

테코톡 : https://youtu.be/pxDlj5jA9z4

블로그 : https://dallog.github.io/dallog-flyway/

<br>



## EntityFlywayConfig

우선 엔티티 관련 데이터베이스를 관리하는 EntityFlywayConfig를 만들었습니다.

일반 적인 경우 아래와 같은 `.yml` 파일만으로 Flyway 관련 설정을 할 수 있습니다.

```yml
spring:
  flyway:
    url: ...
    user: ...
    password: ...
    ...등
```

이는 FlywayAutoConfiguration라는 `org.springframework.boot.autoconfigure`에서 제공하는 AutoConfiguration 중 하나의 도움으로 가능한 것입니다. 

<br>



<img width="395" alt="스크린샷 2023-08-06 오전 1 32 13" src="https://github.com/belljun3395/jongjun/assets/102807742/7fb3b94d-731e-448b-88a1-9051e83eda93">

위 사진은 `org.springframework.boot.autoconfigure` 패키지를 캡처한 사진입니다.

Flyway 말고도 다양한 패키지가 존재한 것을 확인할 수 있고 `*AutoConfiguration`의 클래스를 확인하면 어떠한 설정을 스프링이 대신해서 도와주고 있는지 확인할 수 있습니다.

<br>



### 문제

하지만 이번 프로젝트의 경우 추후 배치와 같은 기능을 추가할 가능성이 크고 이때 배치와 관련된 기록과 엔티티와 관련된 기록은 서로 다른 데이터베이스로 분리되도록 설정하려 합니다.

이는 하나의 DataSource가 아닌 여러 개의 DataSource를 사용한다는 의미고 FlywayAutoConfiguration는 하나의 DataSource에 대한 설정만 제공하여 이를 사용할 수 없었습니다.

<br>



### 해결 방법

그러면 FlywayAutoConfiguration에서 만들어 주는 빈들을 직접 만들면 되지 않을까? (우리는 개발자니까..!!!)

그럼 FlywayAutoConfiguration을 조금 자세히 살펴봅시다.

<br>



#### 클래스

```java
@AutoConfiguration(after = { DataSourceAutoConfiguration.class, JdbcTemplateAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class })
@ConditionalOnClass(Flyway.class)
@Conditional(FlywayDataSourceCondition.class)
@ConditionalOnProperty(prefix = "spring.flyway", name = "enabled", matchIfMissing = true)
@Import(DatabaseInitializationDependencyConfigurer.class)
public class FlywayAutoConfiguration {
  ...
}
```

<br>



#### 빈 목록

```java
@Bean
@ConfigurationPropertiesBinding
public StringOrNumberToMigrationVersionConverter stringOrNumberMigrationVersionConverter() { ... }

@Bean
public FlywaySchemaManagementProvider flywayDefaultDdlModeProvider(ObjectProvider<Flyway> flyways) { ... }

@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(JdbcUtils.class)
@ConditionalOnMissingBean(Flyway.class)
@EnableConfigurationProperties(FlywayProperties.class)
public static class FlywayConfiguration {
  	@Bean
		public Flyway flyway(...) { ... }
  
    @Bean
		@ConditionalOnMissingBean
		public FlywayMigrationInitializer flywayInitializer(...) { ... }
}
```

<br>



우선 클래스 부분에서 설정해야 할 것을 구분해 봅시다.

라고 글을 적었지만 그럴 필요가 없습니다.

이는 StringOrNumberToMigrationVersionConverter과 FlywaySchemaManagementProvider 때문입니다.

해당 클래스들의 선언을 자세히 보면 아래와 같습니다.

```java
static class StringOrNumberToMigrationVersionConverter { ... }

class FlywaySchemaManagementProvider { ... }
```

모두 패키지 외부에서는 사용할 수 없는 것을 확인할 수 있습니다.

이는 FlywayAutoConfiguration를 완전히 제외하고 설정할 수 없다는 뜻입니다.

완전히 우리가 Flyway에 관한 설정을 제어하지는 못하지만, 우리가 해야 할 것은 간단해졌습니다.

Flyway 그리고 FlywayMigrationInitializer를 잘 만들어 제공해 주는 것뿐입니다.

<br>



### 구현

그렇게 완성한 코드는 아래와 같습니다.

```java
@Configuration
public class EntityFlywayConfig {
  
	@Bean(name = FLYWAY)
	public Flyway flyway(
			@Qualifier(value = FLYWAY_CONFIGURATION)
					org.flywaydb.core.api.configuration.Configuration configuration) {
		return new Flyway(configuration);
	}

	@Profile({"!local"})
	@Bean(name = FLYWAY_VALIDATE_INITIALIZER)
	public FlywayMigrationInitializer flywayValidateInitializer(
			@Qualifier(value = FLYWAY) Flyway flyway) {
		return new FlywayMigrationInitializer(flyway, Flyway::validate);
	}

	@Profile({"!local"})
	@Bean(name = FLYWAY_MIGRATION_INITIALIZER)
	public FlywayMigrationInitializer flywayMigrationInitializer(
			@Qualifier(value = FLYWAY) Flyway flyway) {
		return new FlywayMigrationInitializer(flyway, Flyway::migrate);
	}

	@Bean(name = FLYWAY_PROPERTIES)
	@ConfigurationProperties(prefix = BASE_PROPERTY_PREFIX) // 원하는 prefix 설정을 위해 꼭 빈으로 설정할 필요 있음
	public FlywayProperties flywayProperties() {
		return new FlywayProperties();
	}

	@Bean(name = FLYWAY_CONFIGURATION)
	public org.flywaydb.core.api.configuration.Configuration configuration(
			@Qualifier(value = FLYWAY_PROPERTIES) FlywayProperties flywayProperties,
			@Qualifier(value = JpaDataSourceConfig.DATASOURCE_NAME) DataSource dataSource) {

		FluentConfiguration configuration = new FluentConfiguration();
		configuration.dataSource(dataSource);
		flywayProperties.getLocations().forEach(configuration::locations);
		return configuration;
	}
}
```

<br>



### 로컬 프로파일에서는?

위의 코드를 보면 `@Profile({"!local"})`를 통해 로컬 환경에서는 FlywayMigrationInitializer 빈을 올리지 않은 것을 확인할 수 있습니다.

크게 2가지 이유로 위와 같은 선택을 하였습니다.

우선 로컬 환경은 말 그대로 우리가 개발하는 환경입니다.

이때 FlywayMigrationInitializer을 통해 migrate와 validate를 진행한다면 SQL을 변경 때마다 작성하여야 하고 이는 개발 생산성에 낮춘다고 판단하였습니다.

그렇기에 로컬 환경에서는 자유롭게 개발하고 이후 환경부터 SQL을 작성한 후 진행하는 것이 좋다고 판단하였습니다.

<br>



두 번째 이유는 엔티티 클래스에도 테이블의 칼럼에 대한 정보를 작성하는 것을 어느 정도 강제하기 위해서입니다.

사실 Flyway와 같은 툴을 사용해 스키마를 작성한다면 엔티티의 칼럼에 `@ManyToOne`, `@Column(nullable =false)`와 같은 칼럼에 대한 정보를 기재할 필요가 없습니다.

그렇게 되면 칼럼에 대한 정보를 확인할 때 항상 데이터베이스를 확인해야 하는 번거로움이 추가됩니다.

결정적으로 Hiberanate에서도 `spring.jpa.hibernate.ddl-auto=validate`와 같은 설정을 통해 검증할 수 있는데 이를 정확히 수행할 수 없게 됩니다.

<br>



## 마치며

최근 어떤 영상을 보며 문제는 항상 대수롭지 않게 넘어가는 것에서 시작한다는 말을 인상 깊게 들었습니다.

Flyway를 사용하지 않아도 문제 없이 데이터 베이스를 구축할 수 있을 것입니다.

하지만 그 과정이 항상 매끄럽지는 않겠죠?

그러한 것을 대수롭게 넘기지 않고 해결하고자 하면 Flyway의 필요성을 느낄 수 있을 것이라 생각합니다.

감사합니다.