## 스프링에서 JPA2로 데이터 액세스하기

스프링 애플리케이션에 하이버네이트르 적용하는 또 다른 방법은 **하이버네이트를 표준 자바 퍼시스턴스 API의 퍼시스턴스 제공자로 사용하는 것이다.**



### JPA 2.1 소개

JPA 2.1 사양의 목적은 JSE와 JEE 환경에서 ORM 프로그래밍 모델을 표준화하는 것이다.

JPA 2.1 사양은 JPA 퍼시스턴스 제공자가 구현해야하는 개념, 애너테이션, 인터페이스, 기타 서비스로 이루어진 표준 집합을 정의한다.



JPA 내부에서 핵심은 **EntityManager 인터페이스**이며, 인터페이스 구현체는 **EntityManagerFactory 타입의 팩터리**에서 얻을 수 있다.

EntityManager의 주 작업은 **퍼시스턴스 컨텍스트를  관리하는 것으로 퍼시스턴스 컨텍스트가 관리하는 모든 엔터티 인스턴스는 퍼시스턴스 컨텍스트에 저장된다.**

EntityManager의 구성은 퍼시스턴스 유닛으로 정의되며 애플리케이션에는 하나 이상의 퍼시스턴스 유닛을 정의할 수 있다.

하이버네이트를 사용할 대와 비교하면 퍼시스턴스 컨텍스트는 Session 인터페이스로, EntityManagerFactory는 SessionFactory로 생각하면 된다.



#### JPA의 EntityManagerFactory 구성

첫 번째 구성 방법은 `LocalEntityManagerFactoryBean` 클래스를 사용하는 것이다.

이 구성 방법은 퍼시스턴스 유닛 이름만 설정하면 되는 가장 간단한 방법이다.

하지만 DataSource를 주입할 수 없어 분산 트랜잭션을 사용할 수 없으므로 간단한 개발 작업에만 적합하다.



두 번째 구성 방법은 JEE 호환 컨테이너가 제공하는 **엔티티 매니저**를 사용하는 것이다.

JEE 호환 컨테이너에서 애플리케이션 서버는 배포 서술자 정보를 사용해 JPA 퍼시스턴스 유닛을 부트스트랩한다.

이 방법을 사용하면 스프링은 JNDI 룩업으로 엔티티 매니저를 룩업해 사용할 수 있다.



세 번째 구성 방법은 `LocalContainerEntityManagerFactoryBean` 클래스를 사용하는 것이다.

이 방법은 DataSource 주입과 로컬 및 분산 트랜잭션 참여가 가능하다.

```java
@Configuration
@EnableTransactionManagement
@ComponetScan(basePackages = {"..."})
public class JpaConfig {
  @Bean
  public DataSource dataSource() {
    try {
      EmbeddedDatabaseBuilder dbBuilder = new EmbeddedDatabaseBuilder();
      return dbBuilder.setType(EmbeddedDatabaseType.H2)
        							.addScripts("...").build();
    } catch (Exception e) {
      return null;
    }
  }
  
  @Bean
  public PlatformTransactionManager transactionManager() {
    return new JpaTransactionManager(entityManagerFactory());
  }
  
  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
    return new HibernateJpaVendorAdapter();
  }
  
  private Properties hibernateProperties() {
    Properties hibernateProp = new Properties();
    hibernateProp.put("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
    hibernateProp.put("hibernate.format_sql", true);
    hibernateProp.put("hibernate.use_sql_comments", true);
    hibernateProp.put("hibernate.max_fetch_depth", 3);
    hibernateProp.put("hibernate.jdbc.batch_size", 10);
    hibernateProp.put("hibernate.jdbc.fetch_size", 50);
    return hibernateProp;
	}
  
  @Bean
  public EntityManagerFactory entityManagerFactory() {
    LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityFactoryBean();
    factoryBean.setPackagesToScan("...");
    factoryBean.setDataSource(dataSource());
    factoryBean.setJpaVendorAdapter(jpaVendorAdapter());
    factoryBean.afterPropertiesSet();
    return factoryBean.getNativeEntityManagerFactory();
  }
}
```

+ `dataSource` Bean
+ `transactionManager` Bean :
  EntityManagerFactory는 트랜잭션 기반 데이터 액세스 시 사용한 트랜잭션 매니저가 필요하다.
  스프링은 JPA 전용 트랜잭션 매니저를 제공한다.
