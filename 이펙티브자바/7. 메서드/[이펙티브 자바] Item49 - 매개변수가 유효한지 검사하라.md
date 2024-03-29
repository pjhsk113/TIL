# [이펙티브 자바] Item49 - 매개변수가 유효한지 검사하라

**오류는 가능한 빨리 발생한 곳에서 잡아야 한다.** 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 오류 발생 지점을 찾기 어려워진다.

메서드의 매개변수 또한 마찬가지이다. 메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 깔끔한 방식으로 예외를 던질 수 있다.

# 매개변수 검사를 제대로 하지 못하면 발생하는 문제

### 1. 메서드가 수행 중간에 모호한 예외를 던지며 실패할 수 있다.

### 2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.

### 3. 메서드는 문제없이 수행됐지만, 미래의 알 수 없는 시점에 메서드와 관련 없는 오류를 낼 수 있다. (실패 원자성을 어기는 상황)

# public과 protected 메서드는 예외를 문서화하자

@throws 자바독 태그를 사용해 매개변수 값이 잘못됐을 때 던지는 예외를 문서화하자. 일반적으로는 **IllegalArgumentException, IndexOutOfBoundsException, NullpointerException** 중 하나가 될 것이다.

매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야한다. 이런 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다.

보통 아래와 같은 방식으로 제약을 기술한다.

```java
/**
* (현재 값 mod m) 값을 반환한다. 이 메서드는
* 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
*
* @param m 계수(양수여야 한다.)
* @return 현재 값 mod m
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
*/
public BigInteger mod(BigInteger m) {
 if (m.signum() <= 0)
 throw new ArithmeticException("계수(m)은 양수여야 합니다. " + m);
 ... // 계산 
}
```

모든 public 메서드에 적용되는 예외인 경우, 클래스 레벨 주석을 기술하는 것이 훨씬 깔끔한 방법이다. 

위의 예제에서는 매개변수가 null인 경우 NullPointerException에 대한 설명은 BigInteger 클래스 레벨에 기술되어 있기 때문에 메서드에는 해당 내용이 기술 되어있지 않다.

# Null을 검사하는 여러가지 방법

Java7에 추가된 `java.util.Objects.requireNonNull` 메서드를 사용하면 유연하고 편하게 null 검사를 수행할 수 있다.

```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

Java9에서는 Objects에 범위 검사 기능도 더해졌다. `checkFromIndexSize`, `checkFromToIndex`, `checkIndex` 등의 메서드는 null 검사 메서드만큼 유연하지는 않지만, 예외 메시지를 지정할 필요가 없고, 리스트와 배열이고, 닫힌 범위가 아니라면 아주 유용하고 편리하게 사용할 수 있다.

# public 메서드가 아니라면 단언문(assert)를 사용할 수 있다.

공개되지 않은 메서드는 개발자가 메서드 호출을 통제할 수 있다. 따라서 매개변수 유효성을 검증하고 오직 유효한 값만이 메서드에 넘겨진다는 것을 보증해야한다.

단언문 assert를 통해 실행되는 문장이 참이라는 것을 보증할 수 있다.

```java
private static void sort(long a[]. int offset, int length){
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // 계산 수행
}
```

assert는 Java4부터 지원하며 자신이 단언한 조건이 무조건 참이라고 선언한다. 즉, **실행되는 문장이 참(true)이라면 그냥 지나가고, 거짓(false)이라면 AssertionError가 발생한다.**

assert는 런타임에 아무런 효과가 없고 성능 저하도 없다.

# 나중에 쓰기 위해 저장하는 매개변수의 유효성 검사

**메서드가 직접 사용하지는 않지만 나중에 쓰기위해 저장하는 매개변수의 경우 더 신경써서 검사해야 한다.**

입력받은 int 배열의 List 뷰를 반환하는 메서드를 예로 설명할 수 있다.

```java
// int 배열의 List 뷰를 반환하는 정적 팩터리 메서드
static List<Integer> intArrayAsList(int[] a){
    Objects.requireNonNull(a); // null 검사

    return new AbstractList<Integer>() {
        @Override
        public Integer get(int index) {
            return a[index]; // 0x124124 + index
        }

        @Override
        public int size() {
            return a.length;
        }
    };
}
```

만약 위의 메서드에서 `Objects.requireNonNull`을 생략했다면, 클라이언트가 반환된 List를 사용하려 할 때 NullPointerException이 발생한다. 이러한 상황은 해당 List를 어디서 가져왔는지 추적하기 어려워 디버깅이 힘들어진다.

특히, 생성자의 경우 클래스 불변식을 어기는 객체가 만들어지지 않도록 매개변수의 유효성 검사가 꼭 필요하다.

# 메서드 실행 전 매개변수 검사 예외 케이스

메서드 몸체가 실행되기 전에 매개변수를 확인해야 한다는 규칙에도 예외는 있다. **유효성 검사의 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산 과정에서 암묵적으로 검사가 수행되는 경우이다.**

예를 들어, Collections.sort(List)는 두 원소를 비교할 수 있는 타입인지 정렬 과정에서 비교한다. 비교할 수 없는 타입이라면 ClassCastException이 발생한다. 

따라서 정렬에 앞서 원소들이 상호 비교될 수있는 타입인지 검사하는 유효성 검사는 별다른 실익이 없다. **계산 과정에서 암묵적으로 검사가 수행되기 때문이다.**

# 핵심 정리

- 메서드나 생성자를 작성할 때 그 매개변수들에 어떤 제약이 있을지 생각하자.
- 제약들을 문서화하고, 메서드 시작 부분에서 명시적으로 검사하자.