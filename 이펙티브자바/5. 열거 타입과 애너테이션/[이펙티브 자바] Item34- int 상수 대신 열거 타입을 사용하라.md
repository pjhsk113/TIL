# [이펙티브 자바] Item34 - int 상수 대신 열거 타입을 사용하라

Java에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언해서 사용하는 정수 열거 패턴을 사용했다. 하지만 정수 열거 패턴에는 많은 단점이 존재한다. 

# 정수 열거 패턴의 단점

다음과 같은 정수 열거 패턴이 있다.

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

이 패턴의 단점은 무엇일까?

## 첫 째, 타입 안정성을 보장할 수 없으며 표현력도 좋지 않다.

컴파일러 입장에서는 APPLE_FUJI나 ORANGE_NAVEL은 모두 같은 0을 나타내기 때문에 동등 연산자(==)로 비교하더라도 아무런 경고를 발생시키지 않는다. 즉, **APPLE_FUJI가 전달되어야 할 값에 ORANGE_NAVEL가 전달되어도 아무런 문제없이 컴파일된다는 뜻이다.** 따라서 타입 안정성을 보장할 수 없다.

또한, 표현력도 좋지 않다. 정수 열거 패턴은 별도의 namespace를 지원하지 않으므로 접두어를 사용해 이름 충돌을 방지한다. 예를들어, mercury는 수은과 수성을 나타내는 두 가지의 의미를 가지고 있다. 이를 구분하기 위해서는 ELEMENT_MERCURY (수은), PLANET_MERCURY(수성)으로 이름을 구분지어줘야 한다. 

## 둘 째, 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

정수 열거 패턴은 상수를 나열한 것뿐이라 컴파일 후 상수의 값이 바뀐다면 다시 컴파일 해줘야 한다. 컴파일 시 값이 클라이언트 파일에 그대로 새겨지기 때문이다. 만약 상수 값이 변경된 후 다시 컴파일 하지 않는다면, 클라이언트는 엉뚱하게 동작하는 프로그램을 만날 수 있다.

## 셋 째, 문자열로 출력하기 까다롭다.

값을 출력하거나 디버깅할 때 단지 숫자로 값이 표현되기 때문에 그다지 도움이 되지 않는다.

```java
public static final int APPLE_FUJI = 0;
System.out.println(APPLE_FUJI); // 0 출력
```

상수의 의미를 표현하기 위해 문자열 열거 패턴을 사용할 수 있지만, 이는 오히려 더 안좋은 영향을 미칠 수 있다. 문자열 상수의 이름 대신 문자열  리터럴을 그대로 하드코딩하게 만들기 때문인데, 이는 오타가 있어도 컴파일러가 잡을 수 없으니 런타임에 버그를 발생시킬 수 있다.

Java는 위의 단점을 보완함과 동시에 여러 장점을 안겨줄 수 있는 **열거 타입(Enum Type)**을 제안했다.

# 열거 타입(Enum Type)

```java
// 가장 단순한 열거 타입!
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

Java의 열거 타입은 완전한 형태의 클래스이다. 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 또한, 생성자를 제공하지 않으므로 사실상 final이다. 따라서 인스턴스를 생성하거나 확장할 수 없으니, 인스턴스가 하나씩만 존재함을 보장할 수 있다. 이런 특징들을 기반으로 Java의 열거 타입은 강력한 장점을 지닌다. 

## 열거 타입의 장점

### 1. 인스턴스가 하나씩만 존재함을 보장한다.

앞서 말했 듯, 생성자를 제공하지 않기 때문에 인스턴스를 생성하거나 확장할 수 없다. 따라서 인스턴스가 하나씩만 존재함을 보장한다. 따라서 **열거 타입은 인스턴스 통제**된다. 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, **열거 타입은 싱글턴을 일반화한 형태**라고 할 수 있다.

### 2. 타입 안정성을 제공한다.

```java
// 가장 단순한 열거 타입!
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

Apple 타입을 매개변수로 받는 매서드는 Apple 타입의 값만 넘겨 받을 수 있다. 다른 타입을 넘기려 하면 컴파일 오류가 발생한다. 타입이 다른 열거 타입 변수에 할당하려 하거나 다른 열거 타입의 값끼리 동등 연산자(==)로 비교하려는 꼴이기 때문에 당연한 결과다.

### 3. namespace를 제공한다.

열거 타입에는 각자의 이름공간(namespace)이 있어서 이름이 같은 상수도 공존할 수 있다.

```java
// 이름이 같은 상수도 함께 공존할 수 있다.
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { FUJI, PIPPIN, GRANNY_SMITH }
```

또한, 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다. 필드만 공개되기 때문에 상수 값이 클라이언트에 새겨지지 않기 때문이다.

### 4. 열거 타입의 toString메서드는 출력하기에 적절한 문자열을 내어준다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdCG45w%2Fbtq4hIIUaGk%2FmH3I6gXkSJ1rAVxTHJYKg1%2Fimg.png)

