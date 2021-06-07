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