# GCP로 프로젝트 배포하기

*약 3개월 무료로..*



해당 글에서는 [CheckWith](https://www.checkcheckwith.com/)이라는 체크 리스트를 공유할 수 있는 사이드 프로젝트를 만들며 구성한 **약 3개월 무료 인프라 구성**을 소개하려 합니다.



**최종 인프라 구성 미리 보기**

![img](https://blog.kakaocdn.net/dn/FSxdS/btsGkroFZ9G/CDyxzuxCg1D3CUFaPNTbH1/img.png)



## GCP(Google Cloud Platform)

![img](https://blog.kakaocdn.net/dn/bAJ5Hc/btsGiZs5Ukg/2OHrhlT9m9lKKsDEOGAouk/img.png)

인프라 구성을 위해서 GCP를 사용하였습니다.

GCP에서는 3개월간 제한 없이 사용할 수 있는 300달러의 크레딧을 제공합니다.

3개월이라는 기간의 제한이 존재하지만 사이드 프로젝트의 지속 여부를 판단하기에는 충분한 시간이라 판단하였습니다.



## DB

프로젝트를 경험하며 DB가 인프라 비용에 있어 많은 비중을 차지한다는 것을 경험한 바 있기에 DB를 가장 먼저 구성하였습니다.



![img](https://blog.kakaocdn.net/dn/brNjrI/btsGjZy8bCG/L6KS9gaCwIzIXf6tXKVFbk/img.png)

GCP에서는 위와 같이 MySQL, PostgresSQL, SQL Server 3가지의 데이터베이스 엔진을 제공합니다.

MySQL과 PostgresSQL에 따른 가격의 변동이 유의미하게 존재하지 않아 해당 프로젝트에서는 프로젝트를 진행하며 많이 사용해 본 MySQL 엔진을 선택하였습니다.



이후 GCP에서 선택 목록으로 제공하는 사항을 가장 저렴하게 선택하였더니 하루 4.4달러, 월 132달러에 다라는 비용이 청구될 것을 확인할 수 있었습니다.

![img](https://blog.kakaocdn.net/dn/u4vFi/btsGjkjuRfy/Am1fv3W5qKp0XK7R1iful1/img.png)



이에 인스턴스 맞춤 설정을 통해 적절한 가격이 나올 수 있도록 조정하였습니다.



![img](https://blog.kakaocdn.net/dn/bpNjrS/btsGkqwuPoU/jsNkv83MJjaHODUcErGbm0/img.png)

- 이전 : 전용 코어(vCPU 2개, 8GB 메모리)
- 이후 : 공유 코어(vCPU 1개, 1.7GB 메모리)



![img](https://blog.kakaocdn.net/dn/ZxGiE/btsGhRihwx1/azJC1ImksgLfWgpLbYv2mk/img.png)

- 이전 : SSD
- 이후 : HDD



![img](https://blog.kakaocdn.net/dn/6BzSe/btsGh6GiuOA/twK2F493zIDPfWKVTfqsE0/img.png)

이에 최종 DB 구성은 위와 같습니다.

성능에 있어서는 떨어질 수 있지만 하루 1.48달러로 **한 달 44.4달러**의 합리적인 비용으로 DB를 구성할 수 있었습니다.



## VM 인스턴스

### 표준

남은 예산은 약 60달러로 프런트와 백엔드 서버를 최대 30달러로 구성하여야 합니다.

우선 GCP에서 제공하는 머신 구성을 확인해 보면 한달 28.65달러로 e2-medium(vCPU 2개, 코어 1개, 메모리 4GB)의 인스턴스를 구성을 제공하고 있습니다.

![img](https://blog.kakaocdn.net/dn/bLBYPy/btsGhWRmbNC/5PqneKTAOqNdchWFK0ExR1/img.png)



하지만 인스턴스 이외에도 로드벨런서, 고정 IP, DNS와 같이 추가로 필요한 인프라 구성이 남아있었기 위의 구성을 선택할 수 없었습니다.

추가로 서버를 단일 인스턴스로 구성하기보다는 **다중 인스턴스로 구성할 계획**이기에 e2-micro(vCPU 2개, 코어 1개, 메모리 1GB)를 선택하여 인스턴스를 구성하였습니다.

![img](https://blog.kakaocdn.net/dn/vuHZY/btsGiBFWWxQ/qWaLj5sMGhmbK8KQJSbep1/img.png)



### 스팟

스팟 인스턴스를 사용하여 비용을 절감하려는 고민도 하였습니다.

이에 확인한 결과 스팟 인스턴스로 e2-medium를 사용한다면 한 달 13.85달러로 표준과 비교해 절반 가까운 비용으로 인스턴스를 사용할 수 있습니다.

하지만 스팟은 임의로 종료될 수 있기에 안정성이 떨어지고 이에 대비할 수 있는 인프라 구성을 학습하기 이전까지는 표준을 사용하여 안전성을 가져가는 선택을 하였습니다.(ex 관리형 인스턴스 그룹)



그렇다고 스팟 인스턴스를 활용하지 않는 것은 아닙니다.

스팟 인스턴스를 표준 인스턴스와 묶어 다중 인스턴스를 구성하려 합니다.



## 애플리케이션 부하 분산기



![스크린샷 2024-04-01 오후 10 01 33](https://github.com/belljun3395/jongjun/assets/102807742/0070ae03-6b46-4d23-abd3-17f3d3725242)

위의 이미지와 같이 인스턴스 계층 앞에 부하 분산기를 배치하고 해당 부하 분산기에 도메인 네임을 부여하고 인스턴스 그룹을 연결하여 외부의 요청을 처리하려 합니다.



GCP에서는 프런트엔드, 백엔드 구성, 라우팅 규칙을 통해 부하 분산기를 정의할 수 있습니다.



### 프런트엔드



![img](https://blog.kakaocdn.net/dn/AOrGQ/btsGifQGIRo/cXfwkZkSN6k3okwX3rjPUK/img.png)

부하 분산기와 외부의 접점을 의미합니다.

외부에서 부하 분산기에 접근할 프로토콜, IP 주소, 포트를 설정합니다.

이때 해당 프로젝트에서는 도메인을 부하 분산기의 IP와 연결하였기에 **부하 분산기의 IP를 고정**으로 설정하였습니다.

만약 부하 분산기의 IP를 고정하지 않고 도메인과 부하 분산기를 연결한다면 IP의 변경에 따라 도메인의 연결을 수정해야 하는 문제가 발생할 것으로 예상됩니다.

### 백엔드

백엔드는 부하 분산기와 연결될 인스턴스들을 의미합니다.

GCP에서는 이를 백엔드 서비스라 정의합니다.

#### 백엔드 서비스

백엔드 서비스는 인스턴스 그룹이라는 형태로 인스턴스들을 가지고 있습니다.

인스턴스 그룹에 속한 인스턴스는 미리 정의한 **상태 점검** 방법에 의해 백엔드 서비스에서 관리됩니다.

이에 상태 점검에 이상이 없다면 백엔드 서비스는 부하 분산기로 들어온 요청을 인스턴스 그룹에 속한 인스턴스들로 분배합니다.

해당 프로젝트에서는 기본 분배 방식은 **라운드 로빈 방식**을 채택하고 있습니다.

### 라우팅 규칙

라우팅 규칙은 부하 분산기로 들어온 요청을 백엔드로 어떻게 전달할 것인가를 정의합니다.

## 최종 구성 및 비용

![img](https://blog.kakaocdn.net/dn/cQ6AgM/btsGio03gDq/HV02qaPJFkr71lsXz8ULGK/img.png)

최종 인프라 구성은 위와 같습니다.

부하 분산기와 고정 IP의 가격을 확인하지 못하여 아직 한 달 가격을 정확히 예측할 수 없는 만큼 일주일 정도 사용한 이후 비용 추이를 보고 수정을 할 예정입니다.
