# [이펙티브 자바] Item36- 비트 필드 대신 EnumSet을 사용하라

예전에는 열거한 값들이 주로 집합으로 사용될 경우 정수 열거 패턴을 사용해왔다.

```java
// 2의 거듭제곱 값을 할당한 정수 열거 패턴
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    public void applyStyles(int styles) { ... }
}
```

다음과 같이 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이 집합을 비트 필드라 한다.

```java
// 비트 필드
text.applyStyles(STYLE_BOLD | STYLE_ITALIC); 
```

비트 필드를 사용하면 합집합이나 교집합같은 집합 연산을 효율적으로 수행할 수 있지만, **item34에서 언급한 정수 열거 상수의 단점을 그대로 지니며 추가로 다른 문제점도 존재한다.**

# 비트 필드의 단점

### 1. 정수 열거 상수를 출력할 때보다 해석하기 어렵다.

### 2. 모든 원소를 순회하기 까다롭다.

### 3. 최대 몇 비트가 필요한지 예상하고 적절한 타입을 선택해야한다.

다행히 위와 같은 단점을 보완할 방법으로 EnumSet 클래스가 있다.

# EnumSet의 특징

### 1. Set 인터페이스를 완벽히 구현하며 타입 안전하고 어떤 Set 구현체와 함께 사용할 수 있다.

### 2. 내부적으로 비트 벡터로 구현되어 있으며, 대부분의 경우 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 제공한다.

### 3. removeAll과 retainAll과 같은 대량 작업을 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

이러한 EnumSet을 이용해 위에 정수 열거 패턴 코드를 개선해보면 다음과 같다.

```java
public class Text{
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHROUGH
    }

    // 어떤 Set을 넘겨도 되지만, EnumSet이 최선이다.
    public void applyStyles(Set<Style> styles){ ... }
}
```

applyStyles 메서드가 Set<Style>으로 받는 이유는 인터페이스로 받는게 일반적으로 좋은 습관이기 때문이다. 인터페이스를 받음으로써 클라이언트가 EnumSet이 아닌 다른 Set 구현체를 넘기더라도 처리할 수 있다.

```java
// 클라이언트 코드
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

EnumSet의 유일한 단점이라면 불변 EnumSet을 만들 수 없다는 것이다. EnumSet을 불변으로 만들기 위해서는 성능에 조금 손해를 보지만 Collections.unmodifiableSet() 메서드를 사용하면 된다.