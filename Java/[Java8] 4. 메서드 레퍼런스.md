# [Java8] 메서드 레퍼런스

메서드 레퍼런스는 람다 표현식으로 구현할 때 단 하나의 메서드만을 호출하는 경우 축약해서 매개변수 없이 사용할 수 있도록 해준다.

```java
// 람다 표현식
(s) -> System.out.println(s);

// 메서드 레퍼런스
System.out::println;
```

기존에는 직접 구현을 통해 람다를 사용했었다.

하지만 이미 그 기능을 하는 메서드가 존재한다면 메서드 레퍼런스로 더 간단히 표현할 수 있다.

# 메서드 참조하는 방법

## 1. static 메서드 참조

`타입::스태틱 메서드` 형태로 표현한다. 

```java
public class Greeting {
  private String name;

  public static hello(String name) {
    return "hello" + name;
  }
}

public class Run {
  public static void main(String[] args) {
    // 람다로 구현 (아래와 같은 동작)
    UnaryOperator<String> hello1 = (s) -> "hello" + s;
    // 메서드 레퍼런스(스태틱 메서드)
    UnaryOperator<String> hello2 = Greeting::hello;
  }
}
```

## 2. 특정 객체의 인스턴스 메서드 참조

`객체 레퍼런스:: 인스턴스 메서드` 형태로 표현한다.

```java
public class Greeting {
  private String name;

  public hello(String name) {
    return "hello" + name;
  }
}

public class Run {
  public static void main(String[] args) {
    Greeting greeting = new Greeting();
    // UnaryOperator 타입의 메서드 레퍼런스(인스턴스 메서드) 생성
    UnaryOperator<String> hello = greeting::hello;
    // 메서드 레퍼런스 사용
    System.out.println(hello.apply("world"));
  }
}
```

## 3. 생성자 참조

`타입::new` 형태로 표현한다.

```java
public class Greeting {
  private String name;

  public Greeting() {
  }

  public Greeting(String name) {
    this.name = name;
  }
}

public class Run {
  public static void main(String[] args) {
    // 매개변수가 없는 생성자
    Supplier<Greeting> newGreeting = Greeting::new;
    // 매개변수가 있는 생성자
    Function<String, Greeting> nameGreeting = Greeting::new;
  }
}
```

메서드 레퍼런스만을 보고는 어떤 생성자를 참조하고 있는지 파악하기 힘들다. 이럴 때 정적 팩터리 메서드로 생성자에 이름을 줄 수 있다.

```java
public class Greeting {
    private String name;

    private Greeting() {
    }

    private Greeting(String name) {
        this.name = name;
    }

    public static Greeting newInstance() {
        return new Greeting();
    }

    public static Greeting from(String name) {
        return new Greeting(name);
    }
}

public class Run {
  public static void main(String[] args) {
    // 매개변수가 없는 생성자
    Supplier<Greeting> newGreeting = Greeting::newInstance;
    // 매개변수가 있는 생성자
    Function<String, Greeting> nameGreeting = Greeting::from;
  }
}
```

## 4. 임의 객체의 인스턴스 메서드 참조

`타입::인스턴스 메서드` 형태로 표현한다. 우리는 임의의 인스턴스의 메서드 레퍼런스를 사용할 수 있다. String 배열의 요소를 정렬하는 예제로 임의 객체의 인스턴스 메서드 참조에 대한 예를 살펴보자.

우리는 배열의 요소의 정렬 방법을 Comparator를 통해 지정해 줄  수있다. 전통적인 방법으로 다음과 같이 사용해왔다. 

```java
public class Run {
    public static void main(String[] args) {
       // 임의의 인스턴스들
       String[] fruits = {"apple", "orange", "banana", "grape"};
       
        // 익명 클래스 사용
        Arrays.sort(fruits, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.compareToIgnoreCase(o2);
            }
        });
    }
}
```

Java8부터는 Comparator가 Functional Interface로 변경되어 이를 람다 표현식으로 간소화 할 수 있다.

```java
public class Run {
    public static void main(String[] args) {
       // 임의의 인스턴스들
       String[] fruits = {"apple", "orange", "banana", "grape"};
       
       // 람다 사용
       Arrays.sort(fruits, (o1, o2) -> o1.compareToIgnoreCase(o2));
    }
}
```

위의 람다 표현식을 메서드 레퍼런스로 더 축약할 수도 있다. 우리가 구현한 `(o1, o2) -> o1.compareTo(o2)` 은 String에 compareTo 메서드를 사용하고 있다. 따라서 이를 별도 구현할 필요없이 메서드의 레퍼런스만으로 더 축약시킬 수 있다.

```java
public class Run {
  public static void main(String[] args) {
    // 임의의 인스턴스들
    String[] fruits = {"apple", "orange", "banana", "grape"};
   
    // 임의의 인스턴스(String)의 메서드 레퍼런스 사용
    Arrays.sort(fruits, String::compareToIgnoreCase);
  }
}
```