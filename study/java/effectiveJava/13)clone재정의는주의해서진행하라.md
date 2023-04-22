## clone 재정의는 주의해서 진행하라



clone 메서드가 선언된 곳은 Cloneable이 아닌 Object이고, 그마저도 protected로 선언되어 있다.

그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

리플랙션을 사용하면 가능하지만, 100% 성공하는 것도 아니다.

해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문이다.



그리고 Cloneable 인터페이스는 메서드가 없다.

그럼 이는 대체 무슨 일을 하는 것일까?

이는 Object의 protected 메서드인 clone의 동작 방식을 결정한다.

Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 

그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.



### 불변 객체

제대로 동작하는 clone 메서드를 가진 상위 클래스르 상속해 Cloneable을 구현해 보자.

먼저 `super.clone`을 호출한다.

그렇게 얻은 객체는 원본과 완벽한 복제본일 것이다.

즉, 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 갖는다.

**모든 필드가 기본 타입이거나 불변 객체를 참조한다면** 이 객체는 완벽히 우리가 원하는 상태라 **더 손볼 이유가 없다.**



그리고 clone 메서드를 제정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위타입으로 변환하여 **클라이언트가 형변환을 하지 않아도 되게끔 하는 것이 좋다.**



위의 사항들을 반영한 clone 메서드는 아래와 같다.

```java
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```



### 가변 객체

가변 객체인 경우에는 clone 메서드를 구현하는 것이 조금은 복잡해진다.

clone 메서드는 사실상 **생성자와 같은 효과를 낸다.**

그렇기에 clone은 **원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장**해야 한다.



```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INTIAL_CAPACITY = 16;
  
  public Stack() {
    this.elements = new Object[DEFAULT_INTIAL_CAPACITY];
  }
  ...
}
```

위의 Stack 클래스를 복제하기 위해 clone 메서드를 단순히 `super.clone`의 결과를 그대로 반환한다면 

Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, 

**`elements` 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.**

**이는 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기다.**



그렇기에 스택 **내부 정보를 복사해야 한다.**

가장 쉬운 방법은 **`elements` 배열의 clone을 재귀적으로 호출하는 것이다.**

```java
@Override
public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

위와 같이 clone을 정의하기 위해서는 `elements` 필드처럼 **final 한정자로 선언되지 않아야 한다.**

final 필드에는 새로운 값을 할당할 수 없기 때문이다.

하지만 이는 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.

그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.



다른 예제도 살펴보자.

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }
  ...
  
  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch (CloneNotSupportedException e) {
    		throw new AssertionError();
  	}
  }
}
```

위의 예제에는 복제본이 자신만의 버킷 배열을 갖지만, 

**이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.**



그래서 아래와 같이 수정할 필요가 있다.

```java 
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  
  private static class Entry {
    ...
    
    Entry deepCopy() {
      return new Entry(key, value,
                      next == null ? null : next.deepCopy());
    }
  }
  
  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++) {
        if(buckets[i] != null) {
          result.buckets[i] = buckets[i].deepCopy();
        }
      }
      return result;
    } catch (CloneNotSupportedException e) {
    		throw new AssertionError();
  	}
  }
}
```

private 클래스인 HashTable.Entry는 깊은 복사(`deepCopy()`)를 지원하도록 보강하였다.

clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 

다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 수행한다.



이때 Entry의 `deepCopy` 메서드는 자신이 가리키는 연결리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다.

이 방법은 버킷이 너무 길지 않다면 잘 작동한다.

하지만 연결 리스트를 복제하는 방법으로는 좋지 않다.



`deepCopy` 메서드는 위의 예제처럼 재귀 호출로 구현하는 대신 반복자를 써서 순회하는 방향으로 구현할 수도 있다.

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next) {
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  }
  return result;
}
```



마지막으로 `super.clone`을 호출하여 얻은 **객체의 모든 필드를 초기 상태로 설정한 다음, 원복 객체의 상태를 다시 생성하는 고수준 메서드를 호출하는 방법도 있다.**

위의 예에서 `buckets` 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 `키-값` 쌍 각각에 대해 복제본 테이블의 `put(key, value)` 메서드를 호출해 둘의 내용이 똑같게 해줄 수 있다.

이처럼 고수준 API를 활용해 복제하면 보통은 간단하고 제법 우아한 코드를 얻게 되지만, 아무래도 저수준에서 바로 처리할 때보다는 느리다.

또한 **Cloneable 아키텍처의 기초가 되는 필드의 단위 객체 복사**를 우회하기 때문에 **전체 Cloneable 아키텍처와는 어울리지 않는 방식**이기도 하다.



### 복사 생성자와 복사 팩터리

```java
// 복사 생성자
public Yum(Yum yum) {...};

// 복사 팩터리
public static Yum newInstance(Yum yum) {...};
```

언어 모순적이고 위험한 객체 생성 메커니즘을 사용하지 않으며, 

엉상하게 문서화된 규약에 기대지 않고, 

정상적인 final 필드 용법과 충돌하지 않으며, 

불필요한 검사 예외를 던지지 않고, 

형변환도 필요하지 않다.



그리고 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.

이들을 이용하면 클라이언트 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

**그렇기에 Cloneable을 이미 구현한 클래스가 아니라면 복사 생성자와 복사 팩터리 방법을 추천한다.**