+ 컴포넌트 스캔 : 해당 태그를 설정하면 스프링은 설정한 패키지 아래 컴포넌트를 검색한다.
+ JPA `EntityManagerFactory` Bean : 
  emf 빈은 가장 중요한 부분이다.
  먼저 `LocalContainerEntityManagerFacotryBean`을 사용하는 빈을 선언한다.
  빈에는 여러 프로퍼티를 제공한다.
  먼저 `dataSource` 빈을 주입해야 한다.
  이후 jpaVendorAdapter 프로퍼티에는 `HibernateJpaVendorAdapter` 클래스를 설정한다.
  그리고 EntityManagerFactory가 설정한 패키지 아래에서 ORM 애너테이션을 적용한 도메인 객체를 스캔할 수 있도록 설정한다.
  마지막으로 하이버네이트를 퍼시스턴스 제공자로 사용하도록 **jpaProperties** 프로퍼티에 세부 구성을 제공한다.



#### ORM 매핑에 JPA 애너테이션 사용하기

`@Service` 애너테이션은 해당 클래스가 다른 레이어에 비즈니스 서비스를 제공하는 스프링 컴포넌트임을 나타내는 데 사용된다.

`@Repository` 애너테이션은 클래스에 데이터 액세스 로직이 들어 있음을 나타내며 데이터베이스 벤더가 제공하는 예외를 스프링이 자체 `DataAccessException` 계층으로 변환하도록 한다.

`@PersistenceContext` 애너테이션은 `EntityManager`을 주입할 때 사용한다.



### JPA로 데이터베이스 조작하기

#### 데이터 쿼리에 JPQL 사용하기

JPQL은 HQL과 매우 비슷하다.

쿼리를 사용하는 것에 조금의 차이가 있는데 `EntityManager.createNamedQuery()` 메서드를 호출하여 **쿼리 이름**과 원하는 **반환 타입**을 인자로 전달한다는 것이다.



예제를 통해 살펴보면 아래와 같다.

```java
// 하이버네이트 사용
sessionFacotry.getCurrentSession.getNamedQuery("Singer.findAllWithAlbum").list();
// JPA 사용
em.createNamedQuery(Singer.FIND_ALL_WITH_ALBUM, Singer.class).getResultList();
```



그리고 JPA 사양에서는 연관 관계가 있을 때 기본으로 퍼시스턴스 제공자가 연관 관계를 즉시(eagerly) 패치(fetch)하도록 정의한다.

하지만 하이버네이트로 JPA를 구현할 때 기본 패치 전략은 지연(lazy)이다.

그러므로 하이버네이트 JPA 구현체를 사용할 때는 연관 관계를 정의할 때 지연 패치를 명시적으로 정의할 필요가 없다.

하이버네이트의 기본 패치 전략이 JPA 사양과 다르기 때문이다.



#### 명시되지 않은 타입의 결과 쿼리하기

데이터를 조히할 때 사전에 매핑된 엔티티 클래스에 데이터를 저장하기보다는 데이터베이스에서 쿼리를 실행한 뒤 결과를 원하는 대로 가공하고 싶을 때가 있다.

이러한 경우는 쿼리를 실행한 뒤 **ResultSet 객체를 직접 조작함으로써 구현할 수 있다.**



아래 예제코드를 통해 알아보자.

```java
@Transactional(readOnly = true)
public void displayAllSingerSummary() {
  List result = em.createQuery("select s.firstName, s.lastName, a.title from Singer s" 
                               + "left join s.albums a"
                              + "where a.releaseDate=(select max(a2.releaseDate)"
                              + "from Album a2 where a2.singer.id = s.id)")
    													.getResultList();
  int count = 0;
  for (Itreator i = result.iterator(); i.hasNext();) {
    Object[] values = (Object[]) i.next();
    System.out.println(++count + ": " + values[0] + ", " + values[1] + ", " + values[2]);
  }
}
```

위의 예제에서는 `EntityManager.createQuery()` 메서드를 사용해 `Query` 인스턴스를 생성하고 JPQL 문을 전달한 뒤 결과 목록을 조회했다.

JPQL 내에 조회할 컬럼을 명시하면 JPA는 이터레이터를 반환한다.

이 이터레이터 내에 있는 각 항목은 객체의 배열이다.

위의 예제는 이터레이터를 순차적으로 모두 돌며 객체 배열의 각 엘리먼트 값을 출력한 것이다.

각 객체 배열은 ResultSet 객체 내에 있는 레코드에 해당한다.



### 생성자 표현식을 사용한 커스텀 결과 타입 쿼리

커스텀 결과를 쿼리할 때는 JPA를 사용해 각 레코드에서 직접 POJO 객체를 생성할 수 있다.

이렇게 쿼리 결과를 담은 POJO 객체를 여러 테이블의 데이터를 담고 있다는 뜻에서 **뷰(view)**라고 부른다.



이 또한 예제를 통해서 알아보자.

우선 가수 요약 정보를 저장하는 POJO 객체다.

