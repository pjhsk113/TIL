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

![]()

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

# 병렬화의 효과가 좋은 스트림 소스

**ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int ~ long 범위**일 때 병렬화의 효과가 가장 좋다.

위의 자료구조들은 모두 데이터를 원하는 크기로 정확하고 쉽게 나눌수 있다는 공통점을 가지고 있다. 따라서 다수의 스레드에 분배하기 좋다.