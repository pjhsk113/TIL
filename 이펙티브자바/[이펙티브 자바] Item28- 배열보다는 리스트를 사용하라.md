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

### 실체화 가능 타입과 실체화 불가능 타입

배열은 실체화가 가능한 자료형이다. 즉, 런타임에도 타입 정보를 가지고 있어 자신이 담기로한 타입을 인지하고 확인한다는 것이다. 따라서 배열은 컴파일 타임에 형 안전성을 보장할 수 없다.

반면에, 제네릭은 컴파일 타임에 타입을 검사하고 런타임에 소거한다. 따라서 런타임에 원소 타입 정보를 가지고 있지 않고, 원소 타입을 런타임에 알 수 있는 방법이 없다. 

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
// 제네릭 배열 선언
List<String>[] stringLists = new List<String>[1];  // 1 (제네릭 배열)
List<Integer> intList = List.of(42);               // 2 

Object[] objects = stringLists;                    // 3
objects[0] = intList;                              // 4
String s = stringLists[0].get(0);                  // 5
```

위의 코드에서 3번부터 살펴보자. Object[]에 List<String>[](제네릭 배열)을 할당했다.

```java
List<String>[] stringLists = new List<String>[1]; // 1

Object[] objects = List[] stringLists; // 3
```

 List<E>는 Object의 하위 타입이고 배열은 공변이니 아무 문제 없이 할당 가능하다.

4번에서는 List<Integer>의 인스턴스를 Object 배열에 첫 원소로 저장한다.

```java
List<Integer> intList = List.of(42); // 2 

objects[0] = intList; // 4
```

제네릭은 소거 방식으로 구현되기 때문에 List<Integer>의 인스턴스 타입은 List가 된다. 따라서 Object 배열의 첫 번째 원소로 이상없이 저장된다.