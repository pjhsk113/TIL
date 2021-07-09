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

또한, 생성자는 재정의할 수 없으니 다중정의와 재정의가 혼용될 걱정도 없다. 그래도 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우는 피할 수 없으니 그에 따른 **안전 대책**을 배워야 한다.

### 안전 대책!

매개변수 수가 같은 다중정의 메서드가 많더라도 **근본적으로 다르다면 헷갈릴 일이 없다**. 근본적으로 다르다는 것은 두 타입의 값을 어느쪽으로든 형변환할 수 없다는 뜻이다.

이 조건만 만족한다면 어떤 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다. 더 이상 컴파일타임 타입에 영향을 받지 않게 되는 것이다.

# 다중정의시 주의를 기울여야 하는 이유

다음과 같은 코드가 있다. 이 프로그램의 결과 값은 어떻게 출력될까?

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
        }

        for (int i = 0; i < 3; i++) {
            set.remove(i);
        }
        System.out.println(set);
    }

}
```

[-3, -2, -1, 0, 1, 2] 의 값에서 [0, 1, 2]를 지우니까 [-3, -2, -1]이 출력되어야 할 것 같다.

![](https://blog.kakaocdn.net/dn/9qIOX/btq9cIkCc6I/fRI3xsm3Nx0UhwPACNSEM1/img.png)

실제로 [-3, -2, -1]이 출력되면서 테스트가 통과한다. Set의 remove() 메서드의 시그니처는 remove(Object)기 때문에 정상적으로 0 이상의 값을 지운다.

그렇다면 List는 어떨까?

```java
public class SetList {
    public static void main(String[] args) {
       List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            list.remove(i);
        }
        System.out.println(list);
    }

}
```

이 코드도 마찬가지로 [-3, -2, -1]이 출력되어야 할 것 같다.

![](https://blog.kakaocdn.net/dn/DqJDI/btq9f4M3vSb/4P97SZQes3xGARQAAbs2xK/img.png)

하지만 전혀 다른 결과가 출력된다. 왜 [-2, 0, 2]가 출력될까? 

그 이유는 List의 remove가 다중정의되있기 때문이다. 

![](https://blog.kakaocdn.net/dn/0rKco/btq9fluYLoh/ItlEJfrRuBDl0InKKQlYD1/img.png)

![](https://blog.kakaocdn.net/dn/cdXXel/btq9gzsu2FJ/H1WvIdlykk1jnSBaSsUd91/img.png)

위의 코드에서는 remove(Object)가 아닌 remove(int index) 메서드가 선택된다. 따라서 값이 아닌 index의 원소를 제거하기 때문에 [-2, 0, 2]라는 값이 출력되는 것이다.

Java4까지는 Object와 int가 근본적으로 달라 문제가 없었지만, Java5에 오토박싱이 도입되면서 이 개념이 흐트러졌다. 즉, 이제는 int와 Integer가 근본적으로 다르지 않다는 것이다. 이 문제는 remove를 호출할 때 매개변수를 Integer로 형변환 해주면 해결된다.

위의 예시에서 중요한 점은 제네릭과 오토박싱(신규 기능)이 추가되면서 기존의 List 인터페이스가 취약해졌다는 것이다. (다중정의에 의해)

이 예시만으로도 다중정의를 왜 신중하게 사용해야 하는지에 대한 충분한 근거가 된다.

# 핵심 정리

- 일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다.
- 만약 다중정의를 피할 수 없는 상황이라면 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 하자.