```java
@Getter
@Setter
public class SingerSummary implements Serializavle {
  private String firstName;
  private String lastName;
  private String latestAlbum;
  
  publicc SingerSummary(String firstName, String lastName, String latestAlbum) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.latestAlbum = latestAlbum;
  }
}
```

위와 같이 가수 요약 정보를 저장하는 POJO 객체를 만들면 "명시되지 않은 타입의 결과 쿼리하기"의 예제를 아래와 같이 수정할 수 있다.

```java
@Transactional(readOnly = true)
public void displayAllSingerSummary() {
  List result = em.createQuery("select new com....view.SingerSummary("
                               + "s.firstName, s.lastName, a.title) from Singer s" 
                               + "left join s.albums a"
                              + "where a.releaseDate=(select max(a2.releaseDate)"
                              + "from Album a2 where a2.singer.id = s.id)",
                              SingerSummary.class).getResultList();
  return result;
}
```

JPQL 문에서는 new 키워드와 함께 POJO 클래스인 SingerSummary 클래스의 전체 이름을 명시했으며, 조회된 세 개의 애트리뷰트를 클래스 생성자의 인수로 전달했다.

마지막으로 `createQeury()` 메서드의 반환 타입을 명시하려고 SingerSummary 클래스를 해당 메서드에 인수로 전달했다.



#### 데이터 등록하기

JPA로 데이터 등록하는 것은 간단하다.

하이버네이트처럼 JPA도 데이터베이스가 생성한 기본 키 값을 조회할 수 있는 기능을 제공한다.



구체적인 것은 아래 예제 코드를 통해 알아보자.

```java
@Repository
@Transactional 
public class SingerServiceImpl implements SingerService {
  ...
  
  public Singer save(Singer singer) {
    if (singer.getId() == null) {
      em.persist(singer);
    } else {
      em.merge(singer);
    }
    return singer;
  }
}
```

위의 예제처럼 `save`의 경우 먼저 id 값을 확인해봄으로써 객체가 새 엔티티 인스턴스인지를 검사한다.

만약 id가 null이면 객체가 새 엔티티 인스턴스이므로 `EntityManager.persist()` 메서드를 호출한다.

`persist()` 메서드를 호출하면 `EntityManager`는 엔티티를 데이터베이스에 저장하며, 추가된 엔티티를 현재 퍼시스턴스 컨텍스트의 관리 대상 인스턴스로 등록한다.

이후 수정 작업을 하면 `persist()` 메서드 대신 `EntityManager.merge()` 메서드를 호출한다.

`merge()` 메서드가 호출되면 EntityManager는 현재 퍼시스턴스 컨텍스트에서 엔티티의 상태를 수정한다.



### 네이티브 쿼리 사용하기

계층 쿼리와 같은 유형의 쿼리는 특정 데이터베이스에 특화된 쿼리로 **네이티브 쿼리**라고 부른다.

JPA로 네이티브 쿼리를 실행할 수 있다.

네이티브 쿼리를 사용하면 EntityManager은 매핑이나 변환 없이 쿼리를 있는 그대로 데이터베이스에 제출한다.

JPA 네이티브 쿼리를 사용할 때 장점은 ResultSet을 ORM으로 매핑된 엔티티 클래스로 매핑할 수 있다는 것이다.



#### 단순한 네이티브 쿼리 사용하기

```java
@Service
@Repository
@Transactional
public class SingerServiceImpl implements SingerService {
  final static String ALL_SINGER_NATIVE_QUERY =
    "select id, first_name, last_name, birth_date, vesion from singer";
  
  @Transactional(readOnly = true)
  public List<Singer> findAllByNativeQuery() {
    return em.createNativeQuery(ALL_SINGER_NATIVE_QUERY, Singer.class).getResultList();
  }
}
```

`createNativeQuery()` 메서드는 Query 인터페이스를 반환한며, 이 Query 인터페이스는 결과 목록을 조회하는 `getResultList()` 메서드를 제공한다.

JPA 제공자는 쿼리를 실행한 뒤 엔티티 클래스에 정의된 JPA 매핑에 따라 ResultSet 객체를 엔티티 인스턴스로 변환한다.



#### SQL ResultSet 매핑으로 네이티브 쿼리 사용하기

`createNativeQuery()` 메서드를 호출할 때 매핑된 도메인 객체 대신 SQL ResultSet **매핑 이름을 나타내는 문자열**을 인수로 넘길 수 있다.

SQL ResultSet 매핑은 엔티티 클래스에 `SqlResultSetMapping` 애너테이션을 적용해 정의한다.

SQL ResultSet 매핑은 하나 이상의 엔티티와 컬럼 매핑을 가질 수 있다.

