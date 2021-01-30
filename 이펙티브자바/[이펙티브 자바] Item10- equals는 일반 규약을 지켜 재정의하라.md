# [이펙티브 자바] Item10- equals는 일반 규약을 지켜 재정의하라

---

equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있다. 가장 쉬운 길은 아예 재정의하지 않는 것이다.

# 1. equals를 재정의하지 않아도 되는 경우

### 1.1 각각의 객체가 본질적으로 고유할 때

값 표현 객체가 아닌 동작하는 개체를 표현하는 클래스일 때를 말한다. 대표적인 예로 Thread, Controller, Service 등이 이 조건에 부합한다.

### 1.2 인스턴스의 논리적 동치성을 검사할 일이 없을 때

Pattern의 인스턴스가 같은 정규 표현식을 나타내는 지 검사하거나 Random 클래스의 equals 메서드가 큰 의미를 가지지 못하는 것처럼 클라이언트가 이 방식이 필요없다고 판단되면 equals를 재정의 하지 않아도 된다.

### 1.3 상위 클래스에서 재정의한 equlas가 하위 클래스에도 들어맞을 때

하위 클래스에서도 사용하기에 적합한 equlas라면 재정의할 필요 없이 상속받아 사용하면 된다.

### 1.4 클래스가 private 또는 package-private(default)이고 equals 메서드를 호출할 일이 없을 때

equals가 호출될 일이 없어 실수로라도 호출되는 걸 막고 싶다면 다음과 같이 구현하면 된다.

```java
@Override
public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

# 2. equals를 재정의해야 하는 경우

**객체의 식별성**이 아닌 **논리적 동치성**을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때 해야한다. 주로 String이나 Integer와 같은 값 클래스가 이에 해당한다. 

값 클래스의 equals를 재정의 할 때 논리적 동치성을 확인하도록 재정의해두면, 값을 비교하는 것 뿐만아니라 Map의 키나 Set의 원소로 사용할 수 있게 된다.

```java
// 값을 비교
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

### 값 클래스여도 equals 재정의가 필요 없는 경우

값이 같은 인스턴스가 둘 이상 만들어지지 않는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다. 

대표적인 예로 Enum이 있다. 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 **논리적 동치성**과 **객체 식별성**이 똑같은 의미라고 볼 수 있다.

# 3. equals 메서드는 동치 관계를 구현한다.

### 3.1 반사성(reflexivity)

null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true이다. 즉, 모든 객체는 자기 자신과 같아야 한다는 뜻이다.

### 3.2 대칭성(symmetry)

null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.

두 객체에게 서로 같은지 물었을 때 같은 답이 나와야 한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8Fb3t%2FbtqSa9kavdt%2FMke0oRcWimGL8fthz7WP51%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fqkstl%2FbtqSgMPuSwY%2F8tuJPiRPEMxfDsEha3pKHK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FduG9P7%2FbtqSduhkeg1%2FtDY0RRypEF9Ca8cyK2Mj4K%2Fimg.png)

CaseInsensitiveString은 String을 알지만 String은 CaseInsensitiveString이 뭔지 모르기 때문에 (한 방향으로만 정상 동작) 대칭성을 위반하게 된다. 이러한 문제를 방지하려면 CaseInsensitiveString의 equals 메서드가 String 객체와 상호작용하지 않도록 해야한다. 따라서 아래와 같이 구현하면 대칭성을 만족하게 된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwYbmv%2FbtqR1DGDRrd%2FEt4mNlTraKmvwHw4Pv7kJK%2Fimg.png)

### 3.3 추이성(transitivity)

null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true다.

1과 2가 같고 2와 3이 같으면 1과 3도 같아야 한다는 삼단논법이다. 

상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 상황을 생각해보자. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbt1gKe%2FbtqSa9qVuyM%2FEO1kVSxpgulDX52pvBDc2K%2Fimg.png)

좌표를 가지는 Point 클래스가 있다. 이를 상속받아 색상 정보를 추가하는 하위 클래스를 추가해보자.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqWVKO%2FbtqRXInUX0H%2FE3B57RYLWqlzYtdG8QwrF0%2Fimg.png)

이 클래스의 equals 구현을 어떻게 해야할까? 상위 클래스의 equals를 그대로 사용한다면 추가된 color 필드는 비교하지 못한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FUslIC%2FbtqRXIBuVy7%2FEpY6KHZHVRKwKReALT1dq1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FI0xPI%2FbtqSjBGVkEu%2FOAkZkS4d4hnJvFxKrS0adK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbKRQ5i%2FbtqSgLC3IRU%2FJdKsgMAIGlyHDX6yQTadCk%2Fimg.png)

위와 같이 구현하면 대칭성에 위배되는 것이 명확하게 확인된다. 그렇다면 ColorPoint의 equals가 Point와 비교할 때는 색상을 무시하도록하면 해결이 될까?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FB4ft8%2FbtqR3RR9n6X%2FwpJr0zfWoGa2XhfoInWG7k%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFzitQ%2FbtqSdtvUmba%2Fdt0xRVTxbpj9Fi2hUjQgG0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FybJzv%2FbtqSdtbCaU7%2FSAFoOmtUR7yUoytfQiD9BK%2Fimg.png)

대칭성은 보존되지만, 추이성에 위배되는 것을 확인할 수 있다. p1과 p3 비교에서는 색상까지 고려했기 때문이다. 또한 이 방식은 무한 재귀에 빠질 위험성도 있다. 

그렇다면 이러한 문제의 해결책은 무엇일까? 사실 이 문제는 객체 지향 언어에서 동치관계를 구현할 때 발생하는 근본적인 문제다. **객체 생성가능 클래스를 상속하여 새로운 값을 추가하면서 equals 규약을 만족할 방법은 없다.**

