# [이펙티브 자바] Item1- 생성자 대신 정적 팩터리 메서드를 고려하라

---

# 정적 팩터리 메서드의 장점

## 1. 이름을 가질 수 있다.

하나의 시그니처로는 생성자를 하나만 만들 수 있기 때문에 생성자에 넘기는 매개변수와 생성자 자체만으로 반환될 객체의 특성을 제대로 설명할 수 없다.

하지만 정적 팩터리 메서드는 이러한 제약이 없다. 시그니처가 같은 생성자가 여러개 필요할 것 같으면 각각 차이를 잘 드러내는 이름을 지어주어 반환 객체의 특성을 쉽게 묘사할 수 있다.

예를들면,

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f48a3262-6e77-4927-94e0-44d2d2a8f7c9/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210110T102304Z&X-Amz-Expires=86400&X-Amz-Signature=18c489a17c51f8dc249640e1f1eb0b625d9fce6ac9839071a8c714e84aaaedbc&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1f0fc658-af83-414d-a385-3059f4aadeff/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210110T102930Z&X-Amz-Expires=86400&X-Amz-Signature=8457a379b403d99ea7d2fd0ca5e0d4a5b1edfd3f18fda347f500cba502ede144&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

이름을 가진 정적 팩터리 메서드가 소수인 BigInteger를 반환한다는 의미를 더 잘 나타내고 있다.

## 2. 호출될 때마다  인스턴스를 새로 생성하지 않아도 된다.

### **불변 클래스**

- 인스턴스를 미리 만들어 놓거나 인스턴스 캐싱으로 재사용하여 불필요한 객체 생성을 피할 수 있다.

ex) Boolean의 valueOf는 객체를 아예 생성 하지 않는다.

이는 생성 비용이 크거나 같은 객체가 자주 요청되는 상황에서 성능을 향상시켜줄 수 있다. 플라이웨이트 패턴이 이와 비슷한 기법이라 할 수 있다.

### 인스턴스 통제 클래스

- 정적 팩터리 메서드는 같은 객체를 반환하는 방법으로 언제 어느 인스턴스를 살아 있게 할지 통제할 수 있다.
- 인스턴스를 통제하면 클래스를 싱글턴 / 인스턴스화 불가(noninstantiable)로 만들 수 있다.
- 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
- 인스턴스 통제는 플라이웨이트 패턴의 근간이 된다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1bdc3a09-575e-4f96-a782-1efb0d96f59f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210110T102946Z&X-Amz-Expires=86400&X-Amz-Signature=5acde373f2bcb8f57f9ab7b566146934725860f3a46f82982c97105d2c02f40f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4cec3985-8fe9-4ffe-b668-9334c34ebe3b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210110T102958Z&X-Amz-Expires=86400&X-Amz-Signature=dfd999b479ea199f8a7cc63e55d1079a9062a0c73d65df3ed193c6209d89535d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

TRUE와 FALSE 각각 true, false 값을 가지는 인스턴스를 캐싱하고 있고, 값이 true 혹은 false가 들어오면 캐싱된 값이 사용되므로 객체를 생성하지 않는다. 이 덕분에 동치인 인스턴스가 하나뿐임을 보장할 수 있다.

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있다.

## 4. 입력에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

3, 4는 비슷한 맥락의 장점이다. 

정적 팩터리 메서드는 반환 타입의 하위타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 반환할 객체의 클래스를 자유롭게 선택할 수 있기 때문에 유연함을 가진다.

이러한 유연함 덕분에 클라이언트는 실제 구현 클래스가 무엇인지 몰라도 된다.

아래는 위에서 설명한 장점을 잘 나타내는 EnumSet의 예제이다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d9c61164-409a-4eb3-8a0b-98048c48301f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210110T103014Z&X-Amz-Expires=86400&X-Amz-Signature=a681836d06128f1bde63af4f6e71c52a46b5f69762ccc42489aeee1baf9b1e8b&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

universe.length에 의해 EnumSet의 하위 객체인 RegularEnumSet, JumboEnumSet 중 반환 타입이 결정된다. 클라이언트는 EnumSet이 반환되는 것만 알면되고 어떤 객체가 반환 되는지 알 필요가 없다.

---

# 정적 팩터리 메서드의 단점

## 1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

상속을 하기위해서는 public이나 protected 생성자가 필요하다. 따라서 정적 팩터리 메서드만 제공하게되면 하위 클래스를 만들 수 없게 된다는 단점이 있다. 또한, 컬렉션 프레임워크 유틸리티를 상속할 수 없다는 것도 단점이 될 수 있다.

하지만 이 제약은 상속보다 컴포지션을 유도하고 불변 타입으로 만들려면 이제 약을 지켜야 한다는 점에서 오히려 장점으로 볼 수 있다.

## 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 클래스를 인스턴스화할 방법을 알아내야 한다. 따라서 널리 알려진 규약을 따라 이름을 명명해줘야 한다.

### from

매개변수를 하나 받아서 해당 타입의 인스턴스를 반환

``` java
Date d = Date.from(instant);
```

### of

여러 매개변수를 받아 적합한 타입의 인스턴스를 반환

```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

### valueOf

from과 of의 더 자세한 버전

```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

### instance 혹은 getInstance

매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않음

```java
StackWalker luke = StackWalker.getInstance(options);
```

### create 혹은 newInstance

위의 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장

```java
Object newArray = Array.newInstance(classobject, arrayLen);
```

### getType

getInstance와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반활할 객체의 타입이다.

```java
FileStore fs = Files.getFileStore(path)
```

### newType

newInstance와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩털리 메서드가 반환할 객체의 타입이다.

```java
BufferedReader br = Files.newBufferedReaer(path);
```

### type

getType과 newType의 간결한 버전

```java
List<Complaint> litany = Collections.list(legacyLitany);
```

---

# 핵심 정리

> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새를 이해하고 적절히, 적재적소에 사용하는 것이 좋다.