```java
@Entity
@Table(name = "singer")
@SqlResultSetMapping(
	name="singerResult",
	entities=@EntityResult(entityClass=Singer.class)
)
public class Singer implements Serializable {
  ...
}
```

위의 예제에서는 singerResult라는 SQL ResultSet 매핑을 Singer 엔터티 클래스에 정의했으며 entityClass 애트리뷰트를 사용해 매핑할 엔티티를 정했다.

JPA는 여러 엔티티를 사용하는 더욱 복잡한 매핑 기능을 제공하며 엔티티 내 컬럼 수준으로 상세히 매핑하는 기능도 제공한다.



SQL ResultSet 매핑을 지정한 뒤에는 ResultSet 매핑 이름을 사용해 `findAllByNativeQuery()` 메서드를 호출할 수 있다.

```java
@Service
@Repository
@Transactional
public class SingerServiceImpl implements SingerService {
  final static String ALL_SINGER_NATIVE_QUERY =
    "select id, first_name, last_name, birth_date, vesion from singer";
  
  @Transactional(readOnly = true)
  public List<Singer> findAllByNativeQuery() {
    return em.createNativeQuery(ALL_SINGER_NATIVE_QUERY, "singerResult").getResultList();
  }
}
```



### JPA2 크라이티리어 API로 크라이티리어 쿼리 사용하기

JPA 2의 주요 신기능 중 하나는 **강타입 크라이티리어 API 쿼리이다.**

새로 추가된 크라이티리어 API에서 쿼리에 전달되는 **검색 조건(criteria)은 엔티티 클래스의 메타 모델을 기반으로 작성된다.**

그러므로 쿼리에 지정되는 각 검색 조건은 강타입이며 그 결과 실행 시점이 아닌 **컴파일 시점에 에러를 발견할 수 있다.**



JPA 크라이티리어 API에서 **엔티티 클래스 메타 모델**은 엔티티 클래스 이름 뒤에 **언더스코어(_)**를 붙여 나타낸다.

```java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Singer.class)
public abstract class Singer_ {
  public static volatile SinterAttribute<Singer, String> firstName;
  public static volatile SinterAttribute<Signer, String> lastName;
  public static volatile SinterAttribute<Singer, Album> albums;
  public static volatile SinterAttribute<Singer, Instrument> instruments;
  public static volatile SinterAttribute<Singer, Long> id;
  public static volatile SinterAttribute<Singer, Integer> vesion;
  public static volatile SinterAttribute<Singer, Date> birthDate;
}
```

메타 모델 클래스에는 `@StaticMetamodel` 애너테이션이 적용됐으며 매핑된 엔티티 클래스가 애트리뷰트의 값으로 지정되어 있다.

메타 모델 클래스 내에는 애트리뷰트와 애트리뷰트 타입이 선언되어 있다.



아래 예제는 JPA 2 크라이티리어 API 쿼리를 사용해 구현한 `findByCriteriaQuery()` 메서드이다.

```java
@Service
@Repository
@Transactional
public class SingerServiceImpl implements SingerService {
  ...
  @Trnasactional(readOnly=true)
  public List<Singer> findByCriteriaQuery(String firstName, String lastName) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Singer> criteriaQuery = cb.createQuery(Singer.class);
    
    Root<Singer> singerRoot = creteriaQuery.from(Singer.class);
    singerRoot.fetch(Singer_.albums, JoinType.LEFT);
    singerRoot.fetch(Singer_.instruments, JoinType.LEFT);
    
    criteriaQuery.select(singerRoot).distinct(true);
    
    Predicate criteria = cb.conjunction();
    if (firstName != null) {
      Predicate p = cb.equal(singerRoot.get(Singer_.firstName), fistName);
      creteria = cb.and(criteria, p);
    }
   
    if (lastName != null) {
      Predicate p = cb.equal(singerRoot.get(Singer_.lastName), lastName);
      creteria = cb.and(criteria, p);
    }
     
    criteriaQuery.where(criteria);
    
    return em.createQuery(criteriaQuery).getRsultList();
  }
}
```

우선 `EntityManger.getCriteriaBuilder()`를 호출해 CriteriaBuilder 인스턴스를 가져온다.

결과 타입이 Singer가 되도록 Singer를 인수로 전달하며 `CriteriaBuilder.createQuery()`를 호출해서 **타입이 지정된 쿼리를 생성한다.**

엔티티 클래스를 인수로 전달하며 `CriteriaQuery.from()`를 호출한다.



호출결과 지정된 엔티티에 해당하는 **쿼리 루트 객체**가 반환된다.

쿼리 루트 객체는 쿼리 내에서 경로 표현식 기반을 이룬다.

`Root.fetch()` 메서드를 호출해 **엔티티 간 연관관계**를 즉시 패치한다.

JoinType.LEFT 인수는 외부 조인을 나타낸다.

