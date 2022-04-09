# [Java8] Stream API

Java8에 추가된 Stream API는 다량의 데이터 처리나 손쉬운 병렬 처리를 위해 만들어졌다.
Stream API는 일반적으로 데이터를 담고있는 Collection의 개념이 아니라 어떤 시퀀스 요소들의 순차 및 병렬 집계 작업을 지원해주는 API이다.

## Stream의 특징

- 스트림이 처리하는 데이터소스를 변경하지 않는다.(원본 데이터를 훼손하지 않는다.)
- 스트림으로 처리하는 데이터는 오직 한 번만 처리된다.
- 원소의 유한 혹은 무한 시퀀스를 나타낸다. 요소가 무한인 경우 Short Circuit 메서드로 제한할 수 있다.
- 근본적으로 지연 평가(Lazy Evaluation)을 사용한다. 이 덕분에 무한 스트림을 다룰 수 있게된다.
- 손쉬운 병렬처리를 지원한다.

## Stream의 핵심 추상 개념

### 스트림 파이프 라인

원소들로 수행하는 연산 단계를 표현하는 개념으로 Stream을 구성하는 핵심 추상 개념이다.
스트림 파이프 라인은 **소스 스트림으로 시작**해 **중간 연산**을 거치고 **종단 연산**으로 끝난다.

```java
public class Dummy {
    public static void main(String[] args) {
        List<String> nameList = Arrays.asList("a", "b", "c");

        nameList.stream() // 소스 스트림
                .filter(s -> s.startsWith("a")) // 중간 연산
                .forEach(System.out::println); // 종단 연산
    }
}
```

### 중간 연산(중개 오퍼레이션)

중간 연산은 스트림을 어떠한 방식으로 변환한다. 2차원 배열의 요소들을 1차원 배열로 평탄화할 수 있수도 있고, 특정 조건을 만족하는 요소들만 뽑아내는 연산 등을 수행할 수 있다.

중간 연산은 Stream을 리턴하며, Stateless와 Stateful 연산으로 상세하게 나눌 수 있다.
Stateless는 이전 상태(소스 데이터)를 참조하지 않는 연산을 나타내고 Statful은 이전 상태를 참조해야 하는 연산을 나타낸다.

Stateless: **filter, map, faltMap, limit, skip**

Stateful: **distinct, sorted**

### 종단 연산(종료 오퍼레이션)

중단 연산 과정에서 변화된 소스 데이터에 최후 연산을 가한다. 이 과정이 수행되어야만 중간 연산도 수행된다.
종단 연산은 Stream을 리턴하지 않는다.

대표적으로 **collect, allMatch, count, reduce, forEach, min, max** 등의 종단 연산이 존재한다.