# [이펙티브 자바] Item81 - wait와 notify보다는 동시성 유틸리티를 애용하라

고수준의 동시성 유틸리티가 Java5에서 도입되면서 wait와 notify를 사용해야 할 이유가 많이 줄었다. wait와 notify는 올바르게 사용하기 아주 까다로우니고수준 동시성 유틸리티를 사용하자.

# 동시성 유틸리티

java.util.concurrent 동시성 유틸리티는 **실행자 프레임워크, 동시성 컬렉션, 동기화 장치** 이렇게 세 범주로 나눌 수 있다.

## 실행자 프레임워크

앞선 item80에서 실행자 프레임워크에 대한 내용을 다뤘다. 이 내용에 대한 내용은 item80을 참고하자.

## 동시성 컬렉션

List, Queue, Map 등 표준 컬렉션 인터페이스에 동시성을 추가해 구현한 고성능 컬렉션이다. 동기화를 내부에서 수행하여 높은 동시성에 도달할 수 있다. 

![Untitled](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item81%20-%20wait%E1%84%8B%E1%85%AA%20notify%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A9%E1%86%BC%20cdba043c2c4441899088e6e5f0890630/Untitled.png)

내부에서 동기화를 수행하므로 동시성을 무력화 할 수 없고 외부에서 락을 추가로 사용하면 오히려 속도가 느려지니 주의하자.

### 상태 의존적 수정 메서드

동시성 컬렉션에서 동기성을 무력화하지 못하므로 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다. 따라서 **상태 의존적 수정 메서드**가 추가되었다.

> 상태 의존적 수정 메서드는 아주 유용해서 Java8부터는 일반 컬렉션 인터페이스에도 디폴트 메서드 형태로 추가되었다.
> 

                                                                                                                                                                                                                                                                                                         

상태 의존적 수정 메서드의 대표적인 예로 Map의 디폴트 메서드인 `putIfAbsent`를 들 수 있다.

putIfAbset 메서드는 인자로 넘겨진 key가 없을 때 value를 추가한다. 그리고 기존 값이 있으면 그 값을 반환하고 없는 경우 null을 반환한다. 이 메서드 덕분에 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다.

이러한 특성을 이용해 String의 intern의 동작을 흉내 내어 구현한 메서드를 살펴보자.

```java
private static final ConcurrentMap<String, String> map =
        new ConcurrentHashMap<>();

public static String intern(String s) {
    // ConcurrentHashMap은 검색 기능에 최적화 되어있기 때문에 get을 이용하면 훨씬 빠르다.
    String result = map.get(s);

    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

이 메서드는 기존 String.intern 메서드보다 훨씬 빠르게 동작한다.

이렇듯 동시성 컬렉션의 등장은 기존 동기화한 컬렉션을 낡은 유산으로 만들어 버렸다. 동기화된 컬렉션을 동시성 컬렉션으로 교체하는 것만으로도 성능을 개선 할 수 있다. 대표적인 예로, Collections.synchronizedMap을 ConcurrentHashMap으로 교체하는 것만으로도 동시성 애플리케이션의 성능을 극적으로 개선할 수 있다.

이외에도 일부 컬렉션 인터페이스는 작업이 성공적으로 완료될 때까지 기다리도록 설계되었다. 대표적인 예로, BlockingQueue는 실행자 서비스 구현체에서 작업 큐(생산자-소비자 큐)로 사용된다.

## 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게해서 서로의 작업을 조율할 수 있게 해준다. 대표적인 동기화 장치로는 **CountDownLatch**와 **Semaphore**가 있으며 가장 강력한 동기화 장치는 **Phaser**다. 이 외에도 **CyclicBarrier**와 **Exchanger**가 있지만 위의 두 개의 동기화 장치보다는 덜 쓰인다.

### CountDownLatch 알아보기

CountDownLatch는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 하는 역할을 한다.

생성자로 받는 int 값을 받으며, 이 값이 countDown 메서드를 몇 번 호출해야 대기중인 스레드를 깨우는지를 결정한다.

![Untitled](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item81%20-%20wait%E1%84%8B%E1%85%AA%20notify%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A9%E1%86%BC%20cdba043c2c4441899088e6e5f0890630/Untitled%201.png)

CountDownLatch를 잘 활용하면 유용한 기능을 쉽게 구현할 수 있다.

예를 들어 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크를 개발한다고 해보자. 이 프로그램은 메서드 하나로 구성되며, 동시성 수준(실행할 실행자와 동작을 몇개나 동시에 수행할 수 있는지)을 매개변수로 받는다. 

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
```

이제 위 메서드를 활용해 프로그램을 돌려보자.

```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);

    try {
        long time = time(executorService, 10, () -> System.out.println("THREAD START"));
        System.out.println("END TIME : " + time);
    } catch (Exception e) {
        System.out.println(e.getMessage());
    } finally {
        executorService.shutdown();
    }
}
```

