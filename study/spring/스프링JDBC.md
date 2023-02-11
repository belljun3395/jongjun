## 스프링 JDBC

### JDBC 인프라스트럭처

JDBC는 자바 애플리케이션에서 데이터베이스에 있는 데이터에 액세스하는 표준 방법을 제공한다.

JDBC 인프라스트럭처의 핵심은 각 데이터베이스용 드라이버로, JDBC 드라이버를 사용해 자바 코드가 데이터 베이스에 액세스 할 수 있다.



드라이버를 로딩하면 드라이버는 자신을 `java.sql.DriverManager` 클래스에 등록한다.

`DriverManager` 클래스는 드라이버 목록을 관리하며 애플리케이션 데이터베이스에 연결하는데 사용할 수 있도록 정적 메서드를 제공한다.

`DriverManager`의 `getConnection()` 메서드는 드라이버가 구현한 `java.sql.Connection` 인터페이스를 반환한다.



JDBC 프레임워크를 사용할 때 코드 사용이 복잡한 근본 원인은 데이터베이스 **커넥션 관리**로 빈틈 없이 해야 하기 때문이다.

**데이터 베이스 커넥션**은 최소한 사용해야 하는 리소스이며 커넥션을 맺을 때 성능 측면에서 많은 비용이 소모된다.

일반적으로 **데이터베이스는 각 커넥션마다 스레드를 생성하거나 자식 프로세스를 생성한다.**

그래서 동시 접속 수에도 제한이 있으며 과도하게 커넥션을 맺으려고 하면 데이터 베이스가 느려진다.



### 데이터베이스 커넥션과 데이터 소스

스프링은 `javax.sql.DataSource`를 **구현한 빈**을 제공하며, 스프링은 이 빈을 사용해 **데이터 커넥션을 관리**한다.

`DataSource`와 `Connection`은 서로 다른 개념으로**` DataSource`가 ` Connection`을 제공하고 관리한다는 차이가 있다.**



`org.springframework.jdbc.datasource `패키지 아래에 있는 `DriverManagerDataSource`는 `DataSource`의 가장 간단한 구현체이다.

이 클래스가 **데이터베이스 커넥션**을 얻으려면 `DriverManager`를 **호출하기만 한다는 것**을 클래스 이름에서 유추할 수 있을 것이다.

`DriverManagerDataSource`는 **데이터베이스 커넥션 풀을 지원하지 않으므로** 테스트 외에는 **사용하지 않는 것이 좋다.**



아래 코드는 **java 설정을 통해 dataSource를 설정**과 **이를 활용한 테스트 코드**이다.

```java
import org.springframework.jdbc.datasource.SimpleDriverDataSource;

...
  
@Configuration
@PropertySource("classpath:db/jdbc2.properties")
public class DbConfig {
  ...
  
  @SuppressWarnings("unchecked")
  @Lazy
	@Bean
  public DataSource dataSource() {
    try {
      SimpleDriverDataSource dataSource = new SimpleDriverDataSource(); // datasource 패키지 아래 DriverManagerDataSource
      Class<? extends Driver> driver = (Class<? extends Driver>) Class.forName(driverName);
      dataSource.setDriverClass(driver);
      dataSource.setUrl(url);
      dataSource.setPassword(password);
      return dataSource
    } catch (Exception e) {
      return null;
    }
  }
}
```

```java
Connection connection = null;
try {
  connection = dataSource.getConnection();
  PreparedStatement statement = connection.prepareStatement("SELECT 1");
  ResultSet resultSet = statement.executeQuery();
  while (resultSet.next()) {
    int mockVal = resultSet.getInt("1");
    assertTrue(mockVal == 1);
  }
  statment.close();
} catch (Exception e) {
  logger.debug("예상치 못한 에러 발생. ", e)
} finally {
  if (connection != null) {
    connection.close();
  }
}
```

일반 JDBC 코드의 `DataSource를` 사용해도 커넥션 풀과 동일한 효과를 얻을 수도 있지만 여전히 어딘가에서 `DataSource`를 구성해야 했다.

하지만 스프링을 사용하면 `dataSource` 빈 선언과 데이터베이스 커넥션 프로퍼티 정의를 `ApplicationContext` 구성 파일에서 한번에 할 수 있다.

이렇게되면 애플리케이션 코드에서는 `DataSource`의 **실제 구현**이나 `DataSource` **위치**를 **알 수 없게** 된다.

또 데이터베이스 커넥션 관리도 `dataSource` 빈에게 위임해 `dataSource` 빈은 자신이 직접 커넥션을 관리하거나 JEE 컨테이너를 이용해 커넥션을 관리한다.



### JdbcTemplate 클래스

`JdbcTemplate` 클래스는 스프링 JDBC 지원 기능의 핵심이다.

