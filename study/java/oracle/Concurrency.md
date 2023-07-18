## Concurrency



### Processes and Thread

동시성 프로그래밍에는 프로세스와 스레드라는 두 가지 기본 실행 단위가 있다. 

Java 프로그래밍 언어에서 동시성 프로그래밍은 주로 스레드와 관련이 있다. 하지만 프로세스도 중요하다.



컴퓨터 시스템에는 일반적으로 많은 활성 프로세스와 스레드가 있다. 

이는 실행 코어가 하나만 있어 특정 순간에 실제로 실행되는 스레드가 하나뿐인 시스템에서도 마찬가지다. 

단일 코어의 처리 시간은 타임 슬라이싱이라는 OS 기능을 통해 프로세스와 스레드 간에 공유된다.



컴퓨터 시스템에 여러 프로세서 또는 여러 실행 코어를 가진 프로세서가 있는 것이 점점 더 보편화되고 있다.

이는 프로세스와 스레드의 동시 실행에 대한 시스템의 성능을 크게 향상시키지만, 여러 프로세서나 실행 코어가 없는 단순한 시스템에서도 동시 실행이 가능하다.



#### Processes

프로세스에는 독립된 실행 환경이 있다. 

프로세스에는 일반적으로 완전한 비공개 기본 런타임 리소스 세트가 있으며, 특히 각 프로세스에는 고유한 메모리 공간이 있다.

프로세스는 종종 프로그램 또는 애플리케이션과 동의어로 간주된다. 

그러나 사용자가 단일 애플리케이션으로 보는 것은 실제로는 협력하는 프로세스의 집합이다. 

프로세스 간의 통신을 용이하게 하기 위해 대부분의 운영 체제는 파이프 및 소켓과 같은 프로세스 간 통신(IPC) 리소스를 지원한다. 

IPC는 동일한 시스템의 프로세스 간 통신뿐만 아니라 다른 시스템의 프로세스 간 통신에도 사용된다.

대부분의 **Java 가상 머신 구현은 단일 프로세스로 실행된다.** 

Java 애플리케이션은 ProcessBuilder 객체를 사용하여 추가 프로세스를 생성할 수 있다. 



#### Threads

스레드를 경량 프로세스라고 부르기도 한다. 

프로세스와 스레드 모두 실행 환경을 제공하지만 새 스레드를 만드는 데는 새 프로세스를 만드는 것보다 더 적은 리소스가 필요하다.

스레드는 프로세스 내에 존재하며 모든 프로세스에는 하나 이상의 스레드가 있다. 

스레드는 메모리와 열린 파일 등 프로세스의 리소스를 공유한다. 

따라서 효율적이지만 잠재적으로 문제가 될 수 있는 통신이 가능하게 한다.

멀티스레드 실행은 Java 플랫폼의 필수 기능이다. 

모든 애플리케이션에는 하나 이상의 스레드가 있으며, 메모리 관리 및 신호 처리와 같은 작업을 수행하는 "시스템" 스레드까지 포함하면 여러 개의 스레드가 있다. 

하지만 애플리케이션 프로그래머의 입장에서는 메인 스레드라고 하는 하나의 스레드로 시작한다. 



### Thread Objects

각 스레드는 Thread 클래스의 인스턴스와 연결된다. 

Thread 객체를 사용하여 동시 애플리케이션을 생성하는 데는 두 가지 기본 전략이 있다.

+ 스레드 생성 및 관리를 직접 제어하려면 애플리케이션에서 **비동기 작업을 시작해야 할 때마다 스레드를 인스턴스화**하기만 하면 된다.
+ 나머지 애플리케이션에서 스레드 관리를 추상화하려면 **애플리케이션의 작업을 실행자(executor)에게 전달**하면 된다.



#### Defining and Starting a Thread

Thread 인스턴스를 생성하는 애플리케이션은 해당 스레드에서 실행할 코드를 제공해야 한다. 

이를 수행하는 방법에는 두 가지가 있다.

+ **Runnable 객체를 제공**
  Runnable 인터페이스는 스레드에서 실행되는 코드를 포함하기 위한 단일 메서드인 run을 정의한다. 

```java
public class HelloRunnable implements Runnable {
    public void run() {
        System.out.println("Hello from " + Thread.currentThread().getName() + "!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }
}
```

```
Hello from Thread-0!
```



+ **Thread 클래스 상속**
  Thread 클래스 자체는 Runnable을 구현하지만, run 메서드는 아무 것도 하지 않는다. 

```java
public class HelloThread extends Thread {
    
    @Override
    public void run() {
        System.out.println("Hello from " + Thread.currentThread().getName() + "!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }
}
```

```
Hello from Thread-0!
```



두 방법 모두 새로운 스레드를 시작하기위해 start 메서드를 호출하는 것을 확인할 수 있다.

그렇다면 위의 두 방법중 어떤 것을 사용해야 할까? 

첫 번째 방법는 Runnable 객체를 사용하는 것으로, Runnable 객체가 Thread가 아닌 다른 클래스를 서브클래싱할 수 있기 때문에 더 일반적이다. 

두 번째 방법은 간단한 애플리케이션에서 사용하기 더 쉽지만, 작업 클래스가 Thread의 자손이어야 한다는 점에서 제한이 있다. 

이 문서에서는 Runnalbe 태스크와 태스크를 실행하는 Thread 객체를 분리하는 첫 번째 접근 방식에 중점을 둔다. 

이 접근 방식은 더 유연할 뿐만 아니라 나중에 다룰 상위 수준의 스레드 관리 API에 적용할 수 있다.



Thread 클래스는 스레드 관리에 유용한 여러 메서드를 정의한다. 

여기에는 메서드를 호출하는 스레드에 대한 정보를 제공하거나 스레드의 상태에 영향을 주는 정적 메서드가 포함된다. 