![Untitled](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item81%20-%20wait%E1%84%8B%E1%85%AA%20notify%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A9%E1%86%BC%20cdba043c2c4441899088e6e5f0890630/Untitled%202.png)

time에 넘겨진 실행자 서비스는 매개변수로 지정한 동시성 수준만큼으 스레드를 생성할 수 있어야한다. 그렇지 못하면 **스레드 기아 교착상태**가 발생하며 이 메서드는 끝나지 않는다.

```java
/**
 * 스레드 기아 교착상태가 발생하며 time() 메서드가 끝나지 않는다.
 */
public class ConcurrentTimer {
    public static void main(String[] args) throws InterruptedException {
		    ExecutorService executorService = Executors.newFixedThreadPool(10);
		
		    try {
		        // 생성한 스레드는 10개, 매개변수로 넘긴 동시성 수준은 15
		        long time = time(executorService, 15, () -> System.out.println("THREAD START"));
		        System.out.println("END TIME : " + time);
		    } catch (Exception e) {
		        System.out.println(e.getMessage());
		    } finally {
		        executorService.shutdown();
		    }
		}

    public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

> 시간 간격을 잴 때는 System.currentTimeMillis보다 System.nanoTime을 사용하자!
System.nanoTime가 더 정확하고 정밀하다. 또한, 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.
> 

# 레거시의 wait와 notify를 다루는 경우

새로운 코드라면 항상 동시성 유틸리티를 써야 하지만, 레거시 코드의 경우 어쩔 수 없이 wait와 notify를 다뤄야 하는 상황이 생길 수 있다. 이때를 대비해 wait와 notify의 사용 시 주의사항을 알아보자.

## wait 메서드

wait는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다. **Lock 객체의 wait 메서드는 반드시 동기화 영역 안에서 호출해야 한다.**

```java
// wait 메서드를 사용하는 표준 방식
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

**wait 메서드를 사용할 때는 반드시 대기 반복문 관용구를 사용하고 반복문 밖에서는 절대로 호출하지 말자.**

위 코드는 wait 호출 전후로 조건이 만족하는지를 검사한다. 

**대기 전에 조건을 검사**하여 조건에 만족한다면 wait를 건너뛰게 한다. 이것은 **응답 불가 상태를 예방**하는 조치이다. 만약 조건이 충족되었는데 스레드가 notify 혹은 notifyAll 메서드를 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨우는 것을 보장할 수 없다.

한편, **대기 후에 조건을 검사**하여 조건이 만족되지 않았다면 다시 대기하게 한다. 이는 **안전 실패를 막는 조치**이다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 Lock 보호하는 불변식을 꺠질 위험이 있기 때문이다.

### 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황 (주의 사항)

- 스레드가 notify를 호출하여 대기 중인 스레드가 깨어나는 사이 다른 스레드가 Lock을 얻어 상태를 변경하는 경우
- 조건이 만족되지 않았지만 다른 스레드가 실수 혹은 악의적으로 notift를 호출하는 경우(외부에 공개된 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.)
- 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출하는 경우(notifyAll은 모든 스레드를 꺠운다.)
- 대기 중인 스레드가 허위 각성(spurious wakeup)한 경우 - notify없이 깨어나는 경우

## notify와 notifyAll

notify는 스레드 하나만 꺠우며, notifyAll은 모든 스레드를 깨운다. 일반적으로 notifyAll을 사용하는게 안전하고 합리적이다. 

notifyAll을 사용하면 깨어나야 할 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 수 있다. 다른 스레드까지 깨어날 수 있지만, 프로그램의 정확성에 영향을 주진 않는다. 깨어난 스레드들은 조건이 충족되었는지 확인하고, 충족되지 않았다면 다시 대기 상태로 변경될 것이기 때문이다.

다만, 모든 스레드가 같은 조건을 기다리는 상황이고 조건이 충족될 때 하나의 스레드만 혜택을 받는 상황이라면 notify로 최적화할 수 있다.

그럼에도 불구하고 notifyAll을 사용해야하는 이유는 관련 없는 스레드가 wait를 호출하는 공격으로부터 꼭 깨어나야 하는 스레드를 보호할 수 있기 때문이다.(깨어나야 할 스레드가 notify를 삼켜버린다면 영원히 대기하게 될 수 있다.)

# 핵심 정리

- 새로운 코드를 작성한다면 반드시 wait와 notify 대신 **동시성 유틸리티**를 사용하자.
- 레거시를 유지보수한다면 wait는 항상 while문(대기 반복 표준 관용구) 안에서 호출하도록 하자.
- notify보다는 notifyAll을 사용하는 것이 안전하다.
- notify를 사용한다면 응답 불가 상태에 빠지지 않게 각별히 주의하자.