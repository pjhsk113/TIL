# [이펙티브 자바] item90 - 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

Serializable을 구현하기로 결정한 순간부터 생성자 이외의 방법으로 인스턴스를 생성할 수 있게된다. 
이 말은 버그와 보안 문제가 일어날 가능성이 커진다는 뜻이다.

하지만 이러한 위험을 크게 줄여주는 기법이 있다. 바로 **직렬화 프록시 패턴**이다.

## 직렬화 프록시 패턴

직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.

## 직렬화 프록시 패턴의 구조

1. 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다.
    1. 이 중첩 클래스가 바깥 클래스의 직렬화 프록시이다.
2. 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받는다.
3. 생성자는 단순히 인수로 넘어온 인스턴스를 복사한다. (일관성 검사 및 방어적 복사도 필요없다.)
4. 바깥 클래스와 직렬화 프록시 모두 Serializable을 선언한다.

직렬화 프록시 패턴을 만드는 과정을 코드로 살펴보자.

### 직렬화 프록시 ****중첩 클래스 추가

```java
class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    // Peirod의 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

		    private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다.
    }
}
```

### 바깥 클래스에 writeReplace() 메서드 추가

다음으로, 바깥 클래스에 writeReplace 메서드를 추가한다. 
이 메서드는 범용적이니 직렬화 프록시를 사용하는 모든 클래스에 복사해 쓰면된다.

```java
class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 바깥 클래스의 인스턴스를 직렬화 프록시로 변환한다.
     * 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.
     */
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // Peirod의 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

		    private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다.
    }
}
```

writeReplace 메서드는 바깥 클래스의 인스턴스 대신 프록시를 반환하는 역할을 한다.
즉, 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.

이 메서드 덕분에 직렬화 시스템은 절대 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.

### 바깥 클래스에 readObject() 메서드 추가

다음으로 불변식을 훼손하려는 공격을 막기위해 readObject 메서드를 바깥 클래스에 추가하자.

```java
class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 바깥 클래스의 인스턴스를 직렬화 프록시로 변환한다.
     * 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.
     */
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    /**
     * readObject는 Serializable을 구현한 모든 타입을 생성할 수 있고, 그 타입들 안의 모든 코드도 수행할 수 있다.
     * readObject를 통해 불변식을 훼손하려 하면 InvalidObjectException을 던진다.
     */
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }

    // Peirod의 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

		    private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다.
    }
}
```

### 직렬화 프록시 중첩 클래스에 readResolve() 메서드 추가

마지막으로, 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 중첩 클래스에 추가한다. 이 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.

```java
class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 바깥 클래스의 인스턴스를 직렬화 프록시로 변환한다.
     * 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.
     */
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    /**
     * readObject는 Serializable을 구현한 모든 타입을 생성할 수 있고, 그 타입들 안의 모든 코드도 수행할 수 있다.
     * readObject를 통해 불변식을 훼손하려 하면 InvalidObjectException을 던진다.
     */
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }

    // Peirod의 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        /**
         * 역직렬화 시 호출되며, 직렬화 프록시를 바깥 클래스 인스턴스로 변환한다.
         */
        private Object readResolve() {
            return new Period(start, end);
        }

		    private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다.
    }
}
```

readResolve 메서드는 일반 인스턴스를 만들 때와 같은 생성자, 정적 팩터리, 다른 메서드를 사용해 역직렬화된 인스턴스를 생성한다. 
따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 방법을 찾지 않아도 된다.
그냥 정적 팩터리나 생성자가 불변식을 확인해주면 된다.

## 직렬화 프록시 패턴의 장점

1. 가짜 바이트 스트림 공격이나 내부 필드 탈취 공격을 프록시 수준에서 차단할 수 있다. 
2. 필드를 final로 선언해도되므로 직렬화 시스템에서 진짜 불변을 만들 수 있다.
3. 역직렬화 시 유효성 검사를 수행하지 않아도 된다.
4. 역직렬화한 인스턴스와 원래의 직렬화된 클래스가 달라도 정상적으로 동작한다.

특히, 역직렬화한 인스턴스와 원래의 직렬화된 클래스가 달라도 정상적으로 동작한다는 부분이 직렬화 프록시가 가지는 강력한 장점이다. 

대표적으로 EnumSet의 사례를 들 수 있다. 
EnumSet은 정적 팩터리만 제공한다.

클라이언트 입장에서는 EnumSet 인스턴스만 반환하는 것으로 보이지만, 실은 열거 타입의 크기에 따라 두 하위 클래스 중 하나의 인스턴스를 반환한다. 즉, 원소가 64개 이하면 RegularEnumSet을 그보다 크면 JumboEnumSet을 반환한다.

만약, 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화하면 어떻게 될까?

처음 직렬화된 것은 RegularEnumSet이지만 역직렬화될 때는 JumboEnumSet을 반환하면 좋을 것 같다.

EnumSet은 직렬화 프록시 패턴을 사용해서 실제로 이렇게 동작한다. 

![Untitled](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20item90%20-%20%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%85%E1%85%A7%E1%86%AF%E1%84%92%E1%85%AA%E1%84%83%E1%85%AC%E1%86%AB%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%86%AB%E1%84%89%E1%85%B3%20%E1%84%83%2072261e4f315a4dbbbd5d0172eea5e714/Untitled.png)

## 직렬화 프록시의 한계

1. 클라이언트가 마음대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에 적용할 수 없다.
3. 방어적 복사보다 느리다.

위와 같은 객체의 메서드를 직렬화 프록시 readResolve 안에서 호출하면 ClassCastException이 발생한다. 실제 객체는 아직 만들어지지 않았기 때문이다.

## 핵심 정리

- 제 3자가 확장할 수 없는 클래스라면 직렬화 프록시 패턴을 사용하자.