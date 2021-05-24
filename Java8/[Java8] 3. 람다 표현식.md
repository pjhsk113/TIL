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

## 변수 캡쳐(Variable capture)

변수 캡쳐는 람다 표현식에서 외부 변수를 참조할 때 그 값을 복사해두는 것을 말한다.

이해를 위해 다음 예제를 살펴보자.

```java
public class Dummy {
    public static void main(String[] args) {
        Dummy dummy = new Dummy();
        dummy.run();
    }

    private void run() {
        // 지역 변수
        int baseNumber = 10;
        
        IntConsumer printInt = (i) -> {
            // 람다 표현식 외부의 값인 baseNumber를 참조
            System.out.println(i + baseNumber);  
        };

        printInt.accept(10);
    }
}
```

printInt는 baseNumber를 참조하고있다. 이때 변수 캡쳐가 일어난다.

그렇다면 변수 캡쳐는 왜 일어나는 것일까? 그 이유는 람다가 별도의 쓰레드에서 실행 가능하기 때문이다. 지역 변수는 JVM의 메모리 중 Stack 영역에 저장되고 각 쓰레드는 별도의 Stack을 가지고 있다. 즉, 서로 다른 쓰레드간 Stack 영역이 공유되지 않는다는 뜻이다.

만약, 람다의 쓰레드가 종료되기 전 참조하고 있는 baseNumber의 쓰레드가 먼저 종료되어 baseNumber가 Stack에서 사라진다면 이는 오류로 이어질 수 있다. 따라서 변수 캡쳐(Variable capture)를 통해 자신의 스택에 참조하고 있는 변수를 복사해두어 오류를 방지하는 것이다.

이 처럼 값을 복사해오기 때문에 참조되는 변수는 변경되면 안된다. **즉, final이어야 한다**. 참조되는 변수의 값을 변경하려고 하면 동시성 문제가 발생할 수 있어 컴파일 에러가 발생한다. 

![](https://blog.kakaocdn.net/dn/xQDZw/btq5Dac5r9g/U0yKzlQhBFMiA197WrDAk1/img.png)

### effectively final?

Java8 이전에는 final을 명시적으로 선언해줘야 했지만 Java8 이후로는 **참조 변수가 final처럼 동작한다면** 생략 가능하도록 변경됐다. 이를 **effectively final**이라 한다.

**변수 캡쳐**는 람다 표현식뿐아니라 익명 클래스나 내부 클래스에서도 사용되던 기술이다.

```java
private void run() {
        int baseNumber = 10;

        // 내부 클래스
        class LocalClass {
            void printBaseNumber() {
                System.out.println(baseNumber);
            }
        }

        // 익명 클래스
        Consumer<Integer> integerConsumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                System.out.println(baseNumber);
            }
        };
        
        // 람다 표현식
        IntConsumer printInt = (i) -> {
            System.out.println(i + baseNumber);
        };

        printInt.accept(10);
    }
}
```

하지만 람다와 다른점이 한 가지 있는데, 바로 **쉐도잉**이다.

## 쉐도잉

쉐도윙이란 '가려진다'라는 의미다. 같은 변수명이 존재할 경우 스코프에 의해서 외부의 참조가 가려지는 것이다. 

아래 예시를 살펴보자.

![](https://blog.kakaocdn.net/dn/YK0sd/btq5Dac5utD/FRntRLTYb51b3YDpXwuVT1/img.png)

내부 클래스에서 baseNumber를 참조하고 있다. 하지만 이때 내부에 같은 이름을 가진 변수를 선언해보자.

![](https://blog.kakaocdn.net/dn/mxW32/btq5GdNGiai/xNpifBKmjSsoZrKPMBxw8K/img.png)

내부 클래스에서 참조하는 변수가 달라진 것을 확인할 수 있다. 즉, 외부에 존재하는 baseNumber가 내부에 baseNumber에 의해 **가려진** 것이다.

익명 클래스도 마찬가지로 **쉐도잉**된다.

![](https://blog.kakaocdn.net/dn/cFZLDe/btq5yBWy1LL/MEzH2HLXp55czyZHwMzXG1/img.png)

하지만 람다는 조금 다르다. 내부 클래스나 익명 클래스처럼 변수를 가질 수 없는데, 그 이유는 람다의 스코프가 외부 변수와 같기 때문이다. 즉, 같은 스코프에 같은 변수를 2개 선언하려는 것과 같기 때문에 가질 수 없는 것이다.

![](https://blog.kakaocdn.net/dn/bWVeAy/btq5zFZtxxp/neNq3UU4FARX8KhsHV8ki0/img.png)

이처럼 람다 표현식은 익명 클래스, 내부 클래스와는 다른 스코프를 가진다. 따라서 람다는 쉐도잉되지 않으며, 이는 익명 클래스, 내부 클래스와의 차이점이라고 볼 수 있다.