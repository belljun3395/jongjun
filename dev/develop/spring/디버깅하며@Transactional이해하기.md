# 디버깅하며 @Transactional 이해하기



## 시작하며

`@Transactional`에 대해서 다시 한번 공부하며 자료를 찾아보다 아래 링크에 방문하게 되었습니다.

https://sup2is.github.io/2021/03/04/java-exceptions-and-spring-transactional.html

글의 내용도 너무 좋았지만, 글에 달린 댓글이 저의 눈을 사로잡았습니다.

```text
깔끔하게 정리된 포스팅 잘 봤습니다.
@Transactional이 TransactionAspectSupport 을 호출한다는 것은 어떻게 알 수 있을까요?
이렇게 멋지게 찾아가는 방법을 알려주시면 감사하겠습니다!
```

위의 댓글을 보며 저도 제가 무언가를 공부할 때 어떻게 공부하는지 다시 한번 점검해 보았고 `@Transactional`을 알아보며 제가 공부하는 방법을 공유해보려 합니다.



## 환경 준비

### 로깅

저는 강의나 책을 통해 키워드를 획득하고 난 이후 검색을 통해 확인할 수 있는 작은 단위의 예제를 직접 만들어보거나 테스트 코드를 작성하고 디버깅을 하며 조금 더 깊이 이해하려 노력합니다.

우선 디버깅을 하기 위해 아래와 같이 로그 레벨을 설정합니다.

```yml
logging:
  level:
    root: TRACE
```



상당히 많은 양의 로그를 확인할 수 있을 것인데 이때 아래의 로그 이전과 이후를 구분해서 확인하면 좋습니다.

```
Started XXX in XXX seconds (JVM running for 2.539)
```

- 로그 이전: 스프링 부트를 실행하기 위해 미리 설정해야 하는 것에 대한 로그
- 로그 이후: 내가 확인하길 원하는 동작에 대한 로그



### 테스트 코드

```java
@Slf4j
@SpringBootTest
public class TransactionalTest {

	@Autowired
	SaveEntityService saveEntityService;

	@Test
	public void execute() throws Exception {
		log.info(">>>>>>>>>>>> Start transactionalTest");
		saveEntityService.save();
		log.info("<<<<<<<<<<<< End transactionalTest");
	}
}
```

```java
@Slf4j
@TestComponent
@RequiredArgsConstructor
public class SaveEntityService {

    @Autowired TestEntityRepository testEntityRepository;

    @Transactional
    public void save() {
       log.info(">>>>>>>>>>>> Start SaveTransaction.save()");
       TestEntity entity = TestEntity.builder().name("test").build();
       testEntityRepository.save(entity);
       log.info("<<<<<<<<<<<< End SaveTransaction.save()");
    }
}
```

`@DataJpaTest`를 통해 테스트 코드를 작성하지 않고 `@SpringBootTest`를 활용하였습니다.

이는 `@DataJpaTest`이 `@Transactional`을 포함하고 있어 테스트 코드 메서드에 대한 `@Transactional` 여부를 제어하기 위한 선택이었습니다.

프록시 형태로 `@Transactional`가 동작하기 때문에 내부 메서드에서 이를 활용했을 때 발생할 수 있는 문제를 사전에 방지하기 위해 `SaveEntityService`와 같이 별도의 클래스로 분리하였습니다.



## 디버깅

`root`의 로깅 레벨이 `DEBUG`인 상태에서 테스트 코드를 실행하면 상당히 많은 양의 로그를 확인할 수 있습니다.

해당 로그 중 `@Transactional`에 관련이 있어 보이는 로그를 나타내는 클래스는 아래와 같습니다.

- `JpaTransactionManager` / `package org.springframework.orm.jpa`
- `TransactionImpl` / `package org.hibernate.engine.transaction.spi`
- `SQL`



이제 해야 할 것은 로그 레벨을 조정하여 위 클래스의 로그를 더 쉽게 살필 수 있도록 하는 것입니다.

위의 클래스가 속한 패키지를 보고 아래와 같이 로그 레벨을 구체적으로 설정해 주었습니다.

```yml
logging:
  level:
    org:
      springframework:
        orm: TRACE
        transaction: TRACE
      hibernate: TRACE
    sql: TRACE
```



