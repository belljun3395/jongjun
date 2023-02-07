## 컬렉션 API 개선

### 컬렉션 팩토리

#### 리스트 팩토리

`List.of` 팩토리 메소드를 이용해 간단히 리스트를 만들 수 있다.

이때 리스트에 요소를 추가하려고 하면 `java.lang.UnsupportedOperationException` 이 발생한다.

이는 `List.of` 가 **크기를 변경할 수 없는 리스트**를 만든 것이기 때문이다.

이는 컬렉션이 의도치 않게 변하는 것을 막을 수 있기에 나쁜 것은 아니지만 요소 자체가 변하는 것은 막을 수 없다.

그렇기에 데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 사용하기 간편한 팩토리 메서드를 권장 할 수 있다.



#### 집합 팩토리

`Set.of` 팩토리 메서드를 이용해 간단히 집합을 만들 수 있다.



#### 맵 팩토리

`Map.of` 팩토리 메서드에 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있다.

만약 그 값이 많아진다면 `Map.Entry` 객체를 인수로 받는  `Map.ofEntries` 팩토리 메서드를 이용하는 것이 좋다.

```java
Map.ofEntries(Map.Entry("kim", 30), Map.Entry("jong", 20), Map.Entry("jun", 10));
```



### 리스트(List)와 집합(Set) 처리

+ `removeIf`
+ `replaceAll`
+ `sort`

이들은 호출한 컬렉션 자체를 바꾼다.

**새로운 결과를 만드는 스트림**과 달리 이들은 **기존 컬렉션을 바꾼다.**



#### removeIf

```java
for (Transaction transaction : transactions) {
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}
```

위의 코드는 `ConcurrentModificationException`을 일으킨다.

`for-each`루프는 내부적으로 `Iterator`객체를 사용하므로 위의 코드는 다음과 같이 해석된다.

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}
```

+ `transactions.iterator()`

+ `transactions.remove(transaction);`

위의 두 코드를 보면 두 개의 개별 객체가 하나의 컬렉션(`transactions`)을 관리하는 것을 확인할 수 있다.



그리고 반복자(`iterator`)의 상태는 컬렉션의 상태와 동기화되지 않는다.

그렇기에 `Iterator` 객체를 명시적 사용하여 위의 문제를 해결할 수 있다.

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    // transactions.remove(transaction);
    iterator.remove();
  }
}
```



하지만 이러한 과정은 삭제할 요소를 가리키는 프레디케이트 인수를 받는 `removeIf` 메서드를 활용하여 간단히 작성할 수 있다.

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```



#### replaceAll

스트림 API를 사용하여 리스트의 각 요소를 새로운 요소로 바꾸는 것과 replaceAll 메서드의 차이는 replaceAll은 새 문자열 컬렉션을 만드는 것이 아닌 **기존의 컬렉션을 바꾸는 것**이라는 것이다.

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0) + code.substring(1)));
```



### 맵(Map) 처리

#### forEach

맴에서 키와 값을 반복하면서 확인하는 작업은 귀찮다.

그렇기에 자바 8에서 Map 인터페이스는 BiConsumer를 인수로 받는 forEach 메서드를 지원한다.

```java
ageOfFriends.forEach((friend, age) -> System.out.println(firend + "is" + age + "years old"));
```



#### 정렬

+ `Entry.comparingByValue`
+ `Entry.comparingByKey`

위의 두 유틸리티를 이용하여 맵의 항목(`Map.entrySet()`)을 값 또는 키를 기준으로 정렬할 수 있다.

```java
mapsExample.entrySet().stream() // 맵의 항목
  					.sorted(Entry.comparingByKey) // 정렬
  					.forEachOrdered(System.out::println);
```



#### getOrDefault

찾으려는 키가 맵에 존재하지 않으면 null이 반환되며 `NullPointerException`이 일어난다.

하지만 이는 **기본값을 반환**하는 방식으로 해결할 수 있다.

기존의 맵과 동일하게 첫 번째 인수로는 **키**를, 두 번째 인수로는 **기본값**을 받으며 **맵에 키가 존재하지 않거나** **키가 존재하더라도 값이 null이면** 기본 값을 반환한다.



#### 계산 패턴

+ `computeIfAbsent`

+ `computeIfPresent`

+ `compute`

맴에 **키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황**에 위의 계산 패턴을 사용한다.



#### 교체 패턴

+ `replaceAll`
+ `replace`

`replaceAll` 의 경우 `BiFunction`을 적용한 결과로 **각 항목의 값을 교체**한다.

`replace`의 경우 키가 존재하면 맵의 값을 바꾼다.

```java
mapsExample.replaceAll((key, value) -> key.toUpperCase());
```



#### 합침

우선 `putAll`을 사용하여 두 맵을 합칠 수 있다.

**중복된 키가 없다면** 이는 잘 동작한다.



하지만 값을 조금 더 유연하게 합쳐야 한다면 `merge`메서드를 이용할 수 있다.

`merge`는 `BiFunction`을 인수로 받으며 아래와 같이 구성되어 있다.

```java
SomeMap.merge(key, value, (existKeyValue, duplicateKeyValue) ->  ... );
```



### ConcurrentHashMap

#### 리듀스와 검색

+ `forEach`
+ `reduce`
+ `search`

`ConcurrentHashMap` 역시 스트림에서 봤던 것과 비슷한 종류의 세 가지 새로운 연산을 지원한다.

이 연산은 `ConcurrnetHashMap`의 **상태를 잠그지 않고 연산**을 수행한다는 점에 주목해야한다.

따라서 이들 연산에 제공하는 함수는 계산이 진행되는 동안 **바뀔 수 있는 객체, 값, 순서** 등에 **의존하지 않아야 한다.**



또한 이들 연산에 **병렬성 기준값**을 지정해야 한다.

맵의 크기가 주어진 **기준 값보다 작으면 순차적으로 연산을 실행**한다.



#### 계수

`ConcurrentHashMap` 클래스는 맵의 매핑 개수를 반환하는 `mappingCount` 메서드를 제공한다.

기존의 `size` 메서드 대신 새 코드에서는 int를 반환하는 `mappingCount` 메서드를 사용하는 것이 좋다.



#### 집합뷰

`ConcurrentHashMap` 클래스는 `ConcurrentHashMap`을 집합 뷰로 반환하는 keySet이라는 새 메서드르 제공한다.

맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다.

newKeySet이라는 새 메서드를 이용해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.