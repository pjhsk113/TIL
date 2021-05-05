# [이펙티브 자바] Item26- 로 타입(Raw Type)은 사용하지 말라

로 타입(Raw Type)은 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다. 예를 들면 `List list = new ArrayList();` 와 같은 형태를 가진다.

> **타입 매개변수?**
List<E> 에서 <E>에 해당하는 매개변수를 타입 매개변수라 한다.
ex) List<String>에서는 String이 타입 매개변수에 해당한다.

# 로 타입의 문제점

### 타입 안전성과 표현력을 확보할 수 없다.

```java
// 컬렉션의 로 타입
private final Collection stamps = ..... ; // Stamp 인스턴스만 취급한다.

// 실수로 Stamp가 아닌 Coin을 넣는다.
stamp.add(new Coin(..))
```

위의 코드는 아무 오류없이 컴파일되고 실행된다. 컬렉션이 동전을 다시 꺼내기 전까지는 오류를 알아챌 방법이 없다. 즉, 컴파일타임에 오류를 발견할 수 없고 한참 후인 런타임시 오류가 발견된다는 뜻이다. ClassCastException이 발생하면 그때서야 동전을 넣은 지점을 찾기 위해 코드 전체를 살펴봐야하는 상황이 올 수도 있다.

따라서 다음과 같이 타입 매개변수를 입력해줘야 한다.

```java
// 매개변수화된 컬렉션 타입
private final Collection<Stamp> stamps = ..... ;

// 실수로 Stamp가 아닌 Coin을 넣는다.
stamp.add(new Coin(..)) // 컴파일 오류 발생
```

이렇게 선언하면 컴파일러는 stamps에는 Stamp 인스턴스만 넣어야 함을 인식할 수 있고 컴파일된다면 의도대로 동작함을 보장할 수 있다. 즉, **타입 안정성**과 **표현력** 모두를 가지게 된다.

### List와 List<Object>의 차이

임의 객체를 모두 수용한다는 점에서 둘의 동작은 같다. 그렇다면 이 둘의 차이는 무엇일까?

컴파일러에게 명확한 의사전달을 했냐 안했냐의 차이다. List<Object>는 모든 타입을 허용한다는 명확한 의사를 컴파일러에게 전달한 것이다. 제네릭의 **하위 타입 규칙**에 따라 **타입 안정성**을 지킬 수 있게 된다.

다음 예시를 살펴보자.

List를 매개변수로 받는 메서드에는 List<String>을 정상적으로 값을 넘길 수 있다. List<String>은 List와 같은 로 타입의 **하위 타입**이기 때문이다.

```java
public static void main(String[] args) {
	List<String> strings = nenw ArrayList<>();
	unsafeAdd(strings); // List<String>을 넘길 수 있다.
}

// List를 받는 메서드(Raw Type)
private static void unsafeAdd(List list) {
	....
}
```

하지만 List<Object>를 갖는 메서드로 List<String>을 넘기려고 하면 컴파일 에러가 발생한다. List<String>은 List<Object>의 **하위 타입**이 아니기 때문이다.

```java
public static void main(String[] args) {
	List<String> strings = nenw ArrayList<>();
	unsafeAdd(strings); // 컴파일 에러!! List<String>을 넘길 수 없다.
}

// List<Object>를 받는 메서드(매개변수화 타입)
private static void unsafeAdd(List<Object> list) {
	....
}
```

따라서 **타입 안정성**을 지키기위해 매개변수화 타입을 사용하는 것이 좋다.

# 로 타입을 사용하고 싶을 경우에는?

두 개의 Set에 같은 원소가 몇 개 있는지 반환하는 메서드를 만든다고 한다면, 어떤 매개변수가 들어오든 상관없다. 따라서 로 타입을 사용하고 싶은 요구사항이 있을 수 있다.

```java
// 로 타입을 사용해 안전하지 않다.
private int count(Set destination, Set source) {
	int result = 0;
	for (Object o : source) {
		if (destination.contains(o)) {
			result++;
		}
	}
	return result;
}
```

그렇다고 로 타입을 사용하면 안전하지 않다. 이럴 경우에는 어떻게 해야할까?

### 해결 방안

이럴 경우 비한정적 와일드카드 타입을 사용하면 된다. 비한정적 와일드카드 타입은 어떤 타입이든 상관없이 받을 수 있지만 타입의 안정성을 보장해준다. 또한 null 외에는 어떤 원소도 넣을 수 없기 때문에 불변식을 보장해준다.

`List<?>` 와 같은 형태를 가진다.

```java
// 비한정적 와일드카드 타입은 타입 안전하고 유연하다.
private int count(Set<?> destination, Set<?> source) {
	int result = 0;
	for (Object o : source) {
		if (destination.contains(o)) {
			result++;
		}
	}
	return result;
}
```

# 로 타입을 써도 되는 경우

로 타입을 쓰지 말라는 규칙에도 예외는 있다.

### 1. class 리터럴에는 로 타입을 써야 한다.

클래스 리터럴에는 매개변수화 타입을 사용하지 못한다. 예를 들어 List.class, String[].class, int.class는 허용하고 List<String>.class나 List<?>.class는 허용하지 않는다.

### 2. instanceof에 사용해도 괜찮다.

런타임에는 제네릭 타입 정보가 지워진다. 따라서 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다. 그러므로 굳이 지저분한 <?>를 붙일 필요가 없다. 오히려 로 타입이 깔끔하다.

```java
if (o instanceof Set) {
	Set<?> s = (Set<?>) o;
	...
}
```

# 핵심 정리

### 로 타입은 제네릭이 도입되기 전 코드와의 호환성을 위해 제공될 뿐이다.

### 로 타입은 런타임에 예외가 발생할 수 있으니 사용하면 안된다.