## equals를 재정의하려거든 hashCode도 재정의하라



equals를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.

다음은 Object 명세에서 발췌한 규약이다.

+ equals 비교에 사용되는 정보가 변경되지 않는다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
+ eqauls(Object)가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
+ equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다.
  단, 다른 객체에 대해서 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

위에서 `hashCode` 재정의를 잘못했을 때 크게 문제 되는 조항은 두 번째이다.

즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 하는 것이다.



좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다.

이것이 바로 `hashCode`의 세 번째 규약이 요구하는 속성이다.

이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.



좋은 `hashCode`를 작성하는 간단한 요령은 아래와 같다.

1. int 변수 result를 선언한 후 값 c로 초기화한다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
   + 해당 필드의 **해시코드 c를 계산한다.**
     + 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다.
       *여기서 Type은 해당 기본 타입의 박싱 클래스다.*
     + 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다.
       계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 `hashCode`를 호출한다.
       필드의 값이 null이면 0을 사용한다.
     + 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
       이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음 아래 방식으로 갱신한다.
       배열에 핵심 원소가 하나도 없다면 단순히 상수를 사용한다.
       모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
   + 위에서 계산한 해시코드 c로 result를 갱신한다.
     `result = 31 * result + c`
3. result를 반환한다.

이때 파생 필드는 `hashCode` 계산에서 제외해도 된다.

즉, 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 되는 것이다.

또한 equals 비교에 사용되지 않은 필드는 반드시 제외해야 한다.

그렇지 않으면 `hashCode` 규약 두 번째를 어기게 될 위험이 있다.



`result = 31 * result + c`의 경우 필드를 곱하는 순서에 따라 result 값이 달라지게 된다.

그 결과 클래스에 비슷한 필드가 여러 개일 때 해시 코드 효과를 크게 높여준다.

코드를 통해 살펴보자.

```java
public class HashCodeDemo {

    private int oneInt;
    private int twoInt;
}
```

위와 같은 `HashCodeDemo` 클래스의 `hashCode`를 정의해보자!

```java
@Override
public int hashCode() {
  int result = 1;
  result = 31 * result + Integer.hashCode(oneInt);
  result = 31 * result + Integer.hashCode(twoInt);
  return result;
}
```

우선 result를 1로 초기화 하고 c에 해당하는 `Integer.hashCode(nInt)`의 경우 초기화하지 않고 result 계산과정에 바로 계산해주었다.



"이때 파생 필드는 `hashCode` 계산에서 제외해도 된다."

이 문장도 조금 더 알아보자.

그러기 위해 HashCodeDemo를 아래와 같이 조금 수정해보자.

```java
public class HashCodeDemo {

    private int oneInt;
    private int twoInt;
    private HashCodeDemo2 hashCodeDemo2; // 이전 HashCodeDemo와 동일 멤버를 가진 클래스
}
```

위와 같은 경우 파생 필드에 해당하는 경우가 HashCodeDemo2일 것이다.

그렇다면 이를 `hashCode` 계산시 어떻게 처리해야할까?

```java
@Override
public int hashCode() {
  int result = 1;
  result = 31 * result + Integer.hashCode(oneInt);
  result = 31 * result + Integer.hashCode(twoInt);
  result = 31 * result + hashCodeDemo2.hashCode();
  return result;
}
```

위와 같이 HashCodeDemo2의 `hashCode` 계산은 HashCodeDemo2에게 맞기고 그 값만 활용하면 된다.



그렇다면 `hashCode`를 구하기 위해 위와 같은 과정을 매번 해야할까?

이를 한번에 도와주는 `Objects` 클래스의 정적 메서드 `hash`가 있다.

이 메서드를 사용하면 위의 과정을 아래와 같이 한 줄로 줄일 수 있다.

```java
Objects.hash(oneInt, twoInt, hashCodeDemo2);
```



그런데 위의 메서드를 따라가보면 우리가 구현한 것과 크게 다를께 없다.

```java
// Objects
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}

// Arrays
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

그러니 hash는 성능에 민감하지 않은 상황에서만 사용하는 것이 좋을 것이라 한다.



그리고 위를 보면 항상 `hashCode`를 계산하는 것을 확인할 수 있는데 클래스가 불변이고 `hashCode`를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식을 고려할 수 있다.

만약 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 `hashCode`를 계산해둬야 한다.

해시의 키로 사용되지 않는 경우라면 `hashCode`가 처음 불릴 때 계산하는 지연 초기화 전략을 사용할 수 있다.

```java
private int hashCode; // 우선 0으로 초기화

@Override
public int hashCode() {
  int result = hashCode;
  if (result == 0) {
   	result = 1;
    result = 31 * result + Integer.hashCode(oneInt);
    result = 31 * result + Integer.hashCode(twoInt);
    result = 31 * result + hashCodeDemo2.hashCode(); 
  }
  return result;
}
```

이때 성능을 높이기 위해 `hashCode`를 계산할 때 핵심 필드를 생략해서는 안된다.

속도는 빨라지겠지만, 해시 풀질이 나빠져 해시테이블의 성능을 떨어뜨릴 수도 있다.