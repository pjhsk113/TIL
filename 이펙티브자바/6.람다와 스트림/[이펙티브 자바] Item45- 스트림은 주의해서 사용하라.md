# [이펙티브 자바] Item45- 스트림은 주의해서 사용하라

Java8에 추가된 스트림 API는 다량의 데이터 처리를 돕고자 만들어졌다. 스트림 API가 제공하는 핵심 추상 개념은 **스트림**과 **스트림 파이프라인** 이렇게 두 가지다.

# 스트림 API 핵심 추상 개념

### 1. 스트림

데이터 원소의 유한 또는 무한 시퀀스를 나타낸다.

### 2. 스트림 파이프라인

원소들로 수행하는 **연산 단계를 표현**하는 개념이다. 

대표적인 스트림 소스로는 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기 등이 있다.

1. 컬렉션

    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQ8OPJ%2Fbtq66kx9WE1%2Fgc57o7C3uGugzlvvPBwc3k%2Fimg.png)

2. 배열

    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcFaXw3%2Fbtq64RXLa0t%2FfnjAd0HYfNxz86xcS0wW81%2Fimg.png)

3. 파일

    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvX6Ey%2Fbtq64xSJaT8%2FIe1JnwVnm8TGCU5gQra3Ak%2Fimg.png)

4. 정규표현식 패턴 매처

    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcj8u8T%2Fbtq650NsKS4%2FY1KEBq8VCYl4A0KbPGZAyK%2Fimg.png)

5. 난수 생성기
    1. 무한 스트림

        ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F68hFo%2Fbtq66u1QnXI%2FAIkTpRV8z4DcasN1bHXdwk%2Fimg.png)

        or

        ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fn0ckO%2Fbtq64mcSKy3%2FFuBG5uNYeLR6hnNBVEUYKK%2Fimg.png)

    2. 유한 스트림

        ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbfBp5h%2Fbtq65UflNRj%2FDr0woeFfSSP9pvFGlA9SOk%2Fimg.png)

6. 기본 스트림 - IntStream, LongStream, DoubleStream

## 스트림 파이프라인 특징

스트림 파이프라인은 **소스 스트림**으로 시작해 **종단 연산**으로 끝나며, 그 사이에 **중간 연산**이 들어갈 수 있다. 

```java
public class Dummy {
    public static void main(String[] args) {
        List<String> nameList = Arrays.asList("a", "b", "c");

        nameList.stream() // 소스 스트림
                .filter(s -> s.startsWith("a")) // 중간 연산
                .forEach(System.out::println); // 종단 연산
    }
}
```

### 중간 연산

각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 예를 들면, 각 원소에 함수를 적용하거나 특정 조건을 걸어 필터링할 수 있는 식이다.

중간 연산의 종류

**map**

- 입력 T 타입 요소를 R 타입 요소로 변환

**filter**

- 조건을 충족하는 요소를 필터링

**flatMap**

- 중첩된 구조를 한 단계 평탄화하고 단일 원소로 변환한 스트림 생성

**peek**

- 스트림 내의 각각의 요소를 대상으로 특정 연산을 수행

**skip**

- 처음 n개의 요소를 제외하는 스트림 생성

**limit**

- maxSize까지의 요소만 제공하는 스트림 생성

**distinct**

- 스트림 내의 요소의 중복 제거

**sorted**

- 스트림 내 요소를 정렬

### 종단 연산

종단 연산은 중간 연산이 내놓은 스트림에 최후 연산을 가한다. 원소를 정렬해 컬렉션에 담거나 모든 원소를 출력하는 식으로 사용할 수 있다.

종단 연산의 종류

**forEach**

- 스트림을 순회

**reduce**

- 연산을 이용해 모든 스트림 요소를 처리하여 하나의 결과로 만듬

**collect**

- 스트림의 연산 결과를 컬렉션 형태로 모아줌

### 지연 평가(Lazy evaluation)

스트림 파이프라인은 지연 평가된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 연산에 사용되지 않는다. 즉, 종단 연산이 없으면 스트림 파이프라인은 아무일도 하지 않게 된다. 이러한 지연 평가의 특징 덕분에 무한 스트림을 다룰 수 있게된다.

### 스트림 API의 메서드 연쇄

스트림 API는 플루언트 API로 메서드 연쇄를 지원한다. 따라서 파이프라인 하나를 구성하는 호출을 연결해서 하나의 표현식으로 만들 수 있다.

### 순차 실행

기본적으로 스트림 파이프라인은 순차적으로 수행된다. 이를 병렬로 실행하기 위해서는 parallel 메서드를 호출하면 된다. 하지만 효과를 볼 수 있는 상황은 많지 않다.

# 스트림 제대로 사용하기

스트림을 제대로 사용하면 코드가 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수가 어려워진다.

다음 예제를 살펴보자. 스트림을 과용하여 읽기 어려운 코드이다

```java
public class Anagrams {
    public static void main(String[] args) {
        List<String> dictionary = Arrays.asList("Dormitory", "Dirty Room", "Hot water", "Worth tea");

        dictionary.stream()
                .collect(groupingBy(word -> word.chars().sorted()
                                .collect(StringBuilder::new,
                                        (sb, value) -> sb.append((char)value),
                                        StringBuilder::append).toString()))
                .values().stream()
                .filter(group -> group.size() >= 2)
                .map(group -> group.size() +": " + group)
                .forEach(System.out::println);
    }
}
```

위의 코드를 적절한 스트림 사용으로 개선할 수 있다.