다른 메서드들은 스레드 및 스레드 객체 관리와 관련된 다른 스레드에서 호출된다.



#### Pausing Execution with Sleep

Thread.sleep은 현재 스레드가 지정된 기간 동안 실행을 일시 중단하도록 한다. 

이는 애플리케이션의 다른 스레드나 컴퓨터 시스템에서 실행 중인 다른 애플리케이션이 프로세서 시간을 사용할 수 있도록 하는 효율적인 수단이다. 

sleep 메서드는 다음 예제와 같이 페이싱에 사용할 수도 있고, 뒷부분의 SimpleThreads 예제에서와 같이 시간 요구 사항이 있는 것으로 이해되는 다른 스레드를 기다리는 데 사용할 수도 있다.

수면 시간을 밀리초 단위로 지정하는 버전과 나노초 단위로 지정하는 두 가지 오버로드된 수면 버전이 제공된다. 

그러나 이러한 수면 시간은 **기본 OS에서 제공하는 기능에 의해 제한되기 때문에 정확성을 보장할 수 없다.**

또한 나중에 살펴볼 것처럼 인터럽트에 의해 **수면 기간이 종료될 수도 있다.** 

어떤 경우든 수면 모드를 호출하면 지정된 시간 동안 스레드가 정확히 일시 중단된다고 가정할 수 없다.



#### Interrupts

인터럽트는 스레드가 **수행 중인 작업을 중단하고 다른 작업을 수행해야 한다는 표시다.** 

스레드가 인터럽트에 정확히 어떻게 반응할지는 프로그래머가 결정해야 하지만, 스레드가 종료되는 것은 매우 일반적이다. 

**스레드는 중단할 스레드에 대해 스레드 객체에서 인터럽트를 호출하여 인터럽트를 보낸다.** 

**인터럽트 메커니즘이 올바르게 작동하려면 인터럽트된 스레드가 자체 인터럽트를 지원해야 한다.**



#### Supporting Interruption

스레드는 어떻게 interruption을 지원할까?

스레드가 현재 수행 중인 작업에 따라 다르다. 

**스레드가 InterruptedException을 던지는 메서드를 자주 호출하는 경우, 해당 예외를 잡은 후 실행 메서드에서 반환하면 된다.** 

sleep과 같이 InterruptedException을 던지는 많은 메서드는 현재 작업을 취소하고 인터럽트가 수신되면 즉시 반환하도록 설계되있다.



**스레드가 인터럽트 예외를 던지는 메서드를 호출하지 않고 오랫동안 진행되면 어떻게 될까?** 

그러면 인터럽트가 수신되면 참을 반환하는 **Thread.interrupted를 주기적으로 호출해야 한다.** 

예를 들어 아래와 같이 interrupted 메서드를 활용하여 주기적으로 인터럽트가 수신되는지 확인하는 것이다.

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```



#### The Interrupt Status Flag

인터럽트 메커니즘은 인터럽트 상태라는 내부 플래그를 사용하여 구현된다. 

Thread.interrupt를 호출하면 이 플래그가 설정된다. 

스레드가 **정적 메서드 Thread.interrupted를 호출하여 인터럽트를 확인하면 인터럽트 상태가 지운다.** 

```java
public static boolean interrupted() {
    Thread t = currentThread();
    boolean interrupted = t.interrupted;
    if (interrupted) {
        t.interrupted = false; // false로 지운다.
        clearInterruptEvent();
    }
    return interrupted;
}
```



한 스레드가 다른 스레드의 인터럽트 상태를 쿼리하는 데 사용되는 **비정적 isInterrupted 메서드는 인터럽트 상태 플래그를 변경하지 않는다.**

```java
public boolean isInterrupted() {
    return interrupted; // 지우지 않고 그대로 반환한다.
}
```



**관례에 따라 InterruptedException을 던져 종료하는 모든 메서드는 종료할 때 인터럽트 상태를 지워야 한다.** 

그러나 인터럽트를 호출하는 다른 스레드에 의해 인터럽트 상태가 즉시 다시 설정될 가능성은 항상 존재한다.



#### Joins

join 메서드를 사용하면 한 스레드가 **다른 스레드가 완료될 때까지 기다릴 수 있다.** 

```java
t.join();
```

위의 코드의 경우 **t의 스레드가 종료될 때까지 현제 스레드의 실행을 일시 중지하는 것이다.**



join을 오버로드하여 사용하면 프로그래머가 대기 시간을 지정할 수 있다.

하지만 sleep과 같이 join도 타이밍은 OS에 따라 달리지고 join이 지정한 시간만큼 정확히 대기할 것이라 가정해서는 안된다.

그리고 sleep과 마찬가지로 join은 인터럽트에 대해 InterruptedException으로 종료하여 응답한다.



### Synchronization

스레드는 주로 필드와 참조 필드가 참조하는 객체에 대한 액세스를 공유하여 통신한다. 

이러한 형태의 통신은 매우 효율적이지만 **스레드 간섭과 메모리 일관성 오류라는 두 가지 종류의 오류가 발생할 수 있다.** 

이러한 오류를 방지하는 데 필요한 것은 동기화이다.



그러나 동기화는 둘 이상의 스레드가 동일한 리소스에 동시에 액세스하려고 시도할 때 발생하는 스레드 경합을 유발하여 Java 런타임이 하나 이상의 스레드를 더 느리게 실행하거나 심지어 실행을 일시 중단하게 만들 수 있다. 

스타베이션(starvation, 기아)과 라이브락(livelock)은 스레드 경합의 한 형태다.



#### Thread Interference

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }
}
```

위의 Counter 클래스를 여러 스레드에서 참조하는 경우 스레드 간 간섭으로 인해 예상대로 작동하지 않을 수 있다.

**간섭은 서로 다른 스레드에서 실행되지만 동일한 데이터에 대해 작업하는 두 작업이 서로 겹칠 때 발생한다**.

