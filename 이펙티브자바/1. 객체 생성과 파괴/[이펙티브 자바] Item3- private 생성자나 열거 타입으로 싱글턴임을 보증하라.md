# [이펙티브 자바] Item3- private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

# 싱글턴?

**싱글턴**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

## 싱글턴을 만드는 두 가지 방법

### 1. 인스턴스를 **public static final 필드로 만들고 private으로 생성자를 방어한다.**

![](https://blog.kakaocdn.net/dn/bvvprv/btqNr7xlOXb/kw52PpolAP0iAw7dFGqoSK/img.png)

private 생성자는 Singleton.INSTANCE가 초기화될 떄 딱 한번만 호출된다. public이나 protected 생성자가 없기 때문에 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임을 보장한다.

장점 : 간결하기 때문에 누가봐도 싱글턴 클래스임을 한눈에 알아볼 수 있다.

![](https://blog.kakaocdn.net/dn/bzfJYm/btqNpd6DrMe/1PB2A1YDrYmKY1Q72siS30/img.png)

![](https://blog.kakaocdn.net/dn/m4S0g/btqNnwTpQhw/DFpeASfpLzbO7xWDDBoe20/img.png)

단, 리플렉션 API를 사용하면 private 생성자를 호출할 수 있어 새로운 인스턴스를 생성할 수 있다. 이를 해결하기 위해서는 생성자에서 두 번째 객체가 생성될 때 예외를 던지면 된다.

![](https://blog.kakaocdn.net/dn/bqBBg0/btqNnu2jIwa/kuuzKDizz70DXKuERL7Dc0/img.png)

### 2. 정적 팩터리 메서드를 public static 멤버로 제공한다.

![](https://blog.kakaocdn.net/dn/bPbDiu/btqNq0kWYwD/rp8ortkxAEKRTEKVYWkpO0/img.png)

위 1번의 예외(리플렉션 API)는 똑같이 적용된다.

### 장점

1. 정적 팩터리 메서드만 수정하면 언제든 싱글턴이 아니게 바꿀수 있다.
2. 제네릭 싱글턴 팩터리로 만들 수 있다. (타입에 대한 유연한 대처)
3. 공급자(Supplier)로 만들 수 있다.

---

## 두 방식의 문제점

위 둘 중 하나의 방법으로 만든 싱글턴 클래스를 **직렬화**하려면 Serializable을 구현하는 것만으로는 부족하다. **직렬화**한 인스턴스를 **역직렬화**할 때마다 새로운 인스턴스가 반환되기 때문이다.

따라서 직렬화 / 역직렬화시 싱글턴을 보장하기 위해서는 readResolve 메서드를 제공해야한다. 역직렬화는 기본 생성자를 호출하지 않고 값을 복사해서 새로운 인스턴스를 반환한다. 이때 readResolve 메서드를 통해 반환하게 된다. readResolve 메서드에 싱글턴 인스턴스를 반환하도록 하면 역직렬화 시에도 같은 인스턴스를 반환할 수 있게 된다.

```java
private Object readResolve() {
// 싱글턴 인스턴스를 반환하고, 새로운 인스턴스는 가비지 컬렉션에게 맡긴다
		return INSTANCE;
}
```

## 세번째 방법

원소가 하나인 Enum을 선언하여 사용한다.

```java
public enum Singleton {
	INSTANCE;
}
```

Enum은 public 필드 방식과 비슷하지만, 더 간결하고 복잡한 직렬화나 리플렉션 공격에서도 새로운 인스턴스가 생성되는 것을 완벽히 막아준다.

단, 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