이 클래스는 모든 유형의 SQL 문을 실행할 수 있다.



#### DAO 클래스에서 JdbcTemplate 초기화

```java
public class JdbccSingerDao implements SingerDao, InitializingBean {
  private DataSource dataSource;
  private JdbcTemplate jdbcTemplae;
  
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
    this.jdbcTemplate = new JdbcTemplate();
    jdbcTemplate.setDataSource(dataSource);
  }
}
```

JdbcTemplat는 한 번 구성하고 나면 **스레드 세이프**하다.

즉, 스프링 구성에서 JdbcTemplate **인스턴스 하나를 초기화**하고 이 인스턴스를 **모든 DAO 빈에 주입**할 수도 있다.



```java
@Configuration
public class EmbeddedJdbcConfig {
  ...
   
  @Bean 
  public JdbcTemplate jdbcTemplate() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate();
    jdbcTemplate.setDataSource(dataSource());
    return jdbcTemplate;
  }
  
  @Bean
  public SingerDao singerDao() {
    JdbcSingerDao dao = new JdbcSingerDao();
    dao.setJdbcTemplate(jdbcTemplate()); // jdbcTemplate 주입
    return dao;
  }
}
```



### JdbcTemplate을 사용해 보기

#### 단일값 조회

`jdbcTemplate.queryForObject("SQL구문", "문장의 파라미터에 바인딩할 값", "반환 값의 타입")`

```java
@Override
public String findNameById(Long id) {
  return jdbcTemplate.queryForObject(
    "select first_name || '' || last_name from singer where id = ?", 
    new Object[]{id}, 
    String.class
  );
}
```



#### NamedParameterJdbcTempalte로 조회하기

```java
@Override
public String findNameById(Long id) {
  String sql = "select first_name || ' ' || last_name from singer where id = :singerId";
  
  Map<String, Object> namedParameters = new HashMap<>();
  
  namedParameters.put("singerId", id);
  
  return 
    namedParameterJdbcTemplate
    	.queryForObject(
    		sql, 
    		namedParameters, 
    		String.class
  	);
}
```

`NamedParameterJdbcTempalte`에서는 `'?'` 위치 지정자 대신 접두어로 `콜론(:)`이 붙는 **네임드 파라미터를 활용**한다.



#### `RowMapper<T>`를 사용해 도메인 객체 조회하기 

`org.springframework.jdbc.core` 아래 있는 스프링의 `RowMapper<T>` 인터페이스를 사용하면 **JDBC ResultSet**을 간단히 **POJO 객체로 매핑**할 수 있다.

```java
public class JdbcSingerDao implements SingerDao, InitializingBean {
  ...

  @Override
  public List<Singer> findAll() {
    String sql = "select id, first_name, last_name, birthdate from singer";
    return namedParameterJdbcTemplate.query(sql, new SingerMapper());
  }
  
  private static final class SingerMapper implements RowMapper<Singer> {
    @Override
    public Singer mapRow(ResultSet rs, int rowNum) throws SQLException {
      Singer singer = new Singer();
      singer.setId(rs.getLong("id"));
      singer.setFirstName(rs.getString("first_name"));
      singer.setLastName(rs.getString("last_name"));
      singer.setBirthDate(rs.getDate("birth_date"));
      return singer;
    }
  }
}
```

`ResultSet`의 특정 레코드를 원하는 도메인 객체로 변환하는 `mapRow()` 메서드를 구현하였다.



#### ResultSetExtractor를 사용해 중첩 도메인 객체 조회하기

`RowMapper<T>`는 **단일 도메인 객체**에만 로우 매핑할 수 있다.

**좀 더 복잡한 객체 구조**에서는 `ResultSetExtractor` 인터페이스를 사용해야 한다.

```java
public class JdbcSingerDao implements SingerDao, InitializingBean {
  ...
  @Override
  public List<Singer> findAll() {
    String sql = "select s.id, s.first_name, s.last_name, s.birth_date, a.id as album_id, a.title, a.release_date from singer s left join album a on s.id = a.singer_id";
    return namedParameterJdbcTemplate.query(sql, new SingerWithDetailExtractor());
  }
  
  private static final class SingerWithDetailExtractor implements ResultSetExtractorList<<Singer>> {
    @Override
    public List<Singer> extractData(ResultSet rs) throws SQLException, DataAccessException {
      Map<Long, Singer> map = new HashMap<>();
      Singer singer;
      while (rs.next()) {
        Long id = rs.getLong("id");
        singer = map.get(id);
        if (singer == null) {
          Singer singer = new Singer();
		      singer.setId(rs.getLong("id"));
    		  singer.setFirstName(rs.getString("first_name"));
		      singer.setLastName(rs.getString("last_name"));
    		  singer.setBirthDate(rs.getDate("birth_date"));
          singer.setAlbums(new ArrayList<>());
          map.put(id, singer);
        }
        Long albumId = rs.getLong("album_id");
        if (albumId > 0) {
          Album album = new Album();
          album.setId(albumId);
          album.setSingerId(id);
          album.setTitle(rs.getString("title"));
          album.setReleaseDate(rs.getDate("release_date"));
          signer.addAlbum(album);
        }
      }
      return new ArrayList<>(map.values());
    }
    }
  }
}
```

