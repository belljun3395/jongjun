## [Querydsl] 사례공부06 / 페이징 성능 개선하기

*출처 : https://jojoldu.tistory.com/528*



### NoOffset

기존의 페이징 방식이 페이지 번호(offset)와 페이지 사이즈(limit)를 기반으로 한다면

**NoOffset은 페이지 번호(offset)가 없는 더 보기 방식을 이야기한다.**



NoOffset은 기존 페이징 쿼리보다 빠르다.

왜 그럴까?

우선 기존 페이징 쿼리를 살펴보자.

```sql
SELECT *
FROM items
WHERE 조건문
ORDER BY id DESC
OFFSET 페이지번호
LIMIT 페이지사이즈
```

위의 기존 쿼리에서 특히 페이징 쿼리가 뒤로 갈수록 느리다고 하는데 이는 앞에서 읽었던 행을 다시 읽기 때문이다.

뒤로 갈수록 버리지만 읽어야 하는 행의 개수가 많아져서 뒤로 갈수록 느려지는 것이다.



하지만 NoOffset 방식은 조회 시작 부분을 인덱스로 빠르게 찾아 매번 첫 페이지만 읽도록 하는 방식이다.

```sql
SELECT *
FROM items
WHERE 조건문 AND id < 마지막조회ID # 직전 조회 결과의 마지막 id
ORDER BY id DESC # 최신순 정렬
LIMIT 페이지사이즈
```

이전에 조회된 결과를 한 번에 건너뛸 수 있고 마지막 조회의 결과의 ID를 조건문에 사용하는 것으로 이는 매번 이전 페이지 전체를 건너뛸 수 있음을 의미한다.

**즉, 아무리 페이지가 뒤로 가더라도 처음 페이지를 읽은 것과 동일한 성능을 가지게 된다.**

하지만 단점 역시 존재하는데 `where`에 사용되는 기준 key가 중복인 경우 정확한 결과를 반환할 수 없다고 한다.



이러한 NoOffset 페이징을 코드로 구현하면 아래와 같이 구현할 수 있다.

```java
public List<BookPaginationDto> paginationNoOffset(Long bookId, String name, int pageSize) {

    return queryFactory
            .select(Projections.fields(BookPaginationDto.class,
                    book.id.as("bookId"),
                    book.name,
                    book.bookNo))
            .from(book)
            .where(
                    ltBookId(bookId),
                    book.name.like(name + "%")
            )
            .orderBy(book.id.desc())
            .limit(pageSize)
            .fetch();
}

private BooleanExpression ltBookId(Long bookId) {
    if (bookId == null) { // bookId가 null인 경우는 첫 페이지인 경우이다.
        return null; // BooleanExpression 자리에 null이 반환되면 조건문에서 자동으로 제거된다
    }

    return book.id.lt(bookId);
}
```





### 커버링 인덱스 사용하기

*출처 : https://jojoldu.tistory.com/529*



커버링 인덱스란 쿼리를 충족시키는데 필요한 모든 데이터를 갖고 있는 인덱스를 이야기한다.

즉, `select, where, order by, limit, group by` 등에서 사용되는 모든 컬럼이 인덱스 컬럼안에 다 포함된 경우이다.

쿼리문으로 알아보면

```sql
SELECT *
FROM items
WHERE 조건문
ORDER BY id DESC
OFFSET 페이지번호
LIMIT 페이지사이즈
```

와 같은 쿼리를

```sql
SELECT  *
FROM  items as i
JOIN (SELECT id # JOIN 부분의 쿼리가 커버링 인덱스가 사용된 부분
        FROM items
        WHERE 조건문
        ORDER BY id DESC
        OFFSET 페이지번호
        LIMIT 페이지사이즈) as temp on temp.id = i.id
```

처럼 처리한 쿼리를 이야기한다.

위의 쿼리처럼 커버링 인덱스로 걸러낸 row의 id를 통해 실제 `select` 절의 항목을 빠르게 조회하는 방법이다.



