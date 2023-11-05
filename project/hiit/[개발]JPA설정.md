# JPA 설정

<br>

[관련 코드 바로가기](https://github.com/JNU-econovation/hiit-server/blob/main/src/main/java/com/hiit/api/repository/config/EntityJpaDataSourceConfig.java)



<br>



스프링으로 백엔드 개발을 한다면 JPA는 대부분 사용하고 있으리라 생각합니다.

이번 글에서는 Hiit 프로젝트에 JPA 관련 설정을 어떻게 하였는지 살펴봅시다.

<br>



## EntityJpaDataSourceConfig

JPA 관련 설정 역시 스프링의 AutoConfiguration을 사용하지 않고 설정을 구현하였습니다.

이렇게 설정한 이유는 Flyway 설정을 AutoConfiguration을 사용하지 않고 구현한 이유와 유사합니다.

이번 프로젝트에서 핵심 기능을 구현하는 것에만 그치지 않고 로그성 데이터를 수집하는 기능을 추가하려고 생각하고 있습니다.

저는 엔티티와 로그성 데이터는 성격이 다른 데이터라는 판단을 하였고 이를 분리하여 데이터베이스에 저장할 생각입니다.

이러한 이유로 JPA 설정을 스프링에 맡기는 것보다는 구현하는 게 더 좋으리라 판단하였고 구현하게 되었습니다.

<br>



### 무엇을 설정해 주어야 할까?

이번에도 우선 스프링이 JPA를 위해 어떤 설정을 대신하고 있는지 먼저 파악하기 위해 `org.springframework.boot.autoconfigure`를 살펴보았습니다.

우선 우리가 사용하는 JPA는 JDBC를 매핑한 ORM이다는 사실을 다시 한번 리마인드 해야 합니다.

즉, 우리가 찾아야 하는 설정은 JDBC와 JPA 2가지인 것입니다.

<br>



### JDBC



<img width="372" alt="스크린샷 2023-08-06 오후 12 00 56" src="https://github.com/belljun3395/jongjun/assets/102807742/c18ac963-9395-485d-8b89-0dfe6c41f856">

우선 JDBC에서 확인할 수 있는 클래스는 `DataSourceAutoConfiguration` 와 `DataSourceTransactionManagerAutoConfiguration` 입니다.

*다른 `*AutoConfiguration` 도 존재하지만 이번에는 데이터 소스에 관한 것만 살봅시다ㅎㅎㅎ*

<br>



```java
@AutoConfiguration(before = SqlInitializationAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import(DataSourcePoolMetadataProvidersConfiguration.class)
public class DataSourceAutoConfiguration { ... }
```

```java
@AutoConfiguration
@ConditionalOnClass({ JdbcTemplate.class, TransactionManager.class })
@AutoConfigureOrder(Ordered.LOWEST_PRECEDENCE)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceTransactionManagerAutoConfiguration { ... }
```

두 클래스 모두 해당 클래스 내에서 우리가 설정할 수 있는 빈은 없었습니다. (접근 제어자가 default인 설정들만 존재)

<br>



그럼 `@Import` 를 통해 불러온 설정도 확인해 봅시다.

`DataSourceAutoConfiguration`에서 불러온 `DataSourcePoolMetadataProvidersConfiguration`에는 우리가 설정할 수 있는 빈이 없었습니다.

결국 JDBC에서는 우리가 설정할 수 있는 빈이 하나도 없었습니다.

<br>



### JPA

그럼 이제 JPA 부분을 살펴 봅시다.



<img width="391" alt="스크린샷 2023-08-06 오후 12 13 11" src="https://github.com/belljun3395/jongjun/assets/102807742/9509e1dd-6900-4ca1-8343-9afca605bcc9">

JPA에서 우리가 확인해야할 클래스는 `HibernateJpaAutoConfiguration` 입니다.

```java
@AutoConfiguration(after = { DataSourceAutoConfiguration.class })
@ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class, SessionImplementor.class })
@EnableConfigurationProperties(JpaProperties.class)
@Import(HibernateJpaConfiguration.class)
public class HibernateJpaAutoConfiguration { }
```

`HibernateJpaAutoConfiguration` 역시 그 자체로는 설정할 수 있는 빈이 존재하지 않았습니다.

<br>



그럼 이제 `@Import`를 통해 불러온 `HibernateJpaConfiguration`를 살펴봅시다.

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(HibernateProperties.class)
@ConditionalOnSingleCandidate(DataSource.class)
class HibernateJpaConfiguration extends JpaBaseConfiguration { ... }
```

그런데 `HibernateJpaConfiguration`는 클래스의 접근 제어자 자체가 defualt 입니다.

<br>



하지만 `JpaBaseConfiguration`를 상속하고 있기에 이를 살펴보면 우리가 설정할 것이 무엇인지 알 수 있을 것입니다.

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(JpaProperties.class)
public abstract class JpaBaseConfiguration implements BeanFactoryAware { 
	@Bean
	@ConditionalOnMissingBean(TransactionManager.class)
	public PlatformTransactionManager transactionManager( ... ) { ... }
  
	@Bean
	@ConditionalOnMissingBean
	public JpaVendorAdapter jpaVendorAdapter() { ... }
  
	@Bean
	@ConditionalOnMissingBean
	public EntityManagerFactoryBuilder entityManagerFactoryBuilder( ... ) { ... }
  
	@Bean
	@Primary
	@ConditionalOnMissingBean({ LocalContainerEntityManagerFactoryBean.class, EntityManagerFactory.class })
	public LocalContainerEntityManagerFactoryBean entityManagerFactory( ... ) { ... }
}
```

이제야 우리가 설정해야 하는 빈을 찾았습니다.

하나씩 살펴봅시다.

<br>



#### PlatformTransactionManager

```java
@Bean
@ConditionalOnMissingBean(TransactionManager.class)
public PlatformTransactionManager transactionManager(
    ObjectProvider<TransactionManagerCustomizers> transactionManagerCustomizers) { .. }
```

`PlatformTransactionManager`는 데이터베이스 연동 기술에 따라 구현할 수 있도록 추상화된 인터페이스입니다.

*`@ConditionalOnMissingBean(TransactionManager.class)`는 `TransactionManager.class` 가 없는 경우 생성된다는 의미이고 이를 통해서도 추측할 수 있는 사실입니다.*

<br>



JPA를 사용하는 경우 `JpaTransactionManager`라는 구현체를 통해 이를 구현합니다.

*`JpaTransactionManager`는 전달받은 `EntityManagerFactory`를 이용해 트랜잭션을 관리한다고 합니다.*

<br>



#### JpaVendorAdapter

```java
@Bean
@ConditionalOnMissingBean
public JpaVendorAdapter jpaVendorAdapter() { .. }
```



![spring jpa hibernate sql](https://res.cloudinary.com/practicaldev/image/fetch/s--WNRGJoVB--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4lkueej7s72jhmsccjlw.png)

JPA도 JDBC를 편하게 사용할 수 있도록 도와주는 인터페이스이기에 구현체가 필요합니다.

위의 사진에서 볼 수 있듯 Hibernate는 JPA 구현체 중 하나이고 이번 프로젝트에서는 Hibernate를 사용합니다.

그렇기에 `JpaVendorAdapter`에는 Hibernate와 JPA를 이어줄 수 있는 `HibernateJpaVendorAdapter`를 구현해서 주면 된다는 것을 유추할 수 있습니다.

<br>



#### EntityManagerFactoryBuilder & LocalContainerEntityManagerFactoryBean

```java
JpaProperties properties;

@Bean
@ConditionalOnMissingBean
public EntityManagerFactoryBuilder entityManagerFactoryBuilder(JpaVendorAdapter jpaVendorAdapter,
    ObjectProvider<PersistenceUnitManager> persistenceUnitManager,
    ObjectProvider<EntityManagerFactoryBuilderCustomizer> customizers) { 
		EntityManagerFactoryBuilder builder = new EntityManagerFactoryBuilder(jpaVendorAdapter,
				this.properties.getProperties(), persistenceUnitManager.getIfAvailable());
		customizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
		return builder;
}

@Bean
@Primary
@ConditionalOnMissingBean({ LocalContainerEntityManagerFactoryBean.class, EntityManagerFactory.class })
public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder factoryBuilder) { 
		Map<String, Object> vendorProperties = getVendorProperties();
		customizeVendorProperties(vendorProperties);
		return factoryBuilder.dataSource(this.dataSource).packages(getPackagesToScan()).properties(vendorProperties)
				.mappingResources(getMappingResources()).jta(isJta()).build();
}
```

두 클래스 모두 `EntityManagerFactory`를 만들기 위한 클래스입니다.

코드를 보면 우선 `EntityManagerFactoryBuilder`를 통해 JPA요소와 `PersistenceUnitManager`관련 설정을 한 빌더를 만듭니다.

그리고 이 빌더를 `LocalContainerEntityManagerFactoryBean`에서 받아 데이터베이스 소스와 엔티티 스캔 패키지를 설정해 주는 것을 확인할 수 있습니다.

<br>



### 구현

JPA를 구성하는 각 요소에 대한 설명을 맞쳤고 이를 종합하여 해당 프로젝트에서는 아래와 같이 구현하였습니다.

```java
@Configuration
@EnableJpaAuditing
@EnableTransactionManagement
@EnableJpaRepositories(
		basePackages = EntityJpaDataSourceConfig.BASE_PACKAGE,
		transactionManagerRef = EntityJpaDataSourceConfig.TRANSACTION_MANAGER_NAME,
		entityManagerFactoryRef = EntityJpaDataSourceConfig.ENTITY_MANAGER_FACTORY_NAME)
public class EntityJpaDataSourceConfig {

	@Bean(name = DATASOURCE_NAME)
	@ConfigurationProperties(prefix = BASE_PROPERTY_PREFIX + ".datasource")
	public DataSource dataSource() {
		return DataSourceBuilder.create().build();
	}

	@Bean(name = JPA_PROPERTIES_NAME)
	@ConfigurationProperties(prefix = BASE_PROPERTY_PREFIX + ".jpa")
	public JpaProperties jpaProperties() {
		return new JpaProperties();
	}

	@Bean(name = HIBERNATE_PROPERTIES_NAME)
	@ConfigurationProperties(prefix = BASE_PROPERTY_PREFIX + ".jpa.hibernate")
	public HibernateProperties hibernateProperties() {
		return new HibernateProperties();
	}

	@Bean(name = JPA_VENDOR_ADAPTER_NAME)
	public JpaVendorAdapter jpaVendorAdapter() {
		return new HibernateJpaVendorAdapter();
	}

	@Bean(name = ENTITY_MANAGER_FACTORY_BUILDER_NAME)
	public EntityManagerFactoryBuilder entityManagerFactoryBuilder(
			@Qualifier(value = JPA_VENDOR_ADAPTER_NAME) JpaVendorAdapter jpaVendorAdapter,
			@Qualifier(value = JPA_PROPERTIES_NAME) JpaProperties jpaProperties,
			ObjectProvider<PersistenceUnitManager> persistenceUnitManager) {

		Map<String, String> jpaPropertyMap = jpaProperties.getProperties();
		return new EntityManagerFactoryBuilder(
				jpaVendorAdapter, jpaPropertyMap, persistenceUnitManager.getIfAvailable());
	}

	@Bean(name = ENTITY_MANAGER_FACTORY_NAME)
	public LocalContainerEntityManagerFactoryBean entityManagerFactory(
			@Qualifier(value = DATASOURCE_NAME) DataSource dataSource,
			@Qualifier(value = ENTITY_MANAGER_FACTORY_BUILDER_NAME) EntityManagerFactoryBuilder builder) {
		Map<String, String> jpaPropertyMap = jpaProperties().getProperties();
		Map<String, Object> hibernatePropertyMap =
				hibernateProperties().determineHibernateProperties(jpaPropertyMap, new HibernateSettings());
		return builder
				.dataSource(dataSource)
				.properties(hibernatePropertyMap)
				.persistenceUnit(PERSIST_UNIT)
				.packages(BASE_PACKAGE)
				.build();
	}

	@Bean(name = TRANSACTION_MANAGER_NAME)
	public PlatformTransactionManager transactionManager(
			@Qualifier(ENTITY_MANAGER_FACTORY_NAME) EntityManagerFactory emf) {
		return new JpaTransactionManager(emf);
	}
}
```

<br>



### 어노테이션

`EntityJpaDataSourceConfig` 위에 추가적인 어노테이션 때문에 당황했을 수 있을꺼라 생각합니다.

이 역시 하나씩 알아봅시다.

<br>



#### @EnableJpaAuditing

```java
@Inherited
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(JpaAuditingRegistrar.class)
public @interface EnableJpaAuditing { ... }
```

`@EnableJpaAuditing`을 보면 `JpaAuditingRegistrar` 을 불러 등록하는 것을 볼 수 있습니다.

이는 우리가 `@EnableJpaAuditing`가 어떻게 등록되는지 확인하려면 `JpaAuditingRegistrar`를 확인하면 된다는 뜻입니다.

<br>



```java
class JpaAuditingRegistrar extends AuditingBeanDefinitionRegistrarSupport { 

  ...
    
	@Override
	public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry registry) {

		Assert.notNull(annotationMetadata, "AnnotationMetadata must not be null!");
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null!");

		registerBeanConfigurerAspectIfNecessary(registry);
		super.registerBeanDefinitions(annotationMetadata, registry);
		registerInfrastructureBeanWithId(
				BeanDefinitionBuilder.rootBeanDefinition(AuditingBeanFactoryPostProcessor.class).getRawBeanDefinition(),
				AuditingBeanFactoryPostProcessor.class.getName(), registry);
	}
}
```

`JpaAuditingRegistrar.registerBeanDefinitions`을 통해 등록과정이 시작됩니다.

<br>



보다 구체적인 등록은 `JpaAuditingRegistrar.registerAuditListenerBeanDefinition` 에서 일어납니다.

```java
class JpaAuditingRegistrar extends AuditingBeanDefinitionRegistrarSupport { 

  ...
    
	@Override
	protected void registerAuditListenerBeanDefinition(BeanDefinition auditingHandlerDefinition,
			BeanDefinitionRegistry registry) {

		if (!registry.containsBeanDefinition(JPA_MAPPING_CONTEXT_BEAN_NAME)) {
			registry.registerBeanDefinition(JPA_MAPPING_CONTEXT_BEAN_NAME, 
					new RootBeanDefinition(JpaMetamodelMappingContextFactoryBean.class));
		}

		BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(AuditingEntityListener.class);
		builder.addPropertyValue("auditingHandler",
				ParsingUtils.getObjectFactoryBeanDefinition(getAuditingHandlerBeanName(), null));
		registerInfrastructureBeanWithId(builder.getRawBeanDefinition(), AuditingEntityListener.class.getName(), registry);
	}
}

/** ================================================================================================ */
public abstract class AuditingBeanDefinitionRegistrarSupport implements ImportBeanDefinitionRegistrar {
  
  ...
    
	protected void registerInfrastructureBeanWithId(AbstractBeanDefinition definition, String id,
			BeanDefinitionRegistry registry) {

		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(id, definition);
	}
}
```

부모 클래스인 `AuditingBeanDefinitionRegistrarSupport`의 `registerInfrastructureBeanWithId` 메서드를 통해 `BeanDefinitionRegistry`에 등록하여 `@EnableJpaAuditing`을 이용할 수 있도록 도와줍니다.

<br>



<img width="838" alt="스크린샷 2023-08-06 오후 4 38 39" src="https://github.com/belljun3395/jongjun/assets/102807742/b692dc07-cf4b-4037-987b-6b717ef83f12">

위의 사진은 `registry.registerBeanDefinition(id, definition)`에 브레이크 포인트를 걸고 디버깅을 했을 때 나오는 결과입니다.

조금 더 구체적인 과정이 궁금하시다면 디버깅 후 스프링의 동작을 추적해보시는 것을 추천합니다!

<br>



#### @EnableJpaRepositories

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(JpaRepositoriesRegistrar.class)
public @interface EnableJpaRepositories { ... }
```

`EnableJpaRepositories` 역시 `JpaRepositoriesRegistrar` 불러 등록하는 것을 볼 수 있습니다.

<br>



그래서 `JpaRepositoriesRegistrar`를 살펴보면 아래와 같습니다.

```java
class JpaRepositoriesRegistrar extends RepositoryBeanDefinitionRegistrarSupport {

	@Override
	protected Class<? extends Annotation> getAnnotation() {
		return EnableJpaRepositories.class;
	}

	@Override
	protected RepositoryConfigurationExtension getExtension() {
		return new JpaRepositoryConfigExtension();
	}
}

/** ================================================================================================ */

public abstract class RepositoryBeanDefinitionRegistrarSupport
		implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
  ...
	@Override
	public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry,
			BeanNameGenerator generator) {
		Assert.notNull(metadata, "AnnotationMetadata must not be null");
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		Assert.notNull(resourceLoader, "ResourceLoader must not be null");

		// Guard against calls for sub-classes
		if (metadata.getAnnotationAttributes(getAnnotation().getName()) == null) {
			return;
		}

		AnnotationRepositoryConfigurationSource configurationSource = new AnnotationRepositoryConfigurationSource(metadata,
				getAnnotation(), resourceLoader, environment, registry, generator);

		RepositoryConfigurationExtension extension = getExtension();
		RepositoryConfigurationUtils.exposeRegistration(extension, registry, configurationSource);

		RepositoryConfigurationDelegate delegate = new RepositoryConfigurationDelegate(configurationSource, resourceLoader,
				environment);

		delegate.registerRepositoriesIn(registry, extension);
	}
}
```

설정 클래스 정보를 가지고 있는 `configurationSource`을 통해 어노테이션을 통한 설정한 클래스를 통해 설정중이라는 것을 `RepositoryConfigurationDelegate`에 알리고 `delegate` 를 만듭니다.

```java
public RepositoryConfigurationDelegate(RepositoryConfigurationSource configurationSource,
    ResourceLoader resourceLoader, Environment environment) {

  this.isXml = configurationSource instanceof XmlRepositoryConfigurationSource;
  boolean isAnnotation = configurationSource instanceof AnnotationRepositoryConfigurationSource;

  Assert.isTrue(isXml || isAnnotation,
      "Configuration source must either be an Xml- or an AnnotationBasedConfigurationSource");
  Assert.notNull(resourceLoader, "ResourceLoader must not be null");

  this.configurationSource = configurationSource;
  this.resourceLoader = resourceLoader;
  this.environment = defaultEnvironment(environment, resourceLoader);
  this.inMultiStoreMode = multipleStoresDetected();
}
```

그리고 이 `delegate`에게 `registry` 와 `extension`을 전달하여 `registry` 에 레퍼지토리 관련 빈을 등록합니다.

<br>



조금 더 정확히는 아래 코드에서 등록합니다.

```java
public abstract class RepositoryConfigurationExtensionSupport implements RepositoryConfigurationExtension {
	public <T extends RepositoryConfigurationSource> Collection<RepositoryConfiguration<T>> getRepositoryConfigurations(
			T configSource, ResourceLoader loader, boolean strictMatchesOnly) {
    ...
  }
}
```

<br>



<img width="836" alt="스크린샷 2023-08-06 오후 5 13 19" src="https://github.com/belljun3395/jongjun/assets/102807742/a6aef87e-1b9d-4e6d-ad64-e808f59f1de1">

위 사진도 `RepositoryConfigurationExtensionSupport.getRepositoryConfigurations`의 114라인에 브레이크 포인트를 설정하고 찍은 결과입니다.

이 역시 조금 더 구체적인 과정이 궁금하시다면 디버깅 후 스프링의 동작을 추적해보시는 것을 추천합니다!

<br>



#### @EnableTransactionManagement

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement { ... }
```

