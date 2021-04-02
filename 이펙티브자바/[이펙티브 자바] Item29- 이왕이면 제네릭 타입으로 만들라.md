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
            throw new EmptyStackException;
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

Object 배열을 생성한 후 제네릭 배열로 형변환한다. 대놓고 우회하는 방법이다. 

```java
  public Stack() {
        this.elements = (E[]) new Object[16];
  }
```

위와 같이 형변환해주면 컴파일러는 오류대신 경고를 내뱉는다. 

컴파일러는 타입 안전한지 증명할 방법이 없지만 우리는 스스로 검증할 수 있다.