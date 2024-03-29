# [이펙티브 자바] Item70 - 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

자바에서 문제 상황을 알리는 타입(throwable)으로는 검사 예외(checked Exception), 런타임 예외(Runtime Exception), 에러 이렇게 3가지를 제공한다.

언제 무엇을 사용해야 하는지 참고하면 좋은 지침들에 대해 알아보자.

# 검사 예외와 비검사 예외를 구분하는 기본 규칙

> 호출하는 쪽에서 복구하리라 예상되는 상황이라면 검사 예외를 사용하자.

검사 예외를 던지면 호출자가 그 예외를 catch로 처리하거나, 더 바깥으로 전파하도록 강제된다. 따라서 **메서드 선언에 포함되는 예외**는 해당 **메서드를 호출했을 때 발생할 수 있는 유력한 결과**를 나타낸다.

즉, API 설계자는 API 사용자에게 검사 예외(checked Exception)을 넘겨주어 그 상황에서 복구하라고 요구한 것이다. 

따라서 이 상황이 검사 예외와 비검사 예외를 구분하는 기본 규칙이 된다. 

# 비검사 예외(UnChecked Exception)

비검사 throwable은 두 가지로, **런타임 예외**와 **에러**로 나뉜다.

이 둘은 통상적으로 프로그램에서 잡을 필요가 없거나 잡지 말아야 한다. 프로그램에서 비검사 예외를 던진다는 것은 복구가 불가능하거나 더 실행해봐야 잃는게 많다는 의미다.

이런 throwable을 잡지않은 스레드는 적절한 오류 메시지를 뱉으며 중단된다.

## 런타임 예외(Runtime Exception)

> 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자.

런타임 예외는 대부분 전제조건을 만족하지 못했을 때 발생한다. 예를 들면, ArrayIndexOutOfBoundsExceptin은 배열의 전제조건이 만족되지 않았을 경우 발생한다.

**런타임 예외의 문제점**은 검사 예외를 던질것인지 비검사 예외를 던질 것인지 명확히 구분되지 않는다는 것이다. 실제로 메모리가 부족해서 발생하는 문제일 수도, 단순한 프로그래밍 오류일 수도 있기 때문이다. 

따라서 복구될 수 있는 상황인지 아닌지는 API 설계자가 판단해 검사예외를 던질지, 비검사 예외를 던질지를 결정해야 한다. 만약 확신하기 어렵다면 비검사 예외를 던지는 편이 더 났다.

## 에러

에러는 보통 JVM 자원 부족, 불변식 깨짐 등 더 이상 수행할 수 없는 상황을 나타낼 때 사용한다.

**구현하는 비검사 예외(UnChecked Exception)는 모두 RuntimeException의 하위 클래스여야 한다.** Error는 상속하지 말아야 할 뿐 아니라, throw 문으로 직접 던지는 일은 없어야 한다. (AssertionError는 제외)

Exception, RuntimeException, Error를 상속하지 않고 throwable을 만들어 사용할 수 있지만, 절대 그런짓은 하지말자. 검사 예외보다 나을 게 하나도 없으면서 사용자를 헷갈리게 만들기 때문이다.

## 예외의 메서드

예외 역시 어떤 메서드라도 정의할 수 있는 완벽한 객체다. 예외의 메서드는 주로 그 예외를 일으킨 상황에 관한 정보를 코드 형태로 전달하는데 쓰인다.

이런 메서드가 없다면 오류 메시지를 파싱해 정보를 빼내야하는데, 이는 굉장히 나쁜 습관이다.

throwable 클래스들은 오류 메시지 포맷을 상세히 기술하지 않았다. 즉, JVM이나 릴리즈에 따라 포멧이 변경될 수 있다는 뜻이다. 오류 메시지를 파싱해 얻은 코드는 깨지기 쉽고 다른 환경에서 동작하지 않을 수 있다.

**검사 예외는 일반적으로 복구할 수 있는 조건**일 때 발생한다. 따라서 **예외 상황을 벗어나는데 필요한 정보를 알려주는 메서드를 함께 제공하자.**

# 핵심 정리

- 복구할 수 있는 상황이라면 검사 예외(Checked Exception)을, 프로그래밍 오류라면 비검사 예외(UnChecked Exception)을 던지자.
- 확실하지 않다면 비검사 예외를 던지자.
- 검사도 아니고 비검사도 아닌 throwable은 정의하지 말자.
- 검사 예외라면 필요한 정보를 알려주는 메서드도 함께 제공하자.