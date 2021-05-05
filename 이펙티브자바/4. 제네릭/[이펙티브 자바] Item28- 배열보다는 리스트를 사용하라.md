# [이펙티브 자바] Item28- 배열보다는 리스트를 사용하라

배열과 제네릭 타입의 가장 큰 차이는 공변과 실체화다. 배열은 **공변**이며 **실체화가 가능**하고 제네릭은 **불공변**이며 **실체화가 불가능**하다. 

# 배열보다 리스트를 사용해야하는 첫 번째 이유

개발자라면 누구나 컴파일 타임에 오류를 발견하는 것을 선호한다. 배열은 런타임에야 오류를 발견할 수 있지만, 제네릭은 컴파일 타임에 오류를 발견할 수 있다.

아래의 설명을 통해 왜 그런지 알아보자.

### 배열은 공변?

공변은 함께 변한다는 뜻이다. 

Item28에서 말하는 배열의 공변이란 Sub가 Super의 하위 타입이라면 배열 Sub[]는 Super[]의 하위타입이라는 것을 의미한다. 더 쉽게 이해하기 위해 다음 코드를 살펴보자.

```java
// 컴파일에 이상이 없다.
Object[] objectArr = new Long[1]; // 공변
```

Long은 Obejct의 하위 타입이기 때문에 배열 Long 배열은 Object 배열의 하위 타입이다. 따라서 위의 코드는 이상없이 컴파일 된다.

반면에, 제네릭은 서로 다른 타입 Type1과 Type2가 존재할 때 List<Type1>과 List<Type2>는 하위 타입도 상위 타입도 아니다. 

```java
// 컴파일 에러
List<Object> objectList = new ArrayList<Long>(); // 불공변
```

제네릭은 불공변이므로 ArrayList<Long>은 List<Object>의 하위 타입도 상위 타입도 아니다. 따라서 컴파일 타임에 호환되지 않는 타입임을 인지하고 컴파일 에러가 발생하게 된다.

```java
Object[] objectArr = new Long[1]; // 공변
objectArr[0] = "문자열을 넣을 수 있나요?" // 런타임 에러 발생

List<Object> objectList = new ArrayList<Long>(); // 컴파일 에러 발생
```

이게 **배열보다 리스트를 사용**하라고 한 **첫 번째 이유**이다. 

# 배열보다 리스트를 사용해야하는 두 번째 이유

### 제네릭 배열의 문제점

배열은 실체화가 가능한 자료형이다. 즉, 런타임에 타입 정보를 가지고 자신이 담기로한 타입을 인지하고 확인한다는 것이다. 따라서 **배열은 런타임에는 타입 안전하지만 컴파일 타임에는 타입 안전성을 보장할 수 없다.**

반면에, 제네릭은 컴파일 타임에 타입을 검사하고 런타임에 소거한다. 런타임에 원소 타입 정보를 가지고 있지 않고, 원소 타입을 런타임에 알 수 있는 방법이 없다. 따라서 **제네릭은 컴파일 타임에는 타입 안전하지만 런타임에는 그렇지 않다.**

이러한 주요 차이점 때문에 배열과 제네릭을 섞어 쓰기 어렵다. 예를 들어, 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

```java
// 사용 불가!!!!! 컴파일 에러

new List<E>[] // 제네릭 타입 배열

new List<String>[] // 매개변수화 타입 배열

new E[] // 타입 매개변수 배열
```

제네릭 배열이 생성이 허용되지 않는 이유는 타입 안전(Type Safe) 하지 않기 때문이다. 

만약 **제네릭 배열이 허용된다면** 다음과 같은 문제가 발생할 수 있다.

```java
// 제네릭 배열 선언, 컴파일된다고 가정하자.
List<String>[] stringLists = new List<String>[1];  // 1 (제네릭 배열)
List<Integer> intList = List.of(42);               // 2 

Object[] objects = stringLists;                    // 3
objects[0] = intList;                              // 4
String s = stringLists[0].get(0);                  // 5
```

위의 코드에서 3번부터 살펴보자. Object[]에 List<String>[](제네릭 배열)을 할당했다.

```java
List<String>[] stringLists = new List<String>[1]; // 1

Object[] objects = stringLists; // 3
// Object[] objects = (Obejct[]) (List<String>[]) stringLists;
```

 List<E>는 Object의 하위 타입이고 배열은 공변이니 아무 문제 없이 할당 가능하다.

4번에서는 List<Integer>의 인스턴스를 objects 배열에 첫 원소로 저장한다.

```java
List<Integer> intList = List.of(42); // 2 
// 런타임시 타입이 소거되어 List가 된다.

objects[0] = intList; // 4
// objects 배열에는 List<String>[] 이 할당되어있다. 런타임시 List[]가 된다.

// 이해를 돕기위해 치환
stringLists[0] = intList;
// 1. (List<String>) stringLists[0] = (List<Integer>) intList;
// 2. 런타임시 타입이 소거된다.
// 3. (List) stringLists[0] = (List) intList;

// 즉, List[0] = (List) intList 의 형태를 갖는다. 따라서 문제 없이 저장된다.
// List<String> 배열에 List<Integer>의 인스턴스가 저장되었다. (힙 오염)
```

제네릭은 소거 방식으로 구현되기 때문에 List<Integer>의 인스턴스 타입은 List가 된다. 따라서 Object 배열의 첫 번째 원소로 이상없이 저장된다.

문제는 바로 5번이다.

```java
// 런타임시 ClassCastException 발생
String s = stringLists[0].get(0);
// 1. String s = ((List<Integer>) intList).get(0)) ---> intList에는 42가 할당되어있다.
// 2. String s = (String) 42 --> 런타임 시 ClassCastException 발생
```

컴파일러는 get(0)로 꺼낸 원소를 String으로 형변환하는데, 꺼내온 원소는 Integer이므로 런타임에 ClassCastException이 발생한다. 

위와 같은 상황을 방지하기 위해 제네릭 배열 생성을 막아놓은 것이다.

### 배열보다 리스트를 사용해야하는 두 번째 이유

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열 E[] 대신 List<E>를 사용하면 해결된다. 코드가 길어지고 성능이 저하될 수 있지만, 타입 안정성과 호환성은 좋아진다.

아래와 같이 생성자로 Collection을 받고 choose() 메서드로 랜덤으로 선택된 collection element를 리턴한다고 해보자. 

```java
// 제네릭을 필요로 하는 클래스
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
				return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

이 클래스를 사용하려면 메서드 호출을 사용할 때마다 Object를 원하는 타입으로 형 변환 해야하며, 형식이 잘못되면 런타임에 형 변환에 실패한다. 따라서 이 클래스를 제네릭으로 만들어보자.

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = (T[]) choices.toArray();
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

컴파일을 시도하면 다음과 같은 경고가 발생한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcYfGhN%2Fbtq1D0yqukv%2FXkpb9XokH6B7rXsQklK6w0%2Fimg.png)

T가 무슨 타입인지 알 수 없으니 런타임시 형변환이 안전한지 보장할 수 없다는 메시지다. 이 프로그램은 문제 없이 동작한다. 다만, 제네릭에서는 런타임시 타입이 소거되기 때문에 컴파일러가 안전을 보장하지 못할 뿐이다.

위와 같은 비검사 경고를 제거하려면 배열을 리스트로 변경해주면 된다.

```java
public class Chooser<T> {
    private final List<T> choiceArray;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

배열을 리스트로 변경하면서 코드가 조금 늘었고 조금 더 느릴테지만, 비검사 경고 뿐만아니라 런타임에 ClasscastException을 걱정 안해도 되니 그만한 가치가 있다.