`@EnableTransactionManagement`을 역시 `TransactionManagementConfigurationSelector` 을 불러 등록하는 것을 볼 수 있습니다.

<br>



이때 `TransactionManagementConfigurationSelector`를 조금 더 자세히 살펴봅시다.

```java
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(),
						ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {determineTransactionAspectClass()};
			default:
				return null;
		}
	}

	private String determineTransactionAspectClass() {
		return (ClassUtils.isPresent("javax.transaction.Transactional", getClass().getClassLoader()) ?
				TransactionManagementConfigUtils.JTA_TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME :
				TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME);
	}

}
```

이전 클래스들은 `ImportBeanDefinitionRegistrar`를 구현하고 있었는데 `TransactionManagementConfigurationSelector` 는 그렇지 않습니다.

<br>



눈에 뛰는 메서드는 `selectImports`로 클래스 이름을 반환하고 있습니다.

이렇게 반환한 클래스 이름은 `ConfigurationClassPostProcessor`의 `postProcessBeanDefinitionRegistry`를 수행하며 실행되는 `processConfigBeanDefinitions` 메서드 내에서 `ConfigurationClassParser` 타입의 `parser`에 의해 수집됩니다.

이렇게 수집된 클래스 이름을 통해 빈으로 등록합니다.

아래 코드로 확인해 봅시다.

