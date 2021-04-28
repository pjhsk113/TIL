# [이펙티브 자바] Item33- 타입 안전 이종 컨테이너를 고려하라

# 타입 안전 이종 컨테이너 패턴

**컨테이너 대신 키를 매개변수화하고 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 것을 타입 안전 이종 컨테이너 패턴이라고 한다.**

하나의 컨테이너에서 매개변수화되는 대상은 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. 이보다 **유연한 수단이 필요할 때** **타입 안전 이종 컨테이너 패턴을 사용할 수 있다.**

이제 타입 안전 이종 컨테이너 패턴의 예제를 살펴보자.

```java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites{
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type)
}
```

각 타입의 Class 객체를 매개변수화하여 키 역할로 사용한다. 이때, class 리터럴의 타입은 Class가 아닌 Class<T>이다. 그리고 컴파일 타임 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰**이라 부른다.

이번엔 Favorites API를 사용하는 클라이언트 코드를 살펴보자.

```java
// 타입 안전 이종 컨테이너 패턴 - 클라이언트
public static void main(Stringg[] args) {
	Favorites f = new Favorites();

	f.putFavorite(String.class, "JAVA");

	String favoriteString = f.getFavorite(String.class);

	System.out.printf("%s", favoriteString);
}
```

결과로 "JAVA"가 출력된다. 즉, 메서드들이 타입 토큰을 주고받으며 올바른 값을 반환한다. String을 요청했는데 Integer를 반환하는 일은 절대 없다. 또한, 맵과 달리 여러 가지 타입의 원소를 담을 수 있다. 따라서 Favorites는 **타입 안전 이종 컨테이너**라 부를 수 있다.