일반적으로 인덱스를 이용해 조회하여도 쿼리에서 성능 저하를 일으키는 부분은 인덱스를 검색하고 대상이 되는 row의 나머지 칼럼값을 데이터 블록에서 읽을 때다.

그렇기에 커버링 인덱스 방식을 이용하면, `where, order by, offset ~ limit`을 인덱스 검색으로 빠르게 처리하고, 걸러진 row에 대해서만 데이터 블록에 접근하여 성능의 이점을 얻을 수 있게 된다.



하지만 이 역시 단점은 존재하는 데 우선 너무 많은 인덱스가 필요하게 될 수도 있다.

그래서 인덱스가 커질 가능성이 높다.

그리고 데이터양이 많아지고, 페이지 번호가 뒤로 갈수록 NoOffset에 비해 느려진다.



Querydsl에서는 from 절의 서브쿼리를 지원하지 않기 때문에 querydsl-jpa에서 커버링 인덱스를 사용해야 한다면 2개의 쿼리로 분리해서 사용해야 한다.

```java
public List<BookPaginationDto> paginationCoveringIndex(String name, int pageNo, int pageSize) {
        // 커버링 인덱스로 대상 조회
        List<Long> ids = queryFactory
                .select(book.id)
                .from(book)
                .where(book.name.like(name + "%"))
                .orderBy(book.id.desc())
                .limit(pageSize)
                .offset(pageNo * pageSize)
                .fetch();

        // 대상이 없을 경우 추가 쿼리 수행 할 필요 없이 바로 반환
        if (CollectionUtils.isEmpty(ids)) {
            return new ArrayList<>();
        }

        return queryFactory
                .select(Projections.fields(BookPaginationDto.class,
                        book.id.as("bookId"),
                        book.name,
                        book.bookNo,
                        book.bookType))
                .from(book)
                .where(book.id.in(ids))
                .orderBy(book.id.desc())
                .fetch(); // where in id만 있어 결과 정렬이 보장되지 않는다.
}
```

이를 구현한 코드는 위와 같은데 인덱스를 조회하고 그 인덱스를 활용하여 이후 조회 과정을 진행하는 것을 확인할 수 있다.



### 페이지 건수 고정하기

*출처 : https://jojoldu.tistory.com/530*



페이징 기능에는 데이터 조회와 함께 매번 수행되는 것이 `count` 쿼리이다.

이는 조건에 따라 조회되는 결과 건수를 pageSize로 나누어 pageNo를 노출하기 위해 필요하다.

이는 총 몇 건인지 확인하기 위해 전체를 확인해야 하므로 데이터 조회만큼 오래 걸리기도 한다.



이러한 문제를 개선할 방법은 크게 두 가지가 있다고 하는데 그 방법은 아래와 같다.

1. 검색 버튼 사용 시 페이지 건수 고정하기
2. 첫 페이지 조회 결과 cache 하기



#### 검색 버튼 사용 시 페이지 건수 고정하기

이 방법은 **다음과 같은 상황에서 고려**할만한 방법이라고 한다.

- 대부분의 조회 요청이 **검색 버튼 클릭** (즉, 첫 조회)에서 발생하고
- **페이지 버튼을 통한 조회 요청이 소수**일 경우

즉, 다음 페이지로 이동하기 위해 페이지 버튼을 클릭했을 때만 실제 페이지 count 쿼리를 발생시켜 정확한 페이지수를 사용하고,

대부분의 요청이 발생하는 **검색 버튼 클릭 시에는 count 쿼리를 발생시키지 않는 것**이다.

*이는 운영 상황에 맞추어 필요시 도입하는 것이 좋을 것 같다. 그러기 위해서 요청에 대해서 로깅을 잘해둘 필요가 있을 것 같다.*



먼저 **기존 페이징 코드**는 아래와 같다.

