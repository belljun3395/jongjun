### CompletableFuture: 안정적 비동기 프로그래밍

### Future의 단순 활용 

자바 5부터는 **미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공**한다.

비동기 계산을 모델링하는 데 Future를 이용할 수 있으며, Future는 계산이 끝났을 때 결과에 접근할 수 있는 참조를 제공한다.

**시간이 걸릴 수 있는 작업을 Future 내부로 설정**하면 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있다.



Future는 저수준의 스레드에 비해 직관적으로 이해하기 쉽다는 장점이 있다.

**Future를 이용하려면 시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼 다음에 ExecutorService에 제출해야 한다.**



```java
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double>() {
  public Double call() {
    return doSomeLongComputation();
  }
});
doSomethingElse();
try {
  Double result = future.get(1, TimeUnit.SECONDS);
} catch (Exception e) {
  //
}
```

위의 코드는 자바 8 이전의 예제 코드이다.

ExecutorService에서 제공하는 스레드가 시간이 오래 걸리는 작업을 처리하는 동안 다른 스레드로 다른 작업을 동시에 실행할 수 있다.

다른 작업을 처리하다 **시간이 오래 걸리는 작업의 결과가 필요한 시점이 되었을 때** Future의 `get` 메서드로 결과를 가져올 수 있다.

`get` 메서드를 호출했을 때 이미 계산이 완료되어 **결과가 준비되었다면 즉시 결과를 반환**하지만 결과가 **준비되지 않았다면 작업이 완료될 때까지 스레드를 블록시킨다.**

그렇기에 오래 걸리는 작업이 영원히 끝나지 않으면 문제가 된다.

이러한 경우 때문에 `get` 메서드를 오버로드해서 스레드가 대기할 최대 타임아웃 시간을 설정하는 것이 좋다.



#### CompletableFuture 클래스

CompletableFuture 클래스는 아래의 기능을 선언형으로 이용할 수 있도록 자바 8에서 새로 제공하는 클래스이다.

+ **두 개의 비동기 계산 결과를 하나로 합친다.**
+ Future 집합이 실행되는 **모든 태스크의 완료를 기다린다.**
+ Future 집합에서 **가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다.**
+ 프로그램적으로 Future를 완료시킨다.
+ Future 완료 동작에 반응한다.



### 비동기 API 구현

#### 동기 메서드를 비동기 메서드로 변환

자바는 비동기 계산의 결과를 표현할 수 있는 **Future 인터페이스**를 제공한다.

Future는 **결과값의 핸들**일 뿐이며 계산이 완료되면 **`get` 메서드로 결과**를 얻을 수 있다.

비동기 메서드는 즉시 반환되므로 호출자 스레드는 다른 작업을 수행할 수 있다.



이때  비동기 계산과 완료 결과를 포함하는 **CompletableFuture 클래스의 인스턴스**를 사용하면 ExecutorService에 시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼 다음 건내지 않아도 되어 비동기 메서드 구현을 쉽게 할 수 있도록 도와준다.

CompletableFuture 인스턴스는 Future의 구현체로 다른 스레드에서 비동기적으로 계산을 수행하고 오랜 시간이 걸리는 계산이 완료되면 Future에 값을 설정한다.

이를 활용한 예시 코드는 아래와 같다.

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  
  new Thread(() -> {
    double price = calculatePrice(product);
    futurePrice.complet(price);
  }).start();

  return futurePrice;
}
```

이렇게 반환된 Future 역시 나중에 `get` 메서드를 호출하여 결과값을 요청받는다.

이때 Future가 결과값을 가지고 있다면 Future에 포함된 값을 읽거나 아니면 값이 계산될 때까지 블록한다.



#### 비동기 메서드 에러 처리 방법

**예외가 발생한다면 해당 스레드에만 영향을 미친다.**

즉, 에러가 발생해도 비동기적으로 메서드를 구현하였다면 이를 처리하기는 쉽지 않다.

결과적으로 클라이언트가 `get` 메서드가 반환될 때까지 영원히 기다리게 될 수도 있다.

그렇기에 클라이언트는 타임아웃값을 받는 `get` 메서드의 오버로드 버전을 만들어 이 문제를 해결해야 한다.

하지만 타임아웃으로는 왜 에러가 발생했는지 알 수 있는 방법이 없어 **`completeExceptionally` 메서드를 이용해 CompletableFuture 내부에서 발생한 예외를 클라이언트에 전달한다.**



위의 코드를 수정하여 에러 처리를 한다면 아래와 같이 처리할 수 있다.

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  
  new Thread(() -> {
    try {
	    double price = calculatePrice(product);
  	  futurePrice.complet(price);      
    } catch (Exception e) {
      futurePrice.completeExceptionally(e);
    }

  }).start();

  return futurePrice;
}
```



