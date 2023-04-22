## Comparable을 구현할지 고려하라



우선 **compareTo는 Object의 메서드가 아니다.**

**compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.**

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 **자연적인 순서가 있음**을 뜻한다.



모든 객체에 대해 전역 동치관계를 부여하는 `equals` 메서드와는 달리, compareTo는 타입이 다른 객체를 신경 쓰지 않아도 된다.

타입이 다른 객체가 주어지면 간단히 ClassCastException을 던져도 되며, 대부분 그렇게 한다.



> Comparable을 구현한 클래스는 모든 x, y에 대해 `sgn(x.compareTo(y)) == - sgn(y.compareTo(x))` 여야 한다.

compareTo의 첫 번째 규약은 두 객체 참조의 **순서를 바꿔 비교해도 예상한 결과**가 나와야 한다는 얘기다.



>  Comparable을 구현한 클래스는 추이성을 보장해야 한다.

두 번째 규약은 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야한다는 뜻이다.



> Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0`이면 
>
> `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`다.

**크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 뜻**이다.



이는 compareTo 메서드로 수행하는 동치성 검사도 `equals` 규약과 똑같이 **반사성, 대칭성, 추이성**을 충족해야 함을 뜻한다.



> 필수는 아니지만 지키는게 좋은 규약
>
> `(x.compareTo(y) == 0) == (x.equals(y))`

이 규약은 간단히 말하면 compareTo 메서드로 수행한 **동치성 테스트의 결과가 `equals`와 같아야 한다**는 것이다.

이를 잘 지키면 compareTo로 줄지은 순서와 `equals`의 결과가 일관되게 된다.



compareTo 메서드 작성 요령은 `equals`와 비슷하다.

몇 가지 차이점만 주의하면 된다.

Comparable은 **타입을 인수로 받는 제네릭 인터페이스**이므로 compareTo 메서드의 **인수 타입은 컴파일 타임에 정해진다.**

**입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다.**

인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.

또한 null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.

물론 실제로도 인수의 멤버에게 접근하려는 순간 이 예외가 던져진다.



### compareTo 메서드 구현 팁

#### 비교자

compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 **그 순서를 비교**한다.

객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.

compareTo 메서드에서 **Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용**한다.

이때 비교자는 직접 만들거나 자바가 제공하는 것 중에 골라 사용하면 된다.

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
}
```



#### 박싱된 기본 타입 클래스 비교자

그리고 위의 예제처럼 박싱된 기본 타입 클래스에 새로 추가된 **정적 메서드인 `compare`을 이용하면 **

**compareTo 메서드에서 관계 연산자와 `<` 와 `>` 를 사용하는 이전 방식을 사용하지 않을 수 있다.**



#### 비교해야 할 여러 필드

또 클래스에 **핵심 필드가 여러 개**라면 어느 것을 먼저 비교하느냐가 중요해진다.

그렇기에 **가장 핵심적인 필드부터 비교**하는 것이 좋다.

```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);
  if (result == 0) {
    result = Short.compare(prefix, pn.prefix);
    if (result == 0) {
      result = Short.compare(lineNum, pn.lineNum);
    }
  }
  return result;
}
```



### Comparator 인터페이스

자바 8에서는 **Comparator**가 일련의 비교자 생성 메서드와 팀을 꾸려 **메서드 연쇄 방식으로 비교자를 생성**할 수 있게 되었다.

그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는 데 멋지게 사용할 수 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix)
		.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```



### 값의 차를 활용한 비교

그리고 이따금 '값의 차'를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 

두 값이 같으면 0을, 

첫 번째 값이 크면 양수를 반환하는 compareTo나 `compare` 메서드를 마주할 수도 있다.

이는 **정수 오버플로**를 일으키거나 **IEEE 754 부동소수점 계산 방식에 따른 오류**를 낼 수도 있기에 사용하지 않는 것이 좋다.