## [Querydsl] 공식문서03

### General usage

#### Creating queries

Querydsl의 쿼리 구성에는 표현식 인수를 사용하여 쿼리 메서드를 호출하는 작업이 포함된다.

표현식은 일반적으로 도메인 모듈의 생성된 표현식 유형에서 필드에 액세스하고 메서드를 호출하여 구성된다.

코드 생성이 적용되지 않는 경우에는 일반적인 표현식 구성 방법을 대신 사용할 수 있다.



##### Complex predicates

**복잡한 boolean 표현식**을 만들리면 `com.querydsl.core.BooleanBuilder` 클래스를 사용하면된다.

이 클래스는 `Predicate`를 구현하고 cascaded 형식으로 사용 가능하다.

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

`BooleanBuilder`는 변경 가능하며 처음에는 `null`을 나타내고 이후에는 `and` 혹은 `or` 연산 결과를 호출한다.



---

##### Dynamic Query use BooleanExpression

BooleanExpression은 where에서 사용할 수 있는 값으로 `,`는 `and` 조건으로 사용된다.

그리고 `null` 은 조건문에서 제외된다.

그렇기에 동적 쿼리를 작성할 때, **조건이 맞지 않으면 null을 리턴**하고 **조건이 맞다면 BooleanExpression을 리턴**하면 된다.

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

*출처 : https://jojoldu.tistory.com/394*

---



##### Dynamic expressions

`com.querydsl.core.types.dsl.Expressions` 클래스는 동적 표현식 구성을 위한 정적 팩토리 클래스다.

팩토리 메서드는 반환된 유형에 따라 이름이 지정되며 대부분 자체 문서화된다.

일반적으로 표현식 클래스는 동적 경로, 사용자 지정 구문 또는 사용자 지정 연산과 같이 **`fluent DSL` 형식을 사용할 수 없는 경우에만 사용해야 한다.**

```java
QPerson person = QPerson.person;
person.firstName.startsWith("P");

// Q-type을 사용할 수 없는 경우
Path<Person> person = Expressions.path(Person.class, "person");
Path<String> personFirstName = Expressions.path(String.class, person, "firstName");
Constant<String> constant = Expressions.constant("P");
Expressions.predicate(Ops.STARTS_WITH, personFirstName, constant);
```

`Path` 인스턴스는 변수 및 속성을 나타내고, `Constants`는 상수, `Operations`는 연산, `TemplateExpression` 인스턴스는 표현식을 문자열 템플릿으로 표현하는 데 사용할 수 있다.



##### Dynamic paths

Expression 표현식 생성 외에도 Querydsl은 동적 경로 생성을 위한 fluent API를 제공한다.

동적 경로 생성의 경우 `com.querydsl.core.types.dsl.PathBuilder` 클래스를 사용할 수 있다.

이는 EntityPathBase를 상속하고 클래스 생성 및 경로 생성을 위한 별칭 사용의 대안으로 사용할 수 있다.

Expressions API와 비교해 PathBuilder는 알 수 없는 연산이나 사용자 지정 구문에 대한 직접적인 지원을 제공하지 않지만 구문은 일반 DSL에 더 가깝다.

```java
// String property:
PathBuilder<User> entityPath = new
PathBuilder<User>(User.class, "entity");
// fully generic access
entityPath.get("userName");
// .. or with supplied type
entityPath.get("userName", String.class);
// .. and correct signature
entityPath.getString("userName").lower();

// List property with component type:
entityPath.getList("list", String.class).get(0);
// Using a component expression type:
entityPath.getList("list", String.class, StringPath.class).get(0).lower();

// Map property with key and value type:
entityPath.getMap("map", String.class, String.class).get("key");
// Using a component expression type:
entityPath.getMap("map", String.class, String.class, StringPath.class).get("key").lower();

// For PathBuilder validation a PathBuilderValidator can be used. It can be injected in the constructor and will be used transitively for the new PathBuilder
PathBuilder<Customer> customer = new PathBuilder<Customer>(Customer.class, "customer", validator);
```



##### Case expressions

`case-when-then-else` 표현을 만들기 위해  `CaseBuilder` 클래스를 사용한다.

이제 투영 또는 조건에 Case expression을 사용할 수 있다.

---

***Case expression***

*CASE 표현식은 조건을 살펴보고 **첫 번째 조건이 충족되면 값을 반환한다** (예: if-then-else 문).* 

