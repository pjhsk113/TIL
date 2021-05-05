# [이펙티브 자바] Item27- 비검사 경고를 제거하라

제네릭을 사용하기 시작하면 수많은 컴파일러 경고들을 마주치게 된다. 이러한 경고들을 가능한 많이 제거하는 것이 좋다. 경고들을 모두 제거한다면, 그 코드는 타입 안정성이 보장되기 때문이다.

다행히 대부분의 비검사 경고는 쉽게 제거할 수 있다. 대부분의 개발자들이 IDE를 통해 코드를 작성하고 컴파일 하는데, IDE는 코드상에 컴파일 경고가 날 수 있는 부분을 컴파일 전에 미리 알려준다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F9SMth%2Fbtq091dGWVl%2FNt1vwiacVwKvQROgWhl6nK%2Fimg.png)

```java
// 비검사 경고 발생
Set<String> some = new HashSet(); // 타입 매개변수 추론이 어렵다.
```

위의 코드는 가장 간단한 예시로, 자바 7부터 지원하는 `<>` 다이아몬드 연산자를 추가하면 간단히 경고가 제거된다. 

```java
Set<String> some = new HashSet<>(); // 컴파일러가 타입 매개변수를 추론해준다.
```

## 경고를 제거할 수 없지만 타입 안전이 보장되면 경고를 숨겨라.

안전하다고 검증된 비검사 경고를 숨기지 않으면, 진짜 문제가 있는 경고가 나와도 알아차리기 힘들 수 있다. 만약 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애노테이션을 달아 경고를 숨기자.

단, 타입 안전성을 검증하지 않고 경고를 숨기면 절대 안된다. 해당 코드는 경고없이 컴파일되지만, 런타임시 여전히 ClassCastException을 던질 수 있다.

## @SuppressWarnings 애노테이션 사용 시 주의 사항

### 1. 가능한 좁은 범위로 적용해야한다.

해당 애노테이션은 어떤 선언부에도 달 수 있다. 즉, 지역변수, 메서드, 클래스 등 어떤 선언부에도 달 수 있다. 그렇기 때문에 @SuppressWarnings는 최대한 좁은 범위에 적용해야 한다. 보통은 변수 선언, 아주 짧은 메서드 혹은 생성자에 달게 된다. 

범위를 크게 잡아 클래스 전체에 적용했을 경우 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체 범위에 선언해서는 안된다.

ArrayList의 toArray를 예시로 들어보자.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBfxDb%2Fbtq091dG0HM%2Fraq0kbeyeZ58aFGP74cxLk%2Fimg.png)

메서드에 @SuppressWarnings가 선언되어있다. 이를 조금 더 좁은 범위로 좁히려면 아래와 같이 수정하면 된다.

```java
// @SuppressWarnings 범위를 좁히는 방법에 대한 예시.
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		// return에는 선언할 수 없으므로 지역 변수를 생성해 애너테이션을 달아준다.
		@SuppressWarnings("unchecked") T[] result = 
			(T[]) Arrays.copyOf(elementData, size, a.getClass());
		return result;
	}
	System.arraycopy(elementData, 0, a, 0, size);
	if (a.length > size) {
		a[size] = null;
	}
	return a;
}
```

### 2. @SuppressWarnings 애너테이션을 사용할 때는 항상 주석을 남겨라.

해당 코드의 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야한다. 다른 사람이 코드를 이해하는데 도움이 되고, 더 중요하게는 다른 사람이 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다.

```java
// @SuppressWarnings 주석을 꼭 달아주자!
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		// 비검사 경고 제거
		// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
		@SuppressWarnings("unchecked") T[] result = 
			(T[]) Arrays.copyOf(elementData, size, a.getClass());
		return result;
	}
	System.arraycopy(elementData, 0, a, 0, size);
	if (a.length > size) {
		a[size] = null;
	}
	return a;
}
```