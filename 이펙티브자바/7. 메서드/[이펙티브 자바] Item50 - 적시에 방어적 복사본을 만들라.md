# [이펙티브 자바] Item50 - 적시에 방어적 복사본을 만들라

**자바는 안전한 언어다.** C, C++같은 언어에서 흔히 보이는 메모리 충돌 오류에서 안전하고, 자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 불변식이 지켜지기 때문이다.

하지만 자바라도 다른 클랜스로부터의 침범을 다 막을 수 있는 건 아니다. 따라서 **누군가 불변식을 깨뜨리려 한다는 가정하에 방어적으로 프로그래밍을 해야한다.**

# 외부에서 내부를 수정할 수 있는 경우

보통 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다. 하지만 의도치않게 외부에서 내부를 수정하도록 허락하는 상황이 생길 수 있다.

다음과 같이 기간(Period)을 표현하는 클래스 있다. 개발자는 Period 값이 한번 정해지면 변하지 않게 할 의도로 만들었다고 가정해보자.

```java
public final class Period {
    private final Date start;
    private final Date end;
/**
* @param start 시작 시각
* @param end 종류 시각; 시작 시각보다 뒤여야 한다.
* @throws IllegalArgumentException 시작 시각이 종료시각보다 늦을때 발생한다.
* @throws NullPointerException start나 end 가 null 이면 발생한다.
*/
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                start + "가" + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }
 
    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
    ... // 나머지 코드 생략
}
```

얼핏 보면 Period 클래스는 **불변**처럼 보인다. 하지만 어렵지 않게 이 클래스의 불변식을 깨트릴 수 있다. 내부적으로 사용하고 있는 Date 객체가 가변이라는 사실을 이용하면 내부 조작이 가능해 진다.

### 생성자의 매개변수를 변경하여 인스턴스 내부를 공격

```java
// Period 인스턴스의 내부를 공격하는 첫 번째 방법
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);
```

Period 인스턴스의 값들이 한번 정해지면 변하지 않게 할 의도로 만들었지만, 너무 쉽게 내부 값이 변경되었다. 가변인 Date 때문인데, 이 예제를 제외하고도 **Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하지 않는게 좋다.** 

Java8 이후부터는 **Date 대신 불변인 Instant 혹은 LocalDateTime이나 ZonedDateTime을 사용하면 이 문제는 쉽게 해결된다.**

위 예시에서 나타나듯, 악의적으로 혹은 실수로 내부 가변객체를 조작하면 클래스를 오작동하게 만들 수 있다. 그렇다면 **우리는 이를 어떻게 방어할 수 있을까?**

# 방어적 프로그래밍 방법

## 생성자 매개변수의 방어적 복사본을 만들자

위의 예시에서 Period 인스턴스 내부를 보호하려면 생성자에서 받은 **가변 매개변수 각각을 방어적으로 복사**해야 한다. 즉, Period 내부에서는 원본이 아닌 복사본을 사용하는 것이다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    // 수정한 생성자 - 매개변수의 방어적 복사본을 만든다.
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                start + "가" + end + "보다 늦다.");
    }
 
    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
}
```

**매개변수의 유효성을 검사하기 전에 방어적 복사본을 만드는 것이** 부자연스러워 보이겠지만, **멀티 스레딩 환경이라면 반드시 이렇게 작성해야 한다.** 유효성 검사를 수행한 후 복사본을 만드는 그 찰나에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다.

또한, 방어적 복사에 Date의 clone() 메서드를 사용하지 않았다. 그 이유는 clone() 메서드는 final 클래스가 아니라서 상속이 가능한 타입이라면 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있기 때문이다. 따라서 **매개변수가 제 3자에 의해 확정될 수 있는 타입이라면 방어적 복사에 clone을 사용해서는 안된다.**

이렇게 생성자 매개변수를 변경하는 공격은 매개변수 각각의  방어적 복사본을 만들어 방어했다. **하지만 아직 Period의 내부를 공격할 방법은 남아있다.**

### Perirod 접근자(getter)를 이용해 내부를 공격

```java
// Period 인스턴스의 내부를 공격하는 두 번째 방법
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // getter 메서드를 이용해 값을 변경한다.
```

접근자(getter) 메서드를 이용하면 여전히 Period 인스턴스 내부 값을 변경할 수 있다. 이 공격을 막아내려면 **가변 필드의 방어적 복사본을 반환**하면 된다.

### **가변 필드의 방어적 복사본을 만든다.**

```java
public final class Period {
    private final Date start;
    private final Date end;

    // 수정한 생성자 - 매개변수의 방어적 복사본을 만든다.
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                start + "가" + end + "보다 늦다.");
    }
 
    // 수정한 접근자 - 가변 필드의 방어적 복사본을 만든다.
    public Date start() {
        return new Date(start.getTime());
    }
    public Date end() {
        return new Date(end.getTime());
    }
}
```

위의 예시처럼 새로운 접근자까지 갖추면 Period 자신만이 가변 필드에 접근할 수 있고 모든 필드가 객체 안에 완벽히 캡슐화된다. 즉, Period는 완벽한 불변이 되었다.

생성자와는 달리 접근자 메서드에서는 방어적 복사에 clone() 메서드를 사용해도 된다. Period가 가지고 있는 Date 객체는 java.util.Date임이 확실하기 때문이다. 하지만 **일반적으로 인스턴스 복사는 생성자나 정적 팩터리를 쓰는게 좋다.**

# 이 외에 방어적 복사 활용 방법

### 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관하는 경우

클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항상 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다.

만약 변경 될 수 있는 객체라면, 그 **객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제없이 동작할지 생각해봐야 한다.** 이를 확신할 수 없다면, **복사본을 만들어 저장**해야 한다.

예를 들면, 클라이언트에서 넘어온 객체를 인스턴스 내부의 Set이나 Map의 Key로 사용한다면, 추후 넘어온 객체가 변경되었을 때 Set 혹은 Map의 불변식이 깨질 수 있다.

### 가변인 내부 객체가 클라이언트에 반환될 때 안심할 수 없는 경우

클래스가 가변이든 불변이든, 가변인 내부 객체가 클라이언트에 반환될 때 안심할 수 없는 경우라면 원본을 노출하지 말고 방어적 복사본을 반환해야 한다.

길이가 1 이상인 배열은 무조건 가변이므로 내부에서 사용하는 배열을 반환할 때는 항상 방어적 복사를 수행해야 한다.

# 정리

앞서 다룬 내용을 바탕으로 얻을 수 있는 교훈은 **되도록이면 불변 객체를 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다**는 것이다. 방어적 복사에는 성능 저하가 따를 수 밖에 없고, 또 항상 사용할 수 있는 방법도 아니다. 

만약 복사 비용이 너무 크거나 호출자가 내부를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.