*따라서 조건(WHEN)이 참이면 읽기를 중지하고 결과를 반환한다.(THEN)*

*조건이 참이 아닌 경우 ELSE 절에 있는 값을 반환한다.*

----

```java
// #1
QCustomer customer = QCustomer.customer;
Expression<String> cases = new CaseBuilder()
    .when(customer.annualSpending.gt(10000)).then("Premier")
    .when(customer.annualSpending.gt(5000)).then("Gold")
    .when(customer.annualSpending.gt(2000)).then("Silver")
    .otherwise("Bronze");

// #2
QCustomer customer = QCustomer.customer;
Expression<String> cases = customer.annualSpending
    .when(10000).then("Premier")
    .when(5000).then("Gold")
    .when(2000).then("Silver")
    .otherwise("Bronze");
```



#####  Casting expressions

표현식 유형에서 일반 서명을 피하기 위해 유형 계층 구조가 평평해진다. (flattened)

그 결과 생성된 모든 쿼리 유형은 `com.querydsl.core.types.dsl.EntityPathBase` 또는 `com.querydsl.core.types.dsl.BeanPath`의 직접 하위 클래스이며 해당 논리적 상위 유형으로 직접 캐스팅할 수 없게 된다.

직접 Java 형변환 대신 `_super` 필드를 통해 슈퍼타입 참조에 액세스할 수 있다. 

슈퍼 필드는 단일 상위 유형으로 생성된 모든 쿼리 유형에서 사용할 수 있다.

```java
// from Account
QAccount extends EntityPathBase<Account> {
    // ...
}

// from BankAccount extends Account
QBankAccount extends EntityPathBase<BankAccount> {

    public final QAccount _super = new QAccount(this);

    // ...
}

// To cast from a supertype to a subtype you can use the as-method of the EntityPathBase class:
QAccount account = new QAccount("account");
QBankAccount bankAccount = account.as(QBankAccount.class);
```



##### Select literals

literals는 상수 표현식을 통해 참조하여 선택할 수 있다.

---

*조회에 필요한 필드들만 조회하였지만, 그럼에도 더 줄일 수 있는 방법은 무엇이 있을까?*

*가장 쉽게 해볼 수 있는 것이 **이미 선언되어있는 값은 그대로 사용**하는 것다..*

*메소드의 인자값으로 넘어왔거나, 다른 메소드를 통해서 이미 알고 있는 값을 굳이 DB에서 다시 조회해올 필욘 없겠죠?*

*출처 : https://jojoldu.tistory.com/523*

---

Select literals는 위와 같은 경우에 사용하는 것이다.



그 전에 우선 가장 기본적인 방법을 살펴보자.

```java
query.select(Expressions.constant(1),
             Expressions.constant("abc")); // Expressions.constant를 통해 값을 조회해오는 대신 1, abc를 고정적으로 사용 사용한다.
```



**DTO를 사용한다면 아래와 같이 사용할 수 있다.**

```java
query.select(
        Projections.fields(SampleDTO.class,
            Expressions.constantAs("name", QSample.sample.name),
            Expressions.as(Expressions.constant("another name"), "another_name")
        ),
        Expressions.constant(1));
```

+ `Expressions.constantAs(source, alias)` 
  + source는 조회 결과에 사용될 변수값, alias는 **Entity의 필드와 DTO의 필드가 똑같은 이름, 똑같은 타입인 경우 alias로 사용할 수 있다.**
    Entity에서 선언된 필드와 동일한 이름/타입이면 컴파일 체크가 가능한 QClass 의 필드를 사용할 수 있다.



+ `Expressions.as(source, alias)`
  + source는 조회 결과에 사용될 변수값, alias는 **Entity의 필드에는 없는 값이지만 DTO에는 필요한 필드다.**



### Result handling

Querydsl은 결과를 사용자 지정하는 두 가지 방법, 즉 행 기반 변환을 위한 `FactoryExpressions`와 집계를 위한 `ResultTransformer`를 제공한다.

`FactoryExpression` 인터페이스는 **빈 생성, 생성자 호출 및 보다 복잡한 객체 생성에 사용된다.**

Querydsl의 `FactoryExpression` 구현의 기능은 `com.querydsl.core.types.Projections` 클래스를 통해 액세스할 수 있다.

`com.querydsl.core.ResultTransformer` 인터페이스의 경우 GroupBy가 기본 구현이다.



#### Returning multiple columns

Querydsl 3.0부터 다중 열 결과의 기본 유형은 `com.querydsl.core.Tuple`이다.

