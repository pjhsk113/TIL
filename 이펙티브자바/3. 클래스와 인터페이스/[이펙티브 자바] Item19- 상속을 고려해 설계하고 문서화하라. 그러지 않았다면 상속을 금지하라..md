# [이펙티브 자바] Item19- 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

프로그래머의 통제권 밖에 있어 언제 어떻게 변경될지 모르는 외부 클래스를 상속할 때는 주의해야한다. 이러한 클래스를 상속하게 하려면 상속할 때의 주의점을 문서화해야한다.

## 상속을 고려한 설계와 문서화

### 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 (자기사용) 문서로 남겨야 한다.

클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수 있다. 이때 호출한 메서드가 재정의 가능한 메서드라면, 이 사실을 메서드의 API 설명에 명시해야 한다. 더 넓게 말하자면, 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야한다. 

이를 도와주는 용도로 @implSpec를 이용하면 javadoc이 내부 동작 방식을 설명하는 "Implementation Requirements"절을  생성해준다.

ex) AbstractCollection - remove 메서드

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcFGMCD%2FbtqXZIV3byz%2F0wn7EB1sygIKj6GZIwVOAk%2Fimg.png)

내부적으로 iterator의 remove 메서드를 사용하고 있다는 설명과  해당 컬렉션의 iterator 메서드가 반환한 iterator가 remove 메서드를 구현하지 않았다면 UnsupportedoperationException을 반환한다는 내용이 명확히 명시되어있다.

### 클래스의 내부 동작 과정 중간에 끼어들 수 있는 hook을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

효율적인 하위 클래스를 큰 어려움없이 만들수 있게 하려면 내부 동작 과정에 끼어들 수 있는 hook을 protected 메서드로 제공할 수도 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnaPQa%2FbtqX83R8kdo%2F0c1oVlKaFu4VhLiqavGsd0%2Fimg.png)

위의 removeRange 메서드는 이 리스트 또는 부분 리스트의 clear 메소드에서 사용된다고 나와있다. 또한 해당 메서드를 재정의하면 이 리스트 또는 부분 리스트의 clear 메서드를 고성능으로 만들 수 있다고 명시되어있다. 

removeRange 메서드를 제공한 이유는 단지 하위 클래스의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서이다. 그렇다면, 상속용 클래스를 설계할 때 어떤 메서드를 protected로 노출해야 할지는 어떻게 결정할까?

심사숙고해서 잘 예측해보고, 실제 하위 클래스를 만들어 테스트해보는 것이 최선이다. protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적은게 좋다. 다만, 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다.

### 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다. 따라서 배포 전, 반드시 하위 클래스를 만들어 검증해야 한다.

하위 클래스를 만들어 검증 시, 꼭 필요한 protected 멤버를 놓쳤다면 검증 도중에 빈자리가 확연히 드러날 것이다. 반대로 사용되지 않는 protected 멤버는 private이어야 할 가능성이 크다.

이러한 결정들이 클래스의 성능과 기능에 영원히 족쇄가 될 수 있음을 인식해야한다. 따라서 반드시 배포전에 하위 클래스를 만들어 검증하자.

### 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

이 규칙을 어기면 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 이때 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면, 의도대로 동작하지 않는다.

```java
public class Test {
    public Test() {
        overrideMe();
    }

    public void overrideMe() {
        System.out.println("Test method");
    }
}

public class SubTest extends Test{
    private final Date date;

    public SubTest() {
        date = new Date();
    }

    @Override
    public void overrideMe() {
        System.out.println(date);
    }

    public static void main(String[] args) {
        SubTest sub = new SubTest();
        sub.overrideMe();
    }
}
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsfKOU%2FbtqX6gxpvHK%2FR8szSqbLb17ixmQHtNo4B1%2Fimg.png)

하위 클래스의 생성자보다 상위 클래스의 생성자가 먼저 호출되는데, 상위 클래스의 생성자에서 하위 클래스의 재정의된 메서드를 호출하여 Date 값이 초기화 되기전에 접근하기 때문에 null값이 출력된다. 만약 println가 아니었다면, NullpointerException을 반환했을 것이다.

이와 같은 현상은 Cloneable과 Serializeable 인터페이스의 clone 메서드와 readObject 메서드에서도 나타나는데, 두 메서드가 새로운 객체를 생성하는 효과(생성자와 비슷)를 가지기 때문이다. 따라서 이 둘 중 하나의 인터페이스를 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다. 

clone과 readObject 모두 재정의 가능 메서드를 호출할 경우

readObject

→ 하위 클래스의 상태가 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다. 

clone

→ 하위 클래스의 clone 메서드가 복제본의 상태를 수정ㅎ하기 전에 재정의한 메서드를 호출하게 된다.

마지막으로, Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖게되면, 이 메서드들은 proteted로 선언해야 한다. private로 선언할 경우 하위 클래스에서 무시되기 떄문이다.

### 상속용으로 설계하지 않은 클래스는 상속을 금지한다.

상속용으로 설계되지 않은 클래스는 클래스에 변화가 생길 때마다 하위 클래스에 오작동을 일으킬 수 있다. 구체 클래스의 내부만 수정했음에도 이를 확장한 클래스에서 버그가 발생할 수 있기 때문에 상속용으로 설계되지 않았다면 상속을 금지해야한다.

상속을 금지하는 방법은 두 가지가 존재한다. 

1. 클래스를 final로 선언한다.

2. 모든 생성자를 private나 default로 선언하고 public 정적 팩터리를 만든다.

2번째 방법은 내부에서 다양한 하위 클래스를 만들어 사용할 수 있는 유연함을 제공한다. 두 개의 방법 중 어느 방식이든 좋다.

하지만 상속용으로 설계되지 않은 클래스라도 꼭 상속을 허용해야한다면, 재정의 가능한 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기면 된다. 이렇게 한다면 상속해도 그리 위험하지 않은 클래스를 만들 수 있다.

클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거할 수 있는 방법은 **도우미 메서드**를 만드는 것이다. 본문 코드를 private **도우미 메서드로** 옮기고 이 도우미 메서드를 호출하도록 하면 된다.

```java
public class Test {
    public Test() {
        helperMethod();
    }

    public void overrideMe() {
        helperMethod();
    }

    private void helperMethod() {
        System.out.println("Test method");
    }
}
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FY1wCd%2FbtqYbJeMCq7%2FPKck8BCdiYfEa2FlqIV4TK%2Fimg.png)

## 정리

클래스 내부에서 스스로 어떻게 사용하는지를(자기사용 패턴) 모두 문서로 남겨야하며, 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야한다.

다른 프로그래머가 고효율의 하위 클래스를 만들수 있도록 일부 메서드를 protected로 제공해야할 수 있다. 

클래스를 확장해야 할 명확한 이유가 없다면 상속을 금지하는 편이 나을 수 있다.