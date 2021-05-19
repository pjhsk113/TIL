# [이펙티브 자바] Item38- 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

**열거 타입**은 거의 모든 상황에서 **타입 안전 열거 패턴(typesafe enum pattern)**보다 우수하다. 단, 예외가 있다. 열거 타입은 확장을 할 수가 없다는 점이다. 즉, 타입 안전 열거 패턴은 확장을 통해 다음 값을 추가하고 다른 목적으로 사용할 수 있는 반면, 열거 타입은 그렇게 할 수없다.

하지만 대부분 상황에서 열거 타입을 확장하는 것은 좋지 않은 생각이다.

## 일반적으로 열거 타입 확장이 안좋은 이유

1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 아니다.
2. 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
3. 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡해진다.

## 그렇다면 열거 타입 확장이 필요한 경우는 무엇일까?

확장할 수 있는 열거 타입은 **연산 코드**에 잘 어울린다. API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줄 때 유용하게 사용할 수 있다. 열거 타입으로도 이 효과를 구현해낼 수 있다. 바로 **열거 타입이 인터페이스를 구현할 수 있다는 점을 이용하는 것이다.**

다음 예시를 통해 더 자세히 알아보자.

```java
public interface Operation {
	double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

위와 같이 열거 타입인 BasicOperation이 Operation 인터페이스를 연산의 타입으로 사용하게 한다. BasicOperation은 확장할 수 없지만, 인터페이스는 확장할 수 있기 때문에 다른 연산 타입이 필요하다면 Operation을 구현한 다른 열거 타입을 정의해 이를 대체할 수 있다.

예를 들어 앞의 연산 타입을 확장해 지수연산과 나머지 연산을 추가해보자.

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

이처럼 다른 연산 타입이 필요하다면 우리가 해야할 일은 Operation 인터페이스를 구현한 다른 열거 타입을 정의하는 것 뿐이다. 새로 정의한 연산은 기존 연산(BasicOperation)을 사용하던 곳이면 어디에든 사용할 수 있다.

## 확장된 열거 타입의 원소 모두를 사용하게 하는 방법

확장된 열거 타입의 원소 모두를 사용하게 하는 방법은 두 가지가 있다.

1. Class 객체에 한정적 타입 토큰을 넘기는 방법
2. Class 객체 대신 한정적 와일드카드 타입을 넘기는 방법

### 1. Class 객체에 한정적 타입 토큰을 넘기는 방법

먼저 첫 번째 방법에 대해 알아보자. 

```java
public static void main(String[] args) (
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
	}
}
```

확장된 열거 타입의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. 이때 class 리터럴은 한정적 타입 토큰으로 사용된다.

> **<T extends Enum<T> & Operation>**
Class 객체가 열거 타입인 동시에 구현한 인터페이스의 하위 타입이어야 한다.

### 2. Class 객체 대신 한정적 와일드카드 타입을 넘기는 방법

두 번째 대안인 Class 객체 대신 한정적 와일드카드 타입을 넘기는 방법은 다음과 같다.

```java
public static void main(String[] args) (
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```

코드가 덜 복잡하고 test 메서드가 여러 구현 타입의 연산을 조합해 호출할 수 있게 되어 조금 더 유연해졌다. 하지만 특정 연산에서 EnumSet과 EnumMap을 사용하지 못한다.

## 확장 가능한 열거 타입을 흉내 내는 방식의 문제점!

열거 타입끼리 구현을 상속할 수 없다는 사소한 단점이 존재한다. 이 단점은 아무 상태에도 의존하지 않는 경우 인터페이스에 디폴트 메서드를 정의해 극복할 수 있다. 반면, 앞서 살펴본 Operation 예제에서는 연산 기호를 저장하고 찾는 로직이 각각 들어가야한다. 이 예제에서는 중복량이 적어 문제되지 않지만 공유하는 기능이 많다면 별도의 도우미 클래스나 정적 도우미 메서드로 분리해서 중복을 줄여야 한다.

# 핵심 정리

열거 타입은 확장할 수 없지만, 확장할 수 있는 열거 타입이 필요한 경우라면 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 정의해 확장한 열거 타입과 같은 효과를 낼 수 있다.

자바 라이브러리에도 이와 같은 패턴으로 구현한 예제(java.nio.file.LinkOption)가 있듯, 확장할 수 있는 열거 타입이 필요하다면 인터페이스를 사용하자.