![](https://blog.kakaocdn.net/dn/SQaOY/btq4lM4npf0/DKOsj36PBKTAqotMt7ZG11/img.png)

### 5. 임의의 메서드나 필드를 추가할 수 있고, 인터페이스를 구현할 수 있다.

다 섯번째 장점은 태양계의 행성을 모델링하는 사례를 통해 알아보자.

```java
public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23,3.393e6),
    JUPITER(1.899e+27,7.149e7),
    SATURN(5.685e+26,6.027e7),
    URAUS(8.683e+25,2.556e7),
    NEPTUNE(1.024e+26,2.477e7);

    // 임의의 필드
    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면중력

    //중력상수
    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    // 임의의 메서드
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

> 열거 타입은 불변이기 때문에 모든 필드는 final로 선언되어야 한다.

Planet 열거 타입은 **임의의 필드**인 질량, 반지름, 표면중력을 가지고 있다. 또 **임의의 메서드**인 surfaceWeight()를 가지고 있다. 이처럼 열거 타입에 메서드나 필드를 추가하면 고차원의 추상 개념을 완벽히 표현해낼 수 있다.

위의 예제에서는 질량, 반지름, 표면중력 필드를 가지는데 상수가 가지는 값은 질량과 반지름 뿐이다. **표면중력 필드**는 **생성자에서 데이터를 전달받아 인스턴스 필드에 저장**하고 있다. 이처럼 열거 타입 상수 각각을 특정 데이터와 연결지을 수 있다.

그리고 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 values() 메서드를 제공함으로써 강력한 기능을 제공한다.

### 6. 열거 타입의 상수를 제거해도 참조하지 않는 클라이언트는 아무 영향이 없다.

열거 타입의 상수가 제거되도 **제거된 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.** 만약 제거된 상수를 참조하더라도 컴파일 에러가 발생하니 정수 열거 타입에서는 기대할 수 없는 가장 바람직한 대응이라 볼 수 있다.

## 열거 타입을 만들 때는...

열거 타입을 선언한 클래스 혹은 패키지에서만 유요한 기능은 private 혹은 default로 구현해야한다. 이렇게 구현하면 자신을 선언한 클래스 혹은 패키지에서만 사용할 수 있는 기능을 담기 때문이다. 일반 클래스와 마찬가지로 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private 혹은 default로 구현하자.

널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 사용된다면 해당 클래스의 멤버 클래스로 만든다.

## 상수별 메서드 구현

만약 상수마다 동작이 달라져야 하는 상황이라면 어떻게 구현할 수 있을까? 

사칙연산을 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 수행하는 열거 타입 상수가 직접 수행하는 예제를 살펴보자.

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```

정상적으로 동작하지만 이 코드는 쉽게 깨질 수 있다. 새로운 상수가 추가되면 case도 함께 추가되어야 하기 때문이다. 만약 실수로 case를 추가하지 않는다면 **런타임 시 "알 수 없는 연산"이라는 에러**를 만나게 된다.

이러한 문제점을 개선하려면 **상수별 메서드 구현**을 해주면 된다. 열거 타입에 추상 메서드를 선언하고 각 상수에서 자신에 맞게 재정의하는 방법이다.

```java
public enum Operation {
    PLUS{
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}
```

apply라는 추상 메서드로 인해 새로운 상수가 추가되면 재정의를 강제한다. 재정의 되지않으면 컴파일 오류로 알려준다. 이로써 switch-case로 구현했을 때의 단점을 보완할 수 있다. 

또한, 상수별 메서드를 상수별 데이터와 결합하는 방법으로도 구현할 수 있다.

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

위의 코드에서 toString을 상수의 이름이 아닌 연산기호를 반환하도록 재정의한 것을 볼 수 있다. 이처럼 toString 메서드를 재정의하려면, toString이 반환해주는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 것을 고려해보자.

```java
private static final Map<String, Operation> stringToEnum =
            Stream.of(Operation.values())
                    .collect(Collectors.toMap(Operation::toString, operation -> operation));

// 문자열이 주어지면 그에 대한 Operation 상수 반환. 잘못된 문자열이면 null 반환
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

위의 코드에서 Operation 상수가 stringToEnum Map에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다. 열거 타입의 생성자가 실행되는 시점에는 정적 필드가 초기화되기 전이기 때문에 생성자에서 정적 필드를 참조하려고 시도하면 컴파일 에러가 발생한다. 따라서 자신의 인스턴스를 추가하지 못하게하는 제약이 존재하는 것이다.

열거 타입의 정적 필드 중 열거 타입의 생성자에 접근할 수 있는 것은 상수 변수뿐이다.

## 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 이 단점을 보완하려면 **전략 열거 타입 패턴**을 사용하자.

**전략 열거 타입 패턴**은 상수를 추가할 때 전략을 선택하도록 하는 패턴이다. private 중첩 열거 타입을 만들고 계산을 위임한다. 그리고 바깥 열거 타입 생성자에서 전략을 인자로 받게하면 된다.

```java
// 전략 열거 타입 패턴
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    // private 중첩 열거 타입
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

**전략 열거 타입 패턴**은 switch문보다 복잡하지만 안전하고 유연하다. 

다만, ****switch문을 적절하게 활용하면 좋은 선택이 되는 경우도 있다. 바로 **기존 열거 타입에 상수별 동작을 혼합해 넣는 경우이다.**

서드파티에서 가져온 Operation 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요하다고 가정해보자. 아래의 코드는 그 역할을 충실히 할 수 있는 정적 메서드의 예시이다.

```java
public static Operation inverse(Operation op) {
	switch(op) {
	  case PLUS: return Operation.MINUS;
	  case MINUS: return Operation.PLUS;
	  case TIMES: return Operation.DIVIDE;
	  case DIVIDE: return Operation.TIMES;

	  default: throw new AssertionError("알 수 없는 연산: " + op);
}
```

## 열거 타입을 언제 사용해야할까?

### 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자!

ex) 태양계 행성, 한 주의 요일, 체스 말

그리고 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다. 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었기 때문이다.