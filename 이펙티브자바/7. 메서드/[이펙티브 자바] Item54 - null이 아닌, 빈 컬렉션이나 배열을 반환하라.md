# [이펙티브 자바] Item54 - null이 아닌, 빈 컬렉션이나 배열을 반환하라.

컬렉션이나 배열같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 항상 방어 코드를 넣어줘야 한다. 다음 예제를 살펴보자.

```java
private final List<Cheese> cheesesInStock = ...;

/**
* @return 매장 안의 모든 치즈가 목록을 반환한다.
* 단, 재고가 없다면 null을 반환한다.
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}
```

```java
// 클라이언트 코드
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && chheeses.contains(Cheese.STILTON)) {
    System.out.println("good");
}
```

클라이언트 코드에서 null 체크 코드를 추가적으로 작성해야한다. 이 방어 코드를 빼먹으면 오류가 발생할 수 있기 때문에 항상 명시해줘야함으로 코드가 더 복잡해진다.

빈 컨테이너를 할당하는 것에도 비용이 발생하니 null을 반환하는게 더 낫다고 생각할 수 있다. 하지만 이 주장은 두 가지 측면에서 틀렸다고 볼 수 있다.

### 1. 성능 저하의 주범이라고 확인되지 않는 한, 이 정도 성능차이는 무의미하다.

### 2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 반환할 수 있다.

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```

비어있는 불변 컬렉션을 반환하는 **Collections.emptyList()**를 활용하면 **매번 같은 빈(empty) 컬렉션을 반환할 수 있다.** 불변 객체는 자유롭게 공유해도 안전하기 때문에 성능 최적화가 가능해진다.

단, 최적화가 필요없는 일반적인 상황이라면 다음과 같이 사용할 수 있다.

```java
// 빈 컬렉션을 반환하는 올바른 예
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

배열을 사용할 때도 마찬가지이다. null을 반환하지말고 길이가 0인 배열을 반환하자.

```java
// 길이가 0일 수도 있는 배열을 반환하는 올바른 방법
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

만약 이 방식이 성능을 떨어트릴 것 같다면 길이 0짜리 배열을 캐싱해두고 그 배열을 반환하자.

```java
// 최적화
private final static Cheese [] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

길이 0인 배열은 모두 불변이기 때문에 안심하고 사용할 수 있다. 

toArray에 넘기는 배열을 **미리 할당하는 방식**은 오히려 성능을 떨어트릴 수 있다. 단순히 성능 개선이 목적이라면 아래와 같은 방식은 추천하지 않는다.

```java
// 나쁜 예
 return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

# 핵심 정리

null이 아닌, 빈 배열이나 컬렉션을 반환하자. 

null을 반환하는 API는 성능이 좋은 것도 아니고, 사용하기 어려우며 방어 코드도 늘어난다.