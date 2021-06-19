# [이펙티브 자바] Item47- 반환 타입으로는 스트림보다 컬렉션이 낫다

Java8 이전에는 원소를 반환하는 타입을 결정하는 것이 어렵지 않았다. 그런데 Java8에 스트림이 등장하면서 이 선택이 복잡한 일이 되어버렸다. 스트림이 반복(iteration)을 지원하지 않기 때문이다. 

Stream 인터페이스는 Iterable 인터페이스의 추상메서드를 모두 포함하고 정의한 방식대로 동작하지만 Iterable 인터페이스를 확장하지 않았다. 그래서 for-each로 스트림을 반복할 수 없다.

# 스트림을 반복하기 위한 방법

## 어댑터 메서드

**Stream<E>를 Iterable<E>로 중개해주는 어댑터**를 생성해서 사용한다.

```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream){
    return stream::iterable;
}

// 어댑터 메서드를 사용해 반복
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 처리 로직
}
```

어댑터 메서드를 사용하는 경우 타입 추론이 문맥을 파악하기 때문에 별도의 형변환이 필요 없어진다. 이 방법을 사용하면 어떤 스트림도 for-each로 반복할 수 있게된다.

위와 반대로 **Iterable<E>를 Stream<E>로 중개해주는 어댑터**도 손쉽게 구현할 수 있다.

```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static Iterable<E> streamOf(Iterable<E> iterable){
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰인다면 Stream을 반환하도록 하고 반대로 반복문에서만 쓰인다면 Iterable을 반환하자.

하지만 Stream과 반복문을 모두 사용하는 모두를 고려하여 이 둘을 모두 제공하는 것이 더 좋은 방법이다.

## Collection 반환

Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 모두 지원한다. 따라서 원소 시퀀스를 반환하는 **공개 API의 반환타입으로 Collection이나 그 하위 타입을 쓰는게 최선의 방법이다.**

### 컬렉션 반환 시 주의사항

**컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려선 안된다.** 메모리를 잡아먹는 부분이 기하급수적으로 늘어날 수 있기 때문이다. 

예를 들어, 주어진 집합의 멱집합을 반환하는 상황이라고 가정해보자.

{a, b, c}의 멱집합은 { {}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c} } 이다. 멱집합의 원소의 수는 2^3개 이다. 만약 원소 수가 N개라면 멱집합의 원소 개소는 2^n개가 된다.

이를 표준 컬렉션 구현체에 저장하려는 것은 위험한 생각이다. 만약 집합의 원소가 20개라면 이 컬렉션은 2^20만큼의 멱집합 원소를 탐색 후 반환해야한다.

원소 수가 적다면 ArrayList와 같은 표준 컬렉션에 담아 반환해도 되지만, 그렇지 않다면 전용 컬렉션을 구현할지 고민해봐야 한다.

### 전용 컬렉션 구현

**AbstractList를 이용한 전용 컬렉션 구현으로 해결할 수 있다.** 멱집합을 구성하는 각 원소의 인덱스를 비트 벡터로 사용하도록 하여 인덱스의 N번째 비트 값은 멱집합의 해당 원소가 원래 집합의 N번째 원소를 포함하는지 알려준다.

예를 들어, {a, b, c} 집합이 들어오면 멱집합의 원소 수는 8개이므로 인덱스는 0-7이다. 이진수는 000-111로 표현된다. 인덱스를 이진수로 나타내서 각 n번째 자리 값이 원래 집합의 원소 a, b, c를 포함하는지 여부를 알려준다. 즉, {a,b}면 100, {a,c} 면 101로 표현하는 식으로 멱집합을 나타낸다.

```java
// 멱집합 전용 컬렉션
public class PowerSet{
    public static final <E> Collection<Set<E>> of(Set<E> s){
        List<E> src = new ArrayList<>(s);
        // Integer.MAX_VALUE까지만 반환 가능
        if (src.size() > 30) {
            throw new IllegalArgumentException(
                  "집합에 원소가 너무 많습니다.(최대 30개).:" + s);

        return new AbstractList<Set<E>>(){
            @Override
            public int size(){
                // 멱집합의 크기는 2^(원래 집합의 원소 수)
                return 1 << src.size();
            }
            @Override
            public boolean contains(Object o){
                return o instanceof Set && src.containsAll((Set)o);
            }
            @Override
            public Set<E> get(int index){
                Set<E> result = new HashSet<>();
                for(int i = 0; index != 0; i++, index >>= 1){
                    if ((index&1)==1)
                        result.add(src.get(i));
                }
                return result; 
            }
        }
    }
}
```

위의 코드는 멱집합 전용 컬렉션을 구현한 예제인데, 컬렉션을 반환 타입으로 쓸 때의 단점을 잘 보여준다. size() 메서드는 int를 반환하므로 PowerSet.of가 반환되는 시퀀스의 최대 길이는Integer.MAX_VALUE로 제한된다. 반면에 Stream이나 Iterable은 size에 대한 제약이 없다.

### AbstractCollection을 이용하는 경우

AbstractCollection을 활용해 Collection 구현체를 작성한 경우 contains와 size 메서드를 구현해야한다. 만약 **contains와 size 메서드를 구현할 수 없다면 컬렉션보다는 Stream이나 Iterable을 반환하는 편이 낫다.**

## 전용 컬렉션 구현보다 간단한 방법

전용 컬렉션을 구현해도 되지만, 때로는 단순히 구현하기 쉬운 쪽을 선택하기도 한다. 여기 전용 컬렉션 구현보다 좀 더 단순하게 구현할 수 있는 방법이 있다. 

입력 리스트의 모든 부분 리스트를 Stream으로 반환하는 예제를 살펴보자.

```java
// 입력 리스트의 모든 부분리스트를 스트림으로 반환
public class SubList {
    // Stream.concat 메서드는 반환되는 Stream에 빈 리스트를 추가하며, flatMap은 모든 Stream을 하나의 Stream으로 만든다.
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()), 
                             prefixes(list).flatMap(SubList::suffixes));
    }

    // (a, b, c)의 prefixes는 (a), (a, b), (a, b, c) 이다.
    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                        .mapToObj(end -> list.subList(0, end));
    }

    // (a, b, c)의 suffixes는 (c), (b, c), (a, b, c) 이다.
    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size())
                        .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

2중 for 반복문을 이용한 방법 (위와 같은 역할을 한다.)

```java
for (int start = 0; start < src.size(); start++) {
    for (int end = start + 1; end <= src.size(); end++) {
        System.out.println(src.subList(start, end));
    }
}
```

위의 코드(2중 for 반복문)를 중첩 스트림으로 변환

```java
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
        .mapToObj(start -> 
                  IntStream.rangeClosed(start + 1, list.size())
                           .mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
}
```

# 핵심 정리

- 원소 시퀀스를 반환하는 메서드를 작성할 경우, Stream이나 Iterable을 모두 만족할 수 있는 Stream → Iterable과 Iterable → Stream 어댑터를 제공하자.
- 컬렉션을 반환할 수 있다면 컬렉션을 반환하자.
- 원소의 개수가 적다면 ArrayList같은 표준 컬렉션을 사용하고, 그렇지 않다면 전용 컬렉션을 구현하는 것을 고려하자.
- 전용 컬렉션은 코드가 지저분하지만 어댑터보다 빠르다.
- 컬렉션 반환이 불가능하다면 Stream과 Iterable 중 더 자연스러운 것을 반환하자.