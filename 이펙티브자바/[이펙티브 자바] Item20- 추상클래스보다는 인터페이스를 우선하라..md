# [이펙티브 자바] Item20- 추상클래스보다는 인터페이스를 우선시하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스가 있다. 자바 8부터는 인터페이스도 디폴트 메서드를 제공하게되어 인터페이스와 추상 클래스 모두 인스턴스 메서드를 구현 형태로 제공할 수 있게 되었다. 그렇다면 이 둘의 차이점이 무엇일까?

인터페이스와 추상 클래스의 가장 큰 차이는 추상 클래스를 상속받아 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. **단일 상속만 가능한 자바에서 추상 클래스의 하위 클래스는 다른 클래스를 확장할 수 없기 때문에** 새로운 타입을 정의하는데 커다란 제약을 가지게 되는 셈이다. 

반면에 인터페이스는 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

# 인터페이스의 장점

### 1. 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.

인터페이스가 요구하는 메서드만 추가하고 implements 구문만 추가하면 기존 클래스에도 손쉽게 새로운 인터페이스를 구현할 수 있다. 

반면에 기존 클래스에 새로운 추상 클래스를 끼워 넣는 것에는 어려움이 따른다. 두 클래스가 같은 추상 클래스를 상속하려면, 그 추상 크래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 만약 두 클래스에 관계가 없다면 이는 클래스 계층 구조에 혼란을 줄 수 있다.

### 2. 믹스인(mixin) 정의에 안성맞춤이다.

믹스인이란 클래스가 구현할 수 있는 타입으로, 클래스의 주된 기능 외에도 선택적 기능(추가적인 기능)을 혼합할 수 있는 것을 말한다. 따라서 믹스인 인터페이스는 어떤 클래스의 주 기능외에도 다른 일을할 수 있다는 것을 선언하고, 제공해주는 효과를 준다.

추상 클래스는 단일 상속만 지원하기 때문에 한 클래스가 두 부모를 가질 수 없고, 클래스 계층에서 믹스인이 들어 갈 수 있는 합리적인 위치가 없다.

믹스인 인터페이스는 대표적으로 Serializable, Cloneable, Comparable 등이 있다.

### 3. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

현실의 개념중에는 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있는 개념이 있다.

- 자동차
    - 버스
    - 화물차
    - 승용차
    - 스포츠카

하지만 이를 구조적으로 표현하기 어려운 개념들도 존재한다. 가수와 작곡가 그리고 싱어송 라이터를 예로 들수 있다. 이렇게 계층구조가 없는 개념들은 인터페이스로 만들기 편하다. 

가수와 작곡가의 인터페이스가 있다고 가정해보자.

```java
public interface Singer {
	AudioClip sing(Song s);
}

public interface SongWriter{
	Song compose(int chartPosition);
}
```

우리 주변에는 가수 겸 작곡가 싱어송라이터도 존재한다. 이를 클래스가 두 인터페이스를 구현해도 전혀 문제가 되지 않는다. 

```java
public class KimKwangSeok implements Singer, SongWirter {
	@Override
  public void Sing(String s) {

  }
  
	@Override
	public void Compose(int chartPosition) {

  }
}
```

또한, 가수 겸 작곡가라는 새로운 인터페이스를 만들어 낼 수도 있다.

```java
public interface SingerSongWriter extends Singer, SongWriter{
	AudioClip strum();
	void actSensitive();
}
```

위와 같은 구조를 추상 클래스로 만들고자 한다면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어진다. 속성이 n개라면 지원해야할 조합의 수는 2^n개가 된다. 흔히 조합 폭발이라 부르는 현상이 나타날 수 있다.

### 4. 래퍼 클래스 관용구와 함께 사용하면 기능을 향상시키는 안전하고 강력한 수단이 된다.

타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법을 상속뿐이다. 상속은 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.

자바 8부터는 디폴트 메서드를 제공한다. 따라서 구현 방법이 명확한 것이 있다면 이를 디폴트 메서드로 제공해 개발자들의 수고를 덜어줄 수 있다. 

하지만 디폴트 메서드에도 제약은 존재한다. equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공해서는 안된다. 또한 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다. 마지막으로, 본인이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