**즉, 두 작업이 여러 단계로 구성되어 있고 단계의 순서가 겹친다는 의미다.**



Counter의 인스턴스에 대한 연산은 모두 단일에 간단한 상태이기에 인터리빙(interleave)가 불가능해 볼일 수 있다.

그러나 단순한 상태이더라도 가상 머신에 의해 여러 단계로 변환될 수 잇다.

여기서는 가상 머신이 수행하는 구체적인 단계를 살펴보지 않으므로 `c++`이 3단계로 분해될 수 있다는 것만 알면 충분하다.

1. c의 현재 값을 검색
2. 검색된 값을 1씩 증가
3. 증가된 값을 다시 c에 저장



스레드 A가 증가(increment)를 호출하는 동시에 스레드 B가 감소(decrement)를 호출한다고 가정해봅시다.

c의 초기 값이 0이면 인터리브된 작업은 아래의 순서를 따른다.

1. 스레드 A: c를 검색
2. 스레드 B: c를 검색
3. 스레드 A: 검색된 값 증가, 결과는 1
4. 스레드 B: 검색된 값 감소, 결과는 -1
5. 스레드 A: 결과를 c에 저장, 이제 c는 1
6. 스레드 B: 결과를 c에 저장, 이제 c는 -1

스레드 A의 결과가 손실되고 스레드 B가 덮어쓴다. 

이 특정 인터리빙은 한 가지 가능성일 뿐이다. 

다른 상황에서는 스레드 B의 결과가 손실될 수도 있고 오류가 전혀 없을 수도 있다. 

예측할 수 없기 때문에 스레드 간섭 버그를 감지하고 수정하기가 어려울 수 있다.



#### Memory Consistency Errors

메모리 일관성 오류는 서로 다른 스레드가 동일한 데이터에 대해 일관되지 않은 결과를 가질 때 발생한다.



메모리 일관성 오류를 피하기 위한 핵심은 선처리(happens-before) 관계를 이해하는 것이다.

이 관계는 간단하게 말하면, 하나의 표현식에 의한 메모리 쓰기를 다른 연산이 감지할 수 있게 보장하기 위함이다.

이를 이해하기 위해 예제를 살펴보자.

```
int counter = 0;
```

counter 필드는 두 스레드인 A와 B가 공유하고 스레드 A가 counter를 증가시킨다고 가정하자.

그런 다음 얼마 지나지 않아 스레드 B가 counter를 출력한다면 어떻게 될까?



만약 증가와 출력 두 표현식이 동일한 스레드에서 실행되었다면 출력되는 값이 "1"이라 확신할 수 있을 것이다.

하지만 두 표현식이 별도의 스레드에서 실행되는 경우, 프로그래머가 두 표현식 사이에 발생 전 관계를 설정하지 않는 한 스레드 A의 증가 표현식에 의한 counter의 변경을 스레드 B에 출력 표현식이 감지할 수 있을 보장이 없다.

따라서 값이 "1"일 수도 "0"일 수도 있다.



선처리 관계를 형성하기 위한 몇 가지 방법들이 있다. 

그 중에 하나가 다음 섹션에서 살펴볼 게 될 동기화(synchronization)다.

그리고 우리는 이미 선처리 관계를 형성하는 2가지 방법을 살펴보았다.

+ 표현식이 Thread.start 메서드를 호출하였을 때, 해당 표현식과 선처리 관계를 가진 모든 표현식은 새로운 스레드에 의해서 수행되는 모든 표현식과 선처리 관계를 가진다.
+ 어떤 스레드가 종료되어 Thread.join 메서드가 다른 스레드에서 리턴하도록 만든다면, 종료된 스레드에 의해서 수행된 모든 표현식은 성공적은 join 메서드를 뒤따르는 모든 표현식과 선처리 관계를 가진다.
  스레드 내에서 코드의 효과는 join 메서드를 수행한 스레드에서 인지할 수 있게 된다.



#### Synchronized Methods

자바는 동기화된 메서드와 동기화된 표현식, 2가지 기본 동기화 구문을 제공한다.

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

위의 예시처럼 메서드 앞에 synchronized 키워드만 붙이면 동기화된 메서드가 된다.



메서드 동기화는 두 가지 효과를 가진다.

1. 동일 객체 상에서 동기화된 메서드의 2번의 호출이 서로 간섭할 수 없다.

하나의 스레드가 어떤 객체의 동기화된 메서드를 실행하고 있는 중일 때, 그 객체의 동기화된 메서드를 호출하려는 다른 모든 스레드들은 첫 번째 스레드가 해당 객체 상의 작업을 끝날 때까지 실행이 차단된다.

2. 동기화된 메서드는 이어서 발생하는 동일 객체의 동기화된 메서드의 호출과 자동으로 선처리 관계를 수립한다.

따라서 모든 스레드에서 객체의 상태변화를 인지할 수 있다.



동기화된 메서드는 스레드 간섭과 메모리 일관성 오류를 방지하기 위한 간단한 전략이다.

즉, 하나의 객체를 여러 스레드에서 접근할 때, 해당 객체의 변수를 대상으로 한 모든 읽기와 쓰기를 동기화된 메서드를 통해 처리한다.

이 전략은 효과적이지만 라이브니스(liveness)와 관련된 문제를 이르킬 수 있다.



#### Intrinsic Locks and Synchronization

동기화는 내제적(intrinsic) 락 또는 모니터(monitor) 락으로 알려진 내부 락킹 메커니즘에 의해 동작한다.

락은 동기화의 객체 상태에 대한 독접적인 접근을 강제하고 가시성에 핵심인 선처리 관계를 수립하는 두 가지 측면에서 모두 역할을 하고 있다.



모든 객체는 락을 가진다.

관행에 따르면, 어떤 객체의 필드에 대한 독점적이고 일관적인 접근권이 필요한 스레드는 필드에 접근하기 전에 해당 객체에 락을 걸어야 하며 필드에 이용한 작업이 끝나면 락을 풀어줘야 한다.

