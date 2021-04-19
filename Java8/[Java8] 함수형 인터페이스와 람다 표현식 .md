# [Java8] 함수형 인터페이스와 람다 표현식

# 함수형 인터페이스(Functional Interface)

함수형 인터페이스는 **추상 메서드**를 **하나만** 가진 인터페이스를 말한다.

```java
// 함수형 인터페이스!
public interface DoSomething {
    
    void doAnything();
}
```

Java 8에 새롭게 추가된 default 메서드나 static 메서드가 함께 존재한다고 하더라도 추상메서드가 하나라면 함수형 인터페이스이다.

```java
// 함수형 인터페이스!
public interface DoSomething {

    void doAnything();

    default void printDefault() {
        System.out.println("default");
    }

    static void printStatic() {
        System.out.println("static");
    }
}
```

만약 함수형 인터페이스를 정의한다면, Java Standard Library에서 제공하는 **@FunctionalInterface** 애노테이션을 명시적으로 붙여주는 것이 좋다.

```java
@FunctionalInterface
public interface DoSomething {

    void doAnything();
}
```

해당 애노테이션으로 얻을 수 있는 이점은 함수형 인터페이스라는 것을 명확히 명시하는 것과 함수형 인터페이스를 위반했을 경우 컴파일 에러를 바로 알 수 있어 인터페이스를 조금 더 견고하게 관리할 수 있다는 점이다.

![%5BJava8%5D%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%E1%84%92%E1%85%A7%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%20%E1%84%91%E1%85%AD%E1%84%92%E1%85%A7%E1%86%AB%E1%84%89%E1%85%B5%E1%86%A8%209d6735fef47c46a999021256e2e13203/Untitled.png](%5BJava8%5D%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%E1%84%92%E1%85%A7%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%20%E1%84%91%E1%85%AD%E1%84%92%E1%85%A7%E1%86%AB%E1%84%89%E1%85%B5%E1%86%A8%209d6735fef47c46a999021256e2e13203/Untitled.png)

# 람다 표현식(Lambda Expressions)

그렇다면 위에서 정의한 함수형 인터페이스를 어떻게 사용할까? 

Java8 이전에는 아래와 같이 **익명 내부 클래스**를 만들어 사용했다.

```java
public class Test {
    public static void main(String[] args) {
				// 익명 내부 클래스!
        DoSomething doSomething = new DoSomething() {
            @Override
            public void doAnything() {
                System.out.println("Do Anything!");
            }
        };
    }
}
```

하지만 Java8부터는 위와 같은 문법을 줄여쓸 수 있는 **람다 표현식** 문법을 제공한다.

```java
public class Test {
    public static void main(String[] args) {
        DoSomething doSomething = () -> System.out.println("Do Anything!");
    }
}
```

코드량이 줄어들고 가독성도 좋아졌다. 

조금 더 복잡한 코드라도 다음과 같이 줄일 수 있다. 

아래의 코드는 함수형 인터페이스인 Comparator를 구현한 예시이다.

```java
// Before
public class Test {
    public static void main(String[] args) {
        Collections.sort(session, new Comparator<List<Integer>>() {
            @Override
            public int compare(List<Integer> o1, List<Integer> o2) {
                if (o1.get(1) > o2.get(1)) {
                    return 1;
                } else if (o1.get(1) < o2.get(1)){
                    return -1;
                }

                return o1.get(0) - o1.get(1);
            }
        });
    }
}
```

```java
// After
public class Test {
    public static void main(String[] args) {
        Collections.sort(session, (o1, o2) -> {
            if (o1.get(1) > o2.get(1)) {
                return 1;
            } else if (o1.get(1) < o2.get(1)){
                return -1;
            }

            return o1.get(0) - o1.get(1);
        });
    }
}
```

이런 형태의 람다 표현식은 메서드의 파라미터로 전달하거나 람다 자체를 리턴하는 것도 가능하다.

# Java의 함수형 프로그래밍

함수형 프로그래밍 패러다임은 다음 조건을 만족해야한다.

- 일급 객체(일급 시민)
- 순수 함수
- 고차 함수
- 불변성

### 일급 객체 (First Class Object)

다른 엔티티가 일반적으로 사용할 수 있는 모든 작업들을 지원하는 엔티티를 말한다. 변수에 담을 수 있고 파라미터로 전달할 수 있으며, 반환값으로 전달할 수 있다. 

즉, Java에서는 객체를 뜻한다. 람다라는 특수한 형태의 Object를 일급 객체(일급 시민)로 취급하며 함수형 인터페이스라는 약속 아래에서 객체를 함수처럼 작성할 수 있는 것이다.

### 순수 함수

부수 효과가 없는 함수를 말한다. 즉, 입력받은 값이 동일한 경우 결과 값이 같아야한다. 만약 결과가 다르다면 함수형 프로그래밍이라 보기 어렵다. 

함수 외부에 있는 값을 변경하지 않으며, 상태가 없는 함수를 **순수 함수**라 부른다.

아래의 코드는 순수 함수가 아닌 경우를 나타낸다. 

**baseNumber라는 함수 외부의 값을 의존하고 있고 함수 내부에 상태를 가지고 있는 경우**

```java
// 순수 함수 X
public class Test {
    public static void main(String[] args) {
        DoSomething doSomething = new DoSomething() {
            int baseNumber = 10;

            @Override
            public int doAnything(int number) {
                return number + baseNumber; // 함수 외부의 상태 값을 의존
            }
        };
    }
}
```

**함수 외부의 값을 변경하려는 경우**

```java
// 순수 함수 X
public class Test {
    public static void main(String[] args) {
        DoSomething doSomething = new DoSomething() {
            int baseNumber = 10;

            @Override
            public int doAnything(int number) {
                baseNumber++;  // 함수 외부의 값을 변경
                return number + baseNumber;
            }
        };
    }
}
```

### 고차 함수

함수가 함수를 매개변수로 받을 수 있고 함수를 리턴할 수도 있다.

```java
(int value1, String value2) -> value1 + value2.length()
```