```java
public class Anagrams {
    public static void main(String[] args) {
        List<String> dictionary = Arrays.asList("Dormitory", "Dirty Room", "Hot water", "Worth tea");

        dictionary.stream()
                .collect(groupingBy(Anagrams::alphabetize))
                .values().stream()
                .filter(group -> group.size() >= 2)
                .forEach(group -> System.out.println(group.size() +": " + group))
    }

    // 도우미 메서드
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);;
        return new String(a);
    }
}
```

모든 구현을 스트림으로 표현하는 것이 아닌 alphabetize() 메서드를 통해 스트림을 적절한 부분에만 사용했다. 또한, 스트림 변수명을 이해하기 쉽도록 만들었다. 그 결과 이전 코드보다 훨씬 간결하고 이해하기 쉬운 코드가 만들어졌다.

이처럼 무조건 스트림을 사용하기보다는 절충 지점을 잘 찾아 사용하는 것이 중요하다. 

또 주의해야할 점이 있는데, 바로 **char 값들을 처리할 때는 스트림을 사용을 삼가해야 한다는 점**이다. 자바에서는 char용 스트림을 제공하지 않는다. char를 스트림 요소로 사용하면 char가 아닌 int값이 반환된다. 

```java
// char가 아닌 int값이 반환된다.
"hellow World".chars().forEach(System.out::println);
```

이름은 chars인데 int값이 반환되니 헷갈릴 수 있다. 또한 올바른 값을 출력하려면 명시적으로 형변환을 해줘야 한다.

```java
// 명시적인 형변환
"hellow World".chars().forEach(x -> System.out.println((char) x);
```

위와 같은 이유로 성능이 느려질 수 있다. 따라서 **char 값들을 처리할 때는 스트림을 사용을 삼가는 편이 낫다.**

### 스트림을 제대로 사용하는 최선의 방법!

기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 때만 반영하자. 스트림과 반복문을 적절히 조합해서 사용하는 것이 최선이 될 수 있다.

# 스트림 사용이 적절하지 않은 경우

스트림 파이프라인은 연산을 함수 객체로 표현한다. 함수 객체가 수행할 수 없는 계산 로직을 수행해야 하는 경우에는 스트림과 맞지 않는다고 할 수 있다.

### 1. 지역 변수를 읽거나 수정해야하는 경우

람다에서는 final 혹은 effectively final인 변수만 읽을 수 있고, 지역 변수를 수정하는게 불가능하다. 따라서 이러한 연산이 필요한 경우 코드 블럭을 사용해야 한다. 즉, 스트림 대신 반복 코드를 사용해야한다.

### 2. return, continue, break를 수행하거나 메서드 예외를 던져야 하는 경우

람다에서는 위의 나열된 로직 중 어떠한 것도 수행할 수 없다. 따라서 이와 같은 로직이 필요하다면 스트림과 맞지 않는다.

# 스트림 사용이 적절한 경우

1. 원소들의 시퀀스를 일관되게 변환한다.
2. 원소들의 시퀀스를 필터링한다.
3. 원소들의 시퀀스를 연산 후 결합한다.(더하기, 연결, 최솟값 구하기)
4. 원소들의 시퀀스를 컬렉션에 모은다.
5. 원소들의 시퀀스 중 특정 조건을 만족하는 원소를 찾는다.

이러한 일 중 하나를 수행하는 로직이라면 스트림을 적용하기 좋은 후보이다.

# 스트림으로 처리하기 어려운 일

스트림 파이프라인은 **하나의 중간 연산을 거치면 원래의 값(원래의 스트림)을 잃는 구조**를 가지고 있다. 따라서 여러 단계의 파이프라인을 거칠 때 원본 스트림을 사용해야 한다면 스트림으로 처리하기 어렵다.

# 스트림과 반복 중 어느 쪽을 선택해야할까?

스트림과 반복 중 어느쪽을 선택해야 할지 바로 알기 어려운 경우가 많다. 이 경우에는 스트림과 반복 두 가지 방법으로 구현해보고 더 나은쪽을 선택하자.

카드 덱을 초기화하는 예제를 살펴보자. 카드의 무늬(Suit)와 숫자(Rank) 별 모든 카드 조합을 만드는 예제이다.

```java
// 반복 방식
private static List<Card> newDeck() {
  List<Card> result = new ArrayList<>();
  for (Suit suit : Suit.values()) {
    for (Rank rank : Rank.values()) {
      result.add(new Card(suit, rank));
    }
  }

  return result;
}
```

스트림을 사용하지 않는 사람에게 가장 친숙한 방식이다. 그렇다면 이제 같은 동작을 스트림 방식으로 구현한 코드를 살펴보자.

```java
// 스트림 방식
private static List<Card> newDeck() {
  return Stream.of(Suit.values())
       .flatMap(suit -> 
            Stream.of(Rank.values())
                .map(rank -> new Card(suit, rank)))
       .collect(toList());
}
```

스트림이나 함수형 프로그래밍이 익숙하지 않고 확신이 서지 않는다면 반복 방식을, 스트림 방식이 더 나아보이고 동료들도 스트림을 선호한다면 스트림 방식을 사용하자.

즉, 정해진 정답은 없다. 개인 취향과 프로그래밍 환경에 따라 선택하면 된다.

# 핵심 정리

**스트림을 사용했을 때 깔끔하는게 처리할 수 있는 경우**가 있고, **반복 방식으로 더 직관적으로 표현할 수 있는 경우**가 있다. 많은 작업들이 이 둘을 적절히 조합해서 사용할 때 가장 멋지게 해결된다.

어느 쪽을 선택하는 정해진 규칙은 없다. 만약, **어느 방법이 더 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택해 사용하자.**