## int 상수 대신 열거 타입을 사용하라



열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

다시 말해 열거 타입은 인스턴스 통제된다.

싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

그리고 열거 타입은 컴파일 타임 타입 안전성을 제공한다.



열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

Object 메서드을 높은 품질로 구현해 놓았고, Comparable과 Serializable을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해 놓았다.



열거 타입에는 어떤 메서드도 추가할 수 있다.

이는 열거 타입이 가장 단순하게는 그저 상수 모음일 뿐인 열거 타입이지만, 고차원의 추상 개념 하나를 완벽하게 표현할 수도 있다는 것이다.



열거 타입을 만들 때, 열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

그리고 상수별로 다르게 동작하는 코드를 추가하고 싶으면 열거 타입에 추상 메서드를 선언하고 상수별 클래스 몸체, 즉, 각 상수에서 자신에게 맞게 재정의하면 상수별 메서드를 구현을 할 수 있다.

이를 코드로 보면 아래와 같다.

```java
public enum Operation {
    PlUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    },
    ;

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
  
    public abstract double apply(double x, double y);
}
```



열거타입은 상수 이름을 입력 받아 **그 이름(ex "PLUS")**에 해당하는 상수를 반환해주는 **`valueOf(String)` 메서드가 자동 생성된다.**

한편, 열거 타입의 toString 메서드를 재정의하면, **재정의된 toString을 키(key)로 하여 해당 열거 타입 상수를 반환해 주는 `fromString` 메서드를 제공하는 것을 고려해 볼 수 있다.**



이는 `stringToEnum`과 같은 메서드를 통해 재정의한 `toString`을 키로 하고 해당 열거 타입을 값으로 가지고 있는 맵을 만드는 메서드가 필요하다.

이때 열거 타입 상수가 `stringToEnum` 맵에 추가되는 시점은 **열거 타입 상수 생성 후 정적 필드가 초기화될 때이다.**

```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values())
        .collect(toMap(Object::toString, e -> e));
```

이렇게 만든 맵에서 `fromString` 메서드는 아래와 같이 지정한 문자열에 해당하는 열거 타입이 존재한하다면 반환한다.

```java
public static Operation fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol)).orElseThrow(IllegalArgumentException::new);
}
```



한편, 상수별 메서드 구현에는 **열거 타입 상수끼리 코드를 공유하기 어렵다는 단점**이 있다.

이때 **코드를 공유하는 가장 깔끔한 방법은 '전략'을 선택하는 방법이다.**

아래는 전력을 활용한 코드 공유와 그렇지 않은 코드 공유의 예시다.

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minusWorked, int payRate) {
        return payType.pay(minusWorked, payRate);
    }

    enum PayType {
        WEEKDAY {
            int overtimePay(int hours, int payRate) {
                return hours <= MINUS_PER_SHIFT ? 0 :
                    (hours - MINUS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int hours, int payRate) {
                return hours * payRate / 2;
            }
        };
        abstract int overtimePay(int minusWorked, int payRate);
        private static final int MINUS_PER_SHIFT = 8 * 60;

        int pay(int minusWorked, int payRate) {
            int basePay = minusWorked * payRate;
            return basePay + overtimePay(minusWorked, payRate);
        }
    }
}
```

```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINUS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY:
                overtimePay = minutesWorked * payRate / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINUS_PER_SHIFT ?
                    0 : (minutesWorked - MINUS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

**전략을 사용하지 않고 코드를 공유하는 방법**은 간결해 보이지만, **관리 관점에서는 위험한 코드다.**

휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 잊지 말고 쌍으로 넣어줘야 하기 때문이다.



하지만 **'전략'을 사용한다면 코드**는 길어지지만, 관리에 용이해진다.

열거 타입의 상수가 각각의 전략을 가지고 있고 **그 전략에 책임을 할당한면 되는 것이기 때문이다.**



하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.



추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거타입이라도 이 방식을 적용하는 게 좋다.

종종 쓰이지만 열거 타입 안에 포함할 만큼 유용하지는 않은 경우도 마찬가지다.



그럼 열거 타입을 **언제 사용하는 것이 좋을까?**

**필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하는 것이 좋다.**

**이때 열거 타입에 정의된 상수 개수가 영원히 고정일 필요는 없다.**