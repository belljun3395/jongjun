# [FEW] MVP 기능을 구현하며




1.  다중 칼럼 인덱스 사용 시 주의점

: 다중 컬럼칼럼 인덱스에서 각 인덱스를 구성하는 칼럼은 첫 번째 칼럼부터 차례대로 정렬되고, N번째 인덱스 칼럼은 N-1번째 인덱스의 정렬에 의존해서 정렬된다. 따라서 앞선 인덱스를 통해 추출되는 데이터가 적어지는 방향으로 다중 칼럼 순서를 정의해야 한다. (카디널리티가 높은 순에서 낮은 순)



참고 링크

-   [https://studyandwrite.tistory.com/553](https://studyandwrite.tistory.com/553)
-   [https://jojoldu.tistory.com/243](https://jojoldu.tistory.com/243)



2.  where 절, on 절 그리고 실행순서

```sql
create table ARTICLE_MST
(
    id             bigint auto_increment
        primary key,
    member_id      bigint                              not null,
    main_image_url varchar(255)                        not null,
    title          varchar(255)                        not null,
    category_cd    tinyint                             not null,
    created_at     timestamp default CURRENT_TIMESTAMP not null,
    deleted_at     timestamp                           null
);

create table ARTICLE_IFO
(
    article_mst_id bigint     not null
        primary key,
    content        mediumtext not null,
    deleted_at     timestamp  null
);

create index article_mst_idx1
    on ARTICLE_MST (member_id);

create table MAPPING_WORKBOOK_ARTICLE
(
    workbook_id bigint    not null,
    article_id  bigint    not null,
    day_col     int       not null,
    deleted_at  timestamp null,
    primary key (workbook_id, article_id)
);
```

```sql
SELECT 
    article_mst.id AS articleId,
    article_mst.member_id AS writerId,
    article_mst.main_image_url AS mainImageURL,
    article_mst.title AS title,
    article_mst.category_cd AS category,
    article_ifo.content AS content,
    article_mst.created_at AS createdAt,
    mapping_workbook_article.day_col AS day
FROM 
    mapping_workbook_article
JOIN 
    article_mst 
    ON mapping_workbook_article.article_id = article_mst.id
JOIN 
    article_ifo 
    ON article_mst.id = article_ifo.article_mst_id
WHERE 
    mapping_workbook_article.workbook_id = :workbookId;
```

위의 쿼리를 작성할 때 `ARTICLE_IFO`의 `content` 칼럼이 `mediumtext` 타입이기 때문에 최대한 마지막에 해당 칼럼이 조회 결과에 합쳐지기를 바랐습니다.

그래서 select 쿼리의 실행 순서를 알아보았을 때 from/join, where, group by, having, select, order by 순으로 진행된다는 것을 알 수 있었고 on 절에 where 절의 조건을 추가하여 join 되는 대상을 줄이고 싶다는 생각을 하였습니다.



![img](https://blog.kakaocdn.net/dn/bNJyed/btsIAaY410J/fTkDD20kCJTncKwVcgh3w0/img.png)

하지만 `explain`을 통해 실제 실행계획을 확인해 본 결과 위와 같은 순서로 실행됨을 확인할 수 있었습니다.

예상과 달리 `where`절이 먼저 실행되고 `mapping_workbook_article`과 `article_mst`의 조인 마지막으로 `mapping_workbook_article`과 `article_ifo`의 조인이 실행됨을 확인할 수 있었습니다.

where 절의 조건을 on 절에 추가하여 join 대상을 줄이지 않더라도 DB 옵티마이저가 쿼리 실행 순서를 조정하여 효율적으로 실행하고 있음을 확인할 수 있었습니다.

추가로 해당 위의 쿼리를 고민하며 on 절과 where 절은 용도와 의미가 명백히 다르며 의미에 맞게 사용해야 함을 알 수 있었습니다.



참고 링크:

-   실행계획 순서 보는 법 - [https://insanelysimple.tistory.com/424](https://insanelysimple.tistory.com/424)
-   on 절과 where 절 이해하기 - [https://jaehoney.tistory.com/391](https://jaehoney.tistory.com/391)



3.  in 연산자 오용

: where 절에 in 연산자를 여러 개 사용하며 잘못된 쿼리를 작성하였습니다. 해당 쿼리의 목적은 마지막 학습지를 받은 구독자들은 구독을 해지하기 위한 쿼리로 아래와 같습니다.

```sql
/** 잘못된 JooQ 쿼리 */
dslContext.update(subscriptionT)
    .set(subscriptionT.DELETED_AT, LocalDateTime.now())
    .set(subscriptionT.UNSUBS_OPINION, "receive.all")
    .where(subscriptionT.MEMBER_ID.`in`(receiveLastDayMembers))
    .and(subscriptionT.TARGET_WORKBOOK_ID.`in`(targetWorkBookIds))
    .execute()
```

-   memberId: 1 / workbookId: 1 / recieveAll
-   memberId: 1 / workbookId: 2 / ing
-   memberId: 2 / workbookId: 2 / recieveAll
-   memberId: 3 / workbookId: 1 / recieveAll

위와 같은 상황을 가정해 보면 receiveLastDayMembers는 '1,2,3' 그리고 targetWorkBookIds는 '1,2'가 됩니다.

```sql
.where(subscriptionT.MEMBER_ID.`in`(receiveLastDayMembers))
.and(subscriptionT.TARGET_WORKBOOK_ID.`in`(targetWorkBookIds))
```

그리고 위의 where 절의 조건을 해석해 보면 receiveLastDayMembers의 값 중 하나와 targetWorkBookIds의 값 중 하나를 선택한 것을 의미하고 그 결과는 아래와 같습니다.

-   memberId: 1 / workbookId: 1
-   memberId: 1 / workbookId: 2
-   memberId: 2 / workbookId: 1
-   memberId: 2 / workbookId: 2
-   memberId: 3 / workbookId: 1
-   memberId: 3 / workbookId: 2

이는 원하는 결과와 달랐고 in을 사용하지 않는 아래의 쿼리로 수정하였습니다.

```sql
/** 수정한 JooQ 쿼리 batchUpdate로 추가 수정 필요 */
for (receiveLastDayMember in receiveLastDayMembers) {
    dslContext.update(subscriptionT)
        .set(subscriptionT.DELETED_AT, LocalDateTime.now())
        .set(subscriptionT.UNSUBS_OPINION, "receive.all")
        .where(subscriptionT.MEMBER_ID.eq(receiveLastDayMember.memberId))
        .and(subscriptionT.TARGET_WORKBOOK_ID.eq(receiveLastDayMember.targetWorkBookId))
        .execute()
}
```



테스트 코드를 작성하지 않은 것은 아니지만 해당 경우에 대한 테스트 케이스를 놓였습니다.

하지만 로그와 데이터베이스를 점검하는 과정에서 해당 잘못을 확인하고 수정할 수 있었습니다.

![img](https://blog.kakaocdn.net/dn/kMamN/btsIx3gxZth/CXbIjHQJFTlVNnsk3nM2V1/img.png)

![img](https://blog.kakaocdn.net/dn/F9tUn/btsIzA4YCsQ/GE8Wv3rLQI32os8bBEku10/img.png)



4.  DB 대소문자 구분

: `lower_case_table_names` 설정을 통해 DB에서 대소문자 구분 여부를 설정할 수 있다.

-   `lower_case_table_names = 0`: 테이블 생성 및 조회 시 대·소문자 구분
-   `lower_case_table_names = 1`: 입력 값이 대·소문자든 소문자로 인식 소문자 인식 파일 생성
-   `lower_case_table_names = 2`: 윈도우에서 대·소문자를 구분해서 테이블생성

해당 프로젝트에서는 Flyway를 통해 DB 형상을 관리하고 JooQ를 통해 쿼리를 작성하고 있는데 Flyway에서는 소문자를 JooQ에서는 대문자를 사용하여 위 설정을 인지할 수 있었습니다.



5.  JooQ + Flyway 그리고 멀티모듈

: 해당 프로젝트는 멀티모듈 구조로 아래와 같은 모듈을 가지고 있습니다.

-   `api`: api-repo를 통해 조회한 결과를 바탕으로 클라이언트의 요청을 처리한다.
-   `api-repo`: db와 연결을 진행하고 data에서 정의한 테이블 정의를 바탕으로 쿼리를 작성한다.
-   `data`: 테이블을 정의한다.

그리고 JooQ에서 Flyway에서 사용하는 테이블 정의 `.sql`파일을 활용하여 JooQ 클래스를 만들 수 있도록 지원하여 Flyway를 통해 DB 형상을 관리하고 JooQ를 통해 쿼리를 작성하고 있습니다.



JooQ는 해당 프로젝트에서 처음 도전하는 라이브러리로 개발 간 여러 문제를 마주할 수 있었습니다.

-   `api-repo`에서 `data` 모듈의 `.sql` 파일을 참조하지 못하는 문제
-   `api`에서 `api-repo` 모듈을 포함하여도 JooQ 관련 의존성을 추가해야 하는 문제



`api-repo`에서 `data` 모듈의 `.sql` 파일을 참조하지 못하는 문제

: 위와 같이 `data`모듈과 `api-repo`모듈을 분리하며 `data` 모듈에서 정의한 `.sql` 파일을 `api-repo`에서 사용하지 못하는 문제를 마주하였습니다.

```
--- api-repo
----- src
--- data
----- src
----- db
```



해당 문제의 경우 `data` 모듈의 db 파일을 `api-repo` 모듈에 복사하는 태스크를 추가하여 해결하였습니다.

```
/** copy data migration */
tasks.create("copyDataMigration") {
    doLast {
        val root = rootDir
        val flyWayResourceDir = "/db/migration/entity"
        val dataMigrationDir = "$root/data/$flyWayResourceDir"
        File(dataMigrationDir).walkTopDown().forEach {
            if (it.isFile) {
                it.copyTo(
                    File("${project.projectDir}/src/main/resources$flyWayResourceDir/${it.name}"),
                    true
                )
            }
        }
    }
}
```



그리고 해당 테스크를 `compileKotlin`와 `jooqCodegen` 이전에 실행하도록 하는 설정 또한 추가하였습니다.

```
/** copy data migration before compile kotlin */
tasks.getByName("compileKotlin") {
    dependsOn("copyDataMigration")
}

/** jooq codegen after copy data migration */
tasks.getByName("jooqCodegen") {
    dependsOn("copyDataMigration")
}
```



참고 링크:

-   JooQ & Flyway - [https://www.jooq.org/doc/latest/manual/getting-started/tutorials/jooq-with-flyway/](https://www.jooq.org/doc/latest/manual/getting-started/tutorials/jooq-with-flyway/) 



6.  minio와 ncp는 정말 s3와 동일하게 사용할 수 있다.

: 이미지와 같은 기능을 구현할 때 오브젝트 스토어인 s3를 많이 사용합니다. 그렇기에 다른 minio와 ncp와 같은 다른 오브젝트 스토어 역시 s3와 호환을 지원하고 있습니다. 해당 프로젝트에서는 `prd` 환경에서는 오브젝트 스토어로 ncp를 사용하고 `local` 환경에서는 minio를 사용하고 있습니다. 이전 프로젝트에서는 minio와 s3의 호환을 확인하지 못하여 인터페이스와 프로필을 활용하여 두 가지 모두를 구현하였지만 현 프로젝트에서는 s3, ncp, minio 간의 호환성을 확인하였고 s3를 기준으로 하나만 개발할 수 있었습니다.

```kotlin
@Bean
fun s3StorageClient(): AmazonS3Client {
    val awsCredentials = BasicAWSCredentials(accessKey, secretKey)
    return AmazonS3ClientBuilder.standard()
        .withRegion(region)
        .withCredentials(AWSStaticCredentialsProvider(awsCredentials))
        .build() as AmazonS3Client
}
```

```kotlin
@Bean
fun s3StorageClient(): AmazonS3Client {
    val builder = AmazonS3ClientBuilder.standard()
        .withCredentials(
            AWSStaticCredentialsProvider(
                BasicAWSCredentials(
                    accessKey,
                    secretKey
                )
            )
        )
        .withEndpointConfiguration(
            AwsClientBuilder.EndpointConfiguration(
                url,
                region
            )
        )

    builder.build().let { client ->
        return client as AmazonS3Client
    }
}
```

s3 관련 기능 구현 레퍼런스를 찾아보면 위의 코드가 많이 보이는데 아래의 코드와 같이 엔드포인트를 직접 지정해 주면 s3를 지원해 주는 다양한 오브젝트 스토어를 하나의 코드로 사용할 수 있습니다.



7.  다양한 Github Action 및 자동화 도입

-   [담당자 자동 지정 액션](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/workflows/auto-assign-author-to-assignees.yml)
-   [린트 확인 액션](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/workflows/lint.yml)
-   [작업 중 PR 머지 방지 WIP 액션](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/workflows/wip.yml)
-   [자동 라벨 지정 액션](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/workflows/labeler.yml)
-   [GPT 리뷰 액션](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/workflows/gpt_code_review.yml) (지금은 사용 x)
-   [코드 오너 지정](https://github.com/YAPP-Github/24th-Web-Team-1-BE/blob/main/.github/CODEOWNERS)으로 리뷰어 자동 지정



참고 링크:

-   GPT 리뷰 후기 - [https://itchipmunk.tistory.com/592](https://itchipmunk.tistory.com/592)
-   WIP 공식 문서 - [https://github.com/wip/action](https://github.com/wip/action)
-   WIP 적용 후기 - [https://bitlog.tistory.com/55](https://bitlog.tistory.com/55)