스레드는 락을 걸고 락을 풀어주는 시간동안 락을 소유하고 있는 것으로 알려져있다.

어떤 스레드가 락을 소유하고 있는 한, 다른 스레드는 같은 락을 획득할 수 없다.

다른 스레드가 그 락을 얻으려고 할 때 차단 당할 것이기 때문이다.

어떤 스레드가 락을 풀어줄 때, 그 락의 해제와 뒤따르는 그 락의 획득 간에는 선처리 관계가 만들어진다.



##### Locks In Synchronized Methods

스레드는 동기화된 메서드를 호출하면서 자동으로 해당 객체에 락을 걸고 해당 메서드가 리턴할 때 락을 풀어준다.

또한 잡히지 않은 예외가 발생했을 때도 해당 락은 해제된다.



정적 메서드는 객체가 아닌 클래스와 연관되어 있어 동기화된 정적 메서드가 호출될 때는 어떨까?

이 경우는 스레드는 해당 클래스 객체에 대해서 락을 건다.

그러므로 이 락은 정적 필드에 대한 접근을 통제하며 이것이 일반 객체에 대한 락과 다른 부분이다.



##### Synchronized Statements

동기화된 코드를 만들어내는 다른 방법은 동기화된 표현식이다.

동기화된 메서드와 다르게 동기화된 표현식은 반드시 락을 제공하는 객체에 명시해야 한다.

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

위의 예제에서 lastName과 nameCount에 대한 변경을 동기화해야 하지만 다른 객체에 대한 메서드 호출은 동기화하면 안된다.

동기화된 표현식이 없었더라면, nameList.add를 호출하기 위한 별동의 동기화 되지 않은 메서드가 있어야 했을 것이다.



또한 동기화된 표현식은 정교화 동기화로 동시성을 향상 시킬 때 유용한다.



##### Reentrant Synchronization

스레드는 다른 스레드가 소유하고 있는 락을 획득할 수 없다.

그러나 스레드는 자신이 이미 소유하고 있는 락은 획득할 수 있다.

스레드가 같은 락을 여러 번 획득할 수 있도록 허락하는 것은 재진입 동기화를 가능하게 한다.

이는 동기화된 코드가 직간접적으로 동기화된 코드를 포함하고 있는 또 다른 메서드를 호출하면서 이 두 코드가 모두 같은 락을 사용하고 있는 상황으로 설명된다.

재진입 동기화가 없었더라면 동기화된 코드는 자기 자신을 의해서 차단되는 상황을 피하기 위해서 많은 추가 조치가 필요했을 것이다.



#### Atomic Access

프로그래밍에서 원자적 행위는 한 번에 유효하게 일어나는 것을 뜻한다.

원자적 행위는 동중에 멈출 수 없다.

즉, 이것은 완전히 발생하거나 전혀 일어나지 않거나 둘 중에 하나다.

원자적 행위에서는 그 행위가 완료될 때 까지는 그 어떤 부작용도 발생하지 않는다.



우리는 이미 c++와 같은 증가 표현식이 원자적 행위가 아니라는 것을 보았다.

하지만 원자적이라 명시할 수 있는 행위들도 있다.

1. 모든 참조형 변수와 long과 double을 제외한 기본형 변수에 대한 읽기와 쓰기는 원자적이다.
2. volatile로 선언된 모든 변수에 대한 읽기와 쓰기는 long과 double까지 포함해서 원자적이다.



원자적 행위들은 간섭당할 수 없기 때문에 스레드 간섭의 두려움없이 사용될 수 있다. 

하지만 여전히 메모리 일관성 오류는 발생할 수 있기 때문에 이 사실이 원자적 행위를 동기화하고자 하는 모든 요구를 만족하지는 않는다.

 `volatile`한 변수에 대한 모든 쓰기는 뒤따르는 해당 변수에 대한 모든 읽기와 선처리 관계를 형성하기 때문에 `volatile`한 변수를 사용하면 메모리 일관성 문제가 발생하는 위험을 줄여준다. 

다시 말해, `volatile`한 변수에 대한 모든 변경은 다른 쓰레드에서 항상 인지할 수 있다. 

게다가, 쓰레드가 `volatile`한 변수를 읽을 때, 가장 최근의 변경 사항을 인지할 수 있을 뿐만 아니라 그러한 변경을 가져온 부작용도 탐지할 수 있다.



간단하게 원자적 변수에 대한 접근을 이용하는 것이 동기화된 코드를 통해서 접근하는 것 보다 더 효율적이다. 

그러나 메모리 일관성 문제를 피하기 위한 프로그래머의 더 많은 주의를 필요로 한다. 

그러한 추가적인 노력이 가치가 있는지는 애플리케이션의 규모와 복잡도에 달려있다.



### Liveness

#### Deadlock

데드락은 두 개이상으 스레드가 영원히 차단되어 서로를 기다리는 상황을 말한다.



#### Starvation and Livelock

기아와 라이브 락은 데드락 보다 훨씬 덜 흔한 문제지만, 동시 실행 소프트웨어의 모든 개발자가 직면할 수 있는 문제다.



##### Starvation

기아는 스레드가 공유 리소스에 정기적으로 접근할 수 없어 진행하지 못하는 상황을 말한다.

이는 그리디(greedy) 스레드가 공유 리소스를 오랫동안 사용할 수 없게 만들 때 발생한다.



##### Livelock

스레드는 종종 다르 스레드의 동작에 대한 응답으로 동작한다.

다른 스레드의 작업도 다른 스레드의 작업에 대한 응답인 경우 라이브 락이 발생할 수 있다.

교착 상태와 마찬가지로, 라이브 락에 걸린 스레드는 더 이상 진행할 수 없다.

