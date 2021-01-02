# [이펙티브 자바] Item3- private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

# 싱글턴?

**싱글턴**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

## 싱글턴을 만드는 두 가지 방법

### 1. 인스턴스를 **public static final 필드로 만들고 private으로 생성자를 방어한다.**

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f7e6d1e8-cd4c-4aa4-9c22-a2498b42b88b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210102%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210102T052648Z&X-Amz-Expires=86400&X-Amz-Signature=cfd6a167a47dad526dec15c3935c72215383dba5460805e44030e827fcb78dad&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

private 생성자는 Singleton.INSTANCE가 초기화될 떄 딱 한번만 호출된다. public이나 protected 생성자가 없기 때문에 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임을 보장한다.

장점 : 간결하기 때문에 누가봐도 싱글턴 클래스임을 한눈에 알아볼 수 있다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/68da23a1-2b8d-4071-a30c-284bfb5a1626/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210102%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210102T052655Z&X-Amz-Expires=86400&X-Amz-Signature=9b0c8386c3df5ce4f00bdcd3bc0855c27d824ce6808bb9f29c87452588afb88f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/24e6f5d7-497f-4e6d-bf24-3e1d22006b6f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210102%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210102T052659Z&X-Amz-Expires=86400&X-Amz-Signature=c782d65205c0c805c60f3266e6137c8afc85f99eadb09c6cbae087ad9455bd0b&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

단, 리플렉션 API를 사용하면 private 생성자를 호출할 수 있어 새로운 인스턴스를 생성할 수 있다. 이를 해결하기 위해서는 생성자에서 두 번째 객체가 생성될 때 예외를 던지면 된다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e1e038de-18cd-4ba5-83dc-901a14c1bbb8/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210102%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210102T052702Z&X-Amz-Expires=86400&X-Amz-Signature=44e53addaa93d5e656dbe0740fbbc05f0fc50d0dfa0a0c8ee809a7d0b3008c6a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

### 2. 정적 팩터리 메서드를 public static 멤버로 제공한다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/aeb956ce-56db-405a-996e-b3a8f3592acb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210102%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210102T052705Z&X-Amz-Expires=86400&X-Amz-Signature=f2eb028a24a2f42124e99aae711ab26b74b72f4e907bb577cb35351b6bdabdc1&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

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