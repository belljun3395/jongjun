## [Querydsl] 공식문서02

### Tutorials

#### Querying JPA

Querydsl은 일반적으로 도메인 모델 데이터를 질의하기 위한 정적 타입의 문법으로 정의한다.

JDO 그리고 JPA는 Querydsl을 위한 주요 통합 기술이다.



JPA용 Querydsl은 JPQL 및 Criteria 쿼리의 대안이다.

Querydsl은 Criteria 쿼리의 동적 특성과 JPA의 표현력, 그리고 이 모든 것을 타입 세이프한 방식으로 결합한다.



#### Using query types

```java
@Entity
public class Customer {
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setFirstName(String fn) {
        firstName = fn;
    }

    public void setLastName(String ln) {
        lastName = ln;
    }
}
```

Querydsl은 `Customer`라는 간단한 이름의 쿼리 유형을 `Customer`와 동일한 패키지로 생성한다. 

**`QCustomer`는** `Customer` 타입에 대한 대표로 Querydsl 쿼리에서 **정적 타입의 변수로 사용할 수 있다.**

`QCustomer`에는 **정적 필드로 액세스할 수 있는 기본 인스턴스 변수**가 있습니다 

```java
QCustomer customer = QCustomer.customer;
```

혹은 `Custumer` 변수를 아래와 같이 **정의할 수도 있다.**

```java
QCustomer customer = new QCustomer("myCustomer");
```



#### Querying

Querydsl JPA 모듈은 JPA 그리고 Hibernate API 둘 모두를 지원한다.

JPA API를 사용하려면 다음과 같이 JPAQuery 인스턴스화할 수 있다.

```java
// where entityManager is a JPA EntityManager
JPAQuery<?> query = new JPAQuery<Void>(entityManager);
```

Hibernate API를 사용하려면 다음과 같이 HibernateQuery를 인스턴스화할 수 있다.

```java
// where session is a Hibernate session
HibernateQuery<?> query = new HibernateQuery<Void>(session);
```

JPAQuery와 HibernateQuery는 모두 JPQLQuery 인터페이스를 구현한다.

이 장의 예제에서 쿼리는 JPAQueryFactory 인스턴스를 통해 생성된다. 

JPAQueryFactory는 JPAQuery 인스턴스를 얻기 위해 선호되는 옵션이다.



이름이 "Bob"인 Customer을 검색하려면 다음과 같이 쿼리를 작성한다.

```java
QCustomer customer = QCustomer.customer;
Customer bob = queryFactory.selectFrom(customer)
  .where(customer.firstName.eq("Bob"))
  .fetchOne();
```

`selectFrom` 호출은 쿼리 소스와 투영(projection)을 정의하고, `where` 호출은 필터를 정의한다.

그리고 `fetchOne`은 **단일 요소**를 반환하도록 Querydsl에 지시한다.



하나의 소스가 아니라 여러 소스가 있는 쿼리를 만들려면 다음과 같이 쿼리를 사용한다.

```java
QCustomer customer = QCustomer.customer;
QCompany company = QCompany.company;
query.from(customer, company);
```



하나의 조건이 아니라 여러 조건이 있는 쿼리는 다음과 같이 사용한다.

```java
// select customer from Customer as customer
// where customer.firstName = "Bob" and customer.lastName = "Wilson"

// #1
where customer.firstName = "Bob" and customer.lastName = "Wilson"
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob"), customer.lastName.eq("Wilson"));

// #2
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob").and(customer.lastName.eq("Wilson")));
```



#### Using joins

Querydsl은 JPQL에서 다음과 같은 조인을 지원한다 : `inner join`, `join`, `left join` , `right join`

이를 사용하는 방법은 아래와 같다.