그러나 스레드가 차단된 것이 아니라 서로 응답하느라 작업을 재개할 수 없을 정도로 바쁠 뿐이다.



### Guarded Blocks

스레드는 종종 동작을 조정해야 한다. 

가장 일반적인으로 가드 블록을 활용한다.

이러한 블록은 **블록이 진행되기 전에 참이어야 하는 조건을 폴링(Polling)하는 것**으로 시작된다.

*(아래 예시 코드의 `while (empty) { ... }`에 해당한다 생각한다.)*

이를 올바르게 수행하기 위해 따라야 할 여러 단계가 있다.



가드 블록을 사용해 생산자-소비자 애플리케이션을 만들어 보며 조금 더 알아보자. 

이러한 종류의 애플리케이션은 데이터를 생성하는 생산자(Producer)와 데이터를 사용하여 무언가를 수행하는 소비자(Consumer)라는 두 개의 스레드 간에 데이터를 공유한다. 

두 스레드는 공유 객체(Drop)를 사용하여 통신한다. 

소비자 스레드는 생산자 스레드가 데이터를 전달하기 전에 데이터를 검색하려고 시도해서는 안 되며, 생산자 스레드는 소비자가 이전 데이터를 검색하지 않은 경우 새 데이터를 전달하려고 시도해서는 안 된다.



+ **Drop 객체**

```java
public class Drop {

    /** Message sent from producer to consumer. */
    private String message;

    /**
     * True if consumer should wait 
     * for producer to send message,
     * false if producer should wait for
     * consumer to retrieve message. 
     */
    private boolean empty = true;

    public synchronized String take() {
        /** Wait until message is available. */
        while (empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
      
        /** Toggle status. */
        empty = true;

        /**  Notify producer that status has changed. */
        notifyAll();
      
        return message;
    }

    public synchronized void put(String message) {
        /** Wait until message has  been retrieved. */
        while (!empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
      
        /** Toggle status. */ 
        empty = false;
      
        /** Store message. */
        this.message = message;
      
        /** Notify consumer that status has changed. */
        notifyAll();
    }
}
```

+ **Producer 객체**

```java
public class Producer implements Runnable {
    private Drop drop;

    public Producer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };
        Random random = new Random();

        for (int i = 0;
            i < importantInfo.length;
            i++) {
            drop.put(importantInfo[i]);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
        drop.put("DONE");
    }
}
```

+ **Consumer 객체**

```java
public class Consumer implements Runnable {
    private Drop drop;

    public Consumer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        Random random = new Random();
        for (String message = drop.take();
            !message.equals("DONE");
            message = drop.take()) {
            System.out.format("MESSAGE RECEIVED: %s%n", message);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
    }
}
```



+ **실행 코드**

```java
Drop drop = new Drop();
(new Thread(new Producer(drop))).start();
(new Thread(new Consumer(drop))).start();
```



간단히 코드 실행 과정을 나타내 보면 아래와 같다.

1. Drop 객체가 생성된다.
2. Producer가 Drop에 put할 메시지를 전달하고 5초 내에서 램덤하게 대기한다.
3. 메시지를 받을 Drop은 empty인지 상태를 확인하고 이후 작업을 수행한다.
   1. empty라면 메시지를 저장하고 notifyAll을 통해 put이 완료됨을 알린다.
      *(이때 notifyAll을 통해 Consumer, 즉 Drop의 take의 wait가 해지되는 것 같다.)*
   2. empty가 아니라면 wait를 통해 대기한다.
4. Drop의 take가 wait가 해지되면서 저장된 메시지를 Consumer에게 반환해준다.
   이때 다시 notifyAll을 통해 take가 완료됨을 알린다.
5. Consumer은 take가 반환하는 값을 출력한다.
   그리고  5초 내에서 램덤하게 대기한다.
6. 이를 반복한다.



로그와 함께 다시 살펴보자.

```
[Thread-1] INFO Drop take is waiting on Thread-1
```

우선 Drop이 생성되며 take 메서드가 대기중에 있는다. **(1번)**

```
[Thread-0] INFO Producer is on Thread-0 and send [Mares eat oats]
[Thread-0] INFO Drop put is on Thread-0 and will put [Mares eat oats]
[Thread-0] INFO Drop put successfully notify and put [Mares eat oats]
```

Producer가 put할 메시지를 Drop에 전달하고 Drop은 메시지를 저장하고 notifyAll을 통해 전송이 완료됨을 알린다. **(2,3번)**

```
[Thread-1] INFO Drop take waiting is over Thread-1
```

notifyAll을 통해 take 메서드의 wait가 끝난다. **(3-1번)**

```
[Thread-1] INFO Drop take is on Thread-1 and take [Mares eat oats]
[Thread-1] INFO Drop take successfully notify and take [Mares eat oats]
[Thread-1] INFO Consumer is on Thread-1
[Thread-1] INFO MESSAGE RECEIVED: Mares eat oats
```

Drop이 take를 완료 하며 메시지 값을 반환하고 Consumer가 이를 출력한다. **(4,5번)**



**전체 로그**

```
[Thread-1] INFO Drop take is waiting on Thread-1
[Thread-0] INFO Producer is on Thread-0 and send [Mares eat oats]
[Thread-0] INFO Drop put is on Thread-0 and will put [Mares eat oats]
[Thread-0] INFO Drop put successfully notify and put [Mares eat oats]
[Thread-1] INFO Drop take waiting is over Thread-1
[Thread-1] INFO Drop take is on Thread-1 and take [Mares eat oats]
[Thread-1] INFO Drop take successfully notify and take [Mares eat oats]
[Thread-1] INFO Consumer is on Thread-1
[Thread-1] INFO MESSAGE RECEIVED: Mares eat oats
```



Producer와 Consumer에서 " 5초 내에서 램덤하게 대기한다"라는 행위 때문에

이전의 take가 끝나지 않은 상황에서 put이 실행될 수 있다.