한편, 추상 골격 구현 클래스를 함께 제공하면 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다. 인터페이스로 타입을 정의하고 필요한 일부를 디폴트 메서드로 구현한다. 추상 골격 클래스에서는 나머지 메서드들 까지 구현한다. 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료된다. 바로 템플릿 메서드 패턴이다. 이러한 추상 골격 클래스의 좋은 예는 자바 컬렉션 프레임워크에서 AbstractList, AbstractSet, AbstractMap 등이 있다. 

다음은 구체적인 추상 골격 클래스의 예시이다. 커피라는 인터페이스가 있고 이를 구현하는 아이스 아메리카노, 아이스 라떼라는 클래스가 있다.

```java
public interface Coffee {
    void boilWater();
    void putEspresso();
    void putIce();
    void putExtra();
		void makeCoffee();
}
```

```java
public class IceAmericano implements Coffee{
    @Override
    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    @Override
    public void putEspresso() {
        System.out.println("에스프레소를 넣는다.");
    }

    @Override
    public void putIce() {
        System.out.println("얼음을 넣는다.");
    }

    @Override
    public void putExtra() {
        System.out.println("시럽을 넣는다.");
    }
}

public class IceLatte implements Coffee{
    @Override
    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    @Override
    public void putEspresso() {
        System.out.println("에스프레소를 넣는다.");
    }

    @Override
    public void putIce() {
        System.out.println("얼음을 넣는다.");
    }

    @Override
    public void putExtra() {
        System.out.println("우유를 넣는다.");
    }
}
```

아메리카노와 라떼는 마지막에 무엇을 넣느냐에 따라 결정된다. 따라서 추상 골격 클래스를 활용해 위의 코드의 중복을 제거해보자.

```java
public abstract class AbstractCoffee implements Coffee{
    @Override
    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    @Override
    public void putEspresso() {
        System.out.println("에스프레소를 넣는다.");
    }

    @Override
    public void putIce() {
        System.out.println("얼음을 넣는다.");
    }
}
```

```java
public class IceAmericano extends AbstractCoffee implements Coffee {

    @Override
    public void putExtra() {
        System.out.println("시럽을 넣는다.");
    }
}

public class IceLatte extends AbstractCoffee implements Coffee {

    @Override
    public void putExtra() {
        System.out.println("우유를 넣는다.");
    }
}
```

추상 골격 클래스를 통해 중복을 제거하고 읽기 좋은 코드가 되었다. 하지만 이때 카푸치노 클래스를 추가하고 카푸치노 클래스는 MlikCream이라는 클래스를 상속받는다고 가정해보자. 자바는 단일 상속만을 지원하기 때문에 AbstractCoffee 클래스(추상 골격 클래스)를 상속받지 못하게된다. 

이럴때는 private 내부 클래스를 정의하고 내부 클래스가 추상 골격 클래스를 상속하도록 하면된다. 내부 클래스의 메서드를 인스턴스에서 호출하여 우회적으로 추상 골격 클래스를 사용할 수 있게된다. 이를 시뮬레이트한 다중 상속이라 한다.

```java
public class MilkCream {
    public void putCream() {
        System.out.println("우유 크림을 넣는다.");
    }
}
```

```java
public class IceCappuccino extends MilkCream implements Coffee{
    InnerAbstractCoffee innerAbstractCoffee = new InnerAbstractCoffee();

    @Override
    public void boilWater() {
        innerAbstractCoffee.boilWater();
    }

    @Override
    public void putEspresso() {
        innerAbstractCoffee.putEspresso();
    }

    @Override
    public void putIce() {
        innerAbstractCoffee.putIce();
    }

    @Override
    public void putExtra() {
        innerAbstractCoffee.putExtra();
        putCream();
    }

    @Override
    public void makeCoffee() {
        boilWater();
        putEspresso();
        putIce();
        putExtra();
    }

	// private 내부 클래스 - 추상 골격 클래스를 상속한다.
    private class InnerAbstractCoffee extends AbstractCoffee {

        @Override
        public void putExtra() {
            System.out.println("우유를 넣는다.");
        }
    }
}
```

---

### 단순 구현(simple implementation)

단순 구현이란 골격 구현의 작은 변종으로 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다. 쉽게 말해 동작하는 가장 단순한 구현이다.

대표적인 예로는 AbstractMap.SimpleEntry가 있다.

# 핵심 정리

- 일반적으로 다중 구현용 타입으로는 인터페이스가 적합하다.
- 복잡한 인터페이스라면 골격 구현을 함께 제공하는 방법을 고려하자.
- 골격 구현은 가능한 한 인터페이스의 디폴트 메서드로 제공하여 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.