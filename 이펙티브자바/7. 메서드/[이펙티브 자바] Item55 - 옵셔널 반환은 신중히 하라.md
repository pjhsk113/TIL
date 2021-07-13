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

# Optional 사용 선택 기준

null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야하는 기준은 무엇일까?