```java
QCat cat = QCat.cat;
QCat mate = new QCat("mate");
QCat kitten = new QCat("kitten");

// select cat from Cat as cat
// inner join cat.mate as mate
// left outer join cat.kittens as kitten
queryFactory.selectFrom(cat)
    .innerJoin(cat.mate, mate)
    .leftJoin(cat.kittens, kitten)
    .fetch();

// select cat from Cat as cat
// left join cat.kittens as kitten
// on kitten.bodyWeight > 10.0
queryFactory.selectFrom(cat)
    .leftJoin(cat.kittens, kitten)
    .on(kitten.bodyWeight.gt(10.0))
    .fetch();
```



#### General usage

JPQLQeury 인터페이스를 주로 아래와 같이 사용한다.

+ `select` : 쿼리의 투영(projection)을 설정합니다. (queryFactory를 통해 생성한 경우 필요하지 않음)
+ `from` : 쿼리 소스를 추가한다.
+ `innerJoin`, `leftJoin`, `rightJoin`, `on` :  이 구문을 통해 조인 요소를 추가한다. 조인 메서드의 경우 첫 번째 인수는 조인 소스이고 두 번째 인수는 별칭이다.
+ `groupBy` : 인자별로 그룹을 varargs 형식으로 추가한다.
+ `having` : 조건식 표현식의 varags 배열로 '그룹 기준' 그룹화 필터를 추가한다.
+ `orderBy` : 결과의 순서를 주문 표현식의 varargs 배열로 추가한다. 숫자, 문자열 및 기타 비교 가능한 표현식에 asc() 및 desc()를 사용하여 OrderSpecifier 인스턴스에 액세스한다.
+ `limit`, `offset`, `restrict` : 결과의 페이징을 설정한다. 최대 결과에 대한 제한, 행 건너뛰기에 대한 오프셋, 한 번의 호출에 두 가지를 모두 정의하는 제한을 설정한다.



#### Ordering

```java
// select customer from Customer as customer
// order by customer.lastName asc, customer.firstName desc
QCustomer customer = QCustomer.customer;
queryFactory.selectFrom(customer)
    .orderBy(customer.lastName.asc(), customer.firstName.desc())
    .fetch();
```



#### Grouping

```java
// select customer.lastName
// from Customer as customer
// group by customer.lastName
queryFactory.select(customer.lastName).from(customer)
    .groupBy(customer.lastName)
    .fetch();
```



#### Delete clauses

Querydsl JPA는 `delete-where-execute` 형식을 따른다.

```java
QCustomer customer = QCustomer.customer;
// delete all customers
queryFactory.delete(customer).execute();
// delete all customers with a level less than 3
queryFactory.delete(customer).where(customer.level.lt(3)).execute();
```

`where` 호출은 선택 사항이며 실행 호출은 삭제를 수행하고 **삭제된 엔티티의 양을 반환한다.**



#### Update clauses

Querydsl JPA는 `update-set/where-execute` 형식을 따른다.

```java
QCustomer customer = QCustomer.customer;
// rename customers named Bob to Bobby
queryFactory.update(customer).where(customer.name.eq("Bob"))
    .set(customer.name, "Bobby")
    .execute();
```

`set` 호출은 SQL 업데이트 스타일로 속성 업데이트를 정의한다.

`execute`는 업데이트를 수행하고 **업데이트된 엔티티의 양을 반환한다.**



#### Subqueries

Subquery를 만들려면 JPAExpression의 정적 팩토리 메서드를 사용하고 `from`, `where` 등을 통해 쿼리 매개변수를 정의한다.

```java
// #1
QDepartment department = QDepartment.department;
QDepartment d = new QDepartment("d");
queryFactory.selectFrom(department)
    .where(department.size.eq(
        JPAExpressions.select(d.size.max()).from(d)))
     .fetch();

// #2
QEmployee employee = QEmployee.employee;
QEmployee e = new QEmployee("e");
queryFactory.selectFrom(employee)
    .where(employee.weeklyhours.gt(
        JPAExpressions.select(e.weeklyhours.avg())
            .from(employee.department.employees, e)
            .where(e.manager.eq(employee.manager))))
    .fetch();
```



