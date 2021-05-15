# [이펙티브 자바] Item37- ordinal 인덱싱 대신 EnumMap을 사용하라

배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다면 EnumMap을 활용해 해당 코드를 개선해야한다. ordinal 메서드로 인덱스를 얻는 코드는 동작은 하지만 많은 문제점을 내포하고 있기 때문이다.

# ordinal 인덱싱의 문제점

식물의 생애주기를 열거 타입으로 표현한 다음 클래스를 예로 문제점을 살펴보자.

```java
class Plant{
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

		public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString(){
        return name; 
    }
}
```

이제 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기 별로 묶어보자.

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
	plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant plant : garden) {
	plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
}

// 결과 출력 - 레이블을 직접 달아줘야한다.
for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
	System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

위의 코드는 ordinal를 배열의 인덱스로 사용하고 있다. 동작은 하지만 많은 문제점을 내포하고 있다.

### 1. 배열은 제네릭과 호환되지 않으므로 비검사 형변환을 수행해야하고 깔끔하게 컴파일 되지 않는다.

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled.png)

### 2. 배열은 인데스에 대한 의미를 모르니 출력에 레이블을 직접 달아줘야한다.

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%201.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%201.png)

레이블을 달지 않았을 경우

### 3. 정수는 열거타입과 다르게 타입 안전하지 않기 때문에 정확한 정수값을 이용함을 직접 보증해야한다.

이처럼 ordinal 인덱싱은 문제점을 가지고 있다. 이를 해결하기 위한 방법으로는 어떤게 있을까?

# 해결책

java.util의 EnumMap이 멋진 해결책이 될 수 있다. EnumMap은 Enum을 키로 사용하도록 설계된 아주 빠른 Map의 구현체이다.

위에서 문제가 되던 코드를 EmumMap을 활용해 개선해보자.

```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

for (LifeCycle lifeCycle : LifeCycle.values()) {
	plantsByLifeCycle.put(lifeCycle,new HashSet<>());
}

for (Plant plant : garden) {
	plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}

System.out.println(plantsByLifeCycle);
```

EnumMap을 활용한 코드는 이전 코드와 비교해보면 다음과 같은 장점을 가진다.

### 1. 짧고 명료하고 안전하지 않은 형변환을 쓰지 않는다.

### 2. 결과를 출력할 때 별도의 레이블을 달지 않아도 된다. (EnumMap의 toString)

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%202.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%202.png)

### 3. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

### 4. 내부 구현 방식을 안으로 숨겨, Map의 타입 안전성과 배열의 성능 모두를 얻는다.

EnumMap의 생성자는 한정적 타입 토큰 키 타입의 Class 객체를 받는데, 이를 통해 런타입 제네릭 타입 정보를 제공한다.

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%203.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%203.png)

Stream API를 사용해 Map을 관리하면 더 간결하게 코드를 표현할 수 있다.

```java
System.out.println(garden.stream()
                .collect(Collectors.groupingBy(p -> p.lifeCycle)));
```

위 코드는 가장 단순한 형태의 스트림 기반 코드이다. EnumMap이 아닌 고유 Map 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

이 문제를 해결하려면 다음과 같이 원하는 Map 구현체를 명시해 호출하면 된다.

```java
System.out.println(garden.stream()
                .collect(Collectors.groupingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(Plant.LifeCycle.class), Collectors.toSet())));
```

Map의 구현체를 명시했기 때문에 EnumMap을 사용하고 Value는 HashSet으로 구성된다.

또한, Stream을 사용하면 EnumMap만 사용했을 때와는 동작이 살짝 다르다. EnumMap만 사용했을 때는 열거 ㅌ타입 상수 별로 key를 전부 만들지만, Stream을 사용한 버전에서는 존재하는 열거 타입 상수만 key로 만든다.

EnumMap만 사용

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%204.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%204.png)

Stream 사용

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%205.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item37-%20ordinal%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20E%20e69c6c5f136b4e7393118ec41cf8f723/Untitled%205.png)

# 조금 더 복잡한 예제

이번에는 두 가지 상태를 전이와 매핑하도록 구현한 조금 더 복잡한 예제를 살펴보자.

다음은 액체에서 고체로 응고되고 액체에서 기체로 기화되는 두 가지 상태를 매핑하는 구현 예제이다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

앞서 설명한 ordinal 인덱싱의 문제점을 그대로 가지고 있다. 컴파일러는 ordinal과 배열 인덱스의 관계를 알 수 없다. 즉, Phase나 Phase.Transition 열거 타입을 수정하면서 TRANSITIONS를 수정하지 않거나 잘못 수정하면 런타임 오류가 발생한다.

마찬가지로 위의 코드를 EnumMap으로 개선할 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Phase.Transition>>
                m = Stream.of(values()).collect(Collectors.groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                Collectors.toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Phase.Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```