```log
2024-03-03 00:49:54.663  INFO 64109 --- [    Test worker] TransactionalTest                        : >>>>>>>>>>>> Start transactionalTest
2024-03-03 00:49:59.574 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Creating new transaction with name [SaveEntityService.save]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
...
2024-03-03 00:50:00.661 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Opened new EntityManager [SessionImpl(1862912920PersistenceContext[entityKeys=[], collectionKeys=[]];ActionQueue[insertions=ExecutableList{size=0} updates=ExecutableList{size=0} deletions=ExecutableList{size=0} orphanRemovals=ExecutableList{size=0} collectionCreations=ExecutableList{size=0} collectionRemovals=ExecutableList{size=0} collectionUpdates=ExecutableList{size=0} collectionQueuedOps=ExecutableList{size=0} unresolvedInsertDependencies=null])] for JPA transaction
...
2024-03-03 00:50:00.747 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@24455d50]
...
2024-03-03 00:50:06.927  INFO 64109 --- [    Test worker] SaveEntityService                        : >>>>>>>>>>>> Start SaveTransaction.save()
2024-03-03 00:50:12.339 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Found thread-bound EntityManager [SessionImpl(1862912920PersistenceContext[entityKeys=[], collectionKeys=[]];ActionQueue[insertions=ExecutableList{size=0} updates=ExecutableList{size=0} deletions=ExecutableList{size=0} orphanRemovals=ExecutableList{size=0} collectionCreations=ExecutableList{size=0} collectionRemovals=ExecutableList{size=0} collectionUpdates=ExecutableList{size=0} collectionQueuedOps=ExecutableList{size=0} unresolvedInsertDependencies=null])] for JPA transaction
2024-03-03 00:50:14.989 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Participating in existing transaction
...
2024-03-03 01:08:24.463 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Initiating transaction commit
2024-03-03 01:09:23.442 DEBUG 64109 --- [    Test worker] o.s.orm.jpa.JpaTransactionManager        : Committing JPA transaction on EntityManager [SessionImpl(1485974016PersistenceContext[entityKeys=[EntityKey[kim.jongjun.TestEntity#75]], collectionKeys=[]];ActionQueue[insertions=ExecutableList{size=0} updates=ExecutableList{size=0} deletions=ExecutableList{size=0} orphanRemovals=ExecutableList{size=0} collectionCreations=ExecutableList{size=0} collectionRemovals=ExecutableList{size=0} collectionUpdates=ExecutableList{size=0} collectionQueuedOps=ExecutableList{size=0} unresolvedInsertDependencies=null])]
```

위는 로그 레벨을 조정하고 난 이후 확인할 수 있는 로그 중 `JpaTransactionManager`와 관련된 로그입니다.

위의 로그에서 아래와 진행 사항을 나타내는 내용을 전체 검색하여 브레이크 포인트를 설정하여 `@Transactional`이 어떻게 동작하는지 알아봅시다.

- `Creating new transaction with name` 
- `Opened new EntityManager` 
- `Exposing JPA transaction as JDBC`
- `Found thread-bound EntityManager`
- `Participating in existing transaction`
- `Initiating transaction commit`
- `Committing JPA transaction on EntityManager`



### Creating new transaction with name

**브레이크 포인트 위치:** `AbstractPlatformTransactionManager#getTransaction` 

**브레이크 포인트에서 확인한 콜스텍:**

