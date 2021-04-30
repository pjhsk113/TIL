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

결과로 "JAVA"가 출력된다. 즉, 메서드들이 **타입 토큰**을 주고받으며 올바른 값을 반환한다. String을 요청했는데 Integer를 반환하는 일은 절대 없다. 또한, 맵과 달리 여러 가지 타입의 원소를 담을 수 있다. 따라서 Favorites는 **타입 안전 이종 컨테이너**라 부를 수 있다.

이제 Favorites의 구현부를 살펴보자.

```java
// 타입 안전 이종 컨테이너 패턴 - 구현
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();
  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
  }
  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```

위의 예제에서 Map의 타입을 살펴보면 `Map<Class<?>, Object>` 이다. 비한정적 와일드카드 타입이라 Map안에 아무것도 넣을 수 없다고 생각할 수 있지만, 사실은 Map이 아니라 키가 와일드카드 타입이다. 즉, 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻이다. 예를들면 첫 번째는 **Class<String>** 두 번째는 **Class<Integer>** 처럼 될 수 있다.

그리고 `Map<Class<?>, Object>`의 Value 타입을 보면 Object이다. 이는 키와 값 사이의 타입 관계를 보증하지 않는다는 뜻이다. Java의 타입 시스템에서는 이 관계를 명시할 방법이 없다. 하지만 우리는 Class의 cast 메서드를 통해 동적 형변환을 해줌으로써 이 관계가 언제나 성립함을 알 수 있다.

# 타입 안전 이종 컨테이너의 제약

위에서 살펴본 Favorites 클래스**(타입 안전 이종 컨테이너)**에는 알아두어야 할 두 가지의 제약이 존재한다.

### 1. Class 객체를 Raw Type으로 넘기면 타입 안정성이 쉽게 깨진다.

```java
f.putFavorite((Class) Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoriteInteger = f.getFavorite(Integer.class) // ClassCastException
```

위의 코드는 putFavorite를 사용할 때는 정상적으로 동작하지만 getFavorite를 호출하면 ClassCastException이 발생한다. 이 문제를 해결하려면 put을 할 때, 동적 형변환 **type.cast()** 를 해주면 된다. 

### 2. 실체화 불가 타입에는 사용할 수 없다.

실체화 불가 타입인 **List<String>은 저장할 수 없다.** List<String>용 Class 객체를 얻을 수 없기 때문이다. 

만약 List<String>.class와 같은 문법이 허용되었다면, 객체 내부는 아수라장이 될 것이다. List<String>.class와 List<Integer>.class는 결국 List.class를 공유하므로 둘 다 똑같은 타입의 객체 참조를 반환하기 때문이다.

# 타입 토큰 제한, 한정적 타입 토큰

위에서 살펴본 Favorites가 사용하는 타입 토큰은 어떤 Class 객체든 받아들이기 때문에 비한정적이라 할 수 있다. 때로는 이 타입들을 제한하고 싶을 수 있는데, 이때 사용할 수 있는 것이 **한정적 타입 토큰이다**.

애너테이션 API는 이 한정적 타입 토큰을 아주 적극적으로 사용하고있다.

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType)
```

annotationType 인수(annotationClass)는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 위의 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려있다면 애너테이션을 반환하고, 없다면 null을 반환한다. **즉, 애너테이션된 요소는 그 키가 애너테이션 타입인 타입 안전 이종 컨테이너이다.**

만약 Class<?> 타입의 객체를 한정적 타입 토큰을 받는 메서드로 넘기려면 어떻게 해야할까? Class 클래스의 asSubclass 메서드를 사용하면 된다. asSubclass는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환 해준다. 형변환에 성공하면 인수로 받은 클래스 객체를 반환하고, 실패하면 ClassCastException을 반환한다.

# 핵심 정리
- 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 타입 안전 이종 컨테이너를 만들 수 있다.
- 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이를 **타입 토큰**이라 부른다.