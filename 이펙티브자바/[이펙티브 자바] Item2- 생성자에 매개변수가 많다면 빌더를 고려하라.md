# [이펙티브 자바] Item2- 생성자에 매개변수가 많다면 빌더를 고려하라

---

정적 팩터리 메서드와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다. 이러한 제약에 대안으로 프로그래머들은 다음과 같은 방법을 사용했다.

1. 점층적 생성자 패턴
2. 자바빈즈 패턴
3. 빌더 패턴

# 1. 점층적 생성자 패턴

필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개 받는 생성자 ....... 선택 매개변수를 전부 다 받는 생성자까지 점층적으로 늘려가는 방식이다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }

}
```

매개변수만 다른 생성자가 점층적으로 만들어져있다.

이 클래스의 인스턴스를 생성할때는 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출 하면된다.

하지만 **점층적 생성자 패턴을 권장하지는 않는다.** 

1. 매개변수의 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.
2. 실수로 매개변수의 순서를 바꿔 생성했을 경우에 런타임에 엉뚱한 동작을 수행할 수 있고 찾기 어려운 버그를 유발할 수 있다.
3. 확장이 어렵다.

# 2. 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, setter로 원하는 매개변수의 값을 설정하는 방식이다.

```java
public class NutritionFacts {    
		private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

점층적 생성자 패턴에 비해 인스턴스를 만들기 쉽고, 더 읽기 쉬운 코드가 되었지만 자바 빈즈 패턴은 심각한 단점을 가지고있다.

1. 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
2. 일관성이 무너지면서 클래스를 불변으로 만들 수 없게 된다.
3. 하나의 객체를 만들기 위해 여러개의 메서드가 호출된다.

**일관성**이 무너지면 런타임시 디버깅에 어려움을 겪을 수 있고, 불변 클래스로 만들 수 없게 되면서 Thread safe하지 않게된다.

# 3. 빌더 패턴

위 두가지의 방법에 대한 대안이 될 수 있다. **점층적 생성자 패턴**의 안정성과 **자바 빈즈 패턴**의 가독성을 겸비한 것이 빌더 패턴이다.

### 동작설명

1. 필수 매개변수만으로 생성자 혹은 정적 팩터리 메서드를 호출해 빌더 객체를 얻는다.
2. 빌더 객체가 제공하는 setter 메서드들로 원하는 매개변수들을 설정한다.
3. 매개변수가 없는 build 메서드를 호출해 필요한 객체(일반적으로 불변)를 얻는다.

위의 동작을 코드로 살펴보면 다음과 같다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
				    { calories = val;      return this; }
        public Builder fat(int val)
				    { fat = val;           return this; }
        public Builder sodium(int val)
		        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
		        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

위 클래스는 불변이며, 각 setter 메서드는 자신을 반환하기 때문에 연쇄적으로 호출할 수있다. 이를 플루언트 API(fluent API) 혹은 메서드 연쇄(method chaining)라 한다.

이를 사용하는 클라이언트 코드의 형태를 살펴보자.

```java
public static void main(String[] args) {

        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();

    }
```

필수 매개변수를 넣고, 원하는 매개변수를 선택적으로 넣어준 후 bulid()를 통해 객체를 얻고있다. 이 코드는 읽기 쉽게 이루어져 명확한 정보 전달을 하고 있다.

다만, 잘못된 매개변수에 대한 검증은 필요하다. 생성자와 메서드에서 매개변수에 대한 validation check를 하고 build 메서드가 호출하는 생성자에서 여러 매개변수의 **불변식**을 검사하자.

> 불변식 - 런타임시 반드시 만족해야하는 조건

ex)
List의 크기는 반드시 0 이상이어야한다. 
기간을 표현하는 클래스에서 start 필드는 end 필드의 값보다 항상 앞서야한다.

## 빌더 패턴의 활용방법

### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

각 계층의 클래스에 관련 빌더를 멤버로 정의하여 사용한다.

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

위의 코드는 추상 클래스로 선언된 Pizza 클래스이다. 

Pizza의 Builder 클래스는 **재귀적 타입 한정**을 이용하는 제네릭 타입이다. `Builder<T extends Builder<T>>`의 의미는 "Pizza.Builder의 타입 T는 자신을 상속받은 모든 Builder가 될 수 있다." 라는 뜻이다. 즉, T로 표현되는 **타입**이 Builder를 사용 가능해야하므로, Builder를 구현한 클래스만 타입으로 사용해라! 라는 의미이다. 이처럼 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하고 제약을 두는 것을 **재귀적 타입 한정**이라 한다.

또 추상 메서드 `self()` 메서드를 통해 형변환 없이 메서드 연쇄를 지원한다. self 타입이 없는 Java를 위한 이 우회 방법을 **시뮬레이트한 셀프 타입 관용구**라고 한다.

### 공변 반환 타이핑

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
```

Pizza를 상속받는 하위 클래스 NyPizza이다.

이 클래스에서 Builder가 정의한 build 메서드는 `new NyPizza(this)` 를 반환하고 있다.  즉, 하위 클래스를 반환하는 것이다.

NyPizza의 메서드(하위 클래스)가 Pizza의 메서드(상위 클래스)가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 **공변 반환 타이핑**이라한다. 이 기능을 통해 클라이언트는 형변환을 신경쓰지 않고 빌더를 사용할 수 있게된다.

## 빌더 패턴을 사용할 때 고려해야할 점

- 어떤 객체를 만들 때, 그 객체를 만들기 앞서 빌더부터 만들어야 하기 때문에 성능이 민감한 상황에서는 문제가 될 수 있다.
- 점층적 생성자 패턴과 비교했을 때 코드가 장황해서 매개변수가 4개 이상일 때 값어치를 한다.