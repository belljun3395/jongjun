# Repository 계층을 설계하며 고민한 것들

<br>

[관련 코드 바로가기](https://github.com/JNU-econovation/hiit-server/tree/foo/repository/src/main/java/com/hiit/api/repository)



<br>



"그냥 JPA가 해주는 거 잘 사용하면 되는 거 아니야?!"라고 할 수 있지만..ㅎㅎㅎ

이번 프로젝트에서 Repository 계층을 설계하며 고민한 것들을 나누어 보려 합니다.

<br>



## BaseEntity

사실 저는 BaseEntity에 대한 부정적인 시선을 가지고 있었습니다.

자바에서 상속은 하나만 가능하기에 최대한 상속을 사용하지 않는 것을 선호하였습니다.

하지만 이번에는 정말 엔티티에는 엔티티에 필요한 정보만 포함하고 싶어 BaseEntity를 도입하였습니다.

<br>



제가 이번에 BaseEntity에 포함한 정보는 아래와 같습니다.

+ id
+ createAt
+ updateAt
+ deleted

id의 경우 엔티티를 구성하기 위한 필수 요소이기에 포함하였습니다.

그리고 createAt, updateAt, deleted의 경우는 엔티티의 메타 정보로 수정 및 삭제를 파악하기 위해 사용하려 포함하였습니다.

<br>



엔티티의 수정과 삭제 과정을 살펴보며 메타 정보를 어떻게 사용할지 알아봅시다.

우선 엔티티를 생성하면 id가 배정되고 createAt과 updateAt에 동일한 값이 기록될 것입니다.

<br>



그럼 수정되면 어떤 일이 일어날까요?

수정되기 전의 값을 로그 엔티티에 기록하고 값을 수정하고 updateAt을 수정 시간으로 변경할 것입니다.

그냥 값만 수정하면 되는 거지 왜 로그까지 남겨야 할까요?

제가 생각하는 이유는 아래와 같습니다.

+ 엔티티로 기록할 정도의 기록이라면 비즈니스적 가치가 있다고 생각합니다.
  로그로 남기지 않으면 이러한 정보는 사라지게 되는 데 이는 비즈니스적으로도 손해라 생각합니다.
+ 기능을 확장할 수 있습니다.
 로그를 남기지 않고 엔티티에 바로 변경 사항을 적용하면 "이전 기록으로 되돌리기"와 같은 기능으로 확장을 스스로 포기하는 것입니다.

<br>



이제 삭제되는 과정도 알아볼까요?

삭제될 때는 물리적으로 삭제되지는 않습니다.

다만 논리적으로 삭제됩니다.

논리적으로 삭제될 수 있는 이유는 deleted라는 칼럼 때문입니다.

deleted 칼럼의 경우 boolean 값을 가지고 0은 삭제되지 않음을 1은 삭제됨을 뜻합니다.

이렇게 논리적으로 삭제하는 것을 소프트 삭제(soft delete)라고 하며 이를 통해 우리는 수정 때와 비슷한 장점이 있게 됩니다.

+ 물리적으로 삭제하지 않음으로써 비즈니스적 가치가 있는 기록을 남길 수 있습니다.
+ 삭제 복구와 같은 기능을 확장 기능을 포기하지 않을 수 있습니다.

<br>



그럼 이러한 BaseEntity를 어떻게 구현하였는지 살펴봅시다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
@ToString
@MappedSuperclass
@EntityListeners({AuditingEntityListener.class, SoftDeleteListener.class})
@SuperBuilder(toBuilder = true)
public class BaseEntity {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(nullable = false, updatable = false)
	@CreatedDate
	private LocalDateTime createAt;

	@Column(nullable = false)
	@LastModifiedDate
	private LocalDateTime updateAt;

	@Builder.Default
	@Column(nullable = false)
	private Boolean deleted = false;

	public void delete() {
		this.deleted = true;
	}
}
```

나머지 필드는 단번에 이해 가능할꺼라 생각합니다.

그런데 delete 메서드는 왜 있을까요? SoftDeleteListener는 무엇일까요?

delete 메서드는 객체에도 삭제되었다는 것을 반영하기 위해 존재합니다.

그럼 `repository.delete(entity)` 이후에  entity의 delete의 상태를 확인해보면 어떨까요?

SoftDeleteListener가 없다면 기본값인 false일 것입니다.

추후 확인할 @SQLDelete는 삭제 쿼리를 DB 단에서 반영 시켜줄 뿐 객체에는 아무런 변화를 주지 못하기 때문입니다.

<br>



이 문제를 해결해주는 것이  SoftDeleteListener입니다.

```java
public class SoftDeleteListener {

	@PreRemove
	private void preRemove(BaseEntity entity) {
		entity.delete();
	}
}
```

SoftDeleteListener 구현을 보면 @PreRemove라는 어노테이션이 존재합니다.

이 어노테이션을 통해 DB 단에 삭제 쿼리가 나가기 전에 객체의 상태를 변경해 줍니다.

이를 통해 우리는 DB 단에 쿼리가 나가기 전에 delet 필드가 true로 변경된 상태의 객체를 가질 수 있습니다.

<br>



## Entity

그러면 이제 엔티티를 살펴볼까요?

Foo has many Bar 관계를 맺고 있는 Bar 엔티티를 살펴봅시다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
@SuperBuilder(toBuilder = true)
@ToString
@Entity(name = "bar")
@Table(name = BarEntity.ENTITY_PREFIX + "tb")
@SQLDelete(
		sql = "UPDATE bar_tb SET deleted=true , version = version + 1 WHERE id = ? AND version = ?")
@Where(clause = "deleted=false")
public class BarEntity extends VersionBaseEntity {

	public static final String ENTITY_PREFIX = "bar_";

	@Column(name = ENTITY_PREFIX + "name", nullable = false, length = 100)
	private String name;

	@Exclude
	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(
			name = FooEntity.ENTITY_PREFIX + "fk",
			nullable = false,
			foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
	private FooEntity foo;
}
```

우선 VersionBaseEntity는 BaseEntity에 낙관적 락을 위한 version 칼럼이 추가된 BaseEntity 입니다.

*(낙관적 락에 대해서는 이번 프로젝트에 적용할 예정이고 이는 추후 포스팅을 통해 조금 더 자세히 알아봅시다.)*

<br>



### ConstraintMode.NO_CONSTRAINT

그리고 가장 눈에 띄는 것은 `@ForeignKey(value = ConstraintMode.NO_CONSTRAINT)` 라고 생각합니다.

DB ForeignKey의 경우 다음과 같은 단점을 지니고 있다고 합니다.

+ DB 성능이 느려짐
+ 실행계획을 제어할 수 없음
+ 확장이 어려움
+ 테스트의 번거로움

위의 2가지 이유는 아직 제대로 경험할 기회가 없었지만, 확장이 어렵고 테스트의 번거로움은 이전의 프로젝트를 수행하면서도 많이 느꼈던 단점이어서 이번에는 ForeignKey 제약조건을 제외하기로 하였습니다.

<br>



JPA에서 ForeignKey 제약조건을 제외할 방법은 크게 2가지가 있습니다.

우선 JPA 연관관계가 주는 편리함을 포기하고 ForeignKey 관련 컬럼의 타입을 엔티티 관련 타입이 아닌 Long 타입으로 선언하는 것입니다.

이 방법을 사용한다면 앞서 말한 것처럼 JPA 연관관계가 주는 편리함을 포기해야 하고 기본형 타입인 Long 타입을 사용하기에 휴먼에러가 늘 수 있습니다.

Ex) 실수로 FooEntity의 id가 아닌 BazEntity의 id를 넣는 등

