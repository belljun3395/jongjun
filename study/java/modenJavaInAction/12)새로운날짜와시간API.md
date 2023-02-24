## 새로운 날짜와 시간 API

### LocalDate, LocalTime, Instant, Duration, Period 클래스

#### LocalDate와 LocalTime 사용

LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.

이를 사용하는 기본적인 방법은 아래와 같다.

```java
LocalDate date = LocalDate.of(2017, 9, 21);
int year = date.getYear();
Month month = date.getMonth();
int day = date.getDayOfMonth();
DayOfWeek dow = date.getDayOfWeek();
int len = date.lengthOfMonth();
boolean leap = date.isLeapYear();
```

`get` 메서드에 `TemporalField`를 전달해서 정보를 얻는 방법도 있다.

`TemporalField`는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스다.

`ChronoField`는 `TemporalField` 인터페이스를 정의하므로 `ChronoField` 의 열거자 요소를 이용해 원하는 정보를 아래와 같이 얻을 수 있다.

```java
int year = date.get(ChronoField.YEAR); // == date.getYear();
int month = date.get(ChronoField.MONTH_OF_YEAR); // == date.getMonthValue();
int day = date.get(ChronoField.DAY_OF_MONTH); // == date.getDayOfMonth();
```



LocalTime 인스턴스는 시간을 표현하는 불변 객체다.

이를 사용하는 기본적인 방법은 아래와 같다.

```java
LocalTime time = LocalTime.of(13, 45, 20);
int hour = time.getHour();
int minute = time.getMinute();
int second = time.getSecond();
```



추가로 LocalDate, LocalTime 모두 `parese` 메서드에 `DateTimeFormatter`을 전달할 수도 있다.

`DateTimeFormatter`의 인스턴스는 날짜, 시간 객체의 형식을 지정한다.

아래는 이를 활용한 예시와 형식 지정을 위한 포멧형식이다.

```java
String str = "2021-11-05 13:47:13.248";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS");
LocalDateTime dateTime = LocalDateTime.parse(str, formatter);
```

![img](https://blog.kakaocdn.net/dn/omKz7/btrJ70oOkOz/PtU2KFQSkC3FR06jAkkW1K/img.png)



#### 날짜와 시간의 조합(LocalDateTime)

LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.

즉, LocalDateTime은 날짜와 시간을 모두 표현할 수 있으며 날짜와 시간을 조합할 수도 있다.



#### Instant 클래스

Instant 클래스는 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현하는 클래스이다.

즉, 이는 사람을 위한 것이기 보다는 기계 전용의 유틸리티라고 생각하는 것이 좋다.



#### Duration과 Period 

