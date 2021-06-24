# [이펙티브 자바] Item44- 표준 함수형 인터페이스를 사용하라

자바가 람다를 지원하면서 API를 작성하는 모범 사례도 바뀌었다. 대표적으로 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 **템플릿 메서드 패턴**을 예로들 수 있다. 

모던 자바에서는 템플릿 메서드 패턴 대신 **함수 객체를 받는 정적 팩터리나 생성자를 제공하는 방식을 해법으로 제시하고 있다.** 이 말은 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다는 뜻이다. 이 경우에는 **함수형 매개변수 타입을 올바르게 선택**해야 한다.

이미 자바 표준 라이브러리에는 다양한 용도의 표준 함수형 인터페이스를 제공하고 있다. 따라서 용도에 맞는 게 있다면, 직접 구현하기 보다는 표준 함수형 인터페이스를 활용하자.

# 표준 함수형 인터페이스

java.util.function은 총 43개의 함수형 인터페이스를 제공한다. 그 중 다음 기본 인터페이스 6개만 기억하면 다른 인터페이스를 유추하기 쉬워진다. 

### 1. UnaryOperator<T>

T타입 인수 하나를 받아서 T 타입을 반환하는 함수형 인터페이스

```java
public static void main(String[] args) {
	UnaryOperator<Integer> plus10 = (i) -> i + 10;
	UnaryOperator<Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

### 2. BinaryOperator<T>

T 타입 인수 두 개를 받아서 T 타입을 반환하는 함수형 인터페이스

```java
public static void main(String[] args) {
	BinaryOperator<Integer> plus = (a,b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}.
```

### 3. Predicate<T>

T타입을 받아서 boolean을 리턴하는 함수형 인터페이스

```java
public static void main(String[] args) {
	Predicate<String> isHello = (s) -> s.startWith("Hello");
	isHello.test("Hi"); // false
	isHello.test("Hello"); // true
}
```

### 4. Function<T, R>

T 타입을 받아서 R 타입을 리턴하는 함수형 인터페이스다. 인수와 리턴 타입이 다르다.

```java
public static void main(String[] args) {
	Function<Integer, String> print10 = (s) -> String.valueOf(s);
	System.out.println(print10 .apply(10)); // "10" 출력
}
```

### 5. Supplier<T>

T 타입의 값을 제공하는 함수형 인터페이스로 **공급자**라고도 불린다. 인수를 받지 않고 값을 반환한다.

```java
public static void main(String[] args) {
	Supplier<Integer> get10 = () -> 10; // 10을 리턴하겠다!
	get10.get();
}
```

### 6. Consumer<T>

T타입을 받아서 아무 값도 리턴하지 않는 함수형 인터페이스다. 소비자라고도 한다.

```java
public static void main(String[] args) {
	Consumer<Integer> printT = (i) -> System.out.println(i);
	printT.accept(10); // 10 출력
}
```

`추가`

### 7. BiFunction<T, U, R>

두 개의 입력 값(T, U)를 받아서 R 타입을 리턴하는 함수형 인터페이스

```java
public static void main(String[] args) {
	BiFunction<Integer, Integer, Integer> plus = (a, b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

기본 인터페이스는 기본 타입인 int, double, long으로 각 3개씩 변형이 생긴다. 예를 들면, int를 받는 Predicate는 IntPredicate가 되고, long을 받는 BinaryOperator는 LongBinaryOperator가 되는 식이다. 

더 많은 인터페이스를 확인하고 싶다면 [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)을 참고하자.

### 기본 함수형 인터페이스 사용 시 주의사항

**기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말자**. 동작에는 이상이 없지만 계산량이 많을 경우 성능이 처참히 저하된다.

# 그렇다면 코드를 직접 작성해야할 때는 언제일까?

표준 함수형 인터페이스를 사용하는 것이 직접 작성하는 것보다 대부분에 상황에 좋게 작용한다. 하지만 구조적으로 같은 표준 함수형 인터페이스가 있더라도 직접 작성해야 할 때가 있다.

다음 조건 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 고민해보자.

1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.

위 특징을 만족하는 대표적인 예가 바로 Comparator<T> 인터페이스다. 구조적으로는 표준 함수형 인터페이스인 ToIntBiFunction<T, U>와 같지만 Comparator<T>를  ToIntBiFunction<T, U>로 대체하지 않았다. 그 이유는 Comparator가 이름으로 용도를 아주 잘 나타내며, 반드시 지켜야할 규약을 담고있고 유용한 디폴트 메서드를 많이 제공하기 때문이다.

# @FunctionalInterface

**직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하자.** 

직접 만든 함수형 인터페이스에 @FunctionalInterface 애너테이션을 달아야하는 이유는 크게 3가지가 있다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 람다용으로 설계된 것임을 알려준다.
2. 인터페이스가 하나의 추상 메서드만을 담고 있어야 컴파일되게 해준다.
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

# 함수형 인터페이스를 API에서 사용할 때 주의 사항

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안된다. 클라이언트에게 모호함을 안겨주고 올바른 메서드를 알려주기위해 형변환을 해줘야하기 때문이다.

![](https://blog.kakaocdn.net/dn/eKsd8q/btq6V4BTODN/XTMbLIPKFm7ShouB2Ds791/img.png)

![](https://blog.kakaocdn.net/dn/Jo2rw/btq6SvtOaw8/Hu7jHUkilRY8pIGmbEW7mK/img.png)

실제로 ExecutorService의 submit 메서드는 Callable<T>와 Runnable을 받는 것을 다중정의했다. 그래서 가끔씩 올바른 메서드를 알려주기위해 형변환을 해줘야하는 상황이 생길 수 있다.

그러니 서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피하자.