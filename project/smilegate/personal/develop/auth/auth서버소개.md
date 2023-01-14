## auth 서버 소개

auth 서버는 아래의 서비스를 제공합니다.

| 서비스           | HTTP | URL(/members)  | 기능                                                         |
| ---------------- | ---- | -------------- | ------------------------------------------------------------ |
| 회원정보 제공    | GET  | /              | Authorization 헤더를 통해 받은 토큰을 가지고 회원정보를 제공한다. |
| 회원가입         | POST | /join          | Email, Password, Name을 기반으로 회원가입 기능을 제공한다.   |
| 로그인           | POST | /login         | Email, Password를 기반으로 로그인 기능을 제공하며 Authorization 헤더에 access token을, 쿠키로 refresh token을 제공한다. |
| 로그아웃         | POST | /logout        | 로그 아웃 기능을 제공하며 쿠키의 refresh token을 삭제해준다. |
| 토큰 갱신        | GET  | /token/renewal | 쿠기의 refresh token을 통해 access token을 생성하며 그 값을 반환해준다. |
| 롤 갱신          | PUT  | /role          | 갱신된 롤을 적용한 token을 Authorization 헤더와 쿠키를 통해 반환해준다. |
| 이메일 인증 전송 | POST | /email         | 롤 갱신을 위한 email 발송 기능을 제공한다.<br /> 이때 추후 인증 검증에 필요한 uuid를 제공하며 key를 생산한다. |
| 이메일 인증 검증 | POST | /email/key     | 롤 갱신을 위한 email 인증 검증 기능을 제공한다.<br /> 이때  uuid와 key를 기반으로 인증을 진행한다. |



### 로그인

```java
@Override
@Transactional
public MemberInfoDTO login(MemberLoginDTO memberLoginDTO) {
  String email = memberLoginDTO.getEmail();
  String password = memberLoginDTO.getPassword();
  Member member = findMemberBy(email);

  validatePassword(password, member);

  log.info("LOGIN [{}][{}]", member.getId(), LocalDateTime.now());
  return new MemberInfoDTO(member.getId(), member.getEmail(), member.getName(), member.getRole());
}
```

위는 로그인을 수행하는 코드이다.

코드에서 `log.info("LOGIN [{}][{}]", member.getId(), LocalDateTime.now());` 를 볼 수 있는데 이는 로그인 로그를 저장하는 코드이다.

log4j를 다음과 같이 설정하고 

```xml
<appender name="API_ACCESS_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
  <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>${home}/api-access-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <maxFileSize>15mb</maxFileSize>
    <maxHistory>30</maxHistory>
  </rollingPolicy>
  <filter class="ch.qos.logback.classic.filter.LevelFilter">
    <level>INFO</level>
    <onMatch>ACCEPT</onMatch>
    <onMismatch>DENY</onMismatch>
  </filter>
  <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
    <pattern>${FILE_LOG_PATTERN}</pattern>
  </encoder>
</appender>

<logger name="API_ACCESS_APPENDER" additivity="false">
  <level value="INFO"/>
  <appender-ref ref="API_ACCESS_APPENDER"/>
</logger>
```

`@Slf4j(topic = "API_ACCESS_APPENDER")` 와 같이 클래스에 설정하여 로그인 정보를 log 파일 형태로 저장하였다.



위와 같이 구현한 것이 본인에게 의미가 있는 것 이유는 지금까지 로그를 콘솔창을 통해 나타내는 것은 해보았지만 위와 같이 로그 파일로 만드는 것은 해보지 않았기 때문이다.

그래서 지금까지는 로그라는 선택지가 없어 DB에 들어가지 않아도 될 정보까지는 넣는 경향이 있었는데 앞으로는 그러지 않을 수 있을 것 같다.



### 이메일 인증 과정