<br>



하지만  `@ForeignKey(value = ConstraintMode.NO_CONSTRAINT)`를 통한 제약 조건만 제외하는 방식은 위와 같은 단점을 생각할 필요가 없습니다.

JPA 연관관계 내에서 foreignKey 옵션만 바꾸는 것이기에 JPA 연관관계를 그대로 이용할 수 있습니다.

그에따라 타입도 엔티티 타입이기에 휴먼에러를 줄일 수 있습니다.

<br>



### EntitySupporter

그런데 혹시 A has many B 관계에서 자주 볼 수 있는 어떠한 메서드가 안 보일까요?

양방향 편의 메서드는 어디 있을까요?

저는 이번 프로젝트에서는 엔티티에는 정말 테이블과 대응 되는 값만 나타내려고 합니다.

그렇기에 양방향 편의 메서드와 같은 엔티티를 도와주는 메서드는 EntitySupporter 클래스를 만들어 책임을 배정해 주었습니다.

그럼 BarEntity를 도와주는 BarEntitySupporter를 살펴볼까요?

```java
@Component
public class BarEntitySupporter {

	public static BarEntity getIdEntity(Long id) {
		return BarEntity.builder().id(id).build();
	}

	public BarEntity registerFoo(BarEntity source, FooEntity foo) {
		if (Objects.nonNull(source.getFoo())) {
			source.getFoo().getBars().remove(source);
		}
		BarEntity updatedSource = source.toBuilder().foo(foo).build();
		foo.getBars().add(updatedSource);
		return updatedSource;
	}
}
```

