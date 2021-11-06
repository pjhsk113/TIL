# [이펙티브 자바] item87 - 커스텀 직렬화 형태를 고려해보라

클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면, 향후 릴리스에서도 현재의 구현에 영원히 발이 묶이게 된다. 즉, 기본 직렬화 형태를 버리고 싶어도 버릴 수 없게 된다. 실제로 BigInteger 같은 자바 클래스도 같은 문제를 겪고 있다.

기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.

# 기본 직렬화 형태 사용이 합당한 경우

직접 설계해도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 사용해야 한다.

기본 직렬화 형태는 어떤 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 그 객체들이 연결된 위상까지 기술한다. 이상적인 직렬화 형태라면 물리적인 모습과 논리적인 모습만을 표현해야한다. 

객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 사용해도 무방하다.

```java
public class Name implements Serializable {
    
    /**
     * 성. null이 아니어야함
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
}
```

Name 클래스는 이름, 성, 중간이름이라는 3개의 문자열로 구성된 **논리적 구성요소**로 표현되며, 위 코드는 이 논리적 구성요소를 물리적으로 정확히 반영했다.

이러한 형태라면 기본 직렬화 형태를 써도 괜찮지만, **불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.**(lastName과 firstName이 Null이 아님을 보장해야 하기 때문에)