이러한 경우 put은 `while (!empty) {...}` 안에서 wait하며 notifyAll에 의해 empty가 변경되기를 기다린다.

이는 Producer와 Consumer에 이들이 Sleeping 중인지 그리고 Wakeup 하였는지 알려주는 로그를 추가하고 로그를 살펴보자.



```
[Thread-0] INFO Producer is on Thread-0 and send [Does eat oats]
[Thread-0] INFO Drop put is on Thread-0 and will put [Does eat oats]
[Thread-0] INFO Drop put successfully notify and put [Does eat oats]
[Thread-0] INFO Producer is Sleeping
[Thread-1] INFO Consumer is Wakeup
[Thread-1] INFO Drop take is on Thread-1 and take [Does eat oats]
[Thread-1] INFO Drop take successfully notify and take [Does eat oats]
[Thread-1] INFO Consumer is on Thread-1
[Thread-1] INFO MESSAGE RECEIVED: Does eat oats
```

우선 정상적으로 "Does eat oats" 메시지를 받는 로그다.

```
[Thread-1] INFO Consumer is Sleeping // [Does eat oats] 이후 Consumer가 Sleeping 중 이다.
```

Consumer가 Sleeping 중이다.

```
[Thread-0] INFO Producer in is Wakeup
[Thread-0] INFO Producer is on Thread-0 and send [Little lambs eat ivy]
[Thread-0] INFO Drop put is on Thread-0 and will put [Little lambs eat ivy]
[Thread-0] INFO Drop put successfully notify and put [Little lambs eat ivy]
[Thread-0] INFO Producer is Sleeping
[Thread-0] INFO Producer is Wakeup
[Thread-0] INFO Producer is on Thread-0 and send [A kid will eat ivy too] 
```

Consumer가 Sleeping 중이지만 Producer와 Drop은 정상적으로 수행된다.

```
[Thread-0] INFO Drop put is waiting on Thread-0 and wait [A kid will eat ivy too] // Consumer가 아직 Sleeping 중이기에 wait 하고 있다.
```

하지만 Consumer가 Sleeping 중이기에 Drop의 put 과정은 대기하게 된다.

```
[Thread-1] INFO Consumer is Wakeup // Consumer가 일어나며 empty를 변경하였다.
```

Consumer가 Wakeup 된다.

```
[Thread-1] INFO Drop take is on Thread-1 and take [Little lambs eat ivy] // take가 변경을 감지하여 이후 메서드를 실행한다.
[Thread-1] INFO Drop take successfully notify and take [Little lambs eat ivy]
[Thread-1] INFO Consumer is on Thread-1
[Thread-1] INFO MESSAGE RECEIVED: Little lambs eat ivy
```



### Immutable Objects

객체가 생성된 후 상태가 변경되지 않으면 불변 객체로 간주한다. 

불변 객체에 최대한 의존하는 것은 간단하고 안정적인 코드를 작성하기 위한 올바른 전략으로 널리 받아들여지고 있다.

불변 객체는 동시 실행 애플리케이션에서 특히 유용하다. 

상태를 변경할 수 없으므로 스레드 간섭으로 인해 손상되거나 일관되지 않은 상태로 관찰될 수 없다.

프로그래머는 기존 객체를 업데이트하는 대신 새 객체를 생성하는 데 드는 비용에 대해 걱정하기 때문에 불변 객체를 사용하는 것을 꺼리는 경우가 많다. 

객체 생성의 영향은 종종 과대평가되는 경우가 많으며, 불변 객체와 관련된 일부 효율성으로 상쇄될 수 있다. 

가비지 컬렉션으로 인한 오버헤드 감소, 변경 가능한 객체를 손상으로부터 보호하는 데 필요한 코드 제거 등이 이에 해당한다.

다음 하위 섹션에서는 인스턴스가 변경 가능한 클래스를 가지고 이 클래스에서 변경 불가능한 인스턴스를 가진 클래스를 파생한다. 

이를 통해 이러한 종류의 변환에 대한 일반적인 규칙을 제시하고 불변 객체의 몇 가지 장점을 설명한다.



#### A Strategy for Defining Immutable Objects

다음 규칙은 불변 객체를 생성하기 위한 간단한 전략을 정의한다. 

"불변"으로 문서화된 모든 클래스가 이 규칙을 따르는 것은 아나다. 

그렇다고 해서 이러한 클래스를 만든 사람이 엉성하게 만들었다는 의미는 아니다. 

클래스의 인스턴스가 생성된 후에는 절대 변경되지 않는다고 믿는 데에는 타당한 이유가 있을 수 있다. 

그러나 이러한 전략은 정교한 분석이 필요하다.



1. 필드에서 참조하는 필드 또는 객체를 수정하는 메서드인 "setter" 메서드를 제공하지 않는다.
2. 모든 필드를 final 그리고 private 하게 만들어라.
3. 서브클래스가 메서드를 재정의할 수 없도록 하라. 
   이를 수행하는 가장 간단한 방법은 클래스를 final 클래스로 선언하는 것이다. 
   보다 정교한 접근 방식은 생성자를 비공개로 만들고 팩토리 메서드에서 인스턴스를 생성하는 것이다.
4. 인스턴스 필드에 변경 가능한 객체에 대한 참조가 포함된 경우 해당 객체를 변경할 수 없도록 한다.
   1. 변경 가능한 객체를 수정하는 메서드를 제공하지 않느다.
   2. 변경 가능한 객체에 대한 참조를 공유하지 않는다. 
      생성자에게 전달된 외부의 변경 가능한 객체에 대한 참조를 저장하지 말고, 필요한 경우 복사본을 생성하고 복사본에 대한 참조를 저장한다. 
      마찬가지로, 메서드에서 원본을 반환하지 않도록 필요한 경우 내부 변경 가능 객체의 복사본을 생성한다.



