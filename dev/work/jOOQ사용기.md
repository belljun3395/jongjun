# jOOQ 사용기

간단한 프로젝트를 수행할 때에는 JpaRepository가 제공하는 메서드만으로 충분히 쿼리 구현할 수 있었지만 기능이 복잡해지면서 JpaRepository의 `@Query`를 사용해 쿼리를 직접 작성해야 하는 경우가 많아졌습니다.

`@Query`는 문자열을 사용해 쿼리를 작성하기에 타입세이프하지 않고 쿼리 작성에도 불편함이 있습니다.

그래서 이번 프로젝트에서는 타입세이프 하고 복잡한 쿼리도 편리하게 작성하기 위해 jOOQ를 도입하고 사용해 보았습니다.



## jOOQ 사용 세팅

### 의존성

jOOQ는 spring boot starter에 포함되어 있어 아래와 같이 간단히 의존성을 추가할 수 있습니다.

```groovy
implementation org.springframework.boot:spring-boot-starter-jooq
```



### jOOQ CodeGen - Flyway

의존성을 추가하고 난 이후에는 jOOQ DSL을 생성해주어야 합니다. (QueryDSL의 Q클래스와 유사)

jOOQ에서는 다양한 방식으로 jOOQ DSL 생성을 지원합니다.

이번 프로젝트에서는 JPA를 사용하지 않기에 DB 형상 관리를 위해 Flyway 역시 도입한 상태였기에 Flyway를 활용하여 jOOQ DSL을 생성하는 방법을 선택하였습니다.