`CriteriaQuery.select()`를 호출하며 인수로 `Root<Singer>` **인터페이스 타입 객체를 전달해 반환 받을 타입을 지정한다.**



`CiteriaBuilder.conjunction()` 메서드를 호출해 Predicate 인스턴스를 가져온다.

`CiteriaBuilder.conjunction()`은 **제약 사항을 한 개 이상 결합할 때 사용한다.**

Predicate는 표현식으로 정의된 검색 조건을 나타내는 제약사항으로, 단일 또는 복합으로 Predicate를 구성할 수 있다.

검색 조건에 사용할 인자가 null이 아니면 `CriteriaBuilder.and()` 메서드를 사용해 **새 Predicate를 생성한다.**

`equal()` 메서드는 **값이 같음을 나타내는 제약사항**이며, 메서드를 호출할 때는 **해당 제약 사항을 적용할 엔티티 클래스 메타 모델의 애트리뷰트와 비교할 값을 인수로 설정해 `Root.get()`을 호출한다.**

그다음 `CriteriaBuilder.and()` 메서드를 호출하면서 생성한 Predicate를 전달하면 **기존 Predicate에 결합(conjunct)된다.**

적용할 모든 조건과 제약사항으로 Predicate를 생성한 뒤, 쿼리의 where 절에 사용되도록 `CriteriaQuery.where()` **메서드를 호출하면서 해당 Predicate를 전달한다.**



마지막으로 `EntityManager`에 `CriteriaQuery`를 넘긴다.

**그 뒤 EntityManager은 인자로 받은 CriteriaQuery 내용을 바탕으로 쿼리를 생성하고 쿼리를 실행한 결과를 반환한다.**



### 스프링 데이터 JPA

스프링 데이터 JPA 프로젝트는 스프링 데이터 프로젝트의 하위 프로젝트이다.

스프링 데이터 JPA 프로젝트는 JPA를 사용하는 애플리케이션 개발을 단순화하는 데 주목적이 있다.



스프링 데이터 JPA는 몇 가지 주요 기능을 제공하는데 그중 알아볼 기능은 **Repository 추상화**와 **엔티티 클래스의 기본적인 감사 정보를 추적하는 엔티티 리스너**이다.



#### 데이터베이스 조작에 스프링 데이터 JPA Repository 사용하기

스프링 데이터 JPA에서 Repository 추상화는 JPA EntityManager를 래핑해 더 단순화된 JPA 기반의 데이터 액세스 인터페이스를 제공한다.

스프링 데이터 프로젝트 내에서 핵심 인터페이스는 Repository<T, ID extends Serializable> 인터페이스로, 이 인터페이스는 스프링 데이터 Commons 배포판에 포함돼 있는 마커 인터페이스이다.

스프링 데이터는 Repository 인터페이스를 확장한 다양한 인터페이스를 제공한다.

그중 하나가 **CrudRepository 인터페이스**로, Repository 인터페이스와 마찬가지로 스프링 데이터 Commons 프로젝트에 포함돼 있다.



CrudRepository 인터페이스는 많이 사용되는 몇 가지 메서드를 제공한다.

다음은 스프링 데이터 Commons 프로젝트 내에 선언된 CrudRepository 인터페이스 선언 코드다.

```java
@NoRepositoryBean
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
	<S extends T> S save(S entity);
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);
	Optional<T> findById(ID id);
	boolean existsById(ID id);
	Iterable<T> findAll();
	Iterable<T> findAllById(Iterable<ID> ids);
	long count();
	void deleteById(ID id);
	void delete(T entity);
	void deleteAllById(Iterable<? extends ID> ids);
	void deleteAll(Iterable<? extends T> entities);
	void deleteAll();
}

```

스프링 데이터의 Repository 추상화를 사용할 때 매력적인 점은 **메서드 이름에 findByOOO 같은 스프링 데이터 JPA가 제공하는 공통 명명 규칙을 사용할 때는 스프링 데이터 JPA에 네임드 쿼리를 제공할 필요가 없다는 것이다.**

**대신 스프링 데이터 JPA가 '유추'해 메서드 이름을 기반으로 쿼리를 생성한다.**



이러한 Repository 추상화를 사용하려면 이를 스프링 구성에 정의해야 한다.

아래는 Repository 추상화 관련 구성 파일 내용이다.

```java
@Configuration
@EnableTransactionManagement
@ComponetScan(basePackages = {"..."})
@EnableJpaRepositories(basePackages = {"..."}) // 데이터 JPA 기능 활성화
public class DataJpaConfig {
  ...
}
```

이 구성 클래스에서 스프링 데이터 JPA 기능을 활성화하기 위해 사용되 유일한 구성 엘리먼트는 `@EnableJpaRepository` 애너테이션이다.

