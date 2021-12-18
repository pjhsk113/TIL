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

default 메서드가 추가되기 전에는 이러한 불편함을 해소하고 사용의 유틸성을 올리기위해 **추상 골격 클래스**를 사용했다. 

### 추상 골격 클래스

Collection 인터페이스도 마찬가지로 AbstractCollection이란 추상 골격 클래스를 통해 유연함을 제공했다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled.png)

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%201.png)

이러한 구조는 하위 클래스에서 모든 메서드를 구현하지 않아도 되는 유연함을 제공해주지만 단점도 존재한다.
자바는 클래스간 단일 상속만을 지원하기 때문에 추상 클래스인 골격 클래스들도 단일 상속을 피할 수 없다. 
따라서 더 이상의 확장을 할 수 없다는 제약이 생긴다.

이러한 문제를 한 번에 해결하는 방법이 바로 default 메서드이다.

### default 메서드의 장점

default 메서드는 앞서 살펴본 추상 골격 클래스의 기능들을 모두 지원하면서도 다중 상속이라는 확장성까지 가질 수 있게 해준다.(인터페이스는 다중 상속이 가능하다.)

인터페이스에 구현체를 가지게함으로써 해당 인터페이스를 구현한 클래스를 깨트리지 않고 새로운 기능을 손쉽게 추가할 수있다.

대표적으로 Collection 인터페이스의 removeIf()나 stream()이 default 메서드로 구현되어 있다.

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%202.png)

![Untitled](%5BJava8%5D%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20default%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%E1%84%8B%E1%85%AA%20static%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%20ab5676d09f2549dfbc60082690a50a6c/Untitled%203.png)

### default 메서드의 특징