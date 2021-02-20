# [이펙티브 자바] Item18- 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 최선은 아니다. 잘못 사용하면 오류를 내기 쉽다.

## 1. 다른 패키지의 구체 클래스 상속의 단점

> 해당 장의 '상속'은 클래스가 다른 클래스를 확장하는 것을 의미한다.

다른 패키지의 구체 클래스를 상속하는 일은 위험하다. (인터페이스 확장 및 구현 제외)

메서드 호출과는 다르게 상속은 캡슐화를 깨트린다. 하위 클래스가 상위 클래스의 구현에 의존할 수 밖에 없기 때문이다. 상위 클래스의 구현은 변경될 수 있는데, 그러다보면 하위 클래스는 코드 수정 없이도 망가질 수 있다.

```java
// 상속을 잘못 사용한 예
class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    //생성자
    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

public class TestMain {
    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(Arrays.asList("1","2","3"));
    }
}
```

addAll() 메서드를 통해 3개의 원소를 추가하고, getAddCount() 메서드를 통해 추가된 원소의 개수를 출력해보자. 예상대로라면 3이라는 결과가 출력되야 할 것이다. 하지만 실제로는 6을 반환한다.

HashSet의 addAll 메서드는 add 메서드를 통해 구현되어 있기 때문이다. addAll 메서드에서 super.addAll을 호출하면, 재정의한 add 메서드를 삽입할 원소마다 호출한다. 결국 중복 카운트가 되는 것이다.

### 해법아닌 해법(하위 클래스가 깨지기 쉽다)

1. addAll 메서드를 재정의하지 않는다.

→ 자신의 다른 부분을 사용하는 자기사용 여부는 내부 구현 방식이므로 오픈 API가 아니다. 따라서 다음 릴리즈에서 유지될지 알 수없어(삭제 가능) 깨지기 쉽다.

2. addAll을 다른 식으로 재정의한다.

→ 주어진 컬렉션을 순회하며 원소 하나당 add 메서드를 한번씩 호출한다. 하지만 다음과 같은 단점이 있다.

- 상위 클래스의 메서드 동작을 다시 구현하는게 어렵다.
- 시간이 더 든다.
- 오류를 내거나 성능을 떨어뜨릴 수 있다.
- 하위 클래스에서 private을 사용해야한다면 구현이 불가능하다.

### 하위 클래스가 깨지기 쉬운 또 다른 이유

1. 상위 클래스에서 새로운 메서드를 추가할 때 하위 클래스가 깨질 수 있다.

→ 만약 상위 클래스에서 새로운 메서드가 추가되었다면, 하위 클래스에서는 해당 메서드를 통해 허용되지 않은 원소를 추가할 수 있게된다.

2. 상위 클래스에서 추가한 새로운 메서드와 하위 클래스의 메서드 시그니쳐가 같고 반환 타입만 다르다면?

→ 하위 클래스의 메서드는 컴파일 조차 되지 않는다.

## 진짜 해법 - 컴포지션(Composition)

위의 문제를 모두 해결할 수 있는 방법이 있다. 상속하는 대신 새로운 클래스를 만들고, 기존 클래스 객체를 참조하는 private 필드를 하나 두는 것이다. 이러한 설계 방법을 `컴포지션`이라 한다. 컴포지션을 사용하면 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않게 된다.

`컴포지션`: 기존 클래스가 새로운 클래스의 구성요소 쓰인다.

`전달`: 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환

`전달 메서드`: 새 클래스의 메서드

다음 예제는 컴포지션을 활용한 예제이다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }

    public int getAddCount () {
        return addCount;
    }
}
```

다른 인스턴스를 감싸고 있다는 의미에서 InstrumentedSet과 같은 클래스를 **래퍼 클래스**라고 한다.

```java
class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {this.s = s;}
    @Override
    public int size() {    return s.size();}
    @Override
    public boolean isEmpty() {return s.isEmpty();}
    @Override
    public boolean contains(Object o) {return s.contains(o);}
    @Override
    public Iterator<E> iterator() {return s.iterator();}
    @Override
    public Object[] toArray() {return s.toArray();}
    @Override
    public <T> T[] toArray(T[] a) {return s.toArray(a);}
    @Override
    public boolean add(E e) {return s.add(e);}
    @Override
    public boolean remove(Object o) {return s.remove(o);}
    @Override
    public boolean containsAll(Collection<?> c) {return s.containsAll(c);}
    @Override
    public boolean addAll(Collection<? extends E> c) {return s.addAll(c);}
    @Override
    public boolean retainAll(Collection<?> c) {return s.retainAll(c);}
    @Override
    public boolean removeAll(Collection<?> c) {return s.removeAll(c);}
    @Override
    public void clear() {s.clear();}
}
```

재사용할 수 있는 전달 클래스 ForwardingSet은 Set을 구현했고, 내부적으로 Set의 인스턴스를 받는 생성자를 하나 제공한다. 임의의 Set의 **계측 기능**을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다. 재사용할 수 있는 전달 클래스를 인터페이스 당 하나씩 만들어두면 원하는 기능을 덧씌우는 전달 클래스를 손쉽게 구현할 수 있다.

`컴포지션`과 `전달`의 조합은 넓은 의미에서 **위임**이라고 부른다. 단, 엄밀히 따지자면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당된다.

### 주의점

**콜백 프레임워크**

래퍼 클래스는 단점이 별로 없지만, 콜백 프레임워크과 함께 사용하기에는 적합하지 않다. 콜백 프레임워크는 자기 자신의 참조를 다른 객체에 넘겨, 다음 호출 때 (필요할 때) 사용하도록 한다. 하지만 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르기 때문에, 자기 자신의 참조(this)를 넘기고 콜백 때는 래퍼가 아닌 내부 객체를 호출하게된다. 따라서 콜백 과정에서 래퍼 객체는 제외된다. 이를 SELF 문제라고 한다.

## 상속의 원칙

### is-a 관계, B는 확실히 A인가?

상속은 반드시 하위 클래스가 상위 클래스의 `진짜` 하위 타입인 상황에만 쓰여야한다. 즉, is-a 관계가 명확해야한다. 만약 클래스B가 클래스A를 상속하고 싶다면, 'B는 확실히 A인가?' 라는질문에 확실하게 '그렇다'라고 답할 수 있어야 한다.

예를들면 Java의 Stack은 Vector가 아니므로 Stack은 Vector를 확장해서는 안 됐다. 'Stack은 확실히 Vector인가?' 라는 질문에 '그렇다'라고 대답할 수 없기 때문이다.

### 확장하려는 클래스의 API에 아무런 결함이 없는가?

만약 상위 클래스에 결함이 있다면 하위 클래스에 그 결함이 전파되도 괜찮은지를 자문해보자. 컴포지션은 그 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스 API의 **결함**까지 그대로 전파되기 때문이다.

## 핵심 정리

1. 상속은 강력하지만, 캡슐화를 해친다.

2. 상위 클래스와 하위 클래스의 관계가 순수한 is-a 관계일 때만 상속을 사용해야한다.

3. 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 is-a 관계일 때도 문제가 될 수 있다.

4. 상속의 취약점을 피하려면 컴포지션과 전달을 사용하자.

5. 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 컴포지션을 사용해야한다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하기 때문에.