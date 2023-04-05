## [Querydsl] 사례공부01 / Left Outer Join

*출처 : https://jojoldu.tistory.com/342*



우선 OUTER JOIN과 INNER JOIN의 차이부터 알아보자.

**OUTER JOIN은 조건을 만족하지 않아도 기준이 되는 테이블에 해당하는 데이터는 모두 보여준다.**

반면 **INNER JOIN은 조건에 만족하는 데이터만 반환**되는 차이를 볼 수 있다.

즉, LEFT OUTER JOIN이 필요한 경우는 조건을 만족하지 않는 왼쪽 테이블은 모두 보여야 한다는 뜻이다.



블로그에서는 Child, Parent 클래스를 제시하고 이를 하나로 묶는 Family 클래스를 만드는 예제를 소개한다.

이때 Family 클래스는 Child를 가지지 않는 Parent 역시 포함하고 있다. *(자식이 없는 가족도 있을 수 있으니)*

이러한 경우가 Parent는 모두 알 수 있어야 하고 Child는 Parent에 따라 보여지면되기에 왼쪽 테이블은 모두 보여야 하는 LEFT OUTER JOIN이 필요한 경우라 할 수 있다.



이러한 상황에서는 2가지 방법으로 LEFT OUTER JOIN을 수행할 수 있다고 한다.

*Parent 2개, Child는 5개의 데이터를 저장한 상황, 1명의 부모가 5명의 자식을 두고 있음*



### 1. Entity 직접 조회 + Left Outer Join

우선 Parent Entity 를 직접 조회 하되, Child는 Left Outer Join을 하고, 조회된 결과로 Family를 생성하는 방법부터 살펴보자.

```java
@Transactional
public List<Family> findFamily() {
    List<Parent> parents = jpaQueryFactory.selectFrom(QParent.parent)
        .leftJoin(QParent.parent.children, QChild.child)
      	.fetchJoin()
      	.fetch();
  
    return parents.stream()
      						.map(p -> new Family(p.getName(), p.getChildren()))
			      			.collect(Collectors.toList());
}
```

위와 같은 코드로 나온 결과는 어떻게 될까?

우선 질의문은 다음과 같다.

```sql
Hibernate: 
    select
        parent0_.id as id1_3_0_,
        children1_.id as id1_0_1_,
        parent0_.name as name2_3_0_,
        children1_.name as name2_0_1_,
        children1_.parent_id as parent_i3_0_1_,
        children1_.parent_id as parent_i3_0_0__,
        children1_.id as id1_0_0__ 
    from
        parent parent0_ 
    left outer join
        child children1_ 
            on parent0_.id=children1_.parent_id
```

그리고 쿼리 문의 결과를 확인해 보면 아래와 같다.

<img width="361" alt="스크린샷 2023-04-05 오전 8 59 00" src="https://user-images.githubusercontent.com/102807742/229947770-5e72d095-7709-41db-9d07-8164ac54a5cb.png">



그런데 위의 결과를 보고 나면 아래의 코드의 결과가 의도한 결과가 나올 수 있을지 의문이 들 수 있다.

```java
return parents.stream()
              .map(p -> new Family(p.getName(), p.getChildren()))
              .collect(Collectors.toList());
```

아래와 같은 테스트 코드를 통해 알아보자.

```java
List<Family> family = testService.findFamily();
for (Family f : family) {
    List<Child> children = f.getChildren();
    for (Child c : children) {
        System.out.print(c.getId());
    }
    System.out.println();
}
```

```
// 결과
12345
12345
12345
12345
12345
```

의도하였던 결과는 위의 결과이기보다는 "1234"일 것이다.

*물론 의도한 결과가 나올 수 있도록 코드를 작성할 수는 있지만 다음으로 소개할 방법을 통해 간단히 구현할 수 있기에 생략...!*



### 2. Result Aggregation

Result Aggregation이란 **Querydsl의 결과를 특정 키를 기준 삼아 그룹화** 하는 것을 말한다.

그렇다고 **groupBy 쿼리가 나간다는 것은 아니다.**

쿼리 결과를 **메모리에서 집계한다.**

아래와 같이 코드를 수정하고 쿼리를 확인해보자.

```java
public List<Family> findFamily() {
    Map<Parent, List<Child>> transform = queryFactory
            .from(parent)
            .leftJoin(parent.children, child)
            .transform(groupBy(parent).as(list(child)));

    return transform.entrySet().stream()
            .map(entry -> new Family(entry.getKey().getName(), entry.getValue()))
            .collect(Collectors.toList());
}
```

```sql
Hibernate: 
    select
        parent0_.id as id1_3_0_,
        children1_.id as id1_0_1_,
        parent0_.name as name2_3_0_,
        children1_.name as name2_0_1_,
        children1_.parent_id as parent_i3_0_1_ 
    from
        parent parent0_ 
    left outer join
        child children1_ 
            on parent0_.id=children1_.parent_id
```

1번과 동일한 쿼리가 나간다.

하지만 동일한 테스트 코드를 수행하더라도 다른 결과가 나온다.

```
// 결과
12345
```