## ordinal 인덱싱 대신 EnumMap을 사용하라



Enum의 ordinal을 배열 인덱스로 사용하는 경우가 있다.

이때 문제는 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 하고 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.

가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다는 점이다.

정수는 열거 타입과 달리 타입 안전하지 않다.



이러한 문제에 대한 해결책으로 열거 타입을 키로 사용하도록 설계한 EnumMap이라는 Map 구현체가 존재한다.

이는 Enum을 사용하기에 안전하지 않은 형변환을 사용하지 않아도 되고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.

나아가 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 없다.

EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.

내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.



예제를 보면서 조금 더 파악해 보자.

```java
public enum Plant {

    A("a", LifeCycle.ANNUAL),
    B("b", LifeCycle.ANNUAL),
    C("c", LifeCycle.PERENNIAL),
    D("d", LifeCycle.BIENNIAL);


    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL,}


    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
  
    public LifeCycle getLifeCycle() {
        return lifeCycle;
    }
}
```

```java
public class Garden {

    List<Plant> plants;

    public Garden(List<Plant> plants) {
        this.plants = plants;
    }

    public List<Plant> getPlants() {
        return plants;
    }
}
```



**실행 코드**

```java
Garden garden = new Garden(List.of(Plant.A, Plant.B, Plant.C, Plant.D));

/** 1번 */
System.out.println(
    garden.getPlants().stream()
        .collect(
            groupingBy(Plant::getLifeCycle)));

/** 2번 */
System.out.println(
    garden.getPlants().stream()
        .collect(
            groupingBy(
                Plant::getLifeCycle,
                () -> new EnumMap<>(LifeCycle.class),
                toSet())));
```

```
{BIENNIAL=[d], ANNUAL=[a, b], PERENNIAL=[c]}
{ANNUAL=[a, b], PERENNIAL=[c], BIENNIAL=[d]}
```

1번의 경우 EnumMap이 아닌 고유한 맵 구현체를 사용하여 EnumMap을 사용하여 얻는 공간과 성능 이점이 사라진다.

그렇기에 2번 같이 사용하는 것이 맵을 빈번히 사용하는 프로그램에서는 추천한다.