```java
public Page<BookPaginationDto> paginationCount(Pageable pageable, String name) {
    JPQLQuery<BookPaginationDto> query = querydsl().applyPagination(pageable,
            queryFactory
                    .select(Projections.fields(BookPaginationDto.class,
                            book.id.as("bookId"),
                            book.name,
                            book.bookNo,
                            book.bookType
                    ))
                    .from(book)
                    .where(
                            book.name.like(name + "%")
                    )
                    .orderBy(book.id.desc()));

    List<BookPaginationDto> items = query.fetch(); // 데이터 조회
    long totalCount = query.fetchCount(); // 전체 count
    return new PageImpl<>(items, pageable, totalCount);
}

private Querydsl querydsl() {
    return Objects.requireNonNull(getQuerydsl());
}
```

위 코드를 **검색 버튼 클릭 시에는 10개 페이지를 고정으로 노출**하도록 개선하기 위해서는 다음의 코드가 추가되어야 한다.

1. **검색 버튼 클릭한 경우**(`useSearchBtn`)에는 10개 페이지가 노출되도록 TotalCount (`fixedPageCount`) 를 반환한다.
2. **페이지 버튼을 클릭한 경우** 실제 쿼리를 수행해 결과를 반환한다
3. 페이지 버튼을 클릭하였지만, **전체 건수를 초과한 페이지 번호**러 요청이 온 경우에는 **마지막 페이지 결과**를 반환한다.



마지막 3번이 조금 복잡한 로직인데, 이런 경우가 발생하는 이유는 다음과 같다.

- 1번으로 인해서 노출된 페이지 번호는 10개
- 실제 전체 건수와 무방하게 강제로 10개 페이지를 노출시켰기 때문에 사용자는 언제든 10번째 페이지 번호를 클릭할 수 있음
- 10번째 페이지를 클릭했는데, 막상 전체 데이터가 그만큼 안된다면 (ex: 전체 건수가 70개라면 pageSize=10 라서 실제 전체 페이지 수가 7개밖에 안 되는 경우) 노출할 데이터가 없다.



위의 요구 사항을 반영한 페이징을 코드로 확인해 보자.

```java
public Page<BookPaginationDto> paginationCountSearchBtn(boolean useSearchBtn, Pageable pageable, String name) {
    JPAQuery<BookPaginationDto> query = queryFactory
            .select(Projections.fields(BookPaginationDto.class,
                    book.id.as("bookId"),
                    book.name,
                    book.bookNo,
                    book.bookType
            ))
            .from(book)
            .where(
                    book.name.like(name + "%")
            )
            .orderBy(book.id.desc());

    JPQLQuery<BookPaginationDto> pagination = querydsl().applyPagination(pageable, query);

    if(useSearchBtn) { // 검색 버튼 사용시
        int fixedPageCount = 10 * pageable.getPageSize(); // 10개 페이지 고정
        return new PageImpl<>(pagination.fetch(), pageable, fixedPageCount);
    }

    long totalCount = pagination.fetchCount();
    Pageable pageRequest = exchangePageRequest(pageable, totalCount); // 데이터 건수를 초과한 페이지 버튼 클릭시 보정
    return new PageImpl<>(querydsl().applyPagination(pageRequest, query).fetch(), pageRequest, totalCount);
}

Pageable exchangePageRequest(Pageable pageable, long totalCount) {

    /**
      *  요청한 페이지 번호가 기존 데이터 사이즈(10)를 초과할 경우
      *  마지막 페이지의 데이터를 반환한다
      */
    int pageNo = pageable.getPageNumber();
    int pageSize = pageable.getPageSize();
    long requestCount = (pageNo - 1) * pageSize; // pageNo:10, pageSize:10 일 경우 requestCount=90

    if (totalCount > requestCount) { // 실제 전체 건수가 더 많은 경우엔 그대로 반환
        return pageable;
    }

    int requestPageNo = (int) Math.ceil((double)totalCount/pageNo); // ex: 71~79이면 8이 되기 위해
    return PageRequest.of(requestPageNo, pageSize);
}
```



---

이때 `exchangePageRequest()`는 아래와 같이 클래스로 추출할 수도 있다.