![스크린샷 2024-03-03 오전 1 25 40](https://github.com/belljun3395/jongjun/assets/102807742/7c19338a-c50f-438c-8a4d-5961e02a3558)

**콜스텍을 보며:**

`SaveEntityService#save`가 프록시로 실행될 것임을 확인할 수 있습니다.

`TransactionInterceptor#invoke`를 통해 트렌젝션이 포함된 메서드가 최초로 `invoke`됨을 확인할 수 있습니다.

`TransactionAspectSupport#invokeWithinTransaction`가 실행되며 `TransactionAspectSupport#createTransactionIfNecessary`를 통해 필요시 트렌젝션이 생성됨을 확인할 수 있습니다.

이는  `JpaTransactionManager#doGetTransaction`을 통해 수행됩니다.

이를 위해 처음 생성되는 트렌젝션 객체는 `nested transaction`이 허용된  `JpaTransactionObject()` 객체이고 해당 객체를 `AbstractPlatformTransactionManager#startTransaction`에 넘겨 트렌젝션을 시작합니다.



### Opened new EntityManager

**브레이크 포인트 위치:** `JpaTransactionManager#doBegin`

**브레이크 포인트에서 확인한 콜스텍:**

![스크린샷 2024-03-03 오전 1 38 04](https://github.com/belljun3395/jongjun/assets/102807742/c12d4539-fdbb-4589-90c5-43d38f75b90e)



### Exposing JPA transaction as JDBC

**브레이크 포인트 위치:** `JpaTransactionManager#doBegin`

**브레이크 포인트에서 확인한 콜스텍:**

![스크린샷 2024-03-03 오전 2 14 30](https://github.com/belljun3395/jongjun/assets/102807742/bc489e74-55b2-448c-a922-10bb1c48f487)

**콜스텍을 보며:**

`AbstractPlatformTransactionManager#startTransaction`에서 넘겨받은 트렌젝션 객체를 넘기며 `JpaTransactionManager#doBegin`에서 트렌젝션 객체가 트렌젝션을 수행할 수 있도록 필요한 설정을 합니다.

이때 필요한 설정 중 대표적인 것이 엔티티 매니저와 데이터 소스라고 할 수 있습니다.

추가로 해당 과정에서 `TransactionSynchronizationManager`의 `resources`에 커넥션 풀과 엔티티 매니저가 바인딩됩니다.



### Found thread-bound EntityManager

**브레이크 포인트 위치:** `JpaTransactionManager#doGetTransaction`

**브레이크 포인트에서 확인한 콜스텍:**

![스크린샷 2024-03-03 오전 2 31 33](https://github.com/belljun3395/jongjun/assets/102807742/ded4bde4-dd69-4bf7-8348-cd44dc0f0f0d)

**콜스텍을 보며:**

`SaveEntityService#save`가 프록시로 실행됨을 확인할 수 있습니다.

`JpaTransactionManager#doGetTransaction`를 통해 생성되는 객체가 이전과 달리 이미 존재하는 객체임을 확인할 수 있었습니다.

이는 `JpaTransactionManager#doBegin`에서 트랜잭션 객체가 만들어지며 `TransactionSynchronizationManager`에 바인딩된 `resources`를 활용하여 트랜잭션 객체를 만들기 때문입니다.



### Participating in existing transaction

**브레이크 포인트 위치:** `AbstractPlatformTransactionManager#handleExistingTransaction`

**브레이크 포인트에서 확인한 콜스텍:**

<img width="651" alt="스크린샷 2024-03-03 오후 1 51 42" src="https://github.com/belljun3395/jongjun/assets/102807742/45425887-4ae4-4b42-ac24-4e9cb93c6c2a">

**콜스텍을 보며:**

앞서 이미 생성된 트랜잭션 객체임을 확인하였기에 `AbstractPlatformTransactionManager#startTransation` 이 아닌 `AbstractPlatformTransactionManager#handleExistingTransaction`가 실행됨을 확인할 수 있습니다.



### Initiating transaction commit

**브레이크 포인트 위치:** `AbstractPlatformTransactionManager#processCommit`

**브레이크 포인트에서 확인한 콜스텍:**

<img width="643" alt="스크린샷 2024-03-03 오후 1 57 51" src="https://github.com/belljun3395/jongjun/assets/102807742/4705b4ee-92f1-4fe5-b883-57c48deba855">

**콜스텍을 보며:**

```java
// TransactionAspectSupport의 382 ~ 397에 해당하는 코드
TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

Object retVal;
try {
  // This is an around advice: Invoke the next interceptor in the chain.
  // This will normally result in a target object being invoked.
  retVal = invocation.proceedWithInvocation();
}
catch (Throwable ex) {
  // target invocation exception
  completeTransactionAfterThrowing(txInfo, ex);
  throw ex;
}
finally {
  cleanupTransactionInfo(txInfo);
}
```

트랜잭션 내에서 수행해야 하는 동작을 무사히 수행했다는 것은 위의 코드를 문제없이 지나왔다는 것입니다.

이를 DB에 반영하기 위한 커밋을 하는 작업을 `AbstractPlatformTransactionManager#processCommit`을 통해 수행함을 확인할 수 있습니다.



```java
// AbstractPlatformTransactionManager의 738 ~ 744에 해당하는 코드
else if (status.isNewTransaction()) {
  if (status.isDebug()) {
    logger.debug("Initiating transaction commit");
  }
  unexpectedRollback = status.isGlobalRollbackOnly();
  doCommit(status);
}
```

이때 위의 분기 조건에 주목할 필요가 있는데 새롭게 생성된 트랜잭션을 기준으로 커밋을 시도함을 확인할 수 있습니다.



### Committing JPA transaction on EntityManager

**브레이크 포인트 위치:** `JpaTransactionManager#doCommit`

**브레이크 포인트에서 확인한 콜스텍:**

<img width="632" alt="스크린샷 2024-03-03 오후 2 06 38" src="https://github.com/belljun3395/jongjun/assets/102807742/873c2c46-1016-45c3-81ce-d4c7d1d4df7f">

**콜스텍을 보며:**

`JpaTransactionManager#doCommit`에서 실질적인 커밋이 수행됨을 확인할 수 있습니다.



## 배운 것

`JpaTransactionManager` 그리고 `AbstractPlatformTransactionManager`를 위주로 디버깅하며 메서드에 트랜잭션이 어떻게 적용되는지 확인할 수 있었습니다.

`TransactionSynchronizationManager`를 통해 트랜잭션 자원을 어떻게 관리하고 있는지 확인할 수 있었습니다.



## 마치며

공부하다 보면 모호한 상태로 머리에 남아있는 무언가가 많은 것 같습니다.

디버깅은 그런 무언가를 조금은 선명하게 만들어줄 수 있는 좋은 방법의 하나인 것 같습니다.

제가 적은 글이 디버깅하려는 누군가에게 조금의 도움이 되었으면 좋겠습니다.

감사합니다.



---

**참고한 링크**

- [Java의 Exception 그리고 Spring의 @Transactional](https://sup2is.github.io/2021/03/04/java-exceptions-and-spring-transactional.html)
- [Spring - JpaTransactionManager의 동작 원리](https://obv-cloud.com/40)
