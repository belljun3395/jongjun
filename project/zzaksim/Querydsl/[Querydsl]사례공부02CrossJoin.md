## [Querydsl] 사례공부02 / Cross Join

*출처 : https://jojoldu.tistory.com/533*



![image](https://user-images.githubusercontent.com/102807742/229982473-0c572d0d-0895-432b-833d-6b4793b42c18.png)

*사진 출처 : https://www.tutorialgateway.org/sql-joins/*

위의 사진을 보면 Cross Join을 사용하면 많은 연산이 일어날 것을 예상할 수 있을 것이다.

하지만 **"Hibernate의 경우 암묵적인 조인은 Cross Join을 사용하는 경향이 있습니다."**고 한다.

그러한 경우에 Cross Join을 Inner Join과 같이 다른 적절한 Join으로 변경해 주는 것이 성능을 생각하면 좋을 것이다.



이번에도 코드를 통해 조금 더 자세히 알아보자.

```java
@Entity
@Getter
@NoArgsConstructor
public class Phone {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long number;

    @OneToOne
    @JoinColumn(name = "person_id")
    private Person person;

    public Phone(Long number) {
        this.number = number;
    }

    public void setPerson(Person person) {
        this.person = person;
    }
}

@Entity
@Getter
@NoArgsConstructor
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(mappedBy = "person")
    private Phone phone;

    public void setPhone(Phone phone) {
        this.phone = phone;
    }
}
```



위와 같이 정의된 객체들을 Join을 명시적으로 지정해주지 않고 아래와 같이 사용해보자.

```java
@Transactional
public List<Person> crossJoin() {
    return jpaQueryFactory.selectFrom(QPerson.person)
        .where(QPerson.person.id.eq(QPerson.person.phone.id))
        .fetch();
}
```

````sql
Hibernate: 
    select
        person0_.id as id1_4_ 
    from
        person person0_ cross // cross join 발생
    join
        phone phone1_ 
    where
        person0_.id=phone1_.person_id 
        and person0_.id=phone1_.id
````

쿼리를 살펴보면 Hibernate **스스로 cross join**을 수행한 것을 확인할 수 있다.

*이는 Querydsl의 특징이 아니라 Hibernate의 특징이기에 추후 JPA를 사용할 때도 주의하면 좋을 것 같다.*



그렇다면 이는 어떻게 수정할 수 있을까?

**명시적으로 Join을 지정해주면 된다.**

```java
@Transactional
public List<Person> innerJoin() {
    return jpaQueryFactory.selectFrom(QPerson.person)
        .innerJoin(QPerson.person.phone) // inner join 명시적 지정
        .where(QPerson.person.id.eq(QPerson.person.phone.id))
        .fetch();
}
```

```sql
Hibernate: 
    select
        person0_.id as id1_4_ 
    from
        person person0_ 
    inner join
        phone phone1_ 
            on person0_.id=phone1_.person_id 
    where
        person0_.id=phone1_.id
```

inner join 수행하는 쿼리문이 수행되는 것을 확인할 수 있다.