```java
public class FixedPageRequest extends PageRequest {

    protected FixedPageRequest(Pageable pageable, long totalCount) {
        super(getPageNo(pageable, totalCount), pageable.getPageSize(), pageable.getSort());
    }

    private static int getPageNo(Pageable pageable, long totalCount) {
        int pageNo = pageable.getPageNumber();
        int pageSize = pageable.getPageSize();
        long requestCount = pageNo * pageSize; // pageNo:10, pageSize:10 일 경우 requestCount=90

        if (totalCount > requestCount) { // 실제 건수가 요청한 페이지 번호보다 높을 경우
            return pageNo;
        }

        return (int) Math.ceil((double)totalCount/pageNo); // 실제 건수가 부족한 경우 요청 페이지 번호를 가장 높은 번호로 교체
    }
}
```

이를 반영하여 수정한 코드는 아래와 같다.

```java
public Page<BookPaginationDto> paginationCountSearchBtn2(boolean useSearchBtn, Pageable pageable, String name) {
    JPAQuery<BookPaginationDto> query = queryFactory
            .select(Projections.fields(BookPaginationDto.class,
                    book.id.as("bookId"),
                    book.name,
                    book.bookNo,
                    book.bookType
            ))
            .from(book)
            .where(
                    book.name.like(name + "%")
            )
            .orderBy(book.id.desc());

    JPQLQuery<BookPaginationDto> pagination = querydsl().applyPagination(pageable, query);

    if(useSearchBtn) {
        int fixedPageCount = 10 * pageable.getPageSize(); // 10개 페이지 고정
        return new PageImpl<>(pagination.fetch(), pageable, fixedPageCount);
    }

    long totalCount = pagination.fetchCount();
    Pageable pageRequest = new FixedPageRequest(pageable, totalCount);
    return new PageImpl<>(querydsl().applyPagination(pageRequest, query).fetch(), pageRequest, totalCount);
}
```

---



#### 첫 페이지 조회 결과 cache 하기

이 방법은 처음 검색 시 조회된 `count` 결과를 **응답 결과로 내려주어** JS에서 이를 캐싱하고,

매 페이징 버튼마다 `count` 결과를 함께 내려주는 방법이다.

그리고 Repository에서는 요청에 넘어온 항목 중, **캐싱 된 `count`값이 있으면 이를 재사용하고, 없으면 `count` 쿼리를 수행**한다.



그래서 이 방식은 **다음과 같은 상황에서 도움**이 된다.

- 조회 요청이 검색 버튼과 페이지 버튼 **모두에서 골고루 발생**하고
- 실시간으로 데이터 적재 되지 않고, **마감된 데이터**를 사용할 경우

이럴 경우에 사용하신다면 매 페이지 버튼 클릭 시마다 발생하는 `count` 쿼리를 처음 1회만 호출되고 이후부터는 호출되지 않아 `count` 성능을 향상 시킬 수 있다.



이는 **추가적인 쿼리 요청이 최소화된다는 장점**이 있다.

하지만 **첫 페이지 조회가 대부분일 경우에는 효과가 없고 **

**실시간으로 데이터 수정이 필요해 페이지 버튼 반영이 필요한 경우는 사용할 수 없다는 단점**이 있다.



구현의 경우 아래와 같이 할 수 있다.

```java
public Page<BookPaginationDto> paginationCountCache(Long cachedCount, Pageable pageable, String name) {
  // cachedCount : 프런트에서 넘겨주는 count 값
    JPQLQuery<BookPaginationDto> query = querydsl().applyPagination(pageable,
            queryFactory
                    .select(Projections.fields(BookPaginationDto.class,
                            book.id.as("bookId"),
                            book.name,
                            book.bookNo,
                            book.bookType
                    ))
                    .from(book)
                    .where(
                            book.name.like(name + "%")
                    )
                    .orderBy(book.id.desc()));

    List<BookPaginationDto> elements = query.fetch(); // 데이터 조회
    long totalCount = cachedCount != null ? cachedCount : query.fetchCount(); // count 캐시 여부 판단
    return new PageImpl<>(elements, pageable, totalCount);
}

private Querydsl querydsl() {
    return Objects.requireNonNull(getQuerydsl());
}
```