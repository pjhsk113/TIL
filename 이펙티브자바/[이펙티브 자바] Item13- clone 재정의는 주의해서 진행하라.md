# [이펙티브 자바] Item13- clone 재정의는 주의해서 진행하라

---

클래스에서 clone을 재정의 하기위해서는 해당 클래스에 Cloneable 인터페이스를 상속받아 구현한다. 그런데 clone 메서드는 Cloneable 인터페이스가 아닌 Object에 선언되어있다. Cloneable 인터페이스는 아무것도 선언되어있지 않은 빈 인터페이스이다.  메서드 하나 없는 Cloneable 인터페이스는 대체 무슨 일을 할까?

# Cloneable 인터페이스의 역할

Cloneable 인터페이스는 상속받은 클래스가 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)이다.

> mixin interface란?
클래스가 본인의 기능 외에 추가로 구현할 수 있는 자료형으로, 선택적인 기능을 제공한다는 사실을 명시하기위해 쓰인다.

Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다. Cloneable 인터페이스를 구현한 클래스의 clone 메서드를 호출하면 해당 클래스를 필드 단위로 복사하여 반환한다. 만약 이를 구현하지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedExeption`을 던진다.

# Object clone 메서드의 일반 규약

1. x.clone() != x 는 참이다.
2. x.clone().getClass() == x.getClass() 는 참이다.  (반드시 만족해야 하는 것은 아니다.)
3. x.clone().equals(x) 는 참이지만 필수는 아니다.  (반드시 만족해야 하는 것은 아니다.)
4. x.clone().getClass() == x.getClass(), 
super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면 이 식은 참이다. 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

# clone 메서드 재정의

제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다면 super.clone을 호출한다. (클래스에 정의한 모든 필드가 원본과 같다.)

불변 클래스에서는 불필요한 copy를 조장하기 때문에 clone 메서드를 제공하지 않는게 좋다.

```java
@Override public PhoneNumber clone(){
  try{
    return (PhoneNumber) super.clone();
  }catch(CloneNotSupportedException e){
    throw new AssertionError();
  }
}
```

Object의 clone은 Object를 반환하지만 PhoneNumber에서는 PhoneNumber를 반환하게 하였다. 이 방법은 자바의 **공변 반환 타이핑 (convariant return typing)** 도입 덕분에 가능해졌다.

다시 말해서, 재정의한 메서드의 반환타입은 상위클래스 메서드가 반환하는 타입(Object)의 하위 타입(PhoneNumber)일 수 있다. 덕분에 재정의 메서드는 반환될 객체에 대한 더 많은 정보를 제공 할 수 있고, 클라이언트는 형변환을 하지 않아도 된다.

### 가변 객체에서 사용할 경우

만약 복제할 클래스가 가변 객체를 참조한다면, 위의 clone 메서드를 사용하면 안된다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; 
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위의 Stack 클래스에서 clone 메서드가 단순히 super.clone()이 반환한 객체를 그대로 반환하도록 구현한다면, 그 복사본의 size 필드는 올바른 값을 갖겠지만 elements 필드는 원래 Stack 객체와 같은 배열을 참조하게 된다. 그 상태에서 원래 객체나 복사본을 수정하면 다른 하나도 수정되어 불변식이 깨진다. 

clone 메서드는 또 다른 형태의 생성자다. 원래 객체를 손상시키는 일이 없도록 해야 하고, 복사본의 불변식(invariant)도 제대로 만족시켜야 한다.

Stack의 clone 메서드가 제대로 동작하도록 하려면 스택의 내부 구조도 복사해야 한다. 가장 간단한 방법은 elements 배열에도 clone을 재귀적으로 호출하는 것이다.

```java
@Override
    protected Stack clone() {
        try {
            Stack result = (Stack)super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

하지만 elements 필드가 final로 선언되어 있으면 동작하지 않는다. clone 안에서 필드에 새로운 값을 할당할 수 없기 때문이다. 

또한, clone을 재귀적으로 호출하는 것만으로 충분하지 않을 때도 있다. 버킷 배열로 구성된 해시 테이블의 clone 메서드를 예로들수 있다.

해시테이블 내부는 버킷들의 배열과 각 버킷의 키-쌍을 담는 연결리스트의 첫 번째 엔트리를 참조한다. 이때, 버킷 배열을 재귀적으로 복제한다고 하면, 버킷 배열 안에 있는 연결리스트들은 모두 원본과 같게 된다. 따라서 복사본이 자신만의 버킷 배열을 갖긴 하지만, 원본과 동일한 연결 리스트를 참조하게 된다.

이 문제를 해결하려면 각 버킷을 구성하는 연결 리스트까지도 복사해줘야한다. (깊은 복사)

연결 리스트를 복제할 때 재귀적으로 호출하는 방식을 사용하면 연결 리스트가 길 경우 스택 오버플로가 발생하기 쉽다. 리스트 원소마다 스택 프레임을 사용해야하기 때문이다. 이러한 상황을 방지하려면 재귀가 아니라 순환문(반복문)을 사용해 깊은 복사를 해야한다.

# 복제를 대체할 수 있는 좋은 방법

clone을 대체할 수 있는 좋은 방법으로는 `복사 생성자`나 `복사 팩터리`를 제공하는 것이다.

### 복사 생성자

단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

```java
public Yum(Yum yum) {
		....
};
```

### 복사 팩터리

복사 생성자와 유사한 정적 팩터리 메서드이다.

```java
public static Yum newInstance(Yum yum) {
		......
};
```

위 두 방식은 Cloneable/clone보다 좋은 점이 많다.

### 장점

- 위험한 객체 생성 수단(생성자 없이 객체 생성)에 의존하지 않는다.
- 엉성하게 문서화된 규약에 기대지 않는다.
- final 필드 용법과 충돌하지 않는다.
- 불필요한 예외 검사를 요구하지 않는다.
- 형변환이 필요없다.
- 복사 생성자나 팩터리는 해당 메서드가 정의된 클래스가 구현하는 인터페이스를 인자로 받을 수 있다.