##### 팩토리 메서드 supplyAsync로 CompletableFuture 만들기

CompletableFuture는 `supplyAsync` 메서드를 제공하여 CompletableFuture를 간단하게 만들 수 있도록 도와준다.

`supplyAsync` 메서드를 사용하면 아래처럼 한 행으로 CompletableFuture를 만들 수 있다.

```java
public Future<Double> getPriceAsync(String product) {
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

이는 ForkJoinPool의 Executor 중 하나가 Supplier를 실행한 것이다.

그렇기에 두 번째 인수를 받는 `supplyAsync` 메서드를 오보로드해 다른 Executor를 지정할 수 있다.



### 비블록 코드 만들기

#### CompletableFuture로 비동기 호출 구현하기

```java
List<CompletableFuture<String>> priceFutures =
  shops.stream()
  .map(shop -> CompletableFuture.supplyAsync(
  	() -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
  .collect(toList());
```

위의 코드를 보면 CompletableFuture를 포함하는 리스트 `List<CompletableFuture<String>>`를 얻는다.

리스트의 CompletableFuture는 각각 계산 결과가 끝난 상점의 이름 문자열을 포함한다.

하지만 우리가 원하는 반환 형식이 `List<String>`이라면 모든 CompletableFuture 동작이 완료되고 결과를 추출한 다음에 리스트를 반환해야 한다.



두 번째 map 연산을 `List<CompletableFuture<String>>`에 적용할 수 있다.

즉, 리스트의 모든 CompletableFuture에 `join`을 호출해서 모든 동작이 끝나기를 기다린다.

**CompletableFuture 클래스의 `join` 메서드는 Future 인터페이스의 `get` 메서드와 같은 의미를 갖는다.**

**다만 `join`은 아무런 예외도 발생시키지 않는다는 점이 다르다.**



그래서 위의 코드를 `List<String>`를 반환하도록 수정하면 아래와 같다.

```java
List<CompletableFuture<String>> priceFutures =
  shops.stream()
  .map(shop -> CompletableFuture.supplyAsync(
  	() -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
  .collect(toList());

return priceFutures.stream()
  									.map(CompletableFuture::join)
  									.collect(toList());
```

위의 코드를 보면 두 개의 스트림 파이프라인으로 스트림을 처리하였다.

이는 스트림 연산이 게으른 특성이 있으므로 하나의 파이프라인으로 연산을 처리했다면 모든 가격 정보 요청 동작이 동기적, 순차적으로 이루어지는 결과가 되기 때문이다.



---

**하나의 파이프라인으로 연산을 처리한 경우)**

<img width="721" alt="스크린샷 2023-03-01 오후 2 07 04" src="https://user-images.githubusercontent.com/102807742/222050464-077c92d4-4808-4f9d-a652-6959f87862f1.png">

**두 개의 파이프라이능로 연산을 처리한 경우)**

<img width="814" alt="스크린샷 2023-03-01 오후 2 08 53" src="https://user-images.githubusercontent.com/102807742/222050565-3710aa9b-c634-4ae6-893a-8d61f70c57dc.png">

---



#### 커스텀 Executor 사용하기

CompletableFuture는 병렬 스트림에 비해 작업에 이용할 수 있는 다양한 Executor을 지정할 수 있다.

즉 Executor로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 만들 수 있다.



우선 Executor을 만들어 보자.

```java
private final Executor executor = 
  Executors.newFixedThreadPool(100, new ThreadFacotry() {
    public Thread newThread(Runnable r) {
      Thread t = new Thread(r);
      t.setDaemon(true);
      return t;
    }
  })
```

위의 풀은 데몬 스레드를 포함한다.

자바에서 일반 스레드가 실행 중이면 자바 프로그램은 종료되지 않는다.

따라서 어떤 이벤트를 한없이 기다리면서 종료되지 않는 일반 스레드가 있으면 문제가 될 수 있다.

반면 데몬 스레드는 자바 프로그램이 종료될 때 강제로 실행이 종료될 수 있다.

두 스레드의 성능은 동일하다.



이렇게 만든 Executor는 아래와 같이 전달하여 CompleteFuture를 만들 수 있다.

```java
CompletableFuture.supplyAsync(() -> calculatePrice(product), executor);
```

이렇게되면 설정한 스레드의 개수(100)까지는 같은 성능을 유지할 수 있다.



### 비동기 작업 파이프라인 만들기

#### 동기 작업과 비동기 작업 조합하기

우선 예제 코드를 확인하자.

```java
public List<String> findPrices(String product) {
  List<CoompletableFuture<String>> priceFutures = 
    shops.stream()
    			.map(shop -> CompletableFuture.supplyAsync(
          								() -> shop.getPrice(product), executor))
    			.map(future -> future.thenApply(Quote::parse)) // "비동기 요청 흐름 1"
    			.map(future -> future.thenCompose(quote -> // "비동기 요청 흐름 1"의 결과를 사용하는 "비동기 요청 흐림 2"
                                           CompletableFuture.supplyAsync( 
                                           () -> Discount.applyDiscount(quote), executor)) 
         .collect(toList());

		return priceFutures.stream()
               					.map(CompletableFuture::join)
               					.collect(toList());
}
```

`thenApply` 메서드는 CompletableFuture가 끝날 때까지 **블록하지 않는다.**

**즉,  CompletableFuture가 동작을 완전히 완료한 다음에 thenApply 메서드로 전달된 람다 표현식을 적용할 수 있다.**

따라서 `.map(future -> future.thenApply(Quote::parse))` 코드는 `CompletableFuture<String>`을 `CompletableFuture<Quote>`로 변환한다.

**이는 마치 CompletableFuture의 결과물로 무엇을 할지 지정하는 것과 같다.**



자바 8의 CompletableFuture API는 **두 비동기 연산을 파이프라인으로 만들 수 있도록 thenCompose 메서드를 제공한다.**

**thenCompose 메서드는 첫 번째 연산의 결과를 두 번째 연산으로 전달한다.**

즉, 첫 번째 CompletableFuture에 `thenCompose` 메서드를 호출하고 Function에 넘겨주는 식으로 두 CompletableFuture를 조합할 수 있다.

**Function은 첫 번째 CompletableFuture 반환 결과를 인수로 받고 두 번째 CompletableFuture를 반환한다.**

**이때 두 번째 CompletableFuture는 첫 번째 CompletableFuture의 결과를 계산의 입력으로 사용한다.**



#### 독립 CompletableFuture와 비독립 CompletableFuture 합치기

실전에서는 **독립적으로 실행된 두 개의 CompletableFuture 결과를 합쳐야 하는 상황**이 종종 발생한다고 한다.

이런 상황에서 `thenCombine` 메서드를 사용한다.

`thenCombine` 메서드는 Bifunction을 두 번째 인수로 받는다.

Bifunction은 두 개의 CompletableFuture 결과를 어떻게 합칠지 정의한다.

`thenCompose`와 마찬가지로 `thenCombine` 메서드에도 Async 버전이 존재한다.

**thenCombineAsync 메서드에서는 BiFunction이 정의하는 조합 동작이 스레드 풀로 제출되면서 별도의 태스크에서 비동기적으로 수행된다.**



이 또한 예시로 확인해 보면 아래와 같다.

```java
Future<Double> futurePriceInUSD =
  	CompletableFuture.supplyAsync(() -> shop.getPrice(product)) // price 
	  .thenCombine(
				CompletableFuture.supplyAsync(
        () -> exchangeService.getRate(Money.EUR, Money.USD)), // rate
      (price, rate) -> price * rate
));
```

price를 얻는 CompletableFuture와 rate를 얻는 CompletableFuture가 독립적으로 실행되고 `thenCombine` 메서드를 통해 합쳐지는 것을 볼 수 있다.



#### 타임아웃 효과적으로 사용하기

Future의 계산 결과를 읽을 때는 무한정 기다리는 상황이 발생할 수 있으므로 블록을 하지 않는 것이 좋다.

자바 9에서는 CompletableFuture에서 제공하는 몇 가지 기능을 이용해 이런 문제를 해결할 수 있다.

`orTimeout` 메서드는 지정된 시간이 지난 후에 CompletableFuture를 TimeoutException으로 완료하면서 또 다른 CompletableFuture를 반환할 수 있도록 내부적으로 ScheduledThreadExecutor를 활용한다.

이 메서드를 이용하면 계산 파이프라인을 연결하고 여기서 TimeoutException이 발생했을 때 사용자가 쉽게 이해할 수 있는 메시지를 제공할 수 있다.

이를 위의 예제 코드에 적용해보면 아래와 같다.

```java
Future<Double> futurePriceInUSD =
  	CompletableFuture.supplyAsync(() -> shop.getPrice(product)) 
	  .thenCombine(
				CompletableFuture.supplyAsync(
        () -> exchangeService.getRate(Money.EUR, Money.USD)),
      (price, rate) -> price * rate
))
  .orTimeout(3, TimeUnit.SECONDS);
```



### CompletableFuture의 종료에 대응하는 방법

```java
public Stream<CompletableFuture<String>> findPrices(String product) {
  List<CoompletableFuture<String>> priceFutures = 
    shops.stream()
    			.map(shop -> CompletableFuture.supplyAsync(
          								() -> shop.getPrice(product), executor))
    			.map(future -> future.thenApply(Quote::parse))
    			.map(future -> future.thenCompose(quote ->
                                           CompletableFuture.supplyAsync( 
                                           () -> Discount.applyDiscount(quote), executor));
}
```

위의 예제는 앞서 나온 예제를 조금 변형한 것이다.

`findPrices`의 반환 값에 `map` 연산자를 적용하여 CompletableFuture에 동작을 등록하자.

CompletableFuture에 등록된 동작은 CompletableFuture의 계산이 끝나면 값을 소비한다.

CompletableFuture API는 `thenAccept`라는 메서드로 이 기능을 제공한다.

**`thenAccept` 메서드는 연산 결과를 소비하는 Consumer를 인수로 받는다.**

```java 
findPrices("myPhone").map(f -> f.thenAccept(System.out::println));
```

**`thenAccept` 메서드는 CompletableFuture가 생성한 결과를 어떻게 소비할지 미리 지정했으므로 `CompletableFuture<Void>`를 반환한다.**



#### CompletableFuture.allOf

팩토리 메서드 `allOf`는 CompletableFuture 배열을 입력 받아 `CompletableFuture<Void>`를 반환한다.

전달된 모든 CompletableFuture가 완료되어야 `CompletableFuture<Void>`가 완료된다.

**따라서 `allOf` 메서드가 반환하는 CompletableFuture에 `join`을 호출하면 원래 스트림의 모든 CompletableFuture의 실행 완료를 기다릴 수 있다.**



#### CompletableFuture.anyOf

팩토리 메서드 `anyOf`는 CompletableFuture 배열을 입력으로 받아서 `CompletableFuture<Object>`를 반환한다.

**`CompletableFuture<Object>`는 처음으로 완료한 CompletableFuture의 값으로 동작을 완료한다.**