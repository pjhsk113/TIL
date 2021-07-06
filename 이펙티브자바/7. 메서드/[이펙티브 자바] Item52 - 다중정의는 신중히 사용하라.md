# [이펙티브 자바] Item52 - 다중정의는 신중히 사용하라

이름이 같은 메서드가 매개변수의 타입이나 개수만 다르게 갖는 형태를 **다중 정의(overloading)**라고 한다. 이 다중정의를 사용할 때는 신중해야 한다.

## 다중정의(Overloading)

다음과 같이 컬렉션을 구분하기 위한 프로그램이 있다고 생각해보자.

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

잘 동작할 것 같은 코드이다. 하지만 이 프로그램은 **"그 외"**만 3번 출력한다. 다중정의된 classify() 메서드는 컴파일 타임에 어떤 메서드가 호출될 것인지 정해지기 때문이다.

for문 안에 `Collection<?> c`는 런타임에 타입이 결정되고 달라진다. 즉, 컴파일 타임에는 항상 `Collection<?>`타입이라는 것이다.

컴파일 타임에 어떤 classify() 메서드가 호출될 것인지 결정되기 때문에 `classify(Collection<?>)` 메서드만 3번 호출되는 것이다.

이처럼 직관과 어긋나게 동작하는 이유는 **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다.**

이 예제의 의도는 매개변수의 런타임 타입에 기초해 적절한 다중정의 메서드로 자동 분배하는 것이었다. 하지만 의도대로 동작하지 않는다.

이 문제를 해결하려면 메서드를 합친 후 instanceof로 명시적으로 검사를 수행해주면 된다.

```java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "집합" :
            c instanceof List ? "리스트" : "그 외";
}
```

## 재정의(Overriding)

재정의는 상위 클래스의 메서드를 하위 클래스에서 재정의하는 것을 뜻한다. 메서드를 재정의 했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다.

재정의한 메서드는 런타임에 어떤 메서드를 호출할지 정해진다.

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```

위 코드를 실행하면 "포도주", "발포성 포도주", "샴페인"을 차례대로 출력한다. for문에서의 컴파일 타임 타입이 모두 Wine인 것에 무관하게 항상 **가장 하위에서 정의한 재정의 메서드**가 실행되는 것이다. 다중정의 예제에서 의도했던 것처럼 직관과 일치한다.

# 다중정의가 혼동을 일으키는 상황을 피하자

- API 사용자가 매개변수를 넘길 때, 어떤 다중정의 메서드가 호출될지 모른다면 프로그램은 오작동하기 쉽다.
- 헷갈릴 수 있는 코드는 작성하지 말자. (위의 다중정의 예시처럼)
- 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.
- 가변인수를 매개변수로 사용한다면 다중정의는 사용하면 안된다.

이 규칙들만 잘 따르면 다중정의가 혼동을 일으키는 일을 피할 수 있다. 이 외에 다중정의하는 대신 메서드 이름을 다르게 지어주는 방법도 존재한다.

# 생성자 다중정의

생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 된다. 이러한 상황에 **정적 팩터리 메서드**가 적절한 대안이 될 수 있다.

또한, 생성자는 재정의할 수 없으니 다중정의와 재정의가 혼용될 걱정도 없다. 그래도 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우는 피할 수 없으니 그에 따른 안전 대책을 배워야 한다.

### 안전 대책!

매개변수 수가 같은 다중정의 메서드가 많더라도 **근본적으로 다르다면 헷갈릴 일이 없다**. 근본적으로 다르다는 것은 두 타입의 값을 어느쪽으로든 형변환할 수 없다는 뜻이다.

이 조건만 만족한다면 어떤 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다. 더 이상 컴파일타임 타입에 영향을 받지 않게 되는 것이다.