Tuple은 Tuple 행 객체에서 열 데이터에 액세스할 수 있는 타입 세이프 Map과 같은 인터페이스를 제공한다.

```java
List<Tuple> result = query.select(employee.firstName, employee.lastName)
                          .from(employee).fetch();
for (Tuple row : result) {
     System.out.println("firstName " + row.get(employee.firstName));
     System.out.println("lastName " + row.get(employee.lastName));
}}

// QTuple 표현 클래스를 사용해 아래와 같이 사용할 수도 있다.
List<Tuple> result = query.select(new QTuple(employee.firstName, employee.lastName))
                          .from(employee).fetch();
for (Tuple row : result) {
     System.out.println("firstName " + row.get(employee.firstName));
     System.out.println("lastName " + row.get(employee.lastName));
}}
```



#### Bean population

쿼리 결과에 따라 Beans를 채워야 하는 경우 Bean 생성을 사용한다.

```java
List<UserDTO> dtos = query.select(
    Projections.bean(UserDTO.class, user.firstName, user.lastName)).fetch();

// 필드를 setter 대신 직접 사용해야 하는 경우 다음 변형을 대신 사용할 수 있다.
List<UserDTO> dtos = query.select(
    Projections.fields(UserDTO.class, user.firstName, user.lastName)).fetch();
```



#### Constructor usage

생성자 기반 행 변환은 다음과 같다.

```java
List<UserDTO> dtos = query.select(
    Projections.constructor(UserDTO.class, user.firstName, user.lastName)).fetch();
```



일반 생성자 표현식 대신에 생성자 사용에 @QueryProjection 어노테이션을 사용할 수도 있다.

```java
class CustomerDTO {

  @QueryProjection
  public CustomerDTO(long id, String name) {
     ...
  }

}

QCustomer customer = QCustomer.customer;
JPQLQuery query = new HibernateQuery(session);
List<CustomerDTO> dtos = query.select(new QCustomerDTO(customer.id, customer.name))
                              .from(customer).fetch();
```



**QueryProjection 어노테이션이 있는데 엔티티가 아닌 경우** 위의 예제에서와 같이 생성자 투영을 사용할 수 있지만,

**QueryProjection 어노테이션이 있고 엔티티인 경우**쿼리 유형의 **정적 생성 메서드 호출( QModel.create or Projections.constructor)**을 통해 **생성자 투영을 생성해야 한다.**

```java
@Entity
class Customer {

  @QueryProjection
  public Customer(long id, String name) {
     ...
  }

}

QCustomer customer = QCustomer.customer;
JPQLQuery query = new HibernateQuery(session);
List<Customer> dtos = query.select(QCustomer.create(customer.id, customer.name))
                           .from(customer).fetch();

// 코드 생성이 옵션이 아닌 경우 다음과 같이 생성자 투영을 생성할 수 있다.
List<Customer> dtos = query
    .select(Projections.constructor(Customer.class, customer.id, customer.name))
    .from(customer).fetch();
```



#### Result aggregation

`com.querydsl.core.group.GroupBy` 클래스는 **쿼리 결과를 메모리에서 집계하는데 사용할 수 있는 집계 함수**를 제공한다.

*메모리에서 집계하기에 **groupBy 쿼리가 나가는 것이 아니다.***

```java
Map<Integer, List<Comment>> results = query.from(post, comment)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(list(comment)));

// return columns
Map<Integer, Group> results = query.from(post, comment)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(post.name, set(comment.id)));
```



그렇다면 **groupBy 쿼리가 나가게 하려면 어떻게 해야할까?**

```java
// name을 기준으로 groupBy
query.select(QSample.sample.name, QSample.sample.count())
    .from(QSample.sample)
    .groupBy(QSample.sample.name)
    .fetch();

// select
//     sample0_.name as col_0_0_,
// from
//     sample sample0_ 
// group by
//     sample0_.name
```

**그리고 groupBy 쿼리가 나가기에 위의 경우에만 having 쿼리를 사용할 수 있다.**

```java
query.select(QSample.sample.name, QSample.sample.count())
    .from(QSample.sample)
    .groupBy(QSample.sample.name)
  	.having(QSample.sample.name.contains("name"))
    .fetch();

// select
//     sample0_.name as col_0_0_,
// from
//     sample sample0_ 
// group by
//     sample0_.name
// having
//     sample0_.name like ? escape '!'
```