`ResultSet`을 객체 목록으로 변환하는 `extractData()` 매서드를 구현하였다.



### JDBC 조작을 모델링하는 스프링 클래스

+ `MappingSqlQuery<T>` : `MappingSqlQuery<T>` 클래스는 **SQL 쿼리 문자열**과 **mapRow() 메서드**를 한 클래스로 감싸준다.
+ `SqlUpdate` : `SqlUpdate` 클래스는 **모든 데이터 수정 SQL 문**을 래핑할 수 있다.
+ `BatchSqlUpdate` : **배치 수정 조작**을 할 때 사용한다.
+ `SqlFunction<T>` : `SqlFunction<T>` 클래스는 **데이터베이스에 저장된 함수**에 **인자 및 반환 타입을 지정해 호출**할 때 사용한다. 
+ `@Repository`, `@Resource`  :  **JDBC DAO를 설정**할 때 사용한다.



`@Repository`는 **데이터베이스를 조작하는 빈에 적용하도록 특수하게 설계된** 애너테이션인 `@Componet` 애너테이션이다.

`@Reopository`를 적용해 DAO 클래스 빈을 선언한 뒤 이 빈에 dataSource를 아래 코드와 같이 주입할 수 있다.

```java
@Repository("singerDao")
public class JdbcSignerDao implements SingerDao {
  ...
    
  @Resource(name = "dataSource")
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }
  
  ...
}
```

특정 클래스에 `@Repository` 애너테이션이 적용돼 있으면 해당 클래스에는 **데이터 액세스 코드가 들어있다는 뜻**으로 스프링은 각 데이터베이스의 SQL 예외를 좀 더 애플리케이션 친화적인 **`DataAcessException` 계층 예외로 변환한다.**



#### `MappingSqlQuery<T>` 를 사용해 데이터 질의하기

스프링은 `MappingSqlQuery<T>` 클래스를 사용해 쿼리 조작을 모델링한다.

우선 `DataSource`와 SQL 쿼리 문자열을 사용해 `MappingSqlQuery<T>` 생성자를 호출한다.

그 다음 각 `ResultSet` 레코드를 해당 도메인 객체로 변한하는 `mapRow()` 메서드를 구현한다.



우선 `MappingSqlQuery<T>` 클래스는 아래와 같이 구현할 수 있다.

```java
public class SelectAllSingers extends MappingSqlQuery<Singer> {
  private static final String SQL_SELECT_ALL_SINGER = "select id, firtst_name, last_name birth_date from singer";
  
  public SelectAllSingers(DataSource dataSource) {
    super(dataSource, SQL_SELECT_ALL_SINGER);
  }
  
  protected Singer mapRow(ResultSet rs, int rowNum) throws SQLException {
    	Singer singer = new Singer();
      singer.setId(rs.getLong("id"));
      singer.setFirstName(rs.getString("first_name"));
      singer.setLastName(rs.getString("last_name"));
      singer.setBirthDate(rs.getDate("birth_date"));
      return singer;
  }
}
```



 이렇게 구현한 `MappingSqlQuery<T>` 클래스는 아래와 같이 사용할 수 있다.

```java
@Repository("singerDao") 
public class JdbcSignerDao implements SingerDao {
  ...
    
  @Resource(name = "dataSource")
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
    this.selectAllSingers = new SelectAllSingers(dataSource);
  }
  
  @Override
  public List<Singer> findAll() {
    return selectAllSingers.execute(); // MappingSqlQuery가 SqlQuery를 상속한 execute 메서드를 실행시킨 것이다.
  }
}
```

`execute()`메서드는 `SelectAllSingers` 클래스가 `SqlQuery<T>` 추상 클래스에서 간접적으로 상속한 것이다.



이름을 파라미터로 받는 `SelectSingerByFirstName` 클래스는 아래와 같이 생성한다.

```java
public class SelectSingerByFirstName extends MappingSqlQuery<Singer> {
  private static final String SQL_FIND_BY_FIRST_NAME = "select id, first_name, last_name, birth_date from singer where first_name = :first_name";
  
  public SelectAllSingers(DataSource dataSource) {
    super(dataSource, SQL_FIND_BY_FIRST_NAME);
    super.declareParameter(new SqlParameter("first_name", Types.VARCHAR));
  }
  
  protected Singer mapRow(ResultSet rs, int rowNum) throws SQLException {
    	Singer singer = new Singer();
      singer.setId(rs.getLong("id"));
      singer.setFirstName(rs.getString("first_name"));
      singer.setLastName(rs.getString("last_name"));
      singer.setBirthDate(rs.getDate("birth_date"));
      return singer;
  }
}
```



