## [Querydsl] 사례공부05 / Groupby 최적화하기

*출처 : https://jojoldu.tistory.com/477*



우선 MySQL에서는 **Group by를 하면 file sort가 함께 수행**된다고 한다.

인덱스에 있는 칼럼으로 Group by를 한다면 이미 인덱스로 인해 정렬된 상태이기에 큰 문제가 되지 않는다.

하지만 만약 정렬이 필요 없는 Group by라면 자동 정렬은 성능에 악영향을 줄 뿐이다.

**자동으로 정렬이 되지 않**게 하기 위해서 MySQL에서는 **order by null**을 사용한다.



하지만 Querydsl에서는 order by null 구문에 대해서는 아직 지원하지 않는다고 한다.

그래서 아래와 같이 별도의 Order 클래스를 생성하여야 한다고 한다.

```java
public class OrderByNull extends OrderSpecifier {
    public static final OrderByNull DEFAULT = new OrderByNull();

    private OrderByNull() {
        super(Order.ASC, NullExpression.DEFAULT, NullHandling.Default);
    }
}
```

Querydsl의 정렬을 담당하는 OrderSpecifier를 상속하는 OrderByNull를 만든다.

```java
public OrderSpecifier(Order order, Expression<T> target, NullHandling nullhandling) {
    this.order = order;
    this.target = target;
    this.nullHandling = nullhandling;
}
```

이때 OrderSpecifier의 생성자는 위와 같다.

NullExpression.DEFAULT의 경우 null을 그냥 넣게 되면 OrderSpecifier에서 제대로 처리하지 못하여 대신 사용한다고 한다.



그리고 이렇게 만든 클래스를 아래와 같이 사용하면 된다.

```java
public List<Integer> getGroupOne() {
    return queryFactory.select(Expressions.ONE)
            .from(pointEvent)
            .groupBy(pointEvent.pointStatus)
            .orderBy(OrderByNull.DEFAULT) // order by null
            .fetch();
}
```