### High Level Concurrency Objects

위에서 알아보 았던 저수준 API는 매우 기본적인 작업에는 적합하지만 고급 작업에는 더 높은 수준의 API가 필요하다.

특히 오늘날의 멀티프로세서 및 멀티코어 시스템을 완전히 활용하는 대규모 동시 애플리케이션의 경우 더욱 그렇다.

Java 플랫폼 버전 5.0에 도입된 몇 가지 높은 수준의 동시성 기능은 대부분 새로운 java.util.concurrent 패키지에 구현되어 있다. 

Java 컬렉션 프레임워크에는 새로운 동시 데이터 구조도 있다.



#### Lock Objects

동기화된 코드는 간단한 종류의 재진입 잠금에 의존한다. 

이러한 종류의 잠금은 사용하기 쉽지만 많은 제한이 있다. 

보다 정교한 잠금 관용구는 java.util.concurrent.locks 패키지에서 지원된다. 

가장 기본적인 인터페이스인 Lock에 초점을 맞추어보자.



Lock 객체는 동기화된 코드에서 사용하는 암시적 잠금과 매우 유사하게 작동한다. 

암시적 잠금과 마찬가지로, 한 번에 하나의 스레드만 Lock 객체를 소유할 수 있다. 

Lock 객체는 연결된 Condition 객체를 통해 wait/notify 메커니즘도 지원한다.



암시적 잠금에 비해 Lock 객체의 가장 큰 장점은 잠금을 획득하려는 시도를 백아웃할 수 있다는 점이다. 

tryLock 메서드는 잠금을 즉시 사용할 수 없거나 타임아웃이 만료되기 전에(지정된 경우) 백아웃한다. 

lockInterruptibly 메서드는 잠금이 획득되기 전에 다른 스레드가 인터럽트를 보내면 백아웃된다.



#### Executors

앞의 모든 예제에서 새 스레드가 수행하는 작업(Runnable 객체로 정의됨)과 스레드 객체로 정의된 스레드 자체 사이에는 밀접한 연관성이 있었다. 

이는 소규모 애플리케이션에서는 잘 작동하지만 대규모 애플리케이션에서는 스레드 관리 및 생성을 애플리케이션의 나머지 부분과 분리하는 것이 좋다. 

이러한 기능을 캡슐화하는 객체를 Executor라고 한다.



#### Executor Interfaces

java.util.concurrent 패키지는 세 가지 Executor 인터페이스를 정의한다.

+ Executor: 새 작업 시작을 지원하는 간단한 인터페이스이다.
+ ExecutorService: 개별 작업과 executor 자체의 수명 주기를 관리하는 데 도움이 되는 기능을 추가하는 Executor의 하위 인터페이스이다.
+ ScheduledExecutorService: Executor의 하위 인터페이스로, 작업의 향후 및/또는 주기적 실행을 지원한다.

일반적으로 executor 객체를 참조하는 변수는 Executor 클래스 유형이 아닌 이 세 가지 인터페이스 유형 중 하나로 선언된다.



##### The `Executor` Interface

Executor 인터페이스는 일반적인 스레드 생성 관용구를 드롭인 방식으로 대체할 수 있도록 설계된 단일 메서드인 execute를 제공한다. 

r이 실행 가능한 객체이고 e가 실행자 객체인 경우 다음과 같이 대체할 수 있다.

```java
(new Thread(r)).start();
 with
e.execute(r);
```



그러나 execute의 정의는 덜 구체적이다. 

저수준 관용구는 새 스레드를 생성하고 즉시 실행한다. 

실행자 구현에 따라 실행은 동일한 작업을 수행할 수 있지만 기존 작업자 스레드를 사용하여 r을 실행하거나 작업자 스레드를 사용할 수 있게 될 때까지 대기열에 r을 배치할 가능성이 더 높다. 



java.util.concurrent의 executor 구현은 기본 Executor 인터페이스와 함께 작동하지만 보다 고급 ExecutorService 및 ScheduledExecutorService 인터페이스를 최대한 활용하도록 설계되었다.



##### The ExecutorService Interface

ExecutorService 인터페이스는 비슷하지만 더 다재다능한 submit 메서드로 execute를 보완한다.

실행과 마찬가지로 submit은 실행 가능한 객체를 허용하지만, 작업에서 값을 반환할 수 있는 Callable 객체도 허용한다. 

submit 메서드는 Callable 반환 값을 검색하고 Callable 및 실행 가능 작업의 상태를 관리하는 데 사용되는 Future 객체를 반환한다.



ExecutorService는 또한 Callable 객체의 대규모 컬렉션을 제출하는 메서드도 제공한다. 

마지막으로 ExecutorService는 executor의 종료를 관리하기 위한 여러 메서드를 제공한다. 

즉각적인 종료를 지원하려면 작업이 인터럽트를 올바르게 처리해야 한다.



##### The ScheduledExecutorService Interface

ScheduledExecutorService 인터페이스는 지정된 지연 후에 Runnable 또는 Callable 작업을 실행하는 schedule로 부모 ExecutorService의 메서드를 보완한다. 

또한 이 인터페이스는 지정된 작업을 정의된 간격으로 반복적으로 실행하는 scheduleAtFixedRate 및 scheduleWithFixedDelay를 정의한다.



#### Thread Pools

java.util.concurrent의 대부분의 executor 구현은 worker 스레드로 구성된 스레드 풀을 사용한다. 

이러한 종류의 스레드는 실행하는 Callable 및 Runnable 작업과는 별도로 존재하며 여러 작업을 실행하는 데 자주 사용된다.



worker 스레드를 사용하면 스레드 생성으로 인한 오버헤드를 최소화할 수 있다. 

스레드 개체는 상당한 양의 메모리를 사용하므로 대규모 애플리케이션에서 많은 스레드 개체를 할당하고 할당 해제하면 상당한 메모리 관리 오버헤드가 발생한다.