```java
public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor,
  PriorityOrdered, ResourceLoaderAware, ApplicationStartupAware, BeanClassLoaderAware, EnvironmentAware {

    ...
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
     ...
		processConfigBeanDefinitions(registry);
	}

	public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    ...
		ConfigurationClassParser parser = new ConfigurationClassParser(
				this.metadataReaderFactory, this.problemReporter, this.environment,
				this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    ...
    Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
    ....

		this.reader.loadBeanDefinitions(configClasses);
     ...
	}
}
```

코드 상으로는 `this.reader.loadBeanDefinitions(configClasses)`에서 빈으로 등록됩니다.

<br>



```java
class ConfigurationClassBeanDefinitionReader { 
  ...
  public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
		TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();
		for (ConfigurationClass configClass : configurationModel) {
			loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
		}
	}

	private void loadBeanDefinitionsForConfigurationClass(
			ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {

		if (trackedConditionEvaluator.shouldSkip(configClass)) {
			String beanName = configClass.getBeanName();
			if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
				this.registry.removeBeanDefinition(beanName);
			}
			this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
			return;
		}

		if (configClass.isImported()) {
			registerBeanDefinitionForImportedConfigurationClass(configClass);
		}
		for (BeanMethod beanMethod : configClass.getBeanMethods()) {
			loadBeanDefinitionsForBeanMethod(beanMethod);
		}

		loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
		loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
	}
  ...
}
```

<br>



<img width="779" alt="스크린샷 2023-08-06 오후 6 55 30" src="https://github.com/belljun3395/jongjun/assets/102807742/e4788186-7327-42b6-b44b-a1b22184ead2">

위 사진은 빈이 다 로드되고 난 이후인 `ConfigurationClassPostProcessor`의 344번째 줄에 브레이크를 찍고 디버깅한 결과입니다.

<br>



## 마치며

사실 글을 적으면서 이 정도까지 적을 생각을 처음에는 하지 않았습니다.

그런데 적다 보니 부족한 부분이 보이고 보충하다 보니 글이 엄청나게 길어졌네요…. 하하

그치만 평소에 아무런 의심 없이 추가하던 설정들이 어떻게 설정되는지 알아볼 수 있어 좋은 경험이었습니다.

감사합니다.
