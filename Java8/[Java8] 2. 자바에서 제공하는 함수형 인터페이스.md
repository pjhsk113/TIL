# [Java8] 자바에서 제공하는 함수형 인터페이스

# Java가 제공하는 함수형 인터페이스

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

T 타입을 받아서 R 타입을 리턴하는 함수형 인터페이스

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

T 타입의 값을 제공하는 함수형 인터페이스이다. **공급자**라고도 불린다. <T>에는 받아올 값의 타입을 명시한다.

```java
public static void main(String[] args) {
	Supplier<Integer> get10 = () -> 10; // 10을 리턴하겠다!
	get10.get();
}
```

get으로 값을 받아올 수 있다.