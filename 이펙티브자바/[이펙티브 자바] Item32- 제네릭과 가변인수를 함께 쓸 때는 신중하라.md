# [이펙티브 자바] Item32- 제네릭과 가변인수를 함께 쓸 때는 신중하라

# 제네릭과 가변인수를 함께 쓸 때 문제점

가변인수 메서드(varargs)는 제네릭과 함께 Java5에 함께 추가되었다. 함께 추가되었으니 서로 잘 어우러질 거라 기대하지만 사실을 그렇지 않다.

가변인수 메서드를 호출하면 자동으로 배열이 만들어진다. 내부로 감췄어야 할 배열을 클라이언트로 노출하게 되면서 문제가 발생한다. 

제네릭이나 매개변수화 타입을 포함한 가변인수 메서드를 호출하면 컴파일러가 경고를 보낸다.

![](https://blog.kakaocdn.net/dn/tyUS4/btq3gbysI4y/XtEpjywvYo58SAha4vLSA1/img.png)

 아래의 예시처럼 힙 오염이 일어날 수 있기 때문이다. 

```java
// 제네릭과 가변인수를 혼용하면 타입 안전성이 깨진다.
static void dangerous(List<String>...stringLists){
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;    // 힙오염 발생
  String s = stringLists[0].get(0)     // ClassCastException
}
```

타입 안정성이 깨지므로 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

# 왜 제네릭 varargs 배열 매개변수는 경고일까?

Java에서는 제네릭 배열 생성을 허용하지 않는다. 즉, 컴파일 에러가 발생한다. 그런데 왜 제네릭 배열이 만들어지는 제네릭 varargs 배열 매개변수는 허용한 걸까?

그 이유는 실무에서 굉장히 유용하기 때문이다. 그래서 이 모순을 수용한 것이다.

실제로 Java 라이브러리에도 이 모순된 메서드를 여럿 제공한다.

**Arrays.asList**와 **Collections.addAll** 등이 있다.

![](https://blog.kakaocdn.net/dn/cTFmlg/btq3cSsC620/URrmOtpATJljkO89MCzHJK/img.png)

Arrays.asList

![](https://blog.kakaocdn.net/dn/UQNXN/btq3cSlRmf8/lRpTHbN7Uhl5a4MH8CuVf0/img.png)

# 가변인수 메서드의 타입 안전을 보장하는 장치

Java7 부터는 **@SafeVarargs** 애너테이션이 추가되어, **타입 제네릭 가변인수 메서드**가 **타입 안전할 경우 경고를 숨길 수 있게 됐다.** 즉, 메서드 작성자가 제네릭 가변인수 메서드의 타입 안전함을 보장하는 장치로 사용된다. 비검사 경고를 제거했던 **@SuppressWarnings("unchecked")** 처럼 말이다.

위에서 살펴본 **Arrays.asList**와 **Collections.addAll** 등의 메서드도 **@SafeVarargs**로 해당 메서드가 타입 안전함을 보장하고 있다.

![](https://blog.kakaocdn.net/dn/Zmspd/btq3hVhpHuw/DWR8J3IOJ5Nr0VOaMZMyt1/img.png)

![](https://blog.kakaocdn.net/dn/FEWtu/btq3dygoOd4/l2Rp7wvi2vth6WfKbpyVIK/img.png)

# 가변인수 메서드가 안전한지 확인하는 방법

다음 두 조건을 모두 만족하는 제네릭 varargs 메서드는 안전하다고 할 수 있다. **단, 하나라도 어겼다면 수정이 필요하다.**

### 1. varargs 매개변수 배열에 아무것도 저장하지 않는다.

### 2. varargs 배열이 신뢰할 수 없는 코드에 노출되지 않는다.

위의 두 조건이 만족됨을 확인했다면 @SafeVarargs 애너테이션으로 경고를 제거하자.

@SafeVarargs 애너테이션은 재정의한 메서드도 안전할지 보장할 수 없기 때문에, 재정의 할 수 없는 메서드에만 달아야한다.

Java8 - static 메서드, final 인스턴스 메서드 허용
Java9 - static 메서드, final 인스턴스 메서드, private 인스턴스 메서드 허용

# @SafeVarargs 대체 방안

varargs 배열 매개변수@SafeVarargs로 경고를 제거하는 것이 유일한 해결 방법은 아니다. varargs 매개변수를 List 매개변수로 바꿀 수도 있다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists){
  List<T> restul = new ArrayList<>();
  for(List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

이 방식의 장점은 컴파일러가 메서드의 타입 안정성을 검증할 수 있다는 것이다. 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있지만, 우리가 직접 @SafeVarargs 애너테이션을 달지 않아도 된다.