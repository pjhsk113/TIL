# [이펙티브 자바] Item46- 스트림에서는 부작용 없는 함수를 사용하라

스트림을 처음 접하면 이해하기 어렵거나 어떤 장점이 있는지 공감하기 힘들 수 있다. 스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이기 때문이다. 

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 것이다. 이때 각 변환 단계는 이전 단계의 결과를 받아 처리하는 **순수 함수**여야 한다.

> 순수 함수란?

오직 입력만이 결과에 영향을 주는 함수를 말한다. 순수 함수는 다른 가변 상태를 참조하지 않고, 함수 스스로 다른 상태를 변경하지 않는다.

이렇듯 스트림 연산에 전달되는 함수 객체는 모두 부작용(Side Effect)가 없어야 한다.

# 스트림 제대로 활용하기

## forEach

아래 예제는 스트림 API의 이점을 잘 살리지 못한, 스트림 코드를 가장한 반복 코드이다.

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()){
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

**forEach는 연산 결과를 보여주는 역할을 가진 종단 연산**인데, 외부 상태인 freq를 수정하고 있다. 즉, 함수 스스로 외부 상태를 변경하고 연산 결과를 보여주는 일 이상을 하고 있다.

이를 스트림 패러다임에 맞게 수정해보면 다음과 같은 코드가 된다.

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()){
    freq = words.collect(groupingBy(String::toLowerCase, counting()))
}
```

forEach는 종단 연산 중 기능이 가장 적고 가장 덜 스트림스럽다. 또한, 병렬화할 수도 없다. 따라서 **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산할 때는 사용하지 말자.**

## 수집기(collector)

수집기는 스트림을 사용하려면 꼭 배워야하는 새로운 개념이다. 수집기는 java.util.stream.Collectors 클래스의 메서드를 사용하는데, **스트림의 원소들을 축소해서 객체 하나에 모아주는 역할을 한다.** 익숙해지기 전까지 그저 축소 전략을 캡슐화한 블랙박스 객체라고 생각하자.

수집기가 생성하는 객체는 주로 컬렉션이며, toList(), toSet(), toCollection(collectionFactory) 이렇게 총 3가지의 수집기가 있다.