# [이펙티브 자바] Item53 - 가변인수는 신중히 사용하라

인수 개수가 일정하지 않은 메서드를 정의해야 한다면, 가변인수가 반드시 필요하다. 하지만 가변인수는 신중하게 사용해야 한다.

인수가 1개 이상이어야 하는 가변인수 메서드에서 인수를 0개 받을 수 있도록 설계하는 것은 좋지 않다. 인수 개수는 런타임에 배열의 길이로 알 수 있기 때문이다.

예를 들면, 최솟값을 찾는 가변인수 메서드가 있다고 생각해보자.

```java
// 인수 1개 이상어야하는 최솟값을 찾는 가변인수 메서드 - 잘못 구현한 예제
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

위의 예제는 인수를 0개 받는 경우 런타임에 실패한다. 또한, 코드도 지저분하고 args 유효성 검사를 명시적으로 해야한다. min의 초기값을 Integer.MAX_VALUE로 설정하지 않고는 for-each문도 사용할 수 없다.

이 문제는 매개변수를 2개 받도록 강제하면 간단히 해결된다.

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

## 성능에 민감한 상황이라면..

성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문이다.

가변인수의 유연성이 필요하지만, 성능에 대한 비용을 감당할 수 없는 상황이라면 다중정의를 통해 해결할 수 있다.

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a4, int... rest) {}
```

만약 메서드 호출의 95%가 인수를 3개 이하로 사용한다면, 마지막 5번째 다중정의 메서드에 가변인수를 추가해주자. 이렇게 하면 5%의 호출에서만 배열을 생성하기 때문에 특수한 상황에는 성능 최적화를 기대할 수 있게 된다.

이를 활용한 대표적인 예로는 EnumSet의 정적 팩터리가 있다. EnumSet의 정적 팩터리는 위 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.

# 핵심 정리

- 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.