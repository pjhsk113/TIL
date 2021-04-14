# [이펙티브 자바] Item31- 한정적 와일드카드를 사용해 API 유연성을 높이라

Item28에서도 다뤘듯 매개변수화 타입은 **불공변**이다. 

List<Object>에는 어떤 객체든 넣을 수 있지만 Lisit<String>은 String만 넣을 수 있다. 즉, List<String>은 List<Object>가 하는일을 제대로 수행하지 못하니 리스코프 치환 원칙에 어긋나고, 따라서 하위 타입이라 볼 수 없다.

이러한 불공변성 때문에 매개변수화 타입은 유연함이 부족하다. 따라서 불공변의 유연함을 극복하려면 한정적 와일드카드 타입을 사용해야 한다.

# Stack 예제

### E 생산자 매개변수에 와일드카드 타입 : <? extends E>

원소를 스택에 넣는 pushAll 메서드가 있다고 생각해보자.

```java
public class Stack<E> {
    public void pushAll(Iterable<E> src){
        for(E e : src){
            push(e);
        }
    }

    public void push(E e);
}
```

Number을 담을 수 있는 Stack을 만들고 위의 pushAll을 활용해 Integer 타입을 넣어보자.

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ....;

numberStack.pushAll(integers); // 컴파일 오류
```

Integer는 Number의 하위 타입이므로 정상적으로 동작해야할 것 같다. 하지만 실제로는 동작하지 않는다. **제네릭은 불공변이니 위 코드의 Number와 Integer는 하위 타입 관계가 아니다.** 따라서 컴파일 오류가 발생한다.

이러한 결함이 있는 코드를 어떻게 수정하는 것이 좋을까?

바로 **한정적 와일드 카드**를 사용하면 위와 같은 상황에 **유연하게 대처할 수 있다.** **한정적 와일드 카드**를 사용해 E의 하위타입은 어떤 것이든 올 수 있게 수정하면 Stack은 물론 사용하는 클라이언트 코드도 컴파일 에러가 발생하지 않는다.

```java
// E 생산자 매개변수에 와일드 카드 적용
public void pushAll(Iterable<? extends E> src){
		for(E e : src){ 
			push(e);
    }
}
```

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ....;

numberStack.pushAll(integers); // 정상적으로 컴파일된다!
```

### E 소비자 매개변수에 와일드카드 타입 : <? super E>

이번엔 스택안에 모든 원소를 Collection에 옮겨 담는 popAll 메서드가 있다고 해보자.

```java
public class Stack<E> {
    public void popAll(Collection<E> dst) {
			while(!isEmpty()) {
				dst.add(pop());
			}
		}
}
```

Stack<Number>의 원소를 Collection<Object>로 옮겨보자.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ....;

numberStack.popAll(objects); // 컴파일 오류
```

논리적으로는 Number를 Object Collection에 담을 수 있을 것 같다. 하지만 이 코드 역시 동작하지 않는다. 역시 제네릭은 불공변이기 때문에 발생하는 문제이다.

이번 문제는 어떻게 해결할 수 있을까?

바로 한정적 와일드카드를 사용해 원소를 담을 Collection의 타입을 E의 수퍼 타입으로 제한해주면 된다. 

```java
// E 소비자 매개변수에 와일드 카드 적용
public void popAll(Collection<? super E> dst) {
	while(!isEmpty()) {
		dst.add(pop());
	}
}
```

위와 같이 코드를 수정하면 E의 상위 타입 Collection에 stack 원소를 담을 수 있게 된다.

> E 생산자 - <? extends E>
E 소비자 - <? super E>

**이처럼 불공변에 유연성을 극대화하려면 생산자, 소비자용 입력 매개변수에 와일드카드 타입을 사용하면 된다.**

하지만 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 **타입을 정확히 지정**해야 하는 상황으로 **와일드카드 타입을 사용하지 말야한다.**

> 펙스(PECS): Producer-Extends, Consumer-Super
어떤 와일드카드 타입을 써야하는지 헷갈릴 땐 위의 공식을 떠올려보자!

### PECS의 간단 예시

```java
// PECS가 적용되기 전 max 메서드
public static <E extends Comparable<E>> E max(List<E> list)
```

```java
// PECS를 두 번 적용한 max 메서드
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

매개변수의 List<E>는 E 인스턴스를 생산하므로 List<? extends E>로

Comparable<E>는 E 인스턴스를 소비하므로 Comaprable<? super E>로 변경한다.

Comparable은 언제나 소비자이므로 Comparable<? super E>를 사용하는 편이 좋다.

![](https://blog.kakaocdn.net/dn/bXugi5/btq2DEUTBNB/nuqzEygEpfcYaeot03EU0K/img.png)

Collections의 max 메서드

---

## 타입 매개변수와 와일드카드 사용 기준

메서드를 정의할 때 타입 매개변수와 와일드카드 둘 중 어느 것을 사용해도 괜찮을 때가 많다. 예를 들어 리스트의 아이템을 교환하는 메서드가 있다고 가정해보자.

```java
// swap 메서드 타입 매개변수, 와일드카드 둘 다 사용가능하다.
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

위의 메서드에서는 어떤 것을 사용해야할까?

**기본 규칙은 메서드 선언에 타입 매개변수가 한 번만 나올 경우 와일드카드로 대체하는 것이다.** 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

하지만 비한정적 와일드카드 타입 선언에는 한 가지 문제가 있다. 다음 코드를 살펴보자.

```java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i))); // 컴파일 오류
}
```

위의 코드는 방금 꺼낸 원소를 다시 리스트로 넣는 코드인데, 컴파일 오류가 발생한다. 왜 컴파일 오류가 발생할까?

원인은 List<?>에 있다. List<?>는 null값 이외의 원소는 넣을 수 없기 때문에 컴파일 오류가 발생하는 것이다. 이 컴파일 오류는 private 도우미 메서드를 생성하여 간단히 해결할 수 있다.

```java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i , j);
}

// 도우미 메서드
public static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

굳이 더 복잡한 도우미 메서드를 만들어 사용해야하나? 라는 의문이 생길 수 있다. 더 복잡하지만 private 도우미 메서드를 사용하는 이유는 간단하다. public API를 와일드카드 타입으로 제공하고 싶기 때문이다.

클라이언트 입장에서는 내부 구현을 알 필요도 없고 신경쓸 필요도 없다. 또한, 널리 쓰일 public API라면 복잡한 비한정적 타입 매개변수보다 와일드카드를 사용하여 유연함을 높이는게 좋다.

따라서 우리는 공개되는 API를 와일드카드 타입 기반의 선언으로 유지하고, 조금 복잡해지더라도 내부적으로 private 도우미 메서드를 만들라는 것이다.