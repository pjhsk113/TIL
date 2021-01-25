# [이펙티브 자바] Item5- 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

Java의 객체지향은 많은 클래스가 하나 이상의 자원에 의존하게된다. 

이때 의존성을 코드에 명시할 경우 의존성이 클라이언트 코드에 강하게 결합되어 유연하지 않고 테스트 하기 어려워진다.

## 자원을 코드에 직접 명시할 경우 문제점

다음과 같이 정적 유틸로 만들어진 결제 머신 클래스가 있다고 생각해보자.

```java
public class PaymentMachine {
    private static final Pay pay = new KakaoPay();

    private PaymentMachine() { } // 생성자 방어

    public static Payment payment() {
        /* 구현 */
    }
}
```

이 결제 머신은 카카오페이로 결제할 때 아주 잘 동작한다. 카카오페이를 직접적으로 의존하여 카카오페이라는 자원을 가지고 있기 때문이다.

비슷하게, 싱글턴으로 만들어진 클래스도 마찬가지이다.

```java
public class PaymentMachine {
    private final Pay pay = new KakaoPay();

    private PaymentMachine() { } // 생성자 방어
		public static PaymentMachine INSTANCE = new PaymentMachine();

    public Payment payment() {
        /* 구현 */
    }
}
```

하지만 다른 결제 방식을 선택한다면 어떨까? 가령 네이버페이나 제로페이같은 다른 페이로 결제를 시도한다면 결제는 실패할 것이다.

이 결제 머신은 카카오페이만 지원하고 있기 때문이다.

## 해결 방법?

이러한 문제를 해결하기 위해 아래와 같이 클래스를 수정해보자.

```java
public class PaymentMachine {
    private Pay pay = new KakaoPay();

    private PaymentMachine() { } // 생성자 방어

    public Payment payment() {
        /* 구현 */
    }

		public void changePay(Pay pay) {
			this.pay = pay;
		}
}
```

Pay의 final을 제거하고 changePay 메서드를  추가해서 Pay를 재할당할 수 있게 되었다. 

하지만 이 방법은 멀티  쓰레드 환경에서 Pay에 대한 예측을 할 수 없기 때문에 의도하지 않은 오류를 내기 쉽다는 **문제점이 존재**한다. **setter 메서드도 마찬가지이다.**

사용자의 자원에 따라 동작이 달라지는 클래스에서는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다. 대신 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야한다. 즉, 클라이언트 측에서 Pay를 바꿔줄 수 있어야 한다.

## 그렇다면 진짜 해결 방법은?

바로 **의존 객체 주입 패턴**이다. 이 패턴은 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 아주 간단한 패턴이다.

```java
public class PaymentMachine {
    private final Pay pay;

    public PaymentMachine(final Pay pay) {
        this.pay = Objects.requireNonNull(pay);
    }

    public static Payment payment() {
        /* 구현 */
    }
}
```

위와같은 의존 객체 주입 방식은 다음과 같은 장점을 가진다.

1. 자원이 몇개든, 의존 관계가 어떻든 문제없이 잘 동작한다.
2. 불변을 보장하여 여러 클라이언트가 안심하고 자원(의존 객체)을 공유할 수 있다.

**의존 객체 주입 패턴의 비슷한 변형**으로 생성자에 자원 팩터리를 넘겨 줄 수 있다.

> 자원 팩터리란?

호출할 때마다 자원의 인스턴스를 반복해서 만들어주는 객체를 말한다.

자원 팩터리를 완벽하게 구현한 예제로는 Java8에서 소개된 Supplier 인터페이스가 있다.

이를 응용하면, 다음과 같은 클래스가 만들어진다.

```java
public class PaymentMachine {
    private final Pay pay;

    public PaymentMachine(Supplier<? extends Pay> pay) {
        this.pay = pay.get();
    }

    public static Payment payment() {
        /* 구현 */
    }
}
```

일반적으로 한정적 와일드 카드 타입을 통해 팩터리 타입 매개변수를 제한하는 것이 특징이다. 이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

간단한 변경으로 **결제 머신**의 Pay 생성 팩터리를 만들어 주입한 결과, Pay의 하위 타입이라면 무엇이든 생성할 수 있게 되었다. 이제 네이버페이든 제로페이든 다양한 페이를 사용할 수 있는 제대로 된 결제 머신이 된 것이다.