# [이펙티브 자바] Item83 - 지연 초기화는 신중히 사용하라

지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때 까지 늦추는 기법이다. 지연 초기화는 주로 최적화 용도로 쓰이며, 클래스와 인스턴스 초기화 시 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

# 지연 초기화의 특징

지연 초기화는 인스턴스 생성 시 초기화 비용은 줄지만, 그 필드에 접근하는 비용은 커진다. 따라서 초기화가 이루어지는 비율, 실제 초기화에 드는 비용, 호출 빈도에 따라 오히려 성능이 느려질 수 있다.

**인스턴스의 사용 빈도가 낮지만 초기화 비용이 크다면 지연 초기화가 제 역할을 해준다.**

# 멀티 스레드 환경에서 지연 초기화

지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면, 반드시 동기화해야 한다. 그렇지 않으면 심각한 버그로 이어질 수 있다. (item78 - 공유 중인 가변 데이터는 동기화해 사용하라)

멀티 스레드 환경에서는 지연 초기화를 하기가 까다롭기 때문에 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다. 그러니 진짜 지연 초기화가 필요할때까지 사용하지 말자.

# 초기화 방법

## 1. 일반적인 초기화 방법

```java
private final FieldType field = computeFieldValue();
```

final 한정자를 사용한다.

## 2. 지연 초기화 - synchronized 한정자

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

지연 초기화가 초기화 순환성을 깨트릴 것 같다면 synchronized를 단 접근자를 사용하자.

위 두 관용구는 정적 필드에도 똑같이 적용된다.

## 3.  정적 필드 지연 초기화 홀더 클래스 관용구

성능 때문에 **정적 필드를 지연 초기화**해야 한다면 **지연 초기화 홀더 클래스 관용구**를 사용하자. 클래스가 처음 쓰일 때 초기화된다는 특성을 이용한 관용구이다.

```java
// 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

getField() 메서드가 호출되면 FieldHolder.field가 읽히면서 FieldHolder 클래스 초기화를 촉발한다. 동기화를 하지 않기 때문에 성능이 느려질 걱정이 전혀 없다는 장점이 있다.

## 4. 인스턴스 필드 지연 초기화 이중검사 관용구

성능 때문에 **인스턴스 필드를 지연 초기화**해야 한다면 **이중검사 관용구**를 사용하자. 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.

```java
// 인스턴스 필드용 지연 초기화 이중검사 관용구
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null)    // 첫 번째 검사 (락 사용 안 함)
        return result;

    synchronized(this) {
        if (field == null) // 두 번째 검사 (락 사용)
            field = computeFieldValue();
        return field;
    }
}
```

한 번은 동기화 없이 검사하고, 두 번째는 동기화하여 검사한다. 

두 번째 검사에서도 필드가 초기화되지 않았을 때만 필드를 초기화한다. 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 volatile로 선언해야 한다.

> result 지역 변수가 필요한 이유?
필드가 이미 초기화된 상황에서 이 필드가 한번만 읽도록 보장하고 성능을 높여준다.
> 

## 5. 이중검사 관용구의 변종1- 단일검사 관용구

이중검사 관용구에서 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화할 때 사용할 수 있다. 

```java
// 이중검사 관용구의 변종 - 단일검사 관용구(초기화가 중복해서 일어날 수 있다.)
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

## 6.  이중검사 관용구의 변종2- 짜릿한 단일검사 관용구

모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 필드 선언에서 volatile 한정자를 없애도 된다. 이를 짜릿한 단일검사(racy single-check) 관용구라 부른다.

```java
// 이중검사 관용구의 변종 - 짜릿한 단일검사 관용구(field의 선언에 volatile 한정자를 제거했다.)
private FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

이 관용구는 필드의 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다. 

**아주 이례적인 기법으로, 거의 사용되지 않는다.**

# 핵심 정리

- 대부분의 필드를 지연 초기화 대신 곧바로 초기화하자.
- 성능 때문에 지연 초기화를 사용해야 한다면 올바른 지연 초기화 기법을 사용하자.
- 정적 필드의 지연 초기화에는 홀더 클래스 관용구를 사용하자.
- 인스턴스 필드는 이중검사 관용구를 사용하자.(반복 초기화해도 괜찮다면 단일검사 관용구를 사용)