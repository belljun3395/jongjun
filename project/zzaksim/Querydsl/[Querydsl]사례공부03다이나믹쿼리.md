## [Querydsl] 사례공부03 / 다이나믹 쿼리

*출처 : https://jojoldu.tistory.com/394*



우선 다이나믹 쿼리란?

**상황에 따라 조건문을 생성하는 것을 말한다.**



다이나믹 쿼리를 사용하는 방법에는 **BooleanBuilder**와 **BooleanExpression**을 활용하는 방법이 있다.



우선 BooleanBuilder를 알아보자.

BooleanBuilder는 공식문서 **"General usage / Creating queries / Complex predicates"** 에 설명되어 있다.

위의 분류를 기반으로 생각해본다면 BooleanBuilder는 **다이나믹 쿼리를 위해 사용하기 보다는 복잡한 쿼리 조건을 위해 사용하는 것이 좋아보인다.**

그럼 우선 공식문서의 코드를 살펴보자.

```java
public List<Customer> getCustomer(String... names) {
    QCustomer customer = QCustomer.customer;
    JPAQuery<Customer> query = queryFactory.selectFrom(customer);
    BooleanBuilder builder = new BooleanBuilder();
    for (String name : names) {
        builder.or(customer.name.eq(name));
    }
    query.where(builder); // customer.name eq name1 OR customer.name eq name2 OR ...
    return query.fetch();
}
```

BooleanBuilder를 통해 `customer.name eq name1 OR customer.name eq name2 OR ...` 와 같은 복잡한 조건을 추가하는 것을 확인할 수 있다.



그럼 이러한 복잡한 쿼리 조건을 만드는데 사용되는 BooleanBuilder를 어떻게 다이나믹 쿼리로 사용할 수 있는 것일까?

이는 블로그 예시 코드를 통해 알아보자.

```java
@Override
public List<Academy> findDynamicQuery(String name, String address, String phoneNumber) {

    BooleanBuilder builder = new BooleanBuilder();

    if (!StringUtils.isEmpty(name)) {
        builder.and(academy.name.eq(name));
    }
    if (!StringUtils.isEmpty(address)) {
        builder.and(academy.address.eq(address));
    }
    if (!StringUtils.isEmpty(phoneNumber)) {
        builder.and(academy.phoneNumber.eq(phoneNumber));
    }

    return queryFactory
            .selectFrom(academy)
            .where(builder)
            .fetch();
}
```

if문을 통해 필요한 부분만 BooleanBuilder에 추가하며 쿼리를 만드는 것을 확인할 수 있다.

**하지만 이는 쿼리의 형태를 예측하기 어렵다는 단점이 있다.**



이러한 단점을 해결해주는 것이 BooleanExpression이다.

이 역시 블로그 예제를 통해 알아보자.

```java
@Override
public List<Academy> findDynamicQueryAdvance(String name, String address, String phoneNumber) {
    return queryFactory
            .selectFrom(academy)
            .where(eqName(name),
                    eqAddress(address),
                    eqPhoneNumber(phoneNumber))
            .fetch();
}

private BooleanExpression eqName(String name) {
    if (StringUtils.isEmpty(name)) {
        return null;
    }
    return academy.name.eq(name);
}

private BooleanExpression eqAddress(String address) {
    if (StringUtils.isEmpty(address)) {
        return null;
    }
    return academy.address.eq(address);
}

private BooleanExpression eqPhoneNumber(String phoneNumber) {
    if (StringUtils.isEmpty(phoneNumber)) {
        return null;
    }
    return academy.phoneNumber.eq(phoneNumber);
}
```

**각 메서드가 BooleanExpression을 리턴하고 각 메서드에서 조건이 맞지 않는 다면 null을 리턴한다.**

**null을 리턴하면 where에서는 상황에 따라 조건문을 생성한다.**

위의 BooleanBuilder를 사용한 예시의 builder와 

BooleanExpression를 사용한 예시의 

```java
.where(eqName(name),
        eqAddress(address),
        eqPhoneNumber(phoneNumber))
```

를 비교해보면 **가독성이 훨씬 더 좋은 것을 확인 할 수 있다.**