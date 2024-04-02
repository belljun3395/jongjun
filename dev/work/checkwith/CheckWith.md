# CheckWith!

## 기획 의도

종종 무언가를 할 때 **체크 리스트를 공유할 수 있으면 좋겠다**는 생각을 하였습니다.

메신저를 활용하여해야 할 일을 공유할 수 있지만 일의 수행 여부를 표시하기 힘들 뿐 아니라 일이 추가되고 제외된다면 이를 한눈에 파악하기 힘들다는 문제를 가지고 있습니다.

이에 간단히 체크 리스트를 생성하고 체크 목록을 작성하여 이를 공유하여 웹 페이지에서 체크 목록과 각각의 체크 여부를 한눈에 확인할 수 있는 프로젝트를 기획하였습니다.

[when2meet](https://www.when2meet.com/)이 단체의 가능 시간 조사라는 개인의 수고가 필요하였던 작업을 간단히 이벤트 생성하고 이를 공유하여 취합하는 방법으로 잘해결하였다면

[CheckWith](https://www.checkcheckwith.com)은 체크 리스트 공유를 통해 메신저 대화에서는 놓일 수 있는 **체크 목록을 한눈에 파악하기 쉽도록 돕는 것**을 목표로 합니다.

## 기능

현재 제공하고 있는 기능은 아래와 같습니다.

-   체크 리스트를 만든다
-   체크 목록을 추가한다
-   체크한다



### 사용 사례

MT를 가기 위해 마트에서 팀을 나누어 장을 보는 경우를 가정하여 사용 사례를 설명해보려 합니다.



![img](https://blog.kakaocdn.net/dn/nf1hw/btsGjSmFJ8K/hAhzJSokQ1SUhkXPA4NfTk/img.png)

우선 위와 같이 "장보기"라는 이름의 **체크 리스트를 생성**합니다.



![img](https://blog.kakaocdn.net/dn/bQaYN3/btsGkuyQuF5/dcZL2LB1lcO4kZZDQZ3XX1/img.png)

이후 자신의 **신원을 확인할 수 있는 정보를 제출**합니다.



![img](https://blog.kakaocdn.net/dn/b0QCeX/btsGiTsSqWv/zZX1a9rDswm4fAcOCIZx51/img.png)

신원 제출 이후 **목록을 추가**합니다.

목록 추가가 끝나면 해당 **링크를 공유**합니다.



![img](https://blog.kakaocdn.net/dn/bWodoO/btsGhDEwFu8/zXleAYVcuVIdw5N2fmggk0/img.png)

해당 링크를 공유받은 사람은 위와 같이 이미 작성된 **목록을 확인**할 수 있습니다.



![img](https://blog.kakaocdn.net/dn/erfOWt/btsGiqke57q/yh2mkoo6CJWosO297zyKg1/img.png)

목록에 해당하는 물건을 구매하였다면 **체크**하여 이를 표시할 수 있습니다.



## 그래서?

그래서 해당 시리즈에서는 CheckWith이라는 사이드 프로젝트의 개발기를 공유해보려 합니다.

해당 프로젝트는 **하루**라는 짧은 시간에 코드를 작성하고 배포까지의 과정을 완료하였기에 최대한 간단한 방법으로 개발을 진행하였습니다.

그렇기에 부족한 부분과 개선할 부분들이 많이 존재하고 이를 하나씩 개선해 나가는 과정을 기록해보려 합니다.

또한 짧은 시간이었지만 배포 과정까지 완료하였기에 이를 운영해 보며 느끼는 점들도 기록하고 공유해보려 합니다.