## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라



철저한 객체 지향 프로그래머는 아래와 같은 클래스를 싫어해서 필드를 모두 private로 바꾸고 접근자(getter)를 추가한다.

```java
// 싫어하는 class
class Point {
 public double x;
 public double y;
}

// 접근자 추가
@Getter
@Setter
@AllargConstructor
class Point {
 private double x;
 private double y;
}
```

위와 같이 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.



하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.



public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있는데 이때 필드가 불변이라면 직접 노출할 때의 단점이 조금 줄어든다.

하지만 여전히 좋은 생각은 아니다.

API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.

단, 불변식은 보장할 수 있게 된다.