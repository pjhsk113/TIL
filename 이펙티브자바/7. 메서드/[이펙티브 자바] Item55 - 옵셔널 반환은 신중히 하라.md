# [이펙티브 자바] Item55 - 옵셔널 반환은 신중히 하라

Java8 이전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 **예외를 던지거나**, **null을 반환하는** 두 가지 방법이 있었다. 그런데 Java8로 버전이 올라가면서 **Optional<T>**라는 또 하나의 선택지가 생겼다.

# Java8 이전, 메서드가 특정 조건에 값을 반환할 수 없을 때

처리 방법에는 예외를 던지거나 null을 반환하는 두 가지의 선택지가 존재한다. 하지만 이 방법들에는 모두 단점이 존재한다. 

### 1. 예외를 던진다.

- 예외는 진짜 예외적인 상황에서만 사용해야 한다.
- 예외를 생성할 때 스택 추적 전체를 캡처하는 비용이 만만치 않다.

### 2. null을 반환한다.

- 별도의 null 처리 코드를 추가해야 한다.
- null 처리를 무시하고 저장해두면 언젠가 NullPointerException이 발생할 가능성이 있다.

# Java8 이후, 추가된 선택지 Optional<T>

Optional<T>는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 즉, Option<T>는 **원소를 최대 1개 가질 수 있는 불변 컬렉션**이다.

## Optional을 언제 쓸까?

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 상황이라면 Optional<T>를 반환하도록 할 수 있다. 이렇게 하면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다.

### 메서드 반환 타입을 T 대신 Optional<T>로 선언하는 경우 기본 규칙

기본 규칙은 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별히 처리해야 할때 T 대신 Optional<T>를 반환하도록 한다.

## 장점

Optional을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.

## 활용 예제

### **Optional을 사용하지 않은 예제**

max 메서드에 빈 컬렉션이 들어오면 IllegalArgumentException이 발생한다.

```java
// Optional을 사용하지 않은 코드
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("빈 컬렉션");
    }
    
    E result = null;
    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requiredNonNull(e);
        }
    }
    
    return result;
}
```

### Optional을 사용한 예제

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if(c.isEmpty()) {
        return Optional.empty();
    }
    
    E result = null;
    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requiredNonNull(e);
        }
    }
    
    return Optional.of(result);
}
```

빈 컬렉션의 반환 값으로는 Optional.empty()를 이용해 빈 옵셔널을 넘기고, 값이 들어있는 옵셔널은 Optional.of(value)로 생성했다. 이렇듯 Optional을 반환하여 더 유연한 로직을 작성할 수 있다.

### Stream을 활용한 예제

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

스트림의 종단 연산 중 상당수가 옵셔널을 반환한다. 위 코드에서는 max 연산이 옵셔널을 생성한다.

### Optional 반환 시 주의 사항

Optional.of(value)의 값으로 null을 넣으면 NullPointerException이 발생하니 주의하자. 또한, 옵셔널을 반환하는 메서드에서는 절대 null을 반환하면 안된다. null을 반환하지 않으려고 만들어진 Optional을 완전히 무시하는 행위이기 때문이다.

# Optional을 왜 사용할까?

앞서 말했 듯, 메서드가 특정 조건에서 값을 반환할 수 없을 때 null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야하는 기준은 무엇일까?

비검사 예외(Unchecked Exception)를 던지거나 null을 반환한다면 API 사용자가 이를 인식하지 못해 런타임시 예상치 못한 장애가 발생할 수 있다. **하지만 검사 예외(Checked Exception)를 던지면 클라이언트는 예외를 처리하는 로직을 반드시 추가해야한다.**

**옵셔널은 검사 예외(Checked Exception)와 취지가 비슷하다. 즉, 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.**

# Optional 활용

메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야한다. 그중 하나는 기본 값을 설정하는 것이다.

### 1. 기본값을 지정할 수 있다.

```java
String lastWordInLexicon = max(words).orElse("단어 없음..");
```

### 2. 원하는 예외를 던질 수 있다.

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

실제 예외가 아닌 예외 팩터리를 건낸다. 이렇게 하면 예외가 발생하지 않는 한 예외 생성 비용이 들지 않는다.

### 3. 항상 값이 채워져 있는 경우

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

옵셔널에 항상 값이 채워져 있다고 확신한다면 그냥 값을 꺼내 사용하는 방법도 있다. 하지만 잘못 판단한 것이라면 NoSuchElementException이 발생한다.

### 4. 기본값을 설정하는 비용이 아주 큰 경우

Connection과 같이 기본값 설정 비용이 아주 큰 경우라면 Supplier<T>를 인수로 받는 orElseGet을 사용하자. 값이 처음 필요할 때 Supplier<T>를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

```java
Optional.of(name).orElseGet(() -> getName());
```

# 유용한 메서드

더 특별한 쓰임에 대비한 메서드도 있다. 바로 **fliter, map, flatMap, ifPresent** 이다.

앞선 기본 메서드로 처리하기 어려워 보인다면 위의 고급 메서드들이 문제를 해결해줄 수 있을지 검토해보자.

### isPresent

isPresent는 옵셔널 객체 내부의 값이 있는 경우 true, 없는 경우 false를 리턴한다. 이 메서드는 원하는 모든 작업을 수행할 수 있지만 **신중히 사용해야한다**.  위에 언급한 메서드들로 대부분 대체할 수 있으며, 그렇게 하는 것이 더 짧고 명확한 용법에 맞는 코드가 된다. 

따라서 isPresent를 사용하기 보다는 **fliter, map, flatMap, ifPresent**로 대체해 사용하자.

# Optional을 사용하면 안되는 경우(안티 패턴)

반환값으로 옵셔널을 사용한다고 무조건 득이 되는 건 아니다.

## 1. **컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.**

예를 들어, Optional<List<T>>를 반환하기 보다는 빈 List<T>를 반환하는게 좋다. 빈 컨테이너를 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.

## 2. 박싱된 기본 타입을 담은 옵셔널을 반환하면 안 된다.

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 따라서 자바 API는 int, long, double 전용 옵셔널을 제공한다.

바로 OptionalInt, OptionalLong, OptionalDouble이다. **이렇게 대체제까지 존재하니 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.**

## 3. Optional을 컬렉션의 키, 값, 원소나 배열 원소로 사용하지 말자

Optional을 맵의 값으로 사용하면 절대 안된다. 만약 사용한다면 Map안에 키가 없다는 사실을 나타내기 모호한 상황이 발생한다.

- Key 자체가 없는 경우
- Key는 있지만 속이 빈 Optional인 경우

쓸데없이 복잡성만 높아지고 혼란과 오류 가능성만 키울 뿐이니 절대 사용하지 말자.

# 핵심 정리

- 값을 반환하지 못할 가능성이 있고, 하출할 때마다 반환값이 없을 가능성을 염두해야한다면 Optional 반환을 고려해야 하는 상황일 수 있다.
- 옵셔널 반환은 성능 저하가 뒤따르니, 성능에 민감하다면 null이나 예외를 반환하는 식의 적절한 트레이드 오프가 필요하다.
- 옵셔널은 반환값 이외의 용도로 쓰는 경우는 거의 없다.