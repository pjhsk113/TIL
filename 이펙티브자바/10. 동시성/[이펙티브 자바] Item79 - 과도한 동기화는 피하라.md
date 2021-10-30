# [이펙티브 자바] Item79 - 과도한 동기화는 피하라

과도한 동기화는 성능을 떨어뜨리고, 교착상테에 빠드리고, 예측할 수 없는 동작을 낳기도 한다.

**응답 불가(교착 상태)와 안전 실패(데이터 훼손)를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대 클라이언에 양도하면 안 된다.** 

# 외계인 메서드

응답 불가와 안전 실패를 유발할 수 있는 메서드 즉, 동기화된 영역에서 재정의할 수 있는 메서드 혹은 클라이언트가 넘겨준 함수 객체 등을 **동기화된 클래스 관점에서 외계인 메서드라고 한다.** 

동기화된 클래스는 외계인 메서드가 무슨 일을 할지 알지 못하며 통제할 수도 없고 **외계인 메서드**가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

# 외계인 메서드 예제

### ForwardingSet

```java
class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {this.s = s;}
    @Override
    public int size() {    return s.size();}
    @Override
    public boolean isEmpty() {return s.isEmpty();}
    @Override
    public boolean contains(Object o) {return s.contains(o);}
    @Override
    public Iterator<E> iterator() {return s.iterator();}
    @Override
    public Object[] toArray() {return s.toArray();}
    @Override
    public <T> T[] toArray(T[] a) {return s.toArray(a);}
    @Override
    public boolean add(E e) {return s.add(e);}
    @Override
    public boolean remove(Object o) {return s.remove(o);}
    @Override
    public boolean containsAll(Collection<?> c) {return s.containsAll(c);}
    @Override
    public boolean addAll(Collection<? extends E> c) {return s.addAll(c);}
    @Override
    public boolean retainAll(Collection<?> c) {return s.retainAll(c);}
    @Override
    public boolean removeAll(Collection<?> c) {return s.removeAll(c);}
    @Override
    public void clear() {s.clear();}
}
```

### ObservableSet (ForwardingSet을 상속)

```java
// 잘못된 코드 - 동기화 블록 안에서 외계인 메서드를 호출한다.
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```

