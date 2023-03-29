## equals는 일반 규약을 지켜 재정의하라



### equals를 재정의하지 않는 경우

다음 열거한 상황 중 하나에 해당한다면 **equals를 재정의하지 않는** 것이 최선이다.

+ 각 인스턴스가 **본질적으로 고유**하다.
  값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다.
  ex) Thread
+ 인스턴스의 **논리적 동치성을 검사할 일이 없다.**
+ 상위 클래스에서 정의한 equals가 하위 클래스에도 딱 들어맞는다.
+ 클래스가 private거나 package-private이고 equals 메서드를 호출할 일이 없다.



### equals를 재정의해야 하는 경우

다음 열거한 상황에는 **equals를 재정의해야 하는 경우**는 아래와 같다.

**객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.**

주로 값 클래스가 해당한다. **(값 클래스는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 한다.)**



### equals 일반 규약

equals 메서드를 재정의할 때는 일반 규약을 따라야 하는데 Object 명세에 적힌 규약은 아래와 같다.



#### 동치 관계

그전에 Object 명세에서 말하는 동치관계란 무엇일까?

집합을 서로 같은 원소로 이뤄진 부분집합으로 나누는 연산이다.

이 부분집합을 **동치류**라 한다.

**equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.**



#### 반사성

객체는 자기 자신과 같아야 한다는 뜻이다.



#### 대칭성

두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

// #1
cis.equals(s); // true

// #2
s.equals(cis); // false
```

#1과 #2를 보면 서로에 대해 동치 여부를 다르게 답한다.



#### 추이성

첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다.



##### 위반 사례 : 상속

상속하는 클래스의 경우가 추이성을 위배할 수 있는 대표적인 경우라고 한다.

```java
public class Point {
  private final int x;
  private final int y;
  ...
}

public class ColorPoint extends Point {
  private final Color color;
  
  public ColorPoint(int x, int y, Color color) {
    super(x,y);
    this.color = color;
  }
}
```

위의 경우 조심해야하는 상황을 살펴보자.



###### 대칭성 위배

```java
@Override public boolean equals(Object o) {
  if(!(o instanceof ColorPoint))
    return false;
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

위와 같이 ColorPoint의 equals를 정의하는 경우 아래와 같이 대칭성 위배가 일어난다.

```java
cp.equals(p); // false
p.equals(cp); // ture
```



###### 추이성 위배

```java
@Override public boolean equals(Object o) {
  if(!(o instanceof Point))
    return false;
  if(!(o instanceof ColorPoint))
	  return o.equals(this); 
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

위는 **o가 Point인 경우는 Point의 equals를 이용하여 색상을 무시하고 비교**하고 

**o가 ColorPoint이면 색상까지 비교**하도록 equals를 정의한 것이다.

이러한 경우 아래와 같이 추이성 위배가 일어난다.

```java
ColorPoint p1 = new ColorPoint(1,2,Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);

p1.equals(p2); // ture
p2.equals(p3); // ture
p3.equals(p1); // false
```



###### 리스코프 치환 원칙 위배

```java
@Override public boolean equals(Object o) {
  if(o == null || o.getClass() != getClass()) 
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```

***어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.***

***따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.***

하지만 위의 코드는 Point의 하위클래스가 여전히 Point이지만 Point로써 활용되지 못하는 코드이다.



###### 대체 방안

상속 대신 **컴포지션**을 사용하는 방법으로 위의 문제를 해결할 수 있다.

*컴포지션이 **"has"**만 있는 줄 알았는데 **"is part of"**로 판단할 수 있다고 한다.*

```java
public class ColorPoint {
  private final Point point;
  private final Color color;
  
  public ColorPoint(int x, int y, Color color) {
    this.point = new Point(x,y);
    this.color = Objects.requiredNonNull(color);
  }
  
  ...

  // ColorPoint의 Point 뷰를 반환
  public Point asPoint() {
    return point;
  }
  
  @Override public boolean equals(Object o) {
    if(!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```



#### 일관성

두 객체가 같다면 앞으로도 영원히 같아야 한다는 뜻이다.

이때 equals의 판단에 **신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.**

**equals는 항시 메모리에 존재하는 객체만 사용한 결정적 계산만 수행해야 한다.**



### HashCode

equals를 재정의할 땐 hashCode도 반드시 재정의해야 한다.