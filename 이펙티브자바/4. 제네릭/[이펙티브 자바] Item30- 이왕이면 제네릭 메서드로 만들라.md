# [이펙티브 자바] Item30- 이왕이면 제네릭 메서드로 만들라

Item29에서 설명한 것과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환 하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.

이왕이면 제네릭 타입으로 만든 것처럼 메서드도 제네릭 메서드로 만들자.

**매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭**이다. 예를들면 Collections의 알고리즘 메서드(sort, binarySearch)는 모두 제네릭이다.

# 제네릭 메서드 작성법

## 단순한 제네릭 메서드

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>();
	result.addAll(s2);
	return result;
}
```

타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.

위와 같은 제네릭 메서드는 모든 매개변수의 타입이 같아야 동작한다. 따라서 **유연함이 떨어진다.** 이를 한정적 와일드 카드 타입을 사용하여 조금 더 유연하게 개선할 수 있다.

## 제네릭 싱글턴 팩토리 패턴

불변 객체를 여러 타입으로 활용할 수 있게 만들어야할 때가 있다. 하지만 여러 타입으로 만들기 위해서는 요청한 타입 매개변수에 맞게 객체 타입을 바꿔주는 정적 팩터리가 필요하다.

이러한 패턴을 **제네릭 싱글턴 팩토리 패턴**이라고 부른다. 제네릭 싱글턴 팩토리 패턴을 사용하는 예제로 **함수 객체**와 **재귀적 타입 한정**이 있다.

### **함수 객체**

아래의 예시는 함수 객체인 Collections.reversOrder() 이다.

> 함수 객체란 함수 내부에 들어가는 객체를 말한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FH7qB3%2Fbtq12vFkE8j%2FGIVLo4ogzV54iDLwVY2Wzk%2Fimg.png)


![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMP6Pn%2Fbtq17X1CfMX%2F1vlksRwpTuVUZGgjnQToF0%2Fimg.png)

reverseOrder()는 ReverseComparator의 싱글턴 객체 REVERSE_ORDER를 Comparator<T> 타입으로 형변환 해주는 역할을 한다. 이와 같은 형태가 제네릭 싱글턴 팩토리 패턴의 모습이라고 생각하면 된다.

### **재귀적 타입 한정**

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 이를 **재귀적 타입 한정**이라 부른다.

재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable과 함께 쓰인다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

**<E extends Comparable<E>>**는 "모든 타입 E는 자신과 비교할 수 있다."라는 의미를 가지고 있다.

# 정리

타입과 마찬가지로 메서드도 형변환 없이 사용할 수 있는 편이 좋다. 따라서 기존에 형변환을 해줘야하는 메서드가 있다면 제네릭하게 만들어보자