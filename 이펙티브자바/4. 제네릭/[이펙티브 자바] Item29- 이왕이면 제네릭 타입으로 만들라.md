# [이펙티브 자바] Item29- 이왕이면 제네릭 타입으로 만들라

클라이언트가 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 따라서 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 제네릭 타입으로 만들자.

## 예시

### Object 기반 스택을 제네릭 타입으로 변환

```java
public class Stack<E> {
    private E[] elements;
    private int size;

    public Stack() {
        this.elements = new E[16];
    }

    public void push(E e) {
        elements[size++] = obj;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

		......
}
```

Stack에 타입 매개변수를 추가하고 Object를 타입 매개변수로 변경한 후 컴파일해보자.

E와 같은 실체화 불가 타입은 배열을 만들 수 없다.(제네릭 배열 생성 금지 제약) 이와 같은 문제가 있을 경우 두 가지의 우회방법이 존재한다.

### 제네릭 배열을 생성하기 위한 첫 번째 우회방법

**Object 배열을 생성한 후 제네릭 배열로 형변환한다.** 대놓고 우회하는 방법이다. 

```java
  public Stack() {
        this.elements = (E[]) new Object[16];
  }
```

위와 같이 형변환해주면 컴파일러는 오류대신 경고를 내뱉는다. 

컴파일러는 타입 안전한지 증명할 방법이 없지만 우리는 스스로 검증할 수 있다. 위 예제의 경우 elements는 private 필드이고 클라이언트로 반환하거나 다른 메서드에 전달되는 일이 전혀 없다. 또한 push 메서드에 전달되는 원소의 타입은 항상 E이므로 이 비검사 형변환은 확실히 안전하다고 판단할 수 있다.

이처럼 비검사 형변환이 안전하다고 판단되면 @Suppress Warnings 애노테이션을 이용해 해당 경고를 숨긴다.

```java
@SuppressWarnings("unchecked")
 public Stack() {
        this.elements = (E[]) new Object[16];
  }
```

이렇게 사용함으로 얻을 수 있는 이점은 명시적으로 형변환하지 않아도 ClassCastException을 걱정하지 않고 사용할 수 있다는 점이다. 

물론, 단점도 존재한다. 바로 힙 오염이 일어날 수 있다는 점이다. 컴파일 타임에 해당 타입은 Object이지만 런타임시에는 E이기 때문이다.

장점: 명시적으로 형변환을 하지 안아도 된다.

단점: 힙 오염이 일어날 수 있다.

### 제네릭 배열을 생성하기 위한 두 번째 우회방법

**elements 필드의 타입을 E[]에서 Object[]로 변경한다.**

이때도 마찬가지로 경고가 발생한다. 첫 번쨰 방법처럼 우리가 스스로 검증하고 경고를 제거해야한다.

```java
		public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        @SuppressWarnings("unchecked") E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }
```

elements 필드를 Object[]로 변경하는 방법은 힙 오염이 일어나지 않는다는 장점이 있다. 하지만 원소를 pop 할 때마다 형변환을 해주어야 한다는 면에서 단점이라 볼 수도 있다.

장점: 힙 오염이 일어나지 않는다.

단점: pop 호출 시 매번 형변환이 일어난다.