우선 getIdEntity 메서드를 구현하여 JPA 연관관계에서 사용할 객체를 편히 만들 수 있는 메서드를 만들어주었습니다.

그리고 registerFoo 메서드를 통해 기존 BarEntity 객체에 있었던 양방향 연관관계 메서드를 책임을 대체하였습니다.

<br>



이렇게 EntitySupport 클래스를 통해 엔티티는 정말 테이블과 대응 되는 값만 가질 수 있게 되어 개인적으로 이번 프로젝트를 진행하며 한 고민의 결과 중 가장 마음에 드는 결과라 생각합니다!!!ㅎㅎㅎ

<br>



## Repository

이번 프로젝트에서 repository는 3개의 클래스로 구성하였습니다.

+ repository
+ jpaRepository
+ customRepository

<br>



우선 repository는 jpaRepository와 customRepository를 확장합니다.

이를 통해 jpa와 custom repository를 하나의 인터페이스에서 사용할 수 있게 됩니다.

<br>



jpaRepository는 `org.springframework.data.jpa.repository.JpaRepository`를 확장한 인터페이스 입니다.

JpaRepository 인터페이스 확장을 통해  [미리 정의된 키워드](https://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/#jpa.sample-app.finders.strategies)를 통해 메서드만 정의하여 간단히 쿼리를 만들 수 있습니다.

이는 편리한 기능이지만 타입 안전한 상태로 쿼리를 원하는 대로 제어할 수 없다는 단점이 있습니다.

*(@Query를 사용하여 쿼리를 작성할 수 있지만 String으로 쿼리를 작성하여야 하므로 타입 안전하지 않습니다.)*

<br>



customRepository는 JPA에서 제공하는 쿼리가 아닌 제어가 필요한 쿼리를 위한 인터페이스입니다.

이번 프로젝트에서는 customRepository에서 쿼리를 제어하기 위한 구현체로 QueryDsl을 선택했습니다.

제가 QueryDsl을 알아보고 사용하며 느낀 장점은 우선 첫 번째로 타입 안전합니다.

QueryDsl에서 사용할 Q타입을 객체를 만들어 타입 안전하게 쿼리를 작성할 수 있습니다.

두 번째로 동적 쿼리를 작성할 수 있습니다.

BooleanExpression와 같은 표현식을 통해 동적으로 쿼리를 구성할 수 있습니다.

마지막으로 가독성이 좋아집니다.

```sql
select
    fooentity0_.id
    fooentity0_.create_at
    fooentity0_.deleted
    fooentity0_.update_at
    fooentity0_.foo_name
from
    foo_tb fooentity0_ 
where
    (
        fooentity0_.deleted=0
    ) 
    and fooentity0_.foo_name=? 
    and fooentity0_.deleted=? 
order by
    fooentity0_.id desc limit ?

select
    count(fooentity0_.id)
from
    foo_tb fooentity0_ 
where
    (
        fooentity0_.deleted=0
    ) 
    and fooentity0_.foo_name=? 
    and fooentity0_.deleted=?
```

sql로 작성하게 된다면 이렇게 긴 쿼리가 QueryDsl로 작성하면 아래와 같이 간단히 작성할 수 있습니다.

```java
 from(foo)
       .where(foo.name.eq(name), foo.deleted.isFalse())
       .orderBy(foo.id.desc())
       .offset(pageable.getOffset())
       .limit(pageable.getPageSize());
```

<br>



## 마치며

Repository 계층까지의 설계를 마치며 개발하기 위한 기본 준비를 마쳤습니다.

프로젝트를 시작하기 전에 이렇게 많은 시간으로 들여 설계를 생각하고 고민한 경험은 이번이 처음입니다.

이렇게 큰 틀의 설계를 완성해 두니 애매한 부분이 생기면 돌아갈 기준이 생겨 든든하면서도 내가 잘 설계한 것이 맞을까? 하는 걱정이 들기도 합니다.

앞으로 지금까지의 설계를 바탕으로 직접 코드를 작성해 보고 피드백을 받아보면 제가 잘 설계하였는지 알 수 있겠죠...?

<br>



그치만 아직 본격적인 개발을 하기에는 남은 관문들이 많습니다.

정책도 구체화해야하고... 아직 못한 설정들도 해야하고..

해당 내용들도 많이 생각하고 고민해서 블로그에 포스팅할 계획이니 앞으로의 포스팅도 흥미롭게 지켜봐 주셨으면 좋겠습니다.

감사합니다.
