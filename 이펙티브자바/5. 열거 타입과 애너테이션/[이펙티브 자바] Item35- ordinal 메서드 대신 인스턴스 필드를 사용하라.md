# [이펙티브 자바] Item35- ordinal 메서드 대신 인스턴스 필드를 사용하라

열거 타입은 해당 상수가 몇 번째 위치에 존재하는지 반환하는 ordinal() 메서드를 제공한다. ordinal() 메서드는 Enum 클래스 내부에 존재하는 상수인 ordinal을 반환한다.

![](https://blog.kakaocdn.net/dn/dFd68o/btq4nNRG2VI/Z8MfuBExtDUV6bgQWWP2X0/img.png)

Enum의 API 문서를 보면 ordinal에 대해 이렇게 쓰여 있다. **"대부분의 프로그래머는 이 메서드를 쓸 일이 없다. 이는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다."**  따라서 이런 용도가 아니라면 **ordinal 메서드를 절대 사용해서는 안된다.**

# ordinal 메서드를 사용하면 안되는 이유

열거 타입 상수와 연결된 정수값이 필요하면 orrdinal 메서드를 이용하고 싶을 수 있다. 예를 들어, 합주단의 종류를 연주자가 1명인 솔루부터 10명인 디텍트까지 정의한 열거 타입을 살펴보자.

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

상수의 위치(연주자의 수)가 정상적으로 반환되지만 이는 유지보수에 극히 안좋은 코드이다. 만약 상수 선언 순서를 바꾼다면 numberOfMusicians가 제대로 동작하지 않을 것이다. 

```java
// SOLO와 DUET의 위치를 변경했다.
public enum Ensemble {
    DUET, SOLO, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

![](https://blog.kakaocdn.net/dn/b85ITe/btq4rHhDi4p/E4IbtImHm4geILA4zKVjDk/img.png)

SOLO 즉, 1명의 연주자가 출력될 것이라 기대하지만 DUET과 위치가 변경되면서 오작동을 일으킨다.

또한, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다. 8명의 연주자가 연주하는 8중주를 예로 들면, 상수에는 이미 8중주를 나타내는 OCTET이 선언되어 있다. 그런데 만약 똑같이 8명이 연주하는 복4중주(DOUBLE_QUARTET)를 추가하고 싶다면 어떨까? 당연히 추가할 수 없다. 이미 8을 나타내는 OCTET이 존재하기 때문이다.

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, **OCTET**, **DOUBLE_QUARTET**, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}

// 같은 8을 출력하고 싶지만 의도대로 동작하지 않는다!!! 
public static void main(String[] args) {
    System.out.println(Ensemble.OCTET.numberOfMusicians());  // 8 출력
    System.out.println(Ensemble.DOUBLE_QUARTET.numberOfMusicians()); // 9 출력
}
```

이처럼 ordinal() 메서드를 사용하는 것은 코드가 깔끔하지 못할 뿐만 아니라, 실용성이 떨어진다.

# 해결 방법

이 문제의 해결책은 굉장히 간단하다. 열거 타입 상수에 연결된 값을 인스턴스 필드에 저장해서 사용하면 된다.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), **OCTET**(8), **DOUBLE_QUARTET**(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;

    Ensemble(int size) {
      this.numberOfMusicians = size;
    }

    public int numberOfMusicians() { 
      return numberOfMusicians; 
    }
}
```