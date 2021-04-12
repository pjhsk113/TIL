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

**이처럼 불공변에 유연성을 극대화하려면 생산자, 소비자용 입력 매개변수에 와일드카드 타입을 사용하면 된다.**

> E 생산자 - <? extends E>
E 소비자 - <? super E>