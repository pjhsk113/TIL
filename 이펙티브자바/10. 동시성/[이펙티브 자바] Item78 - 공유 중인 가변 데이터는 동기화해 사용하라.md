# [이펙티브 자바] Item78 - 공유 중인 가변 데이터는 동기화해 사용하라

# 동기화(synchronized)

synchronized(동기화) 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 

## 1. 동기화에 대한 오해

많은 프로그래머가 동기화를 배타적 실행을 막는 용도로만 생각한다. 즉, **한 스레드가 변경하는 중이라 상태가 일관되지 않은 순간의 객체를 다른 스레드가 접근하지 못하도록하는 용도로만 생각한다.**

하지만 동기화에는 중요한 기능이 하나 더 있다. **동기화된 메서드나 블록에 들어간 스레드가 같은 락(lock)의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.**

## 2. 동기화가 필요한 이유

Java 언어 명세에는 "long과 double 외의 변수를 읽고 쓰는 동작은 원자적이다."라고 명세되어있다. 이는 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.

하지만 이 명세를 믿고 원자적 데이터를 동기화하지 않는 것은 아주 위험한 행동이다. 스레드가 필드를 읽을 때 항상 **수정이 완전히 반영된 값을 얻는 것을 보장**하지만, 한 스레드가 저장한 값이 **다른 스레드에게 보이는가**는 보장하지 않는다.

따라서 **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.** 한 스레드에서 일어난 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.

## 3. 공유중인 가변 데이터를 동기화하지 않을 때 발생하는 문제

```java
// 잘못된 코드 - 무한루프에 빠진다.
public class StopThread {
    private static boolean stopRequested;

    // 메인 스레드
    public static void main(String[] args) throws InterruptedException {
        // 백그라운드 스레드
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

> Thread.stop은 스레드가 종료될 때 데이터가 훼손될 수 있기 때문에 사용하지말자. (Java11에서는 완전히 제거되었다.)
> 

위의 코드는 언제 종료될까? 코드를 보면 1초 후에 stopRequested를 true로 설정하면서 정상적으로 프로그램이 종료되는 것처럼 보인다. 하지만 이 프로그램은 무한루프에 빠진다.

무한루프에 빠지는 이유는 간단하다. 동기화를 하지 않았기 때문이다. 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게될지 보증할 수 없다. 동기화가 빠지면 JVM은 Hoisting같은 최적화 기법을 적용할 수도 있다.

```java
// 원래 코드
while (!stopRequested)
    i++;

// Hoisting
if (!stopRequested)
    while (true)
        i++;
```

이 결과 프로그램은 **응답 불가 상태**에 빠지게 된다. 이 문제를 해결하려면 어떻게 해야할까?

stopRequested 필드를 동기화해 접근하면 이 문제를 간단히 해결할 수 있다. 

```java
// 스레드가 정상적으로 종료된다.
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    // 메인 스레드
    public static void main(String[] args) throws InterruptedException {
        // 백그라운드 스레드
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

읽기(requestStop)와 쓰기 메서드(stopRequested) 모두에 동기화가 적용되어야만 올바르게 동작함을 보장한다. 이 코드에서는 동기화를 **배타적 실행**과 **스레드 간 통신 안전**이라는 두 가지 기능 중 **스레드간 통신을 목적**으로 사용한 것이다.

## 4. volatile 한정자 (동기화 대안)

위의 예제에서 속도가 더 빠른 대안이 있다. 바로 volatile 한정자를 stopRequeted 필드에 선언하는 것이다. volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
public class StopThread {
    // vaolatile 한정자를 필드에 선언하면 동기화를 생략해도 된다.
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

## 5. volatile 사용 시 주의점

volatile은 주의해서 사용해야한다. 예를 들어 일련번호를 생성할 의도로 작성한 메서드가 있다고 생각해보자.

```java
// 잘못된 코드 - 동기화 필요
private static volatile int nextSerialNumber = 0;
    
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

잘 작동할 것 같은 이 코드의 숨은 문제점은 증가 연산자(++)에 있다. 코드상으로는 하나지만 실제로는 nextSerialNumber에 필드에 두 번 접근한다. 먼저 값을 읽고, 그 다음 새로운 값(1증가 한 값)을 저장한다. 따라서 이 두 번의 접근 사이에 스레드가 들어온다면 값이 증가하기 전의 값을 돌려받게 되고, 잘못된 결과를 계산하는 **안전 실패 오류**가 발생한다.

generateSerialNumber() 메서드에 synchronized를 붙여주면 이 문제는 해결된다. 단, nextSerialNumber 필드의 volatile 한정자는 제거해야 한다.

```java
// 올바르게 동작한다.
private static int nextSerialNumber = 0;
    
public static synchronized int generateSerialNumber() {
    return nextSerialNumber++;
}
```

## 6. 더 좋은 대안!

지금 코드보다 더 최적화를 할 수 있다. 바로 **java.util.concurrent.atomic 패키지의 AtomicLong을 사용하는 것**이다. 이 패키지에는 스레드 안전한 프로그래밍을지원하는 lock-free 클래스들이 담겨 있다.

volatile은 스레드 간 통신 안전만을 지원하지만 AtomicLong은 배타적 실행까지 지원하고 성능도 더 우수하다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

## 7. 동기화의 문제를 해결하는 가장 좋은 대안

사실 가장 좋은 방법은 불변 데이터만 공유하거나 가변 데이터를 공유하지 않는 것이다. **가변 데이터라면 단일 스레드에서만 사용하도록 하자.**

만약 이 정책을 선택했다면, 문서를 남겨 유지보수 과정에도 정책이 계속해서 지켜지도록 하는 것이 중요하다. 

## 8. effectively immutable(사실상 불변)을 활용하자

한 스레드가 데이터를 다 수정한 후 다른 스레드에 데이터를 공유하는 경우 **해당 객체에서 공유되는 부분만 동기화해도 된다.** 이렇게 하면 공유 부분을 수정하기 전까지는 동기화없이 자유롭게 값을 읽어갈 수 있다.

이러한 객체들을 **사실상 불변(effectively immutable)**이라 하고 다른 스레드에 **이런 객체를 건네는 행위를 안전 발행(safe publication)**이라 한다.

객체를 안전 발행하는 방법은 많다. **클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, lock을 통해 접근하는 필드에 저장**해도 된다. 혹은 **동시성 컬렉션**에 저장하는 방법도 있다.

# 핵심 정리

- 여러 스레드가 가변 데이터를 공유한다면 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.
- 공유되는 가변 데이터를 동기화하는데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다.
- (배타적 실행을 제외하고) 스레드간 통신 안전만 필요하다면  volatile 한정자만으로 동기화 할 수 있다.
- 가장 좋은 방법은 가변 데이터를 공유하지 않거나 불변 데이터만 공유하는 것이다.