```java

@Override
public String emailAuth(MemberAuthInfoDTO memberAuthInfoDTO) {

  String uuid = createKey();
  String key = createKey();
  memberAuthInfoDTO.setUuid(uuid);
  memberAuthInfoDTO.setKey(key);
  ListenableFuture<SendResult<String, MemberAuthInfoDTO>> emailAuth = memberAuthInfoDTOKafkaTemplate.send("emailAuth", memberAuthInfoDTO);

  emailAuth.addCallback(new ListenableFutureCallback<>() {
    @Override
    public void onSuccess(SendResult<String, MemberAuthInfoDTO> result) {
      log.debug("Kafka Sent message=[{}] with offset=[{}]", memberAuthInfoDTO, result.getRecordMetadata().offset());
      authMemberInfoRepository.save(new MemberAuthInfo(uuid, key));
    }

    @Override
    public void onFailure(Throwable ex) {
      log.debug("Kafka Unable to send message=[{}] due to : [{}]", memberAuthInfoDTO, ex.getMessage());
    }
  });

  return uuid;
}

private String createKey() {
	...
}

@Override
public boolean validateAuthKey(AuthKeyInfoDTO authKeyInfoDTO) {
  Optional<MemberAuthInfo> authMemberInfoById = authMemberInfoRepository.findById(authKeyInfoDTO.getUuid());
  if (authMemberInfoById.isEmpty()) {
    throw new IllegalStateException("no email auth record");
  }
  MemberAuthInfo authMemberInfo = authMemberInfoById.get();
  return authMemberInfo.getKey()
    .equals(authKeyInfoDTO.getAuthKey());
}

```

우선 이메일 인증에 관한 코드는 위와 같고 조금 더 그 과정을 설명하자면 다음과 같다.

우선 uuid와 key를 만든다.

redis에는 email 인증을 위해 uuid와 key가 저장되는데 이는 추후 key를 검증할 때 uuid를 통해 그 값을 찾는다.

이메일 검증하는데 redis를 사용한 이유는 email 인증을 위한 key-value를 오래 가지고 있을 필요도 저장할 필요도 없다고 생각했다.

그래서 이와 가장 알맞는 DB를 고민하던 중 redis가 떠올랐고 빠르게 접근 가능하며 일정 시간이 지나면 삭제도되는 redis에 이메일 인증을 위한 key-value를 저장하는 것이 좋겠다는 판단을 하여 redis를 사용하게 되었다.



그래서 위와 같이 redis에 uuid와 key를 저장하고 난 이후 kafka에 emailAuth 메시지를 발행하고 key 값은 반환해준다.

이러한 방식으로 구현한 이유는 지금은 email을 통해 인증을 진행하지만 추후 sms를 통해 진행할 수도 github와 같은 방식으로 인증을 진행할 수도 있다.

이러한 확장성을 생각하면 지금은 서비스가 하나지만 notifiation이라는 서버를 만들어 그 하위 기능으로 email을 구현하는 것이 좋겠다는 생각이 들었다.

그렇다면 이제는 rest api로 호출할 것인가 아님 메시지를 통해 처리 할 것인가 문제가 남았다.

이것은 email 서비스에게 반환받을 값이 없기에 구지 rest api를 사용하여 동기적으로 수행할 필요가 없다고 생각하였고 kafka를 통해 비동기적으로 처리하였다.



### 코드리뷰 질문과 반영

---

#### Question 4

이는 jwt 토큰의 경우 payload가 공개된 정보이기 때문에 최대한 적은 정보를 담고 싶어 위와 같이 코드를 구성하였습니다.

jwt 토큰에 어느 정도의 정보가 들어가도 괜찮은지 궁금합니다.

---

위 질문은 jwt 토큰에 들어갈 수 있는 정보가 어떤 범위까지 가능한지 궁금하여 한 질문이다.

대답은 "정해진 것은 없다." 였다.

당연히 password와 같은 중요한 정보는 들어가면 안되지만 그외의 정보는 각 서비스의 정책에 따라 달라지는 것 같다.

개인적으로 기준을 정해보면 누구인지 특정되지 않는 정보 정도는 넣으면 되지 않을까? 하는 생각이 든다.