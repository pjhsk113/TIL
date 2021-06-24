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

## toList()

스트림의 원소를 List에 모아준다.

빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인 예제를 살펴보자.

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

빈도수 `freq::get` 를 역순으로 sorted하고, 단어 10개를 뽑아 List로 모아 반환하는 코드다. 중간 연산 로직만 완성하면 List로 모아 반환하는 건 식은 죽 먹기다.

> 위의 toList()는 Collectors의 메서드로 정적 임포트하여 사용하고 있다. 이렇듯 정적 임포트를 사용하면 코드 가독성이 좋아진다.
collect(Collectors.toList())  → collect(toList())

## toMap()

### toMap(keyMapper, valueMapper) - 인수 2개

맵 수집기 toMap()은 스트림 원소를 Key-Value 형태로 재생산한다. 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 사용하기 적합하다.

toMap(keyMapper, valueMapper)은 각 Key에 매핑되는 keyMapper(함수)와 Value에 매핑되는 valueMapper(함수)를 인수로 받는다.

```java
// toMap(keyMapper, valueMapper) 예제
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toStringg, e -> e));
```

위의 예제는 열거 타입 상수의 문자열 표현을 열거 타입 자체에 매핑하는 구현 예제이다. toMap()은  스트림 원소가 다수의 같은 키를 사용하는 경우 IllegalStateException을 던진다. 주의하자.

### toMap(keyMapper, valueMapper, mergeFunction) - 인수 3개

같은 키를 공유하는 값들이 있는 경우 병함 함수를 통해 기존 값에 합쳐진다. 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.

```java
// toMap(keyMapper, valueMapper, mergeFunction) 예제
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

위 예제는 음악가와 음악가의 베스트 앨범을 연관지어 Map으로 만드는 스트림 코드다. 코드를 읽으면 자연스럽게 의미가 해석된다.

또, 인수가 3개인 toMap의 원소가 충돌할 경우 마지막 값을 취하는 수집기를 만들때도 유용하다.

```java
toMap(keyMapper, valueMapper, (oldValue, newValue) -> newValue)
```

### toMap(keyMapper, valueMapper, mergeFunction, Map의 구현체) - 인수 4개

인수가 4개인 toMap은 마지막 인수로 맵 팩터리를 받는다. 즉, TreeMap이나 EnumMap과 같은 Map의 구현체를 지정할 수 있다. 

## groupingBy

### 가장 일반적인 groupingBy

분류 함수를 입력받고 원소들을 카테고리별로 모아 놓은 Map을 반환한다. 반환된 Map에 담긴 각각의 값은 List다.

```java
words.collect(groupingBy(words -> alphabetize(word)))
```

### 값을 리스트외에 다른 타입으로 반환하는 방법

값을 리스트 외 다른 타입으로 반환하기 위해서는 다운스트림(3번째 인자)을 명시해야 한다. 다운스트림의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다.

```java
// 다운스트림으로 counting()을 건낸다.
words.collect(groupingBy(String::toLowerCase, counting())
```

### 다운스트림에 더해 맵 팩터리를 지정하기

맵 팩터리를 지정해 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있게 만들 수 있다.

일반적으로 3번째 인자로 맵 팩터리가 와야하지만, 맵 팩터리(mapFactory) 매개변수가 다운스트림 매개변수보다 앞에 놓인다. 

```java
// 2번째 인자는 맵 팩터리, 3번째 인자는 다운스트림
words.collect(groupingBy(String::toLowerCase, TreeMap::new, counting())
```

## joining

### 인자가 없는 joining()

원소들을 연결하는 역할을 한다. joining은 문자열 등의 CharSequence 인스턴스의 스트림에만 적용할 수 있다.

### 인자가 1개 joining(delimiter)

구분문자를 매개변수로 받아 문자열 연결 부위에 삽입하여 연결한다.

```java
// came, saw, conquered joining
words.collect(joining('/'));

// came/saw/conquered 출력
```

### 인자가 3개 joining(perfix, delimiter, suffix)

접두문자, 구분문자, 후미문자를 입력받아 문자열을 연결한다.

```java
// came, saw, conquered joining
words.collect(joining('[', ',', ']'));

// [came, saw, conquered] 출력
```

## 이외의 다양한 메서드

maxBy, minBy, summing, averaging, reducing, filtering, mapping, flatMapping, collectingAndThen 등... 굉장히 다양한 메서드가 존재한다. 대부분의 프로그래머는 이 존재를 몰라도 된다. 설계 관점에서 보면, 스트림의 기능을 일부 복제하여 다운스트림 수집기를 작은 스트림처럼 동작하게 한 것이기 때문이다.

# 핵심 정리

- 스트림 파이프라인 프로그래밍의 핵심은 부작용(side effect)없는 함수 객체다.
- 스트림을 올바르게 사용하려면 수집기를 잘 알아둬야한다.(toList, toSet, toMap, groupingBy, joining)