스레드 풀의 일반적인 유형 중 하나는 고정 스레드 풀이다. 

이 유형의 풀에는 항상 지정된 수의 스레드가 실행 중이며, 사용 중인 스레드가 종료되면 자동으로 새 스레드로 대체된다. 

작업은 내부 대기열을 통해 풀에 제출되며, 대기열은 스레드보다 활성 작업이 더 많을 때마다 추가 작업을 보유한다.



고정 스레드 풀의 중요한 장점은 이를 사용하는 애플리케이션의 성능이 점진적으로 저하된다는 점이다. 

이를 이해하기 위해 각 HTTP 요청이 별도의 스레드에서 처리되는 웹 서버 애플리케이션을 생각해 보자. 

애플리케이션이 모든 새로운 HTTP 요청에 대해 새 스레드를 생성하고 시스템이 즉시 처리할 수 있는 것보다 많은 요청을 수신하면 모든 스레드의 오버헤드가 시스템 용량을 초과하면 애플리케이션이 갑자기 모든 요청에 대한 응답을 중지한다. 

생성할 수 있는 스레드 수에 제한이 있으면 애플리케이션은 HTTP 요청이 들어오는 즉시 처리하지는 않지만 시스템이 견딜 수 있는 한도 내에서 요청을 처리한다.



고정 스레드 풀을 사용하는 executor을 생성하는 간단한 방법은 java.util.concurrent.Executors에서 newFixedThreadPool 팩토리 메서드를 호출하는 것dl다. 

이 클래스는 다음과 같은 팩토리 메서드도 제공한다.

+ newCachedThreadPool 메서드는 확장 가능한 스레드 풀이 있는 실행기를 생성한다. 
  이 실행기는 수명이 짧은 작업을 많이 실행하는 애플리케이션에 적합하다.
+ newSingleThreadExecutor 메서드는 한 번에 단일 작업을 실행하는 실행기를 만든다.
+ 여러 팩토리 메서드가 위 실행기의 ScheduledExecutorService 버전이다.



위의 팩토리 메서드에서 제공하는 실행기가 사용자의 요구 사항을 충족하지 않는 경우, java.util.concurrent.ThreadPoolExecutor 또는 java.util.concurrent.ScheduledThreadPoolExecutor의 인스턴스를 구성하면 추가 옵션을 사용할 수 있다.



#### Fork/Join

포크/조인 프레임워크는 여러 프로세서를 활용할 수 있도록 도와주는 ExecutorService 인터페이스의 구현이다. 

이 프레임워크는 재귀적으로 더 작은 조각으로 나눌 수 있는 작업을 위해 설계되었다. 

목표는 사용 가능한 모든 처리 능력을 사용하여 애플리케이션의 성능을 향상시키는 것이다.



다른 ExecutorService 구현과 마찬가지로, 포크/조인 프레임워크는 스레드 풀의 워커 스레드에 작업을 배포한다. 

포크/조인 프레임워크는 작업 훔치기 알고리즘을 사용하기 때문에 구별된다. 

할 일이 부족한 작업자 스레드는 아직 사용 중인 다른 스레드에서 작업을 훔칠 수 있다.



포크/조인 프레임워크의 중심은 AbstractExecutorService 클래스의 확장인 ForkJoinPool 클래스다. 

ForkJoinPool은 핵심 작업 훔치기 알고리즘을 구현하고 ForkJoinTask 프로세스를 실행할 수 있다.



포크/조인 프레임워크를 사용하여 멀티프로세서 시스템에서 동시에 수행되는 작업에 대한 사용자 지정 알고리즘을 구현하는 것 외에도 Java SE에는 포크/조인 프레임워크를 사용하여 이미 구현된 일반적으로 유용한 기능이 몇 가지 있다. 

Java SE 8에 도입된 이러한 구현 중 하나는 java.util.Arrays 클래스에서 parallelSort() 메서드에 사용된다. 

이 메서드는 sort()와 유사하지만 포크/조인 프레임워크를 통해 동시성을 활용한다. 

대규모 배열의 병렬 정렬은 멀티프로세서 시스템에서 실행할 때 순차 정렬보다 빠르다. 

포크/조인 프레임워크의 또 다른 구현은 Java SE 8 릴리스에 예정된 프로젝트 람다의 일부인 java.util.streams 패키지의 메서드에서 사용된다.



#### Concurrent Collections

java.util.concurrent 패키지에는 Java 컬렉션 프레임워크에 여러 가지 추가 기능이 포함되어 있다. 

이러한 추가 기능은 제공되는 컬렉션 인터페이스로 아래와 같이 분류 할 수 있다.

+ BlockingQueue는 전체 큐에 추가하거나 빈 큐에서 검색을 시도할 때 차단하거나 시간 초과하는 선입선출 데이터 구조를 정의한다.
+ ConcurrentMap은 유용한 원자 연산을 정의하는 java.util.Map의 서브인터페이스다. 
  이러한 연산은 키가 있는 경우에만 키-값 쌍을 제거하거나 바꾸고, 키가 없는 경우에만 키-값 쌍을 추가한다. 
  이러한 연산을 원자 연산으로 만들면 동기화를 방지하는 데 도움이 된다. 
  ConcurrentMap의 표준 범용 구현은 HashMap의 동시 아날로그인 ConcurrentHashMap이다.
+ ConcurrentNavigableMap은 근사 일치를 지원하는 ConcurrentMap의 서브 인터페이스다. 
  ConcurrentNavigableMap의 표준 범용 구현은 TreeMap의 동시 아날로그인 ConcurrentSkipListMap다.

이러한 모든 컬렉션은 컬렉션에 개체를 추가하는 작업과 해당 개체에 액세스하거나 제거하는 후속 작업 간의 발생 전 관계를 정의하여 메모리 일관성 오류를 방지하는 데 도움이 된다.