#### SqlUpdate를 사용해 데이터 수정

아래는 `JdbcSingerDao` 클래스를 `SqlUpdate`를 사용해 구현한 것이다.

```java
public class UpdateSinger extends SqlUpdate {
  private static final String SQL_UPDATE_SINGER = "update singer set first_name=:first_name, last_name=:last_name, birth_date=:birth_date where id=:id";
  
  public UpdateSinger(DataSource dataSource) {
    super(dataSource, SQL_UPDATE_SINGER);
    super.declareParameter(new SqlParameter("first_name", Types.VARCHAR));
    super.declareParameter(new SqlParameter("last_name", Types.VARCHAR));
    super.declareParameter(new SqlParameter("birth_date", Types.DATE));
    super.declareParameter(new SqlParameter("id", Types.INTEGER));
  }
}
```



#### BatchSqlUpdate를 사용하는 배치 조작

```java
public class InsertSingerAlbum extends BatchSqlUpdate {
  private static final String SQL_INSERT_SINGER_ALBUM = "insert into album (singer_id, title, release_date) values (:singer_id, :tilte, :release_date)";
  
  private static final int BATCH_SIZE = 10;
  
  public InsertSingerAlbum(DataSource dataSource) {
    super(dataSource, SQL_INSERT_SINGER_ALBUM);
    
    declareParameter(new SqlParameter("singer_id", Types.INTEGER));
    declareParameter(new SqlParameter("title", Types.VARCHAR));
    declareParameter(new SqlParameter("release_date", Types.DATE));

    setBatchSize(BATCH_SIZE);
  }
}
```

`BatchSqlUpdate` 클래스는 **스레드 세이프하지 않으므로** 호출할 때마다 **새로운 인스턴스**를 생성한다.

그 다음 생성한 인스턴스를 `SqlUpdate`와 마찬가지로 사용한다.

`SqlUpdate`와 주요 차이점은 `BatchSqlUpdate`는 수행할 등록 조작을 **어느 정도(BATCH_SIZE) 모아뒀다가 데이터베이스에 보내 일괄적으로 실행**한다는 것이다.

모인 레코드 수가 **배치 크기에 도달하면** 스프링은 대기 중인 레코드를 대량으로 **데이터베이스에 등록한다.**



#### SqlFunction으로 저장함수 호출하기

스프링은 JDBC를 사용해 **저장 프로시저**나 **저장함수**를 간단히 실행할 수 있는 클래스를 제공한다.

```sql
DELIMITER //
CREATE FUNCTION getFirstNameById(in_id INT)
	RETURNS VARCHAR(60)
	BEGIN
		RETURN (SELECT first_name FROM singer WHERE id = in_id);
	END //
DELIMITER;
```

```java
public class StoredFunctionFirstNameById extends SqlFunction<String> {
  private static final String SQL = "select getfirstnamebyid(?)";
  
  public StoredFunctionFirstNameById(DataSource dataSource) {
    super(dataSource, SQL);
    declareParameter(new SqlParameter(Types.INTEGER));
    complie();
  }
}
```



### 스프링 부트 JDBC

스프링 부트는 다음과 같은 빈을 자동으로 등록한다.

+ `DataSource`
+ `JdbcTemplate`
+ `NamedParameterJdbcTemplate`
+ `PlatformTransactionManager(DataSourceTransactionManager)`



스프링 부트는 `src/main/resource` 디렉터리 아래에 있는 임베디드 **데이터베이스 초기화 파일을 검색한다**.

스프링 부트는 `schema.sql` 파일에 `SQL DDL` 문이, `data.sql` 파일에 `DML` 문이 들어있을 것으로 얘상한다.

스프링 부트는 시작 시 이 두 파일을 데이터베이스 초기화에 사용한다. 



데이터베이스 초기화에 사용할 파일 이름은 `src/main/resources` 아래에 있는 **`application.properties` 파일에 지정**할 수 있다.

스프링 부트 애플리케이션이 사용할 SQL 파일 이름이 들어있는 구성 파일은 다음과 같다.

+ `spring.datasource.schema=db/schema.sql`
+ `spring.datasource.data=db/test-data.sql`

스프링 부트는 기동 시에 **기본적으로 데이터베이스를 초기화**하지만 `application.properties` 파일에 **`spring.datasource.initialize=false` 프로퍼티를 추가해 자동 초기화를 막을 수 있다.**
