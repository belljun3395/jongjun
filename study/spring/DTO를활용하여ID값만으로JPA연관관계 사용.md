## DTO를 활용하여 ID값 만으로 JPA 연관관계 사용하기



우선 예제에 활용할 엔티티는 아래와 같다.

```java
@Data
@Entity
@NoArgsConstructor
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    
  	public Member(Long id) {
      this.id = id;
  	}
    
  	public Member(String name) {
      this.name = name;
  	}
}
```

```java
@Data
@Entity
@NoArgsConstructor
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(columnDefinition = "TEXT")
    private String text;

    @ManyToOne
    @JoinColumn(name = "member_fk")
    private Member member;
	 
  	public Post(String text, Member member) {
        this.text = text;
        this.member = member;
    }
}
```



### 객체를 활용한 JPA 연관관계 사용

```java
@Service
@RequiredArgsConstructor
public class PostService {

    private final PostRepository postRepository;
    private final MemberRepository memberRepository;

    @Transactional
    public void save(Long memberId) throws NotFoundException {
        Member member = memberRepository.findById(memberId).orElseThrow(NotFoundException::new);
        Post post = new Post("lalalalalala", member);
        postRepository.save(post);
    }
}
```

위의 코드는 Spring을 처음하는 사람에게 가장 익숙할 수 있는 JPA 연관관계를 사용하는 코드이다.

위의 코드를 사용하며 아래와 같은 sql 쿼리가 생성된다.

```sql
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        member member0_ 
    where
        member0_.id=?
Hibernate: 
    insert 
    into
        post
        (member_fk, name) 
    values
        (?, ?)
```

내가 필요한 것은 member의 id 값인데 member의 name까지 함께 조회하는 것을 확인할 수 있다.



### DTO를 활용하여 ID값 만으로 JPA 연관관계 사용

DTO는 JPA를 사용하면서 내가 원하는 칼럼만 조회해 올 수 있도록 도와준다.

우선 내가 원하는 정보가 담긴 DTO를 아래와 같이 만들자.

```java
@Getter
@Setter
public class MemberIdDTO {

    private Long id;

    public MemberIdDTO(Long id) {
        this.id = id;
    }
}
```



```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    String MEMBER_DTO_PACKAGE_LOCATION = "org.example.domain.";

    @Query(value = "select new "
        + MEMBER_DTO_PACKAGE_LOCATION
        + "MemberIdDTO(m.id) from Member m where m.id = :id")
    Optional<MemberIdDTO> findByIdMemberDTO(@Param("id") Long id);
}
```

그리고 이렇게 만든 DTO를 위와 같이 @Query에서 활용하여 메서드를 만들어 준다.



```java
@Service
@RequiredArgsConstructor
public class PostService {

    private final PostRepository postRepository;
    private final MemberRepository memberRepository;

    @Transactional
    public void save(Long memberId) throws NotFoundException {
        MemberIdDTO memberIdDTO = 
          memberRepository.findByIdMemberDTO(memberId).orElseThrow(NotFoundException::new);

        Member member = new Member(memberIdDTO.getId());
        Post post = new Post("lalalalalala", member);
      
        postRepository.save(post);
    }
}
```

```sql
Hibernate: 
    select
        member0_.id as col_0_0_ 
    from
        member member0_ 
    where
        member0_.id=?
Hibernate: 
    insert 
    into
        post
        (member_fk, name) 
    values
        (?, ?)
```

위에서 DTO를 활용하여 만든 메서드 덕분에 sql 쿼리는 member의 id만 조회하는 쿼리로 원하는 쿼리가 생성되었다.

하지만 위의 Service 코드 중 `new Member(memberIdDTO.getId())` 이 부분이 어딘가 어색해 보인다.

뭔가 id만을 가지고 새로운 Member을 생성하여 나머지 정보는 null로 Member의 정보가 갱신될 것 같다는 생각을 가지게 된다.



그렇다면 테스트를 통해 직접 확인해보자.

```java
@SpringBootTest
class PostServiceTest {

    @Autowired
    PostService postService;

    @Autowired
    MemberRepository memberRepository;
  
 		static Long MEMBER_ID;

    @BeforeEach
    void setUp() {
        Member member = new Member("jun");
        memberRepository.save(member);
        MEMBER_ID = member.getId();
    }
  
    @Test
    void save() throws NotFoundException {
        postService.save(MEMBER_ID);
    }
}
```

<img width="497" alt="스크린샷 2023-04-25 오후 7 08 36" src="https://user-images.githubusercontent.com/102807742/234245324-3c92deeb-0558-4c5f-bb8a-fd688dbec817.png">

위의 사진은 테스트 이후에 member테이블을 조회한 결과다.

null로 정보가 갱신되지 않고 그대로 정보가 유지되어 있는 것을 확인할 수 있다.



이렇게 DTO를 활용하여 ID 값만으로 JPA 연관관계를 활용해 보았는데 @Query를 매번 작성한다고 생각하면 상당히 불편할 것이다.

이는 Querydsl을 활용하면 간편히 해결 할 수 있다.

`findByIdMemberDTO`에 해당하는 Querydsl 코드만 확인해 보면 아래와 같다.

```java
  @Transactional
  public Optional<Long> getMemberId(Long id) {
      Long memberId = jpaQueryFactory
          .select(QMember.member.id)
          .from(QMember.member)
          .where(QMember.member.id.eq(id))
          .fetchOne();

      return Optional.ofNullable(memberId);
  }
```