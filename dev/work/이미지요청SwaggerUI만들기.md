# 이미지가 포함된 multipart/form-data 요청 Swagger-UI 만들기



org.webjars:swagger-ui 의존성을 사용하여 Swagger-UI를 만든다면 이미지가 포함된 `multipart/form-data` 요청에 대해서는 Swagger 스펙을 작성하기 힘들다는 문제가 있습니다.

```
mockMvc
    .perform(
        post(URL)
            .contentType(MediaType.APPLICATION_JSON)
            .content(content))
    .andExpect(status().is2xxSuccessful())
        .andDo(
            document(
                "postMember",
                resource(
                        ResourceSnippetParameters.builder()
                                .description("회원 가입을 한다.")
                                .tag(TAG)
                                .requestSchema(Schema.schema("PostMemberRequest"))
                                .responseSchema(Schema.schema("PostMemberResponse"))
                                .responseFields(
                                        Description.common(
                                                new FieldDescriptor[] {
                                                    fieldWithPath("data")
                                                            .type(JsonFieldType.OBJECT)
                                                            .description("데이터"),
                                                    fieldWithPath("data.id")
                                                            .type(JsonFieldType.NUMBER)
                                                            .description("회원 ID"),
                                                    fieldWithPath("data.nickname")
                                                            .type(JsonFieldType.STRING)
                                                            .description("닉네임"),
                                                    fieldWithPath("data.profile")
                                                            .type(JsonFieldType.STRING)
                                                            .description("프로필 이미지"),
                                                    fieldWithPath("data.accessToken")
                                                            .type(JsonFieldType.STRING)
                                                            .description("액세스 토큰"),
                                                    fieldWithPath("data.refreshToken")
                                                            .type(JsonFieldType.STRING)
                                                            .description("리프레시 토큰")
                                                }))
                                .build())));
```

평소 `MockMvc`를 활용하여 위와 같은 컨트롤러 테스트 코드를 통해 Swagger-UI를 만들어왔는데 이미지가 포함된 `multipart/form-data` 요청의 경우 요청 스펙을 정의할 수 없습니다.

위의 코드에서는 `post(URL).contentType(MediaType.APPLICATION_JSON).content(content)`의 `content`를 통해 요청 스펙을 정의할 수 있지만 `multipart/form-data` 요청을 위한 `multipart(URL).file((MockMultipartFile)request.source).contentType(MediaType.MULTIPART_FORM_DATA_VALUE).content(content)`에서는 아래의 예외와 함께 요청 스펙을 정의해주지 않습니다.

```
org.springframework.restdocs.payload.PayloadHandlingException: Cannot handle multipart/form-data content as it could not be parsed as JSON or XML
```



하지만 Swagger-UI에서 `multipart/form-data`을 지원하지 않는 것은 아닙니다.

![img](https://blog.kakaocdn.net/dn/bl9smy/btsIhtrYChE/KeJRJky4QYaMTBcmKkmLf1/img.png)

[https://swagger.io/docs/specification/describing-request-body/multipart-requests/](https://swagger.io/docs/specification/describing-request-body/multipart-requests/)

공식 문서를 확인해 보면 위와 같이 지원하고 있는 것을 확인할 수 있습니다.

그렇다면 우리는 org.webjars:swagger-ui 의존성이 Swagger-UI를 만드는 과정을 파악하고 위의 필요한 `multipart/form-data` 스펙을 추가하면 됩니다.



![img](https://blog.kakaocdn.net/dn/bIFKnU/btsIhw3j0jg/EFZ8yTEcB1hBNgc4CVbjz1/img.png)

확인해본 결과 org.webjars:swagger-ui의 generateSwaggerUI 태스크는 그 결과로 위 사진과 같은 결과물을 만들어 내고 `index.html`에서는 `swagger-spec.js`에서 정의한 스펙을 Swagger-UI로 제공하고 있음을 확인할 수 있었습니다.

이에 `swagger-spec.js`에 원하는 `multipart/form-data` 스펙을 직접 추가한다면 원하는 Swagger-UI를 완성할 수 있을 것이라 생각하였습니다.



이에 저는 아래 테스크를 추가하여 `multipart/form-data` 스펙을 `swagger-spec.js`에 추가해 주었습니다.

```
tasks.register("generateApiSwaggerUI", Copy::class) {
    dependsOn("generateSwaggerUI")
    val generateSwaggerUISampleTask = tasks.named("generateSwaggerUI", GenerateSwaggerUI::class).get()
    from(generateSwaggerUISampleTask.outputDir)
    into("$projectDir/src/main/resources/static/docs/swagger-ui")
    doLast {
        // swagger-spec.js 위치 지정
        val swaggerSpecSource = "$projectDir/src/main/resources/static/docs/swagger-ui/swagger-spec.js"
        file(swaggerSpecSource).writeText(
            // 원하는 위치에 MULTIPART/FORM-DATA_SPEC을 추가
            file(swaggerSpecSource).readText().replace(
                TARGET_SPEC,
                TARGET_SPEC + MULTIPART/FORM-DATA_SPEC
            )
        )
    }
}

val MULTIPART/FORM-DATA_SPEC = "" +
    "        \"requestBody\" : {\n" +
    "            \"content\" : {\n" +
    "                \"multipart/form-data\" : {\n" +
    "                    \"schema\" : {\n" +
    "                        \"type\" : \"object\",\n" +
    "                        \"properties\" : {\n" +
    "                            \"source\" : {\n" +
    "                                \"type\" : \"string\",\n" +
    "                                \"format\" : \"binary\"\n" +
    "                            }\n" +
    "                        }\n" +
    "\n" +
    "                    }\n" +
    "                }\n" +
    "            }\n" +
    "        },"
```

그 결과 원하는 Swagger-UI를 생성하여 팀의 클라이언트 개발자에게 전달할 수 있었습니다.



완성된 API Swagger-UI: [https://api.fewletter.site/docs/swagger-ui/index.html](https://api.fewletter.site/docs/swagger-ui/index.html)

관련 구현 PR: [https://github.com/YAPP-Github/24th-Web-Team-1-BE/pull/123](https://github.com/YAPP-Github/24th-Web-Team-1-BE/pull/123)