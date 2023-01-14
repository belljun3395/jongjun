## config 서버 소개

아직 config 서버를 재대로 알고 사용하지는 못했다. 

그래서 이번에는 어떻게 사용하였는지 위주로 글을 작성하려 한다.



우선 config 서버의 resource의 구조는 다음과 같다.

```
resource
|-- config
|---- auth.properties
|---- ...
|-- bootstrap.yml
```

bootstrap.yml이라는 본인은 자주 사용하진 않았던 파일 명이 나왔다.

알아보니 bootstrap이 가장 우선권을 가지고 있고 이후는 `application -> application-{profile} -> ...` 와 같은 순서로 우선권을 지닌다고 한다.



config 서버는 이러한 설정 파일들을 한 곳에 모아두는 서버이다.

이렇게 한 곳에 설정 파일을 모아두면 장점이 설정 파일을 여러 곳에서 공유하여 사용할 수 있다.

당장만 하더라도 token과 관련된 설정 파일을 auth와 zuul 서버가 공유하여 사용하였다.

```yml
spring:
  application:
    name: auth
  profiles:
    active: default ,token, jwt
  cloud:
    config:
      uri: http://localhost:8071
```

```yml
spring:
  application:
    name: zuul
  profiles:
    active: jwt
```

이는 config 서버의 application-jwt를 공유하고 있는 설정이다.



이렇게 공통 설정을 공유할 수 있다는 것 말고도 무중단 설정 변경을 할 수 있는 등 다양한 장점이 있지만 그 부분에 대해서는 아직 파악이 되지 않아 추후 구현해보도록 할 것이다.