그렇다고 instanceof 대신 getClass를 하라는 것은 아니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeokeoN%2FbtqRVqOG1kv%2FJQVfEPKon5wf7GaU9Kurl1%2Fimg.png)

이 방법은 괜찮아 보이지만 실제로 활용할 수 없다. 아래 예제를 보면 올바르지 않다는 것을 알 수 있다.

```java
//단위 원 상의 모든 점을 포함하도록 unitCircle 초기화 
private static final Set<Point> uniCircle;
static{
    unitCircle = new HashSet<Point>();
    unitCircle.add(new Point(1, 0));
    unitCircle.add(new Point(0, 1));
    unitCircle.add(new Point(-1, 0));
    unitCircle.add(new Point(0, -1));
}

public static boolean onUnitCircle(Point p){
    return unitCircle.contatins(p); 
}
```

```java
public class CounterPoint extends Point{
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y){
        super(x, y);
        counter.incrementAndGet();
    }   

    public int numberCreated() { return counter.get(); } 
}
```

**리스코프 대체 원칙**은 어떤 자료형의 중요한 속성은 하위 자료형에도 그대로 유지되어서, 그 자료형을 위한 메서드는 하위 자료형에도 잘 동작해야 한다는 원칙이다. 이에 따르면 Point의 하위 클래스는 정의상 여전히 Point이기 때문에 어디서든 Point로 활용되어야 한다. 

그런데 CounterPoint 객체를 onUnitCircle 메서드의 인자로 넘기는 경우를 생각해보자. Point 클래스의 equals 메서드가 getClass를 사용하고 있다면, onUnitCircle 메서드는 CounterPoint 객체의 x나 y값에 상관없이 무조건 false를 반환할 것이다. 

이는 onUnitCircle 메서드가 이용하는 HashSet 같은 컬렉션이 객체 포함여부를 판단할 때 equals를 사용하기 때문이며, CounterPoint객체는 어떤 Point객체와도 같을 수 없기 때문이다.

## 우회방법

### 상속 대신 컴포지션을 사용하라

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbeabQg%2FbtqRVpWsete%2FlpNTid9HfjcJzEsh9FTkEk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzaxIy%2FbtqRVo4iBa0%2FuwOEhsbANPs3zzaB6MG0Nk%2Fimg.png)

Point를 상속받는 구조 대신 Point를 private 필드로 두고, Point를 반환하는 뷰 메서드를 public으로 추가하는 방식이다.

### 추상 클래스의 하위 클래스 사용

추상 클래스는 객체를 생성할 수 없으므로 하위 클래스끼리 비교가 가능해진다.

### 3.4 일관성(consistency)

null이 아닌 모든 참조 값 x, y에 대해 xequals(y)를 반복해서 호출하면 항상 같은 값을 반환해야 한다.

가변이든 불변이든 신뢰할 수 없는 자원들을 비교하는 equals 구현을 삼가해야한다. 이 제약을 어기면 일관성 조건을 만족시키기가 굉장히 어려워진다. 예를들면 java.net.URL의 equals 메서드는 URL에 대응되는 호스트의 IP 주소를 비교하여 equals의 반환값을 결정한다. 문제는 호스트 이름을 IP 주소로 변환하려면 네트워크에 접속해야 하는데, 그 결과가 언제나 같다고 보장할 수 없다.

### 3.5 null-아님

null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.

**묵시적 null 검사**

instanceof 연산자는 첫 번째 피연산자가 null이면 무조건 false를 반환하므로 null를 따로 할 필요가 없다.

```java
// 묵시적 null 검사
@Override public boolean equals(Obejct o){
// instanceof 자체가 타입과 무관하게 null이면 false 반환함.
  if(!(o instanceof MyType))
    return false;
  MyType mt = (MyType) o;
}
```

**명시적 null 검사**

```java
// 명시적 null 검사 - 필요 없음
@Override public boolean equals(Object o){
  if( o == null){
    return false;
  }
}
```

# 4. 양질의 equals 메서드 구현 방법

### 4.1 == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.

### 4.2 instanceof 연산자로 입력이 올바른 타입인지 확인한다.

### 4.3 입력을 올바른 타입으로 형변환한다.

### 4.4 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

# 5. equals 구현 시 주의사항

### 5.1 타입 별 비교 방법

**float, doble을 제외한 기본타입** : == 연산자로 비교하기

**참조타입**: equals 메서드로 비교

**float, double** : Float.compare(float, float), Double.compare(double, double)로 비교 (부동 소수 값) 이 필드에 equals를 쓸 수 있지만, 오토 박싱이 일어날 수 있어 성능상 좋지 않다.

**배열:**  앞선 지침들을 활용한다. 모두가 핵심 필드라면 Arrays.equals()를 사용한다.

### 5.2 null 정상 값으로 취급하는 참조 타입 필드일 경우

Object.equals(Object, Object)로 비교해 NPE 발생을 방지한다.

### 5.3 필드의 표준형을 저장

비교하기 복잡한 필드는 필드의 표준형을 저장한 후 비교한다. **불변 클래스에 제격이다.**

### 5.4 비용이 싼 필드를 먼저 비교하라

필드 비교순서에 따라 equals의 성능이 좌우되기도 한다. 

다를 가능성이 크거나 비교 비용이 싼 필드를 우선적으로 비교하자.

### 5.5 equals를 재정의할 때는 hashCode도 반드시 재정의하자

### 5.6 Object 외의 타입을 매개변수로 받는 equals는 선언하지 말자