`@EnableJpaRepository` 애너테이션에 basePackages 애트리뷰트로 패키지를 지정하면, 지정한 패키지내에 있는 커스텀 Repository 확장 클래스가 스캔되면며 저장소 빈으로 생성된다.



그렇기에 이제는 **EntityManager 대신**에 **Repository 인터페이스를 기반으로 자동 생성한 Repository 빈을 서비스 클래스에 주입**하기만 하면 되며, 그다음부터는 스프르이 데이터 JPA가 필요한 모든 로우 레벨 조작을 대신 수행한다.



### JpaReopsitory 사용하기

CrudRepository 외에도 커스텀 저장소를 더 쉽게 만들 수 있는 훨씬 진보된 스프링 인터페이스가 있다.

**JpaRepository 인터페이스**로, **배치, 페이징, 정렬 기능**을 제공한다.

(JpaRepository는 CrudRepository를 상속한다.)



### 스프링 데이터 JPA로 커스텀 쿼리 사용하기

복잡한 애플리케이션을 개발할 때는 스프링이 "간섭"할 수 없는 커스텀 쿼리를 사용해야할 때가 있다.

이때는 `@Query` 애너테이션을 적용해 쿼리를 명시적으로 선언해야 한다.



아래는 이에 관한 예제 코드이다.

```java
@Query("select a from Album a whre a.title like %:title%")
List<Album> findByTitle(@Param("title") String title);
```

`@Query` 애너테이션이 적용된 메서드의 인자 이름과 네임드 파라미터가 동일할 때는 `@Param` 애너테이션을 생략할 수 있다.

하지만 메서드 인자 이름일 다를 때는 스프링이 쿼리 내 네임드 파림터에 값을 주입할 인자의 이름을 `@Param` 애너테이션으로 지정해햐 한다.



#### 엔티티 클래스의 변화 추적

**대부분 애플리케이션에서는 사용자가 관리하는 비즈니스 데이터를 대상 기본적인 감사(audit)활동을 지속적으로 해야한다.**

일반적으로 감사 정보는 데이터를 생성한 사용자, 데이터가 생성된 시각, 마지막으로 수정된 시각, 마지막으로 수정한 사용자를 포함한다.



스프링 데이터 JPA 프로젝트는 **JPA 엔티티 리스너**의 형태로 감사 기능을 제공해 **감사 정보를 자동으로 추적할 수 있는 기능을 제공**한다.

스프링 4까지는 해당 기능을 사용하려면 엔티티 클래스가 `Persistable<ID>` 인터페이스를 상속한 `Auditable<U, ID extends Serizaliable, T extends TemporalAccessor>`를 구현하거나 해당 인터페이스를 구현한 다른 클래스를 상속해야 했다.



하지만 스프링 5부터는 더 이상 `Auditable<U, ID extends Serializable>` 인터페이스를 구현핳지 않아도 된다.

감사와 관련된 모든 정의를 **애너테이션으로 대신할 수 있기 때문이다.**

```java
@Entity
@Getter
@ToString
@EntityListners(AuditingEntityListener.class)
public class SingerAudit implements Serializable {
  @Id
  @GenerateValue(strategy = IDENTITY)
  private Long id;
  
  @Version
  private int version;
  
  private String firstName;
  
  private String lastName;
  
  @Temporal(TemporalType.DATE)
  private Date birthDate;
  
  @CreateDate
  @Temporal(TemporalType.TIMESTAMP)
  private Date createdDate;
  
  @CreatedBy
  private String createdBy;
  
  @LastModifiedBy
  private String lastModifiedBy;
  
  @LastModifiedDate
  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;
}
```

`@EntityListeners(AuditingEntityListener.class)` 애너테이션은 퍼시스턴스 컨텍스트 내에서 singerAudit 엔티티에만 적용되는 `AuditingEntityListener`를 등록한다.

두 개 이상의 엔티티 클래스가 필요한 복잡한 예제에서 `@MappedSuperclass` 애너테이션인 적용된 추상 클래스로 분래해 감사 기능을 작성하며, 해당 클래스에도 `@EntityListeners(AuditingEntityListener.clas)` 애너테이션을 적용한다.



이처럼 여러 엔티티를 감사하는 계층 구조에 SingerAudit이 포함돼 있을 때는 아래와 같은 추상 클래스를 상속할 수 있다.

```java
@Getter
@Setter
@MappedSuperClass
@EntityListners(AuditingEntityListener.class)
public abstract class AuditableEntity<U> implements Serializable {
  @CreateDate
  @Temporal(TemporalType.TIMESTAMP)
  private Date createdDate;
  
  @CreatedBy
  private String createdBy;
  
  @LastModifiedBy
  private String lastModifiedBy;
  
  @LastModifiedDate
  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;
}
```



