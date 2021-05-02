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