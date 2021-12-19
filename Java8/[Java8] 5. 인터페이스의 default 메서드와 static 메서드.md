# [Java8] 인터페이스의 default 메서드와 static 메서드

Java8부터는 인터페이스에 추상 메서드 선언뿐만 아니라 구현체를 제공할 수 있게 되었다.

## default 메서드

### dfault 메서드의 필요성

Java8 이전에는 인터페이스에 추상메서드가 추가되면 해당 클래스를 구현한 모든 클래스에서 다시 재정의를 해줘야하는 불편함이 있었다.

예를 들어, 다음과 같은 구조를 가진 클래스가 있다고 생각해보자.

```java
public interface Printable {

    void print();	

}
```

```java
public class PrintName implements Printable {

    @Override
    public void print() {
        System.out.println("print name");
    }
}

public class PrintAge implements Printable {

    @Override
    public void print() {
        System.out.println("print age");
    }
}
```

이때 인터페이스에 새로운 추상 메서드가 추가된다면 Printable을 구현한 모든 클래스는 이를 구현해야한다. 구현하지 않으면 컴파일 에러가 발생한다.

```java
public interface Printable {

    void print();	

    // 새로운 추상 메서드 추가!
    void printSomething();
}
```

```java
// 컴파일 오류 - printSomething()을 구현해야한다.
public class PrintName implements Printable {

    @Override
    public void print() {
        System.out.println("print name");
    }
}

// 컴파일 오류 - printSomething()을 구현해야한다.
public class PrintAge implements Printable {

    @Override
    public void print() {
        System.out.println("print age");
    }
}
```

지금처럼 2개의 클래스라면 별 문제가 되지 않겠지만, 해당 인터페이스를 구현한 클래스가 100개라면 어떨까?
100개의 클래스 모두에 추가된 추상 메서드를 구현해야한다. 

하지만 default 메서드를 활용하면 간단하게 해결된다.

```java
public interface Printable {

    void print();	

    // default 메서드 추가
    default void printSomething() {
        System.out.println("print something");
    }
}
```

Printable을 구현한 모든 클래스는 printSomething()이라는 메서드를 가지게 된다.

### 추상 골격 클래스

복잡한 인터페이스라면 구현의 유틸성을 올리기 위한 방법으로 **추상 골격 클래스**를 생각해볼 수 있다.
Collection 인터페이스도 마찬가지로 AbstractCollection이란 추상 골격 클래스를 통해 유연함을 제공했다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled.png)

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%201.png)

이러한 구조는 하위 클래스에서 모든 메서드를 구현하지 않아도 되는 유연함을 제공해주지만 단점도 존재한다.
자바는 클래스간 단일 상속만을 지원하기 때문에 추상 클래스인 골격 클래스들도 단일 상속을 피할 수 없다. 
따라서 더 이상의 확장을 할 수 없다는 제약이 생긴다.

이러한 문제도 default 메서드로 해결할 수 있다.

### default 메서드의 장점

default 메서드는 앞서 살펴본 추상 골격 클래스의 기능들을 모두 지원하면서도 다중 상속이라는 확장성까지 가질 수 있게 해준다.(인터페이스는 다중 상속이 가능하다.)

인터페이스에 구현체를 가지게함으로써 해당 인터페이스를 구현한 클래스를 깨트리지 않고 새로운 기능을 손쉽게 추가할 수있다.

대표적으로 Collection 인터페이스의 removeIf()나 stream()이 default 메서드로 구현되어 있다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%202.png)

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%203.png)

default 메서드와 추상 골격 클래스를 잘 활용하면 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.

### default 메서드의 특징

**default 메서드는 구현체에서 재정의할 수 있다.**

다중 상속한 인터페이스의 default 메서드가 중복되는 경우 해당 구현체에는 컴파일 오류가 발생한다. 
여러개의 default 메서드 중 어떤 메서드를 사용해야할지 컴파일러는 알 수 없기 때문이다.
따라서 이러한 경우 default 메서드를 재정의해야 한다.

```java
public interface Foo {
    default void printSomething() {
        System.out.println("something foo");
    }
}

public interface Bar {
    default void printSomething() {
        System.out.println("something bar");
    }
}

public class Anything implements Foo, Bar {

    @Override
    default void printSomething() {
        System.out.println("something");
    }
}
```

**default 메서드를 다시 추상 메서드로 변경할 수 있다.**

특정 default 메서드를 제공하고 싶지 않은 경우 다음과 같이 다시 추상 메서드로 변경할 수 있다.

```java
public interface Foo {
    default void printSomething() {
        System.out.println("something foo");
    }
}

public interface Bar extends Foo {
    void printSomething();
}

public class Anything implements Bar {

    @Override
    default void printSomething() {
        System.out.println("something");
    }
}
```

위와 같이 구현하면 Bar를 구현하는 모든 구현체는 printSomething()을 재정의해야 한다.

### default 메서드 주의사항

default 메서드는 해당 인터페이스를 구현한 클래스를 깨트리지 않고 새로운 기능을 추가할 수 있게 해주지만, 항상 그 기능이 올바르게 동작한다는 보장은 해주지 않는다. default 메서드는 구현 클래스가 모르게 추가된 기능이기 때문에 리스크(런타임 오류)가 존재한다.

따라서 default 메서드를 추가할 때는 항상 **@implSpec을 활용한 문서화**를 해야한다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%204.png)

equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공해서는 안되며 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다.

## static 메서드

static 메서드는 유틸리티나 헬퍼 메서드를 제공할 때 많이 사용한다.
default 메서드와 달리 재정의가 불가능하다.

```java
public interface Printable {

    static void printAnything() {
        System.out.println("print anything!");
    }
}

public class Application {
    public static void main(String[] args) {
        Printable.printAnything();
    }
}
```

기본적으로 클래스의 정적 메서드와 같으며, 가장 대표적으로 List.of를 static 메서드의 예로 들 수 있다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%205.png)