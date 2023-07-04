## 비트 필드 대신 EnumSet을 사용하라



우선 비트에 대해서 다시 한번 리마인드 해보자.

```java
public class Bits {
    public static final int ONE = 1 << 0;
    public static final int TWO = 1 << 1;
    public static final int THREE = ONE | TWO;

    public static void main(String[] args) {
        System.out.println(ONE);
        System.out.println(TWO);
        System.out.println(THREE);
    }
}
```

자바에서 역시 `<<, >>`와 같은 연산자를 사용하여 비트를 다룰 수 있다.

위의 THREE의 경우 ONE과 TWO를 OR 연산하여 나타낸 것을 볼 수 있다.



비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제를 가지고 있다.

+ 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
+ 필드 하나에 녹아 있는 모든 원소를 순회하기도 어렵다.
+ 최대 몇 비트가 필요한지 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.



그렇기에 상수 집합을 사용할 때 EnumSet 클래스는 비트 필드의 좋은 대안이다.

EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 준다.

Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.

하지만 EnumSet의 내부는 비트 벡터로 구현되어 있다.

즉, 원소가 총 64개 이하라면, 즉 대부분의 경우 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다는 것이다.



아래 예제는 비트 필드를 활용한 방법과 EnumSet을 활용한 방법에 관한 예제다.

```java
public class Choices {
    public static final int FIRST = 1 << 0;
    public static final int SECOND = 1 << 1;
    public static final int THIRD = 1 << 2;

    public void applyText(int bits) {
        if ((bits & THIRD) != 0) {
            System.out.println("THIRD is chosen");
        }

        if ((bits & SECOND) != 0) {
            System.out.println("SECOND is chosen");
        }

        if ((bits & FIRST) != 0) {
            System.out.println("FIRST is chosen");
        }
    }

    public static void main(String[] args) {
        Choices choices = new Choices();
        choices.applyText(SECOND | THIRD);
    }
}
```

```java
public class Choices {
    public enum Choice {
        FIRST, SECOND, THIRD
    }

    public void applyText(Set<Choice> choices) {
        if (choices.contains(Choice.THIRD)) {
            System.out.println("THIRD is chosen");
        }

        if (choices.contains(Choice.SECOND)) {
            System.out.println("SECOND is chosen");
        }

        if (choices.contains(Choice.FIRST)) {
            System.out.println("FIRST is chosen");
        }
    }

    public static void main(String[] args) {
        Choices choices = new Choices();
        choices.applyText(EnumSet.of(Choice.SECOND, Choice.THIRD));
    }
}
```

위의 코드를 통해 확인할 수 있듯 EnumSet을 활용하면 타입 안전하면서 성능까지 챙길 수 있는 것을 확인할 수 있다.