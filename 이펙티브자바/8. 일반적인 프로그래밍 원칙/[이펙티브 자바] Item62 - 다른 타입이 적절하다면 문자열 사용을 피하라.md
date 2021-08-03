# [이펙티브 자바] Item62 - 다른 타입이 적절하다면 문자열 사용을 피하라

문자열은 텍스트를 표현하도록 설계되었다. 하지만 워낙 흔하고 자바가 잘 지원해주기 때문에 설계 의도와 다르게 사용될 때가 많다.

# 문자열을 쓰지 않아야 할 사례들

## 1. 문자열로 다른 값 타입을 대신하는 경우

받는 데이터가 수치를 나타낸다면 int, float, BigInteger 등을, Y / N을 나타낸다면 boolean을 사용하는 것이 적절하다. 즉, 기본 타입이든 참조 타입이든 알맞는 값 타입이 있다면 그것을 사용하고, 없다면 새로 만들어서 사용하자.

## 2. 문자열로 열거 타입을 대신하는 경우

아이템 34에서도 설명했듯, 상수를 열거할 때는 문자열보다 열거 타입을 사용하자.

## 3. 문자열로 혼합 타입을 표현하는 경우

혼합된 데이터를 하나의 문자열로 표현하는 것은 좋지 않은 생각이다. 

```java
// 혼합 타입을 문자열로 표현.. 좋지 않다!
String compoundKey = className + "#" + i.next();
```

위 예제는 각 요소를 개별로 접근하기 위해 문자열을 파싱한다. 따라서 느리고, 귀찮고, 오류 가능성도 커진다. 또한, 적절한 equals, toString, compareTo 메서드를 제공할 수 없고, 오직 String이 지원하는 기능에만 의존해야 한다.

이런 경우 보통 private 정적 멤버 클래스로 선언하는 편이 좋다.

```java
public class Compound {
    private CompoundKey compoundKey;

    private static class CompoundKey {
        private String className;
        private String delimiter;
        private int index;

        public CompoundKey(String className, String delimiter, int index) {
            this.className = className;
            this.delimiter = delimiter;
            this.index = index;
        }
    }
}
```

## 4. 문자열로 권한을 표현하는 경우

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);
    
    // (키가 가르키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```

위 예시는 스레드 지역변수 기능 클래스이다. 각 스레드가 자신만의 변수를 갖게 해주는 기능을 가지고 있다.

스레드 구분용 키로 문자열을 받는다. 이 방식의 문제점은 무엇일까?

바로 스레드 구분용 문자열 키가 전역 namespace에서 공유한다는 점이다. 만약 문자열 키가 겹친다면 의도치않게 같은 변수를 공유하게 된다. 또한, 의도적으로 같은 키를 이용해 다른 클라이언트의 값을 가져올 수도 있다. 이러한 권한 기능을 문자열로 표현하면 보안에도 취약하고 제대로 동작하지도 않는다.

### 해결 방법

문자열로 권한을 표현하기 보다는 별도의 클래스로 분리하는게 좋은 해법이 될 수 있다.

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가

    public static class Key {
        key() {}
    }

    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		    return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

Key 클래스로 권한을 구분했다. 앞선 문제점을 모두 해결해 주지만 아직 개선할 여지가 있다.

### 리팩터링

set / get 메서드는 더 이상 정적 메서드일 필요가 없다. 따라서 Key의 인스턴스 메서드로 변경한다. 이렇게 하면 KEy는 더 이상 스레드 지역변수를 구하분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.

결과적으로 톱레벨 클래스인 ThreadLocal 클래스는 별달리 하는 일이 없어지므로 중첩 클래스 Key의 이름을 ThreadLocal로 변경해보자.

```java
// Key를 ThreadLocal로 변경
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

이 API는 get으로 얻은 Object를 형변환해서 사용해야하므로 타입 안전하지 않다. 이는 매개변수화 타입을 추가해 타입 안전하게 만들수 있다.

```java
// 매개변수화로 타입안전성 확보
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

# 핵심정리

- 더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열보다 그 타입을 사용하자.