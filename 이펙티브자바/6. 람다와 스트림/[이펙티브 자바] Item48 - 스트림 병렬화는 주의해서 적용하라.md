# [이펙티브 자바] Item48 - 스트림 병렬화는 주의해서 적용하라

Java8부터는 parallel 메서드만 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원한다. 동시성 프로그램을 작성하기는 쉬워지고 있지만, 올바르고 빠르게 작성하는 일은 여전히 어려운 일이다.

동시성 프로그래밍을 할 때는 **안전성**과 **응답 가능 상태**를 유지해야한다. 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.

## 파이프라인 병렬화로 성능 개선을 기대하기 어려운 경우

```java
// 스트림을 사용해 처음 20개의 메르센 소수를 생성하는 프로그램
public class MersennePrime {

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                //.parallel()
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```

![](https://blog.kakaocdn.net/dn/ANQtn/btq8aauxGwK/XJnwnvhlTZ2sC4RlwWwdDK/img.png)

20개의 메르센 소수를 생성하는 프로그램을 실행해보니 14.7초가 걸린다. 이 프로그램의 성능을 올리는 방법으로 병렬처리를 떠올려서, parallel()을 호출해 병렬처리를 한다고 생각해보자. 이 프로그램의 성능을 올라갈까? 

```java
// 스트림을 사용해 처음 20개의 메르센 소수를 생성하는 병렬 프로그램
public class MersennePrime {

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel()
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```

안타깝게도 이 프로그램은 아무것도 출력하지 못하고, **응답 불가 상태**가 된다. **스트림이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문**이다. 또한, **데이터소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.**

즉, 스트림 파이프라인을 마구잡이로 병렬화하면 오히려 성능이 나빠질 수 있다.

# 효율적인 병렬화

## 1. 병렬화의 효과가 좋은 스트림 소스

**ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int ~ long 범위**일 때 병렬화의 효과가 가장 좋다.

## 2. 병렬화 효과가 좋은 자료구조의 공통점

### 2-1. 정확하고 쉽게 나눌 수 있다.

**위의 자료구조들은** 모두 **데이터를 원하는 크기로 정확하고 쉽게 나눌수 있다는 공통점**을 가지고 있다. 따라서 다수의 스레드에 분배하기 좋다. 나누는 작업은 **Spliterator**가 담당하며, Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있다.

### 2-2. 참조 지역성이 뛰어나다.

또 다른 공통점은 **참조 지역성**이 뛰어나다는 것이다. 즉, 이웃한 원소의 참조들이 **메모리에 연속해서 저장**되어 있기 때문에 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 유리하다. 

> 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다. 기본 타입 배열은 데이터 자체가 메모리에 연속적으로 저장되기 때문이다.

## 종단 연산

종단 연산의 동작 방식은 병렬 수행 효율에 영향을 준다. 일반적으로 종단 연산 중 **병렬화에 가장 적합한 것은 축소(reduction)**다. 축소란 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다.

축소 연산은 Stream의 reduce 메서드 중 하나, 혹은 min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해서 수행한다. 또한, anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.

반면에, 종단 연산 중 가변 축소를 수행하는 collect 메서드는 병렬화에 적합하지 않다. 컬렉션을 합치는 부담이 크기 때문이다.

# 병렬화 올바르게 사용하기

## 안전 실패(safety failure)

스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다. 이를 **안전 실패**라고 한다. **안전 실패**는 병렬화한 파이프라인이 사용하는 mappers, filters 등 **함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있다.**

Stream 명세는 함수 객체에 관한 엄중한 규약을 정의해놨다. 예를 들어, reduce 연산에 건내지는 누적기와 결합기 함수는 반드시 결합법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야 한다. 이러한 요구사항을 지키지 못하는 상태에 병렬로 파이프라인을 수행하면 실패로 이어지기 쉽다.

## 병렬화를 사용할 가치가 있는지 판단하자.

파이프라인이 수행하는 진짜 작업이 **병렬화에 드는 추가 비용을 상쇄하지 못한다면** **성능 향상은 미미할 수 있다.**

실제로 성능 향상이 이루어졌는지 확인하려면 스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱해보자. 이 값이 최소 수십만은 되어야 성능 향상을 기대할 수 있다.

**스트림 병렬화는 오직 성능 최적화 수단임을 기억하자. 성능을 테스트해서 병렬화를 사용할 가치가 있는지 판단해야한다.**

## 병렬화를 효과적으로 사용한 예제

조건이 잘 갖춰지면 parallel 메서드 호출 하나로 굉장한 성능 향상을 기대할 수 있다. 아래의 예제는 병렬화 하기 좋은 조건을 갖춘 소수 계산 스트림 파이프라인 코드이다.

```java
// 소수 계산 스트림 파이프 라인 - 병렬화 안한 경우
public class ParallelPrime {

    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }
}
```

![](https://blog.kakaocdn.net/dn/cSqSp6/btq789XJWwH/CdwKaW9Bei6zWgKiBmOyzK/img.png)

**10^6을 계산하는데 약 5초**, **10^7을 계산하는데 44.5초**가 걸렸다. 이러한 연산에 parallel() 메서드를 추가해서 병렬화해보자. 

```java
// 소수 계산 스트림 파이프 라인 - 병렬화
public class ParallelPrime {

    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .parallel()
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }
}
```

![](https://blog.kakaocdn.net/dn/cKWWfK/btq7894tyqE/Bnw48IoqfLU3DVEtbJkyak/img.png)

**10^6을 계산하는데 약 2.9초**, **10^7을 계산하는데 18.6초**가 걸렸다. parallel() 메서드 호출을 하나 추가한 것 뿐이지만 **시간이 2배 이상 단축**됐다.

### Random한 수들로 이루어진 스트림 병렬화

- **SplittableRandom 인스턴스**를 활용해 병렬화하자. 성능이 선형으로 좋아진다.
- **Random을 사용하는 경우 병렬화를 해선 안된다.** 모든 연산을 동기화하기 때문에 최악의 성능으로 이어질 수 있기 때문이다.

무작위 수 병렬화 연산 속도

**SplittableRandom > ThreadLocalRandom > Random**

# 핵심 정리

- 계산도 정확하고 성능도 좋아질 거라는 확신 없이는 스트림 파이프라인 병렬화를 하지말자.
- 수정 후 코드가 여전히 정확한지 확인하고 병렬화 사용 가치가 있는지 확인하자.
- 위의 조건들이 확실해졌을 때, 병렬화 버전 코드를 운영 코드에 반영하자.