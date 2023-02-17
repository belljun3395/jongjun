## null 대신 Optional 클래스



### Optional 클래스 소개

Optional은 선택형값을 캡슐화하는 클래스다.

값이 있으면 Optional 클래스는 값을 감싸준다.

만약 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.



Optional을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분해준다.

하지만 모든 null 참조를 Optional로 대치하는 것은 바람직하지 않고 Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이라 생각하면 된다.

즉, 메서드의 시그니처만 보고도 선택형값인지 여부를 구별할 수 있도록 하는 것입니다.



### Optional 적용 패턴

#### Optional 객체 만들기

##### 빈 Optional

정적 팩토리 메서드 `Optional.empty`로 빈 `Optional` 객체를 얻을 수 있다.

##### null이 아닌 값으로 Optional 만들기

정적 팩토리 메서드 `Optional.of` 로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.

그렇기에 null을 넣는다면 NullPointException이 발생하게 된다.

##### null 값으로 Optional 만들기

정적 팩토리 메서드 `Optional.ofNullable` 로 null 값을 저장할 수 있는 Optional을 만들 수 있다.



#### 디폴트 액션과 Optional 언랩

+ `get()`은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드다.
  래핑된 값이 있으면 해당 값을 반환하고 값이 없으면 `NoSuchElementException`을 발생시킨다.
  그렇기에 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니면 get 메서드를 사용하지 않는 것이 바람직하다.
+ `orElse` 메서드를 이용하면 Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.
+ `orElseGet`는 `orElse` 메서드에 대응하는 게으른 버전의 메서드다.
  Optional에 값이 없을 때만 Supplier가 실행된다.
+ `orElseThrow`는 Optional 이 비어있을 대 예외를 발생시킨다.
+ `ifPresent`를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.
