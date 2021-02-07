# [이펙티브 자바] Item17- 변경 가능성을최소화하라

---

변경 가능한 클래스로 만들 타당한 이유가 없다면, 가능한 한 **변경 불가능한 불변 클래스**로 만드는 것이 좋다.

불변 클래스란 객체가 생성된 시점부터 파괴되는 시점까지 그 내부 값을 수정(변경)할 수 없는 클래스를 말한다. 대표적인 예로는 String, Wrapper Class, BigInteger, BigDecimal 등이 있다. 불변 클래스는 설계, 구현, 사용이 쉬우며, 오류가 생길 여지가 적고 안전하다.

## 불변 클래스 생성 5가지 규칙

### 1. 객체 생태를 변경하는 메서드를 제공하지 않는다. (Setter)

### 2. 클래스를 확장할 수 없도록 한다.

하위 클래스에 의해 객체의 상태를 변하게 하는 사태를 막아준다.

### 3. 모든 필드를 final로 선언한다.

설계자의 의도를 명확히 드러내는 방법이다.

### 4. 모든 필드를 private로 선언한다.

필드가 참조하는 가변 객체를 직접 접근해 수정하는 일을 막아준다. 기본 타입 필드나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 내부 표현을 바꾸지 못한다는 단점이 있으므로 권하지 않는다.

### 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

가변 객체를 참조하는 필드가 있다면, 그 객체의 참조를 얻을 수 없도록 해야한다. getter가 해당 필드를 그대로 반환하면 안되며, 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행해야한다.

```java
public final class Complex{
    private final double re;
    private final double im;

    public Complex(double re, double im){
        this.re = re;
        this.im = im;
    }

		// 새로운 인스턴스 생성후 반환
    public Complex add(Complex c){
        return new Complex(re + c.re, im + c.im);
    }

    public Complex subtract(Complex c){
        return new Complex(re - c.re, im - c.im);
    }

    ...
}
```

위의 예시에서는 사칙연산 메서드가 자기 자신을 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다. 대부분의 불변 클래스가 이러한 함수형 프로그래밍 패턴을 따르고 있다. 이러한 방식으로 프로그래밍하면 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.

> 함수형 프로그래밍
피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴

## 불변 객체의 장점

### 1. 불변 객체는 단순하다.

생성될 때 부여한 한 가지 상태를 파괴될 때 까지 그대로 간직하기 때문이다. 따라서 프로그래머는 불변 객체를 믿고 사용할 수 있다.

### 2. 스레드 안전하다.

불변 객체는 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다. 근본적으로 스레드 안전하기 때문에 따로 동기화 할 필요가 없다.

### 3. 객체를 안심하고 공유할 수 있다.

스레드간의 영향을 받지 않기 때문에 한번 생성된 자원은 재활용할 수 있다. 따라서 자주 사용되는 인스턴스를 캐싱하는 정적 팩터리 메서드를 제공하자. 메모리 사용량과 가비지 컬렉션 비용이 감소하는 효과를 볼 수 있다.

자유롭게 객체 공유를 할 수 있다는 점은 방어적 복사도 필요없다는 것을 의미한다. 복사해도 원본과 같으니 복사의 의미가 사라진다.  → clone 메서드나 복사 생성자 메서드가 필요없다.

## 불변 객체의 단점

### 1. 값이 다른 경우 반드시 독립적인 객체로 만들어야한다.

```java
String s1 = "a";
String s2 = "b";
String s3 = s1 + s2; // 새로운 객체 생성
```

값이 단순하다면 문제 없겠지만, 값의 가짓수가 많다면 이를 모두 만드는데 큰 비용을 치뤄야한다. 

예를 들면, 백만 비트짜리 BigInteger의 비트 하나를 바꾼다고 가정해보자. 하나의 비트만 다른 백만 비트짜리 인스턴스가 생성된다. 이는 크기에 비례해 시간과 공간을 잡아먹는다.  → O(N)

이처럼 원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 발생할 수 있다. 이 문제를 해결하기위해 **가변 동반 클래스**를 제공한다.

**다단계 연산이 예측이 될 때**

가변 동반클래스를 package-private 클래스로 만든다.

ex) BigInteger의 BitSieve, MutablebigInteger, SignedMutableBigInteger

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNt8is%2FbtqVV2XqDin%2FkG2QKFxnLdOhkX0g9uykXk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fo3MPM%2FbtqWc9NG6Nm%2FsIoLW5d1i9KzxjcC8dTKCK%2Fimg.png)

**다단계 연산이 예측 안될 때**

가변 동반클래스를 public으로 둔다.

ex) String의 StringBuffer, StringBuilder

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzBa33%2FbtqV356cRTL%2FUWPpK9UpupNmgOkkSTQ5sk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlM77O%2FbtqV0EPjRsc%2FQEJzoRmwFIK5FixfpWyVd0%2Fimg.png)

## 불변 클래스 설계 방법

클래스가 불변임을 보장하기 위해서는 자신을 상속하지 못하게 해야한다.

### 1. final 클래스

가장 쉽게 상속을 금지하는 방법

### 2. 정적 팩터리 메서드

final 클래스 보다 조금 더 유연한 방법이다.

package-private 혹은 private 생성자를 만들고 public 정적 팩터리 메서드를 제공한다.

public이나 protected 생성자가 없기 때문에 다른 클래스에서는 이 클래스를 확장 할 수 없게 된다.

## 메서드를 재정의 가능하게 설계된 불변 객체 사용시 주의점

값이 불변이어야 클래스의 보안을 지킬 수 있다면 주의해야한다. 인수로 전달받는 객체가 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면 이 인수를 가변이라고 생각하고 **방어적 복사**를 사용해야한다.

## 불변 객체의 기준 완화

"모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없어야한다." 는 너무 과한감이 있다.

따라서 "어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없어야 한다."로 완화할 수 있다. 계산 비용이 큰 값을 나중에 계산하여 final이 아닌 필드에 캐싱해둔다. 다음에 같은 값이 요청되면 캐싱해둔 필드를 반환하여 계산 비용을 줄일 수 있다.

### 주의 사항

추가적으로 한가지 더 주의할 것은 직렬화에 관계된 부분이다. 불변 클래스가 Serializable 인터페이스를 구현하도록 했고, 해당 클래스에 변경 가능 객체를 참조하는 필드가 있다면, readObject 메서드나 readResolve 메서드를 반드시 제공해야 한다. 아니면 ObjectOutputStream.writeUnshared나 ObjectInputStream.readInshared 메서드를 반드시 사용해야 한다.

그렇지 않다면 공격자가 이 클래스로부터 가변 인스턴스를 만들어낼 수 있다.

## 정리

### 1. 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

### 2. 단순한 값 객체는 항상 불변으로 만들자. 무거운 값 객체라면 불변 클래스와 쌍을 이루는 각변 동반 클래스를 public으로 제공하자.

### 3. 불변으로 만들 수 없는 클래스라도 변경 가능 부분을 최소한으로 줄이자.

### 4. 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.

### 5. 생성자는 붋련식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.