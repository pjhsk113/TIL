# [이펙티브 자바] Item42- 익명 클래스보다는 람다를 사용하라

예전에는 함수 객체를 만드는 주요 수단으로 익명 클래스를 많이 사용했다.

> 함수 객체
추상 메서드를 하나만 담은 인터페이스의 인스턴스

```java
// 익명 클래스를 함수 객체로 사용 - 낡은 기법이다.
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2){
    return Integer.compare(s1.length(), s2.length());
  }
});
```

하지만 이 방식은 낡은 기법이고 코드가 너무 길어서 함수형 프로그래밍에 적합하지 않다.

Java8부터는 추상 메서드가 하나인 인터페이스는 특별한 대우를 받게되었다. 지금은 **함수형 인터페이스**로 부르는 이 인터페이스를 **람다 표현식**으로 만들 수 있게 된것이다.  

```java
// 람다식을 함수 객체로 사용 - 익명 클래스를 대체했다.
Collections.sort(words,
        (s1,s2) -> Integer.compare(s1.length(), s2.length()));
```

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하고 의도를 명확히 드러낸다는 장점이 있다. 또한, 컴파일러가 함수형 인터페이스의 타입을 추론해주기 때문에 타입을 생략할 수 있다. 따라서 **타입을 명시해야 코드가 더 명확할 때를 제외하고 타입을 생략하는 것이 좋다.**

다만 주의해야할 점은 컴파일러가 제네릭을 통해 타입 정보를 얻는다는 것이다. 즉, 제네릭을 사용하지 않으면 컴파일러는 타입 정보를 추론할 수 없게 된다.

# 열

# 열거 타입과 람다 표현식

```java
// 추상 메서드 구현
public enum Operation {
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

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);
}
```

위의 예제에서는 추상 메서드를 선언하고 상수 별로 다르게 동작하는 연산 코드를 구현했다. 이를 람다로 표현하면 훨씬 간결하고 깔끔하게 개선할 수 있다.

```java
// 람다 구현
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    OperationLambda(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y){
        return op.applyAsDouble(x,y);
    }
}
```

> DoubleBinaryOperator ?
Java에서 제공하는 함수형 인터페이스이다. double 타입 인수 2개를 받아 double을 리턴한다.

## 열거 타입에서 람다 사용시 주의사항

### 1. 람다는 이름이 없기 때문에 문서화를 할 수 없다.

람다는 간결한 코드와 의도를 명확히 드러낼 때 강점을 가진다. 따라서 코드 자체로 동작이 명확히 설명되지 않는거나 람다로 표현한 코드가 세 줄 이상이라면  람다를 사용하지 말자.

### 2. 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다.

열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론된다. 하지만 인스턴스는 런타임에 만들어지기 때문에 인스턴스 멤버에 접근할 수 없게 된다. 이 경우에는 상수별 클래스 몸체를 사용해야 한다.

# 람다로 대체 불가능한 부분

람다의 시대가 열리면서 익명 클래스는 설 자리가 크게 좁아진게 사실이다. 하지만 람다로 대체할 수 없는 부분이 존재한다.

### 1. 람다는 함수형 인터페이스에서만 쓰인다.

함수형 인터페이스란 추상 메서드가 한 개인 인터페이스를 말한다. 즉, 추상 메서드가 하나 이상이라면 람다를 사용할 수 없다.

### 2. 추상 클래스의 인스턴스를 만들 때는 익명 클래스를 사용해야 한다.

추상 클래스에서는 람다를 사용할 수 없기 때문에 익명 클래스를 사용해야 한다.(추상 메서드가 여러개인 인터페이스도 마찬가지)

### 3. 람다의 this는 바깥 인스턴스를 가리킨다.

람다의 스코프(scope)는 익명 클래스와 다르다. 람다는 바깥 인스턴스와 스코프가 같다. 하지만 익명 클래스는 더 좁은 범위의 스코프를 가진다. 따라서 익명 클래스의 this는 익명 클래스의 인스턴스 자신을 가리키고, 람다의 this는 바깥 인스턴스를 가리킨다. 만약 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 사용해야 한다.