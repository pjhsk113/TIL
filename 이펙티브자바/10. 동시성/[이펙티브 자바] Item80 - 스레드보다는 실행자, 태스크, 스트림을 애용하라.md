# [이펙티브 자바] Item80 - 스레드보다는 실행자, 태스크, 스트림을 애용하라

과거에는 클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 비동기적으로 처리하기 위해 단순한 작업 큐를 사용했다. 하지만 이는 안전 실패나 응답 불가에 대한 예외 코드 작성 등으로 수 많은 코드 작업이 발생했다.

다행히도 실행자 프레임워크의 등장으로 이제 이러한 수고를 덜 수 있게 되었다.

# 실행자 프레임워크(Executor Framework)

실행자 프레임워크는 java.util.concurrent 패키지에 속해있으며, 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다. 기존보다 모든 면에서 뛰어난 작업 큐를 이제는 단 한 줄로 생성할 수 있다.

```java
// 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 태스크를 넘김
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```

# 실행자 서비스의 주요 기능

### 1. 특정 태스크가 완료되기를 기다린다.

get() 메서드를 통해 태스크가 완료되기를 기다릴 수 있다.

```java
ExecutorService exec = Executors.newSingleThreadExecutor();

exec.submit(()  -> s.removeObserver(this)).get(); 
```

### 2. 태스트 모음 중에 어느 하나 혹은 모든 태스크가 완료되기를 기다린다.

```java
// 태스크 모음 중 어느 하나를 기다린다.
exec.invokeAny(tasks);

// 모든 태스크가 완료되기를 기다린다.
exec.invokeAll(tasks);
```

### 3. 실행자 서비스가 종료되기를 기다린다.

```java
exec.awaitTermination(10, TimeUnit.SECONDS);
```

### 4. 완료된 태스크의 결과를 차례로 받는다.

```java
ExecutorService exex = Executors.newFixedThreadPool(2);
ExecutorCompletionService executorCompletionService = 
    new ExecutorCompletionService(exex);

.....

for (int i = 0; i < 2; i++) {
    executorCompletionService.take().get();
}
```

### 5. 태스크를 특정 시간 혹은 주기적으로 실행하게 한다.

```java
ScheduledThreadPoolExecutor scheduledExecutor =
                new ScheduledThreadPoolExecutor(1);
```

큐를 둘 이상의 스레드가 처리하게 하고 싶다면 다른 정적 팩터리를 이용해 다른 종류의 실행자 서비스를 생성하면 된다. 스레드의 개수는 고정할 수 있고 필요에 따라 늘어나거나 줄어들게도 설정할 수 있다.

이러한 실행자는 java.util.concurrent.Executors의 정적 팩터리들을 이용해 생성할 수 있다.

# 실행자 서비스를 사용하기 까다로운 경우

일반적으로 가벼운 애플리케이션의 서버에서는 Executors.newCachedThreadPool을 사용하는 것이 적합하다. 하지만 무거운 프로덕션 서버에서 newCachedThreadPool를 사용하면 좋지 않은 상황이 일어날 수도 있다.

newCachedThreadPool은 요청받은 태스크를 큐에 쌓지 않고 바로 처리하며 사용 가능한 스레드가 없다면 새로 스레드를 생성하기 때문에 CPU 사용률이 100%에 치닫는 상황이 생길 수 있다. 이때 새로운 태스크가 도착할 때마다 다른 스레드를 계속 생성하니 상황이 더욱 악화 될 것이다.

따라서 무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThredPool을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 좋다.

# 스레드를 직접 다루는 것을 삼가자

작업 큐를 손수 만드는 일과 스레드를 직접 다루는 일은 삼가야 한다.

스레드를 직접 다루면 스레드가 작업 단위와 수행 매커니즘 역할을 모두 수행하지만, 실행자 프레임워크를 사용하면 작업 단위와 실행 매커니즘이 분리된다. 

작업 단위를 나타내는 핵심 추상 개념은 태스크이고, 이 태스크는 Runnable과 Callable로 나눌 수 있다.(Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.)

태스크를 수행하는 일반적인 매커니즘이 바로 실행자 서비스다. 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 언제든 변경할 수 있다. 이 말인 즉, 실행자 프레임워크가 작업 수행을 담당해준다는 것이다.

Java7부터 지원하는 fork-join 태스크를 예로 들 수 있다.

ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPoll을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다. 따라서 모든 스레드가 최대한의 가용성을 뽑아낼 수 있게 되고 CPU를 최대한 활용해 높은 처리량과 낮은 지연시간을 가질 수 있게 된다.

우리가 직접 스레드를 직접 작성하고 튜닝해서 ForkJoinTask와 같은 성능을 뽑아내기란 어려운 일이다. 하지만 단순히 ForkJoinPool 이라는 특별한 실행자 서비스를 실행하면 적은 노력으로 많은 이점을 누릴 수 있게 된다.