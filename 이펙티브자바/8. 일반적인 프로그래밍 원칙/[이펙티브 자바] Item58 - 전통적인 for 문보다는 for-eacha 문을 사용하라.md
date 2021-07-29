# [이펙티브 자바] Item58 - 전통적인 for 문보다는 for-eacha 문을 사용하라

가능한 모든 곳에서 전통적인 for 문보다는 for-each 문을 사용하는 것이 좋다. 전통적인 for 문과 비교했을 때 for-each문은 명확하고, 유연할 뿐아니라 버그를 예방해주는 효과가 있다. 또 성능 저하도 없다.

```java
// 전통적인 for 문 - 컬렉션 순회
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
}
```

```java
// 전통적인 for 문 - 배열 순회
for (int i = 0; i < a.length; i++) {
    int z = a[i]; // a[i]로 무언가를 한다.
}
```

전통적인 for 문은 while문 보다는 낫지만 가장 좋은 방법은 아니다. 우리에게 필요한 건 원소뿐인데 for 문 안의 반복자와 인덱스가 코드를 지저분하게 만든다. 또한, 컬렉션인 경우 코드의 형태가 달라지기 때문에 주의를 기울여야 한다.

# 향상된 for 문 (for-each)

전통적인 for 문의 단점들을 보완한 반복문 관용구이다.

```java
List<String> list = List.of("a", "b", "c");

// 향상된 for 문 - 컬렉션 순회
for (String s : list) {
	System.out.println(s);
}
```

```java
String[] arr = new String[]{"a", "b", "c"};

// 향상된 for 문 - 배열 순회
for (String s : arr) {
	System.out.println(s);
}
```

반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다. 또한, 하나의 관용구로 배열과 컬렉션을 모두 처리할 수 있으니 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.

## for-each의 강점

컬렉션이 중첩되는 경우 for-each 문의 강점이 더욱 커진다. 

```java
// before
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();) {
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext();) {
        // 오류 발생!! i.next()가 너무 많이 호출됨.
        deck.add(new Card(i.next(), j.next()));
    }
}
```

deck.add(new Card(i.next(), j.next())); 에서 NoSuchElementException이 발생한다. 안쪽 for 문에서 i.next()가 너무 많이 호출되고 있기 때문이다.

이를 for-each 문으로 수정해보면 명확하고 간결한 코드로 만들 수 있다.

```java
// after
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Suit suit : suits) {
    for(Rank rank : ranks) {
        deck.add(new Card(suit, rank));   
    }
}
```

# for-each 문을 사용할 수 없는 세 가지 상황

### 1. 파괴적인 필터링

컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우 for-each 문은 사용할 수 없다. 자바 8부터는 컬렉션의 removeIf 메서드를 통해 컬렉션을 명시적으로 순회하지 않을 수 있다.

```java
// removeIf 사용예시 - 짝수만 제거
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9));
numbers.removeIf(n -> { n % 2 == 0 });
```

### 2. 변형

리스트나 배열을 순회하면서 원소 값 일부 혹은 전체를 교체해야 한다면 반복자나 배열의 인덱스를 사용해야 한다.

### 3. 병렬 반복

여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

위 세 가지 상황에 속한다면 전통적인 for 문을 사용하자.

# Iterable 구현을 고려해볼 만한 상황

for-each 문은 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

Iterable을 구현하는 것은 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현을 고려해보자.