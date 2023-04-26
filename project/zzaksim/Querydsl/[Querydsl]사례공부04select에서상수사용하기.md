## [Querydsl] 사례공부04 / select에서 상수 사용하기

*출처 : https://jojoldu.tistory.com/523*



우선 쿼리 성능을 개선 할 수 있는 아래와 같은 방법이 있다.

- check for indexes
- work with the smallest data set required
- **remove unnecessary fields** and tables and
- remove calculations in your JOIN and WHERE clauses.

이 중에서도 조회하는 컬럼의 수를 초소화하는 방법을 알아보자.

이는 Entity로 조회 결과를 가져오기 보다는 Dto로 필요한 필드만 가져오길 권장하는 이유기도 한다고 한다.



조회하는 컬럼의 수를 줄이는 방법으로 단순히 5개의 컬럼을 꼭 필요한 3개로 줄이는 방법이 있을 수 있다.

하지만 이보다 더 줄일 수 있는 방법은 이미 선언되어있는 값을 그대로 사용하는 것이다.

메소드의 인자값으로 넘어왔거나, 다른 메소드를 통해 이미 알고 있는 값의 경우 굳이 DB에서 다시 조회할 필요가 없다고 한다.



Querydsl에서는 상수를 사용하기 위해 아래와 같은 방법을 사용한다.

### 1. constantAs

우선 아래와 같은 조회용 Dto가 있다고 생각하자.

```java
@Getter
@NoArgsConstructor
public class BookPageDto {
    private String name;
    private int pageNo;
    private int bookNo;
}
```

그리고 constantAs를 활용한 조회 코드이다.

```java
public List<BookPageDto> getBookPage (int bookNo, int pageNo) { // bookNo를 인자값으로 받는다.
    return queryFactory
            .select(Projections.fields(BookPageDto.class,
                    book.name,
                    Expressions.constantAs(bookNo, book.bookNo) // constantAs를 활용하여 bookNo를 Dto에 전달한다.
                )
            )
            .from(book)
            .where(book.bookNo.eq(bookNo))
            .offset(pageNo)
            .limit(10)
            .fetch();
}
```

```sql
Hibernate: 
    select
        book0_.name as col_0_0_ 
    from
        book book0_ 
    where
        book0_.book_no=? limit ?
```

쿼리를 확인해 보면 위처럼 name만 조회해오는 것을 확인할 수 있다.



### 2. constant

위의 조회에서 "pageNo는?"하는 의문이 있을 것이다.

```java
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private int pageNo;
}
```

위의 Book 엔티티를 확인해보면 pageNo는 존재하지 않는다.

이렇게 엔티티에 존재하지않지만 Dto에는 존재하는 값을 넣기 위해서는 constant를 활용해야 한다.

위의 조회문의 Projections.fields에 추가될 코드는 아래와 같다.

`Expressions.as(Expressions.constant(pageNo), "pageNo")` 

`Expressions.constant(pageNo)` 를 통해 조회 결과에 사용될 상수를 지정하고 `Expressions.as(상수값, "pageNo")` 를 통해 상수값 alias를 지정한다.
