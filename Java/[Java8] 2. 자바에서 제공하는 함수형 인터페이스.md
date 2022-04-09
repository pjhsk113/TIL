# [Java8] 자바에서 제공하는 함수형 인터페이스

# Java가 제공하는 주요 함수형 인터페이스

[java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)은 Java에서 자주 사용되는 함수형 인터페이스를  미리 정의해둔 패키지이다. 대표적으로 다음과 같은 함수형 인터페이스가 존재한다.

- Function<T, R>
- BiFunction<T, U, R>
- Consumer<T>
- Supplier<T>
- Predicate<T>
- UnaryOperator<T>
- BinaryOperator<T>

각 함수형 인터페이스에 대해 알아보자.

## Function<T, R>

T 타입을 받아서 R 타입을 리턴하는 함수형 인터페이스이다.

Integer 타입을 받아서 Integer 타입을 리턴하는 간단한 예제를 살펴보자.

```java
public static void main(String[] args) {
	Function<Integer, Integer> plus10 = (i) -> i + 10;
	System.out.println(plus10.apply(1)); // 11 출력
}
```

apply()를 통해 매개변수를 전달할 수 있다. 또한, 함수를 조합할 수 있는 디폴트 메서드인 andThen()과 compose()를 제공한다. andThen에 넘겨진 함수를 입력 매개변수에 나중에 적용하고 compose로 넘겨진 함수는 입력 매개변수에 먼저 적용한다. 쉽게 이해하기 위해 코드로 살펴보자.

```java
public static void main(String[] args) {
	Function<Integer, Integer> plus10 = (i) -> i + 10;
	Function<Integer, Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

andThen과 compose에는 multiply2라는 함수를 넘긴다. 이는 고차함수의 특성을 나타낸다.

andThen을 사용한 부분은 **(plus10(2)) * multiply2** 같이 표현할 수 있다. 입력 매개변수를 puls10에 먼저 적용한 후에 multiply2를 적용한다. 반면에 compose는 **(multiply2(2)) + plus10** 과 같이 나타낼 수 있다. 입력매개변수를 multiply2에 먼저 적용한 후, plus10을 적용한다.

## BiFunction<T, U, R>

두 개의 입력 값(T, U)를 받아서 R 타입을 리턴하는 함수형 인터페이스

BiFunction을 활용해 간단하게 사칙 연산을 수행하는 예제를 살펴보자.

```java
public enum Operator {
    PLUS ("+", (firstValue, secondValue) -> firstValue + secondValue),
    MINUS ("-", (firstValue, secondValue) -> firstValue - secondValue),
    MULTIPLY ("*", (firstValue, secondValue) -> firstValue * secondValue),
    DIVIDED ("/", (firstValue, secondValue) -> {
        if (secondValue == 0) {
            throw new IllegalArgumentException(ExceptionMessage.DIVIDE_BY_ZERO);
        }
        return firstValue / secondValue;
    });

    private final String operator;
    private final BiFunction<Integer, Integer, Integer> expression;

    /*
    * BiFunction은 첫번째 인자, 두번째 인자를 받아 세번째 인자로 리턴해주는 Functional Interface 이다.
    * */
    Operator(String operator, BiFunction<Integer, Integer, Integer> expression) {
        this.operator = operator;
        this.expression = expression;
    }

    /* operator로 Mapping되는 simbol과 firstValue, secondValue를 계산한 값을 리턴한다. */
    public static int result(String operator, int firstValue, int secondValue) {
        return findSymbols(operator).expression.apply(firstValue, secondValue);
    }

    /* filter를 통해 연산 기호를 찾는다. */
    public static Operator findSymbols(String operate) {
        return Arrays.stream(values())
                .filter(operator -> operator.operator.equals(operate))
                .findAny()
                .orElseThrow(() -> new IllegalArgumentException(ExceptionMessage.NOT_ARITHMETIC_SIMBOL));
    }
}
```

BiFunction을 활용해 추출된 연산 기호에 맞는 연산을 수행하는 로직이다. firstValue(T)와 secondValue(U)를 입력 받아 연산이 수행된 값(R)을 리턴한다.

## Consumer<T>

T타입을 받아서 아무값도 리턴하지 않는 함수형 인터페이스이다. **소비자**라고도 한다.

```java
public static void main(String[] args) {
	Consumer<Integer> printT = (i) -> System.out.println(i);
	printT.accept(10); // 10 출력
}
```

accept를 통해 값을 전달할 수 있다.

## Supplier<T>

T 타입의 값을 제공하는 함수형 인터페이스이다. **공급자**라고도 불린다. T에는 받아올 값의 타입을 명시한다.

```java
public static void main(String[] args) {
	Supplier<Integer> get10 = () -> 10; // 10을 리턴하겠다!
	get10.get();
}
```

get으로 값을 받아올 수 있다.

## Predicate<T>

T타입을 받아서 boolean을 리턴하는 함수형 인터페이스이다.

```java
public static void main(String[] args) {
	Predicate<String> isHello = (s) -> s.startWith("Hello");
	isHello.test("Hi"); // false
	isHello.test("Hello"); // true
}
```

test()로 값을 넘길 수 있으며, 함수 조합용 메서드인 or, and, negate 등을 통해 조합을 할 수 있다. 

## UnaryOperator<T>

앞서 살펴본 Function<T, R>의 특수한 형태로, Function<T, R>의 입력 타입 T와 리턴타입 R이 같을 경우 UnaryOperator<T>로 사용할 수 있다. 

```java
// before
public static void main(String[] args) {
	Function<Integer, Integer> plus10 = (i) -> i + 10;
	Function<Integer, Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

```java
// after
public static void main(String[] args) {
	UnaryOperator<Integer> plus10 = (i) -> i + 10;
	UnaryOperator<Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

위 두개의 코드는 같은 동작을 나타낸다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcEZeBW%2Fbtq4bXTAHgw%2FAWBuljcwgGcaAksLPoMXRk%2Fimg.png)

UnaryOperator는 Function을 상속받았기 때문에 디폴트 메서드인 andThen()과 compose() 모두 사용할 수 있다.

## BinaryOperator<T>

BinaryOperator<T>는 BiFunction<T, U, R>의 특수한 형태로써, 입력과 출력의 타입이 다를 거라는 가정하에 만들어진 BiFunction과는 다르게 타입이 하나로 사용될 것을 고려해 만들어진 함수형 인터페이스이다.

```java
public static void main(String[] args) {
	// BiFunction<Integer, Integer, Integer> plus = (a, b) -> a + b;
	BinaryOperator<Integer> plus = (a,b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

BiFunction<Integer, Integer, Integer>과 BinaryOperator<Integer>는 같은 동작을 한다.

# 그외에 함수형 인터페이스

Java는 위에서 설명한 주요 함수형 인터페이스 외에도 굉장히 많은 함수형 인터페이스를 제공하고있다.(**BiConsumer<T, U>, DoubleBinaryOperator, BooleanSupplier**.... 등)

[java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)에서 더 다양한 함수형 인터페이스 종류를 알아볼 수 있다. 앞서 공부한 주요 함수형 인터페이스를 이해하고 살펴보면, 다른 함수형 인터페이스도 쉽게 이해할 수 있다.