# 기본 직렬화 형태 사용이 합당하지 않은 경우

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
}
```

논리적으로 이 클래스는 일련의 문자열을 표현한다. 물리적으로는 문자열들을 이중 연결 리스트로 연결했다. 이처럼 **객체의 물리적 표현과 논리적 표현의 차이가 큰 객체에 기본 직렬화 형태를 적용하면 크게 네 가지 면에서 문제가 생긴다.**

### 1. 공개 API가 현재 내부 표현 방식에 영구히 묶인다.

기본 직렬화 형태를 사용하면 private 클래스인 StringList.Entry가 공개 API가 되버린다. 추후 내부 표현방식을 바꾸더라도 StringList의 클래스는 여전히 연결 리스트로 표현된 입력을 처리할 수 있어야한다. 즉, 연결 리스트를  더 이상 사용하지 않더라도 관련 코드를 절대 제거할 수 없게 된다.

### 2. 너무 많은 공간을 차지한다.

위 코드의 직렬화 형태는 연결 리스트의 모든 Entry와 연결 정보를 기록한다. 하지만 Entry와 연결 정보는 내부 구현에 속하니 직렬화 형태에 포함할 가치가 전혀 없다. 그럼에도 불구하고 이러한 정보들이 포함되기 때문에 직렬화 형태가 너무 커져 디스크에 저장하거나 네트워크 전송 속도가 느려진다.

### 3. 시간이 너무 많이 걸릴 수 있다.

 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수밖에 없다. 따라서 객체의 형태에 따라 순회에 시간이 너무 많이 걸릴 수도 있다.

### 4. 스택 오버플로를 일으킬 수 있다.

기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 과정에서 스택 오버플로를 일으킬 수 있다. 실행할 때마다 스택 오버플로가 발생하는 리스트의 최소 크기가 달라질 수 있다. 즉, 플랫폼에 따라 발생할 수도, 발생하지 않을 수도 있다.

## 어떻게 해결할 수 있을까? - 커스텀 직렬화

위와 같은 문제점들을 해결하고 StringList를 위한 합리적인 직렬화 형태로 만드는 방법은 무엇일까?

단순히 리스트가 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 수준이면 될 것이다. 즉, 물리적인 상세 표현을 배제하고 논리적인 구성만 담는 것이다.

```java
// 합리적인 커스텀 직렬화 형태를 갖춘 StringList
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {...}
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     * 
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
     	//기본 직렬화를 수행한다.
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 커스텀 역직렬화를 수행한다.
        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        //기본 역직렬화를 수행한다.
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // 커스텀 역직렬화 부분
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for(int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
}
```

writeObject와 readObject가 직렬화 형태를 처리한다. StringList의 인스턴스 필드에 일시적이란 뜻의 transient 한정자를 붙여 기본 직렬화 형태에 포함되지 않음을 명시했다.

StringList의 필드가 모두 transient더라도 writeObject와 readObject는 각각 가장 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 이렇게 해야 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가되더라도 호환되기 때문이다.(신버전 인스턴스를 직렬화한 후 구버전 역직렬화하면 새로 추가된 필드는 무시되기 때문에 상호 호환이 가능해짐)

커스텀 직렬화를 활용하면 앞서 살펴본 문제점들을 해결할 수 있게 된다. 

원래 버전의 StringList보다 절반의 공간만을 차지하며, 수행 속도도 두 배 정도 빨라진다. 또 스택 오버플로가 발생하지 않기 때문에 직렬화의 크기 제한이 없어진다.

# 커스텀 직렬화 시 주의 사항

## 객체의 불변식이 깨지는 객체는 직렬화에 주의해야 한다.

해시테이블을 예로 생각해보자. 해시테이블은 key-value 엔트리를 담은 해시 버킷을 차례로 나열한 형태로 구성된다. 버킷에 어떤 엔트리를 담을지는 해시코드가 결정하는데, 이러한 해시코드는 구현 방식에 따라 달라질 수 있다.

따라서 세부 구현에 따라 불변식이 깨지는 객체로 볼 수 있고 이 객체는 정확성을 깨트린다. 이러한 객체에 기본 직렬화를 사용하면 심각한 버그로 이어진다. 이런 객체를 기본 직렬화한 후 역직렬화하면 불변식이 심각하게 훼손된 객체들이 생길 수 있기 때문이다.

## 객체의 논리적 상태와 관련된 필드는 모두 transient 한정자로 선언하자.

커스텀 직렬화에서 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 필드는 모두 직렬화된다. 따라서 transient를 선언해도 되는 필드에는 모두 transient 한정자를 붙여야 한다.

여기에는 JVM을 실행할 때마다 값이 달라지는 네이티브 자료구조를 가지는 필드(long 필드)나 캐시된 해시 값과 같은 다른 필드에서 유도되는 필드 등이 해당된다.

해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 tranisent 한정자를 생략해야 한다.

## 기본 직렬화 사용 시 transient 필드는 기본값으로 초기화됨을 잊지말자.

객체 참조는 null로 int는 0으로, boolean 필드는 false로 초기화된다. 기본값을 그대로 사용해서는 안 된다면 readObject 메서드에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원하자. 혹은 지연 초기화를 사용하는 방법도 있다.

## 동기화 메커니즘을 직렬화에도 적용하자.

기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.

모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 writeObject도 synchronized로 선언해야 한다.

```java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```

writeObject 메서드 안에서 동기화 하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다. 그렇지 않으면 교착상태가 발생할 수 있다.

## 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시하자.

직렬 버전 UID를 명시하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제를 해결할 수 있다. 또한, 런타임에 이 값을 생성하는 시간을 단축시켜 성능이 빨라진다.

```java
public class SerializableClass implements Serializable {
    private static final long serialVersionUID = 5130240674615883456L;
}
```

만약 직렬 버전 UID가 없는 기존 클래스를 구부전으로 직렬화된 인스턴스와 호환성을 유지한채 수정하고 싶다면, 구버전에서 생성된 UID 값을 그대로 사용해야 한다. 

반면에 기본 버전 클래스와 호환성을 끊고 싶다면 단순히 UID 값을 바꿔주면 된다. 이렇게 하면 기존 버전 직렬화 인스턴스를 역직렬화할 때 InvalidClassException이 던져진다.

구버전으로 직렬화된 인스턴스들과 호환성을 끊으려는 경우를 제외하고 직렬 버전 UID를 절대 수정하지 말자.