관찰자는 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
@FunctionalInterface
public interface SetObserver<E> {
    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

이제 집합에 추가된 0부터 99까지 정수 값을 출력하다가, 그 값이 23이면 자기 자신을 제거(구독해지)하는 관찰자를 추가해보자.

```java
public class Dummy {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if(e == 23) {
                    s.removeObserver(this);
                }
            }
        });

        for(int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```

이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 구독해지한 다음 종료될 것이라 예상된다. 하지만 실제로 실행해 보면 23까지 출력한 후 ConcurrentModificationException을 던진다.

![](https://blog.kakaocdn.net/dn/TsfMv/btrjqnoQ1aJ/DnbIUw5WT6HSUZJ0m47yHk/img.png)

이렇게 예외가 발생하는 이유는 added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다.

added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출한다. 이때 문제가 발생하게 된다. 리스트에서 원소를 제거하려는데, 이 리스트를 순회하는 도중이기 때문에 허용되지 않는 동작으로 인식한다.

notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록이므로 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지는 막지 못한다.

정리하자면,

1. main에서 set.add()가 호출되면 ObservableSet의 재정의된 add()가 호출된다.
2. 재정의된 add는 notifyElementAdded()를 호출한다.
3. notifyElementAdded()에서는 관찰자 목록(List<SetObserver<E>>)을 순회하며 added()를 호출한다.
4. main에서 익명 함수로 정의한 added가 호출된다. 해당 added는 특정 조건에서 removeObserver 메서드를 호출한다.
5. 이때, removeObserver()가 호출되면 콜백으로 되돌아와 자신을 수정하는 것을 막지 못하므로 원소가 삭제된다.
6. notifyElementAdded()에서 순회 중인 동기화 블록에서 ConcurrentModificationException이 발생한다.(동기화가 걸려있음에도 원소가 삭제됨 → 동기화 오류)

# 예제2

이번에는 구독해지를 하는 관찰자를 작성할 때, 직접 removeObserver를 호출하는 것이 아닌, 실행자 서비스(ExecutorService)를 사용해 다른 스레드에게 부탁할 것이다.

```java
// 백그라운드 스레드를 사용하는 예제
public class Dummy {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if(e == 23) {
                    ExecutorService exec = Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get(); //여기서 lock이 걸려서 못들어감
                        //하지만 메인 스레드는 너의 작업을 기다리고 있어
                    } catch(ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });
        
        for(int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```

이 프로그램은 예외가 발생하진 않지만, 교착상태에 빠져버린다. 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자에 Lock을 시도하지만 메인 스레드가 Lock을 가지고 있기 때문에 Lock을 얻을 수 없다.

메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리고 백그라운드 스레드는 메인 스레드의 lock을 획득하기 위해 기다리고 있는 상태, 바로 교착상태다.

만약 앞선 예제들의 리소스(관찰자)가 일관된 상태가 아닌 임시로 불변식이 깨진 상태라면 어떨까?

Java의 Lock은 재진입을 허용하므로 교착상태에는 빠지지 않는다. 하지만 예외를 발생시킨 예에서는 외계인 메서드를 호출하는 스레드가 이미 Lock을 획득하고 있고 다음 재진입에서도 Lock을 획득한다. 이는 Lock이 제 역할을 하지 못하는 것을 나타내고 재진입 가능 락으로 인해 **응답불가(교착상태)가 될 상황을 안전 실패(데이터 훼손) 상태로 변질시킬 위험을 나타낸다.**

재진입 가능 락은 교착상태는 방지할 수 있지만 근본적인 해결 방법은 되지 못한다.

# 해결 방법

## 1. 동기화 블록 바깥으로 외계인 메서드 옮기기

외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 위와 같은 문제들을 다행히 어렵지 않게 해결할 수 있다. notifyElementAdded 메서드에서라면 List<SetObserver<E>>를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다. 이 방식을 사용하면 앞선 예제(예외 발생과 교착상태)의 증상들이 사라진다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element); //외계인 메서드를 동기화 블록 바깥으로 옮겼다.
    }
}
```

## 2. CopyOnWriteArrayList 사용하기

위의 1번 방법보다 더 나은 방법이 바로 CopyOnWriteArrayList를 사용하는 것이다. 

```java
// 명시적으로 동기화한 곳이 사라졌다.
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }
	
    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(observer);
    }
	
    public void notifyElementAdded(E element) {
        for (SetObserver<E> observer : observers) {
            observers.added(this, element);
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```

이렇게 하면 내부를 변경하는 작업은 복사본을 만들어 수행하고, 내부 배열은 절대 수정되지 않기 때문에 Lock을 사용할 필요가 없어 성능도 매우 빨라진다.

# 동기화의 정확성과 효율성 개선

외계인 메서드는 얼마나 오래 실행될지 알 수 없기 때문에 동기화 영역에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야 한다.

따라서 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선시켜준다.

> 동기화 블록 바깥으로 외계인 메서드 옮기기 예제처럼 동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출(open call)이라 한다.
> 

## 동기화의 기본 규칙

동기화 영역에서는 가능한 한 일을 적게하는 것이 기본 규칙이다. Lock을 얻고, 공유 데이터를 검사하고, 필요하면 수정하고, Lock을 놓는다.

만약 오래걸리는 작업이라면 동기화 영역 바깥으로 옮기는 방법을 찾아보자.

## 동기화의 성능 개선

멀티코어가 일반화된 오늘날, 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 동기화의 진짜 비용이라고 할 수 있다.

가변 클래스를 작성한다면 아래의 두 개의 선택지 중 하나를 따르자.

1. 동기화를 전혀 고려하지 말고, 사용하는 클래스가 외부에서 알아서 동기화하게 하자.
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. (단, 클라이언트가 외부에서 전체에 락을 거는 것보다 효율성이 좋을 때만)

Java의 라이브러리 중 java.util은 첫 번째 방법을, java.util.concurrent는 두 번째 방법을 택했다. 만약 선택하기 어렵다면 동기화하지 말고 "스레드 안전 하지 않다."라고 명시하자.

### 클래스 내부에서 동기화하기로 했다면..

락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원해 동시성을 높일 수 있다.

- 락 분할(Lock Splitting)
    - 하나의 클래스에서 기능적으로 Lock을 분리해서 사용하는 것(읽기 전용 Lock, 쓰기 전용Lock)
- 락 스트라이핑(Lock Striping)
    - 자료구조 관점에서 한 자료구조 전체가 아닌 일부분에 락을 적용하는 것
- 비차단 동시성 제어(NonBlocking Concurrency Control)

# 핵심 정리

- 교착 상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자.
- 동기화 영역 안에서의 작업은 최소한으로 줄이자.
- 가변 클래스 설계 시에는 스스로 동기화가 필요한지에 대해 고민해보자.
- 멀티코어 세상인 지금은 과도한 동기화를 피하는게 중요하다.
- 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화 여부를 문서에 명확히 명시하자.