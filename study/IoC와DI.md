## IoC와 DI

### IoC

IoC는 Inversion Of Control의 약어이다.

이는 제어의 역전이라는 뜻이다.

그렇다면 "제어(Control)의 관점이 전도(Inversion)된다는 것은 무엇일까?"



스프링은 스프링 컨테이너를 통해 IoC가 가능하도록 지원해준다고 한다.

우리는 무의식적으로 IoC를 적용하고 있었다.



그렇기에 우선 **IoC가 적용되기 이전에는 어떠한 방식으로 제어**를 하였는지 살펴보자.

예제의 경우 [마틴파울러가 작성한 IoC 설명](https://www.martinfowler.com/articles/injection.html)의 예제를 사용할 것이다.

```java
public class MovieLister {

    private MovieFinder finder;

    public MovieLister() {
        this.finder = new MovieFinderImpl(); 
    }
}
```

위의 코드를 보면 MovieLister는 자신에 대한 제어권을 모두 자신이 가지고 있다.

MovieLister 자신이 MovieFinder에 대한 제어를 하고 있는 것을 볼 수 있다. (`this.finder = new MovieFinderImpl()`)



<img width="823" alt="스크린샷 2023-03-29 오후 5 43 29" src="https://user-images.githubusercontent.com/102807742/228478179-2029ff6a-711a-43c0-8530-bcdbb2b4573d.png">

그렇기에 의존성을 확인해 보아도 뭔가 찜찜한 부분이 있다.

분명 MovieFinder라는 인터페이스를 통해 MovieFinder와 결합도를 낮췄지만 MovieLister은 MovieFinderImpl을 알고 있어야 한다.

*DIP도 위반하고 있다! 더 중요한 모듈이 덜 중요한 모듈에 의존하고 있다.*



이는 finder를 플러그인되도록 할 수 없게 만든다.

*이때 플러그인 된다는 것은 구현 클래스가 컴파일 타임에 결정되는 것이 아니라 런타임에 결정되는 것을 뜻한다. ([컴파일 타임과 런타임의 의미](https://spaghetti-code.tistory.com/35))*



위의 예시가 **컴파일 타임**에 구현 클래스가 결정되는 경우이고

**런타임**에 결정되는 경우를 **슈도코드**를 통해서 알아보자.

```java
public class MovieLister {

    private MovieFinder finder;

    public MovieLister() {
        this.finder = OOO.알려줘finder();
    }
}
```

위의 코드는 MovieLister가 생성되는 시점에 `000의 알려줘finder메서드`를 통해 어떤 구현 클래스로 결정되는지 알 수 있다.



### DI

지금까지의 내용을 보면 객체의 다른 것에 대한 제어가 아닌 객체의 의존성 부분의 제어만 언급하고 있는 것을 느낄 것이다.

그렇기에 마틴 파울러는 IoC라는 범용적인 용어는 사람들이 혼동할 가능성이 크기에 DI라는 이름을 만들었다고 한다.

즉, DI는 IoC중 하나라 생각할 수 있다. *(`DI ( IOC`, **DI 말고 Service Locator와 같은 개념도 있다**.)*



예제를 다시 살펴보자.

```java
public class MovieLister {

    private MovieFinder finder;

    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
}

@Getter
public class Assembler {

    private MovieLister movieLister;
    private MovieFinder movieFinder;

    public Assembler() {
        movieFinder = new MovieFinderImpl();
        movieLister = new MovieLister(movieFinder);
    }
}
```

MovieLister는 생성자를 통해서 MovieFinder의 구현체를 받을 준비만 하고 있다.

그 구현체는 Assembler에서 생성되고 MovieLister의 생성자를 통해 의존성 주입된다. (`movieLister = new MovieLister(movieFinder)`)

<img width="752" alt="스크린샷 2023-03-29 오후 6 16 32" src="https://user-images.githubusercontent.com/102807742/228487241-e598e1aa-9b88-4d48-a552-e59bc6560c9c.png">

위의 DI를 적용하지 않은 경우와 달리 MovieLister가 MovieFinder의 구현체를 알지 못한다는 것을 알 수 있다.

*DIP도 만족! 더 중요한 모듈이 덜 중요한 모듈에 의존하지 않을 수 있게 되었다.*



예제의 경우 Assembler라는 클래스를 통해 DI를 수행하였는데 스프링에서는 Assembler의 역할을 하는 컨테이너가 존재한다.

그렇기에 우리가 편하게 스프링을 통해서 개발을 할 수 있었던 것이다.



추가로 위의 예제는 생성자 방식으로 DI를 진행하였는데 수정자를 통해 DI를 진행할 수도 있다고 한다.



#### DI의 장점

*(출처 :  https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)*

이러한 DI의 장점은 다음과 같다.

1. **의존성이 줄어든다.**
1. **재사용성이 높은 코드가 된다.**
1. **테스트하기 좋은 코드가 된다.** 
1. **가독성이 높아진다.**