[jooq with flyway 공식 문서](https://www.jooq.org/doc/latest/manual/getting-started/tutorials/jooq-with-flyway/)



jOOQ DSL 생성을 위해선 우선 플러그인을 추가해 줍니다.

```groovy
id "org.jooq.jooq-codegen-gradle" version '3.19.10'
```



그리고 아래와 같은 코드를 추가합니다.

```kotlin
sourceSets {
    main {
        java {
            val mainDir = "src/main/kotlin"
            val jooqDir = "src/generated"
            srcDirs(mainDir, jooqDir)
        }
    }
}
```

```kotlin
jooq {
    configuration {
        generator {
            database {
                name = "org.jooq.meta.extensions.ddl.DDLDatabase"
                properties {
                    property {
                        key = "scripts"
                        value = "" // script 위치
                    }
                    property {
                        key = "sort"
                        value = "flyway" // flyway 방식으로 정렬할 것임을 선언
                    }
                    property {
                        key = "unqualifiedSchema"
                        value = "none"
                    }
                    property {
                        key = "defaultNameCase"
                        value = "as_is"
                    }
                }
            }

            generate {
                isDeprecated = false
                isRecords = true
                isImmutablePojos = true
                isFluentSetters = true
                isJavaTimeTypes = true
            }

            target {
                packageName = "jooq.jooq_dsl"
                directory = "src/generated"
                encoding = "UTF-8"
            }
        }
    }
}
```

이후 Gradle을 리프레시하고 태스크를 확인해 보면 jOOQ DSL을 생성해 주는 `jooqCodegen` 태스크가 추가되어 있는 것을 확인할 수 있습니다.



이때 jOOQ는 Flyway의 무언가를 활용하여 jOOQ DSL을 만든 것은 아니고 database를 `org.jooq.meta.extensions.ddl.DDLDatabase` 로 설정한 것과 `sort : flyway`에서 알 수 있듯 `*.sql`를 Flyway가 `*.sql`를 읽는 순서로 읽는 것입니다.



_추가로 JPA를 활용한 jOOQ DSL 생성 방법은 [링크](https://sightstudio.tistory.com/68) 참고하면 됩니다._



### jOOQ JavaConfig

`org.springframework.boot.autoconfigure.jooq`가 존재하기에 따로 jOOQ 관련 JavaConfig를 만들지 않고도 `application.yml`을 통해 설정할 수 있습니다.

하지만 직접 jOOQ 관련 설정을 한다면 아래와 같이 코드를 작성할 수 있습니다.

```kotlin
import com.few.api.repo.common.ExceptionTranslator
import com.few.api.repo.common.NativeSQLLogger
import com.few.api.repo.common.PerformanceListener
import org.jooq.SQLDialect
import org.jooq.impl.DataSourceConnectionProvider
import org.jooq.impl.DefaultConfiguration
import org.jooq.impl.DefaultDSLContext
import org.jooq.impl.DefaultExecuteListenerProvider
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import javax.sql.DataSource

@Configuration
class JooqConfig(
    private val dataSource: DataSource,
) {
    @Bean
    fun dsl(): DefaultDSLContext {
        return DefaultDSLContext(configuration())
    }

    @Bean
    fun configuration(): DefaultConfiguration {
        val jooqConfiguration = DefaultConfiguration()
        jooqConfiguration.set(connectionProvider())
        jooqConfiguration.set(DefaultExecuteListenerProvider(exceptionTransformer())) // ExecuteListener를 상속한 클래스 + 기본 ExecuteListener로 설정
        jooqConfiguration.set(NativeSQLLogger(), PerformanceListener()) // ExecuteListener를 상속한 클래스
        jooqConfiguration.set(SQLDialect.MYSQL)
        return jooqConfiguration
    }

    @Bean
    fun connectionProvider(): DataSourceConnectionProvider {
        return DataSourceConnectionProvider(dataSource)
    }

    @Bean
    fun exceptionTransformer(): ExceptionTranslator {
        return ExceptionTranslator()
    }
}
```

참고 : [https://sightstudio.tistory.com/66](https://sightstudio.tistory.com/66)



## jOOQ 사용

`DSLContext`와 `jooq.jooq_dsl.tables`에 생성된 jOOQ DSL을 활용하여 jOOQ를 사용할 수 있습니다.

```sql
create table MEMBER
(
    id          bigint auto_increment        primary key,
    email       varchar(255)                        not null,
    type_cd     tinyint                             not null,
    created_at  timestamp default CURRENT_TIMESTAMP not null,
    modified_at timestamp default CURRENT_TIMESTAMP not null,
    deleted_at  timestamp                           null,
    constraint email
    unique (email)
);
```

위와 같은 MEMBER 테이블을 기준으로 간단히 jOOQ를 사용한 쿼리 테스트 작성하면 아래와 같습니다.



### 생성

```kotlin
@Test
@Transactional
fun `새로운 정보를 저장합니다`() {
    // given
    val email = "test2@gmail.com"
    val typeCd: Byte = 1

    // when
    val result = dslContext.insertInto(Member.MEMBER)
        .set(Member.MEMBER.EMAIL, email)
        .set(Member.MEMBER.TYPE_CD, typeCd)
        .execute()

    // then
    assert(result > 0)
}

@Test
@Transactional
fun `이메일이 중복되는 경우 저장에 실패합니다`() {
    // when & then
    assertThrows<DuplicateKeyException> {
        dslContext.insertInto(Member.MEMBER)
            .set(Member.MEMBER.EMAIL, EMAIL)
            .set(Member.MEMBER.TYPE_CD, TYPECD)
            .execute()
    }
}

@Test
@Transactional
fun `이메일 값을 입력하지 않은면 저장에 실패합니다`() {
    // when & then
    assertThrows<DataIntegrityViolationException> {
        dslContext.insertInto(Member.MEMBER)
            .set(Member.MEMBER.TYPE_CD, TYPECD)
            .execute()
    }
}
```

### 조회

```kotlin
@Test
fun `이메일 일치 조건을 통해 정보를 조회합니다`() {
    // when
    val result = dslContext.selectFrom(Member.MEMBER)
        .where(Member.MEMBER.EMAIL.eq(EMAIL))
        .and(Member.MEMBER.DELETED_AT.isNull())
        .fetchOne()

    // then
    assert(result != null)
    assert(result!!.email == EMAIL)
    assert(result.typeCd == TYPECD)
    assert(result.description.equals(JSON.json("{}")))
    assert(result.createdAt != null)
    assert(result.deletedAt == null)
}

@Test
fun `이메일 불일치 조건을 통해 유저를 조회합니다`() {
    // when
    val result = dslContext.selectFrom(Member.MEMBER)
        .where(Member.MEMBER.EMAIL.ne("test2@gmail.com"))
        .and(Member.MEMBER.DELETED_AT.isNull())
        .fetch()

    // then
    assert(result.isNotEmpty())
}
```

### 수정

```kotlin
@Test
@Transactional
fun `이메일을 수정합니다`() {
    // given
    val newEmail = "test2@gmail.com"

    // when
    val update = dslContext.update(Member.MEMBER)
        .set(Member.MEMBER.EMAIL, newEmail)
        .where(Member.MEMBER.EMAIL.eq(EMAIL))
        .and(Member.MEMBER.DELETED_AT.isNull())
        .execute()

    val result = dslContext.selectFrom(Member.MEMBER)
        .where(Member.MEMBER.EMAIL.eq(newEmail))
        .and(Member.MEMBER.DELETED_AT.isNull())
        .fetchOne()

    // then
    assert(update > 0)
    assert(result != null)
    assert(result!!.email == newEmail)
}
```

### 삭제

```kotlin
dslContext.deleteFrom(Member.MEMBER).execute()
```



### 프로젝트

추가로 이번 프로젝트에서 jOOQ를 사용해 작성한 복잡한 쿼리도 소개합니다.

-   아티클의 본문 내용이 커 ARTICLE\_MST와 ARTICLE\_IFO로 분리하여 테이블을 설계한 상황에서 모든 아티클 정보를 조회하기 위해 ARTICLE\_MST와 ARTICLE\_IFO를 조인하여 조회하는 쿼리

```kotlin
dslContext.select(
      articleMst.ID.`as`(SelectWorkBookArticleRecord::articleId.name),
      articleMst.MEMBER_ID.`as`(SelectWorkBookArticleRecord::writerId.name),
      articleMst.MAIN_IMAGE_URL.`as`(SelectWorkBookArticleRecord::mainImageURL.name),
      articleMst.TITLE.`as`(SelectWorkBookArticleRecord::title.name),
      articleMst.CATEGORY_CD.`as`(SelectWorkBookArticleRecord::category.name),
      articleIfo.CONTENT.`as`(SelectWorkBookArticleRecord::content.name),
      articleMst.CREATED_AT.`as`(SelectWorkBookArticleRecord::createdAt.name),
      mappingWorkbookArticle.DAY_COL.`as`(SelectWorkBookArticleRecord::day.name)
  ).from(articleMst)
      .join(articleIfo)
      .on(articleMst.ID.eq(articleIfo.ARTICLE_MST_ID))
      .join(mappingWorkbookArticle)
      .on(mappingWorkbookArticle.WORKBOOK_ID.eq(workbookId))
      .and(mappingWorkbookArticle.ARTICLE_ID.eq(articleMst.ID))
      .where(articleMst.ID.eq(articleId))
      .and(articleMst.DELETED_AT.isNull)
      .fetchOneInto(SelectWorkBookArticleRecord::class.java)
```



-   ARTICLE\_VIEW\_COUNT 테이블에서 중복된 키가 있으면 VIEW\_COUNT를 1개 증가시키고 그렇지 않으면 1을 저장하는 Upsert 쿼리

```kotlin
dslContext.insertInto(ARTICLE_VIEW_COUNT)
    .set(ARTICLE_VIEW_COUNT.ARTICLE_ID, query.articleId)
    .set(ARTICLE_VIEW_COUNT.VIEW_COUNT, 1)
    .set(ARTICLE_VIEW_COUNT.CATEGORY_CD, query.categoryType.code)
    .onDuplicateKeyUpdate()
    .set(ARTICLE_VIEW_COUNT.VIEW_COUNT, ARTICLE_VIEW_COUNT.VIEW_COUNT.plus(1))
    .execute()
```



## 트러블 슈팅

### 사전 지식

#### jOOQ 버전에 따른 차이

jOOQ는 3.19 버전부터 Gradle을 사용한 jOOQ DSL을 생성하는 방법이 달라졌습니다.

기존의 방식은 공식문서 나와있는 아래의 코드를 통해 생성하거나

```kotlin
GenerationTool.generate(new Configuration()
    .withJdbc(new Jdbc()
        .withDriver('org.h2.Driver')
        .withUrl('jdbc:h2:~/test-gradle')
        .withUser('sa')
        .withPassword(''))
    .withGenerator(new Generator()
        .withDatabase(new Database())
        .withGenerate(new Generate()
            .withPojos(true)
            .withDaos(true))
        .withTarget(new Target()
            .withPackageName('org.jooq.example.gradle.db')
            .withDirectory('src/main/java'))))
```

Etienne Studer이란 개발자가 만든 [nu.studer.jooq](https://github.com/etiennestuder/gradle-jooq-plugin?tab=readme-ov-file)를 사용하여 jOOQ DSL을 생성하였습니다.



하지만 3.19 버전부터는 jOOQ에서 `org.jooq.jooq-codegen-gradle`을 제공하고 이를 사용해서 jOOQ DSL를 생성합니다.

실제로 `jooqCodegen`테스크 이후 생성된 `DefaultCatalog`를 확인해 보면 아래와 같은 코드를 발견할 수 있습니다.

```kotlin
/**
 * A reference to the 3.19 minor release of the code generator. If this
 * doesn't compile, it's because the runtime library uses an older minor
 * release, namely: 3.19. You can turn off the generation of this reference
 * by specifying /configuration/generator/generate/jooqVersionReference
 */
private static final String REQUIRE_RUNTIME_JOOQ_VERSION = Constants.VERSION_3_19;
```



#### SpringBoot 3.2.5 버전에서의 jOOQ

SpringBoot 3.2.5 버전에서 `org.springframework.boot:spring-boot-starter-jooq`에 포함되는 jOOQ의 기본 버전은 3.18.14 입니다.

[spring boot 3.2.5 의존성 버전](https://docs.spring.io/spring-boot/docs/3.2.5/reference/html/dependency-versions.html#appendix.dependency-versions)



### 문제

저는 이번에 jOOQ를 스프링 멀티모듈 환경에서 사용하였습니다.

`api-repo` 모듈에서 jOOQ DSL 및 DAO 클래스를 생성하고 `api` 모둘에서 이를 활용하는 방식으로 모듈을 설계하였습니다.

첫 jOOQ 사용 설정 당시 개인이 만든 `nu.studer.jooq`가 아닌 공식인 `org.jooq.jooq-codegen-gradle`를 활용하여 jOOQ DSL을 생성하고 싶었습니다.

이에 SpringBoot 3.2.5에서 지원하는 3.18.14 버전의 jOOQ가 아닌 3.19 버전의 jOOQ를 별도로 지정하였습니다.

이를 위해 `api-repo` 모듈에 `org.springframework.boot:spring-boot-starter-jooq` 뿐만 아니라 아래의 의존성도 추가하였습니다.

```kotlin
/** jooq */
api("org.springframework.boot:spring-boot-starter-jooq")

/** add for use 3.19.9 */
api("org.jooq:jooq:3.19.9")
api("org.jooq:jooq-meta:3.19.9")
api("org.jooq:jooq-codegen:3.19.9")
jooqCodegen("org.jooq:jooq-meta-extensions:3.19.9")
```

하지만 아래와 같은 에러가 발생하였습니다.

![img](https://blog.kakaocdn.net/dn/k7eRb/btsIJDVsMQi/TKpMRQIuUUto1QgNvXDlT1/img.png)



사진의 에러를 통해 `api` 모듈에 jOOQ 관련 의존성이 추가되지 않았음을 의심할 수 있었고 임시로 `api-repo`에서와 동일한 jOOQ 의존성을 추가하여 일시적으로 문제를 해결할 수 있었습니다.

하지만 이로 인해 `api` 모듈에서도 jOOQ 의존성이 추가되어 모듈 간 분리가 이루어지지 않게 되었고 제대로 된 해결책을 찾아야 했습니다.

이에 임시로 추가한 jOOQ 의존성을 제거하고 다시 `api` 모듈의 의존성을 살펴보는 도중 `api-repo` 모듈과 jOOQ 의존성이 다르게 들어간 것을 확인할 수 있었습니다.

![img](https://blog.kakaocdn.net/dn/bINKyp/btsIIHRR4GV/xc3ba89IYuBCzfTF1VQim1/img.png)

![img](https://blog.kakaocdn.net/dn/cnB0tN/btsIJXzrvEn/7F1PyeDYcRNZt5AnrKXKV0/img.png)

![img](https://blog.kakaocdn.net/dn/VhBjS/btsIJAEKdeK/02gD6PvgdiQ5dZI3HuSXKK/img.png)

이렇게 서로 다른 jOOQ 버전이 각각의 모듈에 포함되어 있음을 확인하고 jOOQ 버전에 따른 차이를 확인한 이후 `api-repo`에서 jOOQ 3.19 버전으로 만든 jOOQ DSL 클래스를 `api` 모듈에서 jOOQ 3.18 버전으로 사용하려 했기에 일어나는 문제임을 알 수 있었습니다.



### 해결

이에 저는 두 가지 해결 방법이 있다고 생각했습니다.

1.  `api` 모듈과 `api-repo` 모듈의 jOOQ 관련 의존성을 3.19로 동일하게 설정한다.
2.  jOOQ DSL 생성 방식을 `nu.studer.jooq`를 사용하는 방법으로 변경한다.



저는 위의 두 가지 방법 중 첫 번째 방법을 선택하였습니다.

그 이유는 아래와 같습니다.

-   공식 플러그인을 사용할 수 있다. (`org.jooq.jooq-codegen-gradle`)
-   추후 개발 간 SpringBoot과 jOOQ 모두 버전을 낮출 가능성보다 높일 가능성이 크다.

jOOQ 관련 의존성의 통일은 dependency-management 플러그인을 통해 아래와 같이 설정할 수 있었습니다.

```kotlin
dependencyManagement {
    dependencies {
        /**
         * spring boot starter jooq 3.2.5 default jooq version is 3.18.14.
         * But jooq-codegen-gradle need over 3.19.0.
         *  */
        dependency("org.jooq:jooq:${DependencyVersion.JOOQ}")
    }
}
```



![img](https://blog.kakaocdn.net/dn/1Y8JQ/btsIJHwPcp6/3KhzNPEudTqlZ6r2YnmTtK/img.png)



### 소감

개발하면서 의존성을 추가할 때는 각 버전의 차이를 알고 적절하게 도입해야 한다는 것은 알고 있었지만 버전과 관련된 문제를 직접 겪어본 적은 없어 이번 경험을 통해 앞으로 의존성을 사용할 때는 버전 간 차이점도 알아보고 조금 더 신중하게 이를 적용해야겠다는 생각을 할 수 있었습니다.