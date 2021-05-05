# [이펙티브 자바] Item14- Comparable을 구현할지 고민하라

---

Comparable을 구현하면 손쉽게 컬렉션을 정렬할 수 있다. Comparable 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 좁쌀만한 노력으로 누릴수 있는 것이다. 따라서 알파벳, 숫자, 연대와 같이 순서가 명확한 클래스를 작성한다면 반드시 Comparable을 구현하자.

## compareTo 메서드의 일반 규약

compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다.

compareTo는 기준 객체와 주어진 객체의 순서를 비교한다. 

반환값 기준

- 기준 객체 < 주어진 객체 → -1 반환
- 기준 객체 == 주어진 객체 → 0 반환
- 기준 객체 > 주어진 객체 → 1 반환

### 첫번째 규약

`x.compareTo(y) < 0` 일때 `y.compareTo(x) > 0`이다. 

따라서 `x.compareTo(y)` 가 Exception이라면 `y.compareTo(x)` 또한 Exception이 발생해야 한다.

### 두번째 규약

삼단논법.

`x.compareTo(y) < 0` 이고 `y.compareTo(z) < 0` 이라면 `x.compareTo(z) < 0` 이다.

### 세번째 규약

크기가 같은 객체끼리는 어떤 객체와 비교하든 항상 같아야한다.

`(x.compareTo(y) == 0)`  == `(x.compareTo(y))`

일반적으로, Comparable 인터페이스를 구현하면서 이 조건을 만족하지 않는 클래스는 반드시 그 사실을 명시해야 한다.

compareTo와 equals가 일관되지 않은, 즉 `(x.compareTo(y) == 0)`  == `(x.compareTo(y))` 가 지켜지지 않으면 정의된 동작과 엇박자를 낼 수 있다.

compareTo와 equals의 일관성이 지켜지지 않은 BigDecimal 클래스를 예로 생각해보자.

HashSet 객체를 만들어 거기에 new BigDecimal("1.0")과 new BigDecimal("1.00")로 만든 객체들을 추가해 보자. 그러면 집합에는 두 개의 객체가 추가된다. 이 두 객체를 equals로 비교하면 서로 다르다고 판정되기 때문이다. 하지만 HashSet 대신 TreeSet을 사용하면 집합에는 하나의 객체만 삽입된다. compareTo로 비교하면 그 두 객체는 같은 객체이기 때문이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbD0AN4%2FbtqU52RqO6B%2Fby7krm8cEMlSZzuQtbNkkk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIf5lf%2FbtqVbuzxIBr%2FjIOgFEHxg3V67p6CR1aTX0%2Fimg.png)

## compareTo 작성 요령

equals와 비슷하다. 다만 몇 가지 차이점만 주의하면 된다.

compareTo와 equals의 차이점

1. Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 컴파일타임에 인수 타입이 결정된다. 따라서 equals에서 했던 타입 확인이나 형 변환이 필요없다. 
2. null을 인자로 받는다면 NullPointerException을 발생시키면된다.

compareTo 메서드는 equals와 마찬가지로 객체 참조 필드를 비교할 때 compareTo 메서드를 재귀 호출해서 순서를 비교한다. 

### Comparator 생성 메서드

자바 7부터 compareTo 메서드에서 관계 연산자(<, >) 를 사용하는 방식을 추천하지 않는다.

자바 8에서는 Comparator 인터페이스가 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 이 비교자들을 compareTo 메서드를 구현하는데 활용할 수 있다. 코드가 깔끔하고 간결해지지만, 약간의 성능 저하가 뒤따를 수 있으니 주의해야한다.

```java
// 비교자 생성 메서드를 활용한 비교자
private static final Comparator<PhoneNumber> COMPARATOR = 
		comparingInt((PhoneNumber pn) -> pn.areaCode)
				.thenComparingInt(pn -> pn.prefix)
				.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
		return COMPARATOR.compare(this, pn);
}
```

- Comparator 생성자 메서드에서 타입 추론을 하지 않기 때문에 타입을 지정해줘야 한다.

### 값의 차를 기준으로 비교하는 비교자 - 추이성 위배

```java
static Compartor<Object> hashCodeOrder = new Comparator<>(){
  public int compare(Object o1, Object o2){
    return o1.hashCode() - o2.hashCode();
  }
}
```

이 방식은 사용하면 안된다. 정수 오버플로우를 일으키거나 부동 소수점 계산 방식에 따른 오류를 낼 수 있다.

### 값의 차를 기준으로 비교하는 비교자 - 대안

두 가지의 방식을 추천한다.

1. 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
			return Integer.compare(o1.hashCode(), o2,hashCode());
	}
};
```

2. 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = 
		Comparator.comparingint(o -> o.hashCode());
```

# 핵심 정리

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스를 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 하자.

compareTo 메서드에서 필드 값을 비교할 때 관계 연산자를 사용하지 말자. 오류가 발생하기 쉽다. 이 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparable 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.