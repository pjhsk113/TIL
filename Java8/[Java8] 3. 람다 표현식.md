# [Java8] 람다 표현식

람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다. 즉, 컴파일러의 추론에 의지해서 코드를 단순화하는 것이다.

람다 표현식의 기본 구조는 다음과 같다.

```java
(i) -> i + 10;
```

**(인자 리스트) -> { 바디 }**의 형태를 가진다. 위의 예제처럼 바디가 한 줄로 표현될 경우 { }와 return문을 생략할 수 있다. 하지만 여러 줄이라면 { }와 return문을 명시적으로 선언해줘야 한다.

```java
(x, y) -> {
   System.out.println("x + y");
   return x + y;
}
```

## 인자 리스트

### 1. 인자가 없을 경우

구현해야하는 추상 메서드가 파라미터를 아무것도 받지 않는 경우에 사용한다.

```java
Supplier<Integer> getSomeNumber = () -> 10;
```

### 2. 인자가 하나인 경우

```java
UnaryOperator<Integer> plus10 = (i) -> i + 10;
```

### 3. 인자가 두개인 경우

```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
```

인자 리스트에 타입을 컴파일러가 추론하기 때문에 생략 가능하다. 하지만 명시할 수도있다.

```java
BinaryOperator<Integer> sum = (Integer a, Integer b) -> a + b;
```

## 변수 캡쳐