이를 사용하기 위해서는 추가적인 구성 작업이 필요한데 추가되는 사항은 아래와 같다.

```java
@Configuration
@EnableTransactionManagement
@ComponetScan(basePackages = {"..."})
@EnableJpaRepositories(basePackages = {"..."})
@EnableJpaAuditing(auditorAwareRef = "auditorAwareBean")
public class AuditConfig {
  ...
}
```

`@EnableJpaAuditing(auditorAwareRef = "auditorAwareBean")` 애너테이션은 JPA 감사 기능을 활성화 한다.



```java
@Coomponet
public class AuditorAwareBean implements AuditorAware<String> {
  public Optional<String> getCurrentAuditor() {
    return Optional.of("...");
  }
}
```

`AuditorAwareBean` 클래스는 `AuditorAware<T>` 인터페이스를 구현했으며 인터페이스 타입으로 String을 전달했다.

**실제 개발을 할 때는 보안 인프라스트럭처에서 사용하는 정보를 가져와야 한다.**

예를들어 스프링 시큐리티를 사용할 때는 SecurityContextHolder 클래스에서 사용자 정보를 가져올 수 있을 것이다.



### 하이버네이트 엔버스로 엔티티 버전 관리하기

**엔터프라이즈 애플리케이션에서는 핵심 비즈니스 데이터에는 각 엔티티의 버전 관리해야하는 요구사항이 존재한다.**

이를 위해서는 두 가지 방법이 존재한다.

첫 번째는 데이터베이스 트리거를 생성해 수정 작업 전에 변경 이전 레크드를 이력 테이블에 등록하는 것이다.

두 번째는 데이터 액세스 레이어에 로직을 개발하는 것이다.

두 방법 모두 단점이 존재한다.

트리거를 사용하는 방법은 특정 데이터 베이스 플랫폼에 종속되는 문제점이 있으며 직접 로직을 구현하는 방법을 꽤 불편하고 에러가 발생하기 쉬운 문제가 있다.



하이버네이트 엔버스는 엔티티 버전 관리 자동화에 특화된 하이버네이트 모듈이다.

엔버스는 다음과 같은 두 가지 감사 전략을 제공한다.

+ 기본 :
  엔버스가 레코드 수정 관련 칼럼을 관리한다.
  레코드가 등록되거나 수정될 때마다 데이터베이스 시퀀스나 테이블에서 조회한 개정 번호를 가진 새 레코드를 이력 테이블에 등록한다.
+ 유효성 검증 감사 :
  각 이력 레코드에 **시작 개정 번호와 마지막 개정 번호**를 저장한다.
  레코드를 등록하거나 수정할 때마다 시작 개정 번호를 새 레코드 이력 테이블에 등록한다.
  동시에 이전 이력 레코드는 마지막 개정 번호로 업데이트된다.
  이전 이력 레코드의 마지막 개정 정보를 수정할 때 **타임스탬프**를 기록하도록 엔버스를 구성할 수도 있다.

유효성 감사 전략을 사용하면 더 많은 데이터베이스 수정이 발생하지만 이력을 조회하는 속도는 훨씬 빨라진다.

마지막 개정 타임스탬프도 이력 레코드에 기록되므로 이력 데이터를 조회할 때 특정 시점의 레코드 스냅샵을 식별하기 더 쉬워진다.



#### 엔티티 버전 관리 테이블 추가

엔티티 버전 관리 기능을 사용하려면 테이블 몇 개를 추가해야 한다.

**먼저 버전 관리를 할 엔티티의 테이블마다 이력 테이블을 생성해야 한다.**



**유효성 감사 전략을 사용하려면 아래의 네 개의 칼럼을 이력 테이블에 추가해야 한다.**

하이버네이트 엔버스는 개정 번호와 개정이 발생한 시점의 타임스탬프를 관리하는 별도 테이블이 필요하다.

| 감사 전략             | 데이터 타입 | 설명                                             |
| --------------------- | ----------- | ------------------------------------------------ |
| AUDIT_REVISION        | INT         | 이력 레코드의 시작 개정 번호                     |
| ACTION_TYPE           | INT         | 조작 유형, 가능한 값은 추가(0), 수정(1), 삭제(2) |
| AUDIT_REVISION_END    | INT         | 이력 레코드의 마지막 개정 번호                   |
| AUDIT_REVISION_END_TS | TIMESTAMP   | 마지막 개정이 수정된 시점의 타임스탬프           |



#### 엔티티 버전 관리에 사용할 EntityManagerFactory 구성

하이버네이트 엔버스는 EJB 리스너의 형태로 구현되었다.

`LocalContainerEntityManagerFactory` 빈에서 해당 리스너를 구성할 수 있다.

```java
@Configuration
@EnableTransactionManagement
@ComponetScan(basePackages = {"..."})
@EnableJpaRepositories(basePackages = {"..."})
public class DataJpaConfig {
  ...
  private Properties hibernateProperties() {
    Properties hibernateProp = new Properties();
    hibernateProp.put("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
    hibernateProp.put("hibernate.format_sql", true);
    hibernateProp.put("hibernate.use_sql_comments", true);
    hibernateProp.put("hibernate.max_fetch_depth", 3);
    hibernateProp.put("hibernate.jdbc.batch_size", 10);
    hibernateProp.put("hibernate.jdbc.fetch_size", 50);
    // Hibernate Envers 관려 프로퍼티
    hibernateProp.put("org.hibernate.envers.audit_table_suffix", "_H");
    hibernateProp.put("org.hibernate.envers.revision_field_name", "AUDIT_REVISION");
    hibernateProp.put("org.hibernate.envers.audit_strategy", "org.hibernate.envers.strategy.VlidityAuditStrategy");
    hibernateProp.put("org.hibernate.envers.audit_strategy_validity_end_rev_field_name", "AUDIT_REVISION_END");
    hibernateProp.put("org.hibernate.envers.audit_strategy_validity_store_revend_timestamp", "True");
    hibernateProp.put("org.hibernate.envers.audit_strategy_validity_revend_timestamp_field_name", "AUDIT_REVISION_END_TS");
    return hibernateProp;
	}
  ...
}
```

엔버스 감사 이벤트 리스너인 `org.hibernate.envers.event.AuditEventListener`는 **다양한 퍼시스턴스 이벤트의 발생을 기다린다.**

**리스너는 등록 뒤, 수정 뒤, 또는 삭제 뒤 이벤트를 가로체 엔티티 클래스의 수정 전 스냅샷을 이력 테이블에 저장한다.**

**리스너는 또 연관 관계 수정 이벤트인 pre-collection-update, pre-collection-remove, pre-collection-recreate를 기다렸다가 이벤트가 발생하면 엔티티 클래스의 연관 관계에 발생한 수정 작업을 처리한다**.

엔버스는 감사 대상 엔티티와 연관 관계 내에 있는 엔티티의 수정 이력을 추적할 수 있다.



#### 엔티티 버전 관리와 이력 조회를 가능하게 하기

엔티티 버전 관리를 활성화하려면 엔티티 클래스에 `@Audited` 애너테이션을 적용하기만 하면 된다.

`@Audited` 애너테이션은 **클래스 레벨**에 적용할 수  있으며, **엔버스는 애너테이션이 적용된 엔티티의 모든 칼럼에 발생한 변화를 감사한다.**

특정 필드를 감사 대상에서 제외하려면 해당 컬럼에 **@NotAudited**를 적용한다.



그리고 엔티티에 위와 같이 엔버스를 적용한 이후에는 아래와 같이 사용할 수 있다.

```java
@Service
@Transactional
public class SingerAuditServiceImple implements SeingerAuditService {
  @Autowired
  private SingerAuditRepository singerAuditRepository;
  @PersistenceContext
  private EntityManager entityManager;
  ...
  @Transactional(readOnly=true)
  public SingerAudit findAuditByRevision(Long id, int revision) {
    AuditReader auditReader = AuditReaderFactory.get(entityManager);
    return auditReader.find(SingerAudit.class, id, revision); // 엔버스를 통해 이전 기록 조회
  }
}
```



### 스프링 부트 JPA

지금까지 엔티티, 데이터베이스, 저장소, 서비스를 포함한 모든 항목을 일일이 구성했다.

스프링 부트 스타터 아티팩트는 이를 편리하게 구성할 수 있도록 도와준다.

스프링 부트 JPA 스타터는 스프링 부트 JPA에 의존하며 사전에 구성이 완료된 임베디드 데이터베이스와 함께 제공된다.

개발자가 추가로 해야하는 구성은 클래스패스에 의존성을 추가하는 것뿐이다.

**스프링 부트 JPA 스타터에는 퍼시스턴스 레이어 추상화에 사용할 하이버네이트도 포함되어 있다.**

**스프링 Repository 인터페이스도 자동으로 감지된다.**



### JPA를 사용할 때 고려사항

아직 JPA를 사용해 **저장 프로시저**를 호출하는 방법을 살펴보지 않았다.

JPA는 완벽하고 강력한 ORM 데이터 액세스 표준이며 스프링 데이터 JPA나 하이버네이트 엔버스 같은 서드파티라이브러리를 사용해 다양한 횡단 관심사를 비교적 쉽게 구현할 수 있다.

