# [이펙티브 자바] item88 - readObject 메서드는 방어적으로 작성하라

### 방어적 복사(가변 필드)를 사용하는 불변 클래스의 Serializable 구현

[item50](https://jjingho.tistory.com/99)에서는 불변 날짜 범위 클래스를 만드는데 가변 Date 필드를 이용했다. 따라서 불변식을 지키기 위해 생성자와 접근자에 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어졌다.

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

만약 이 클래스를 직렬화하기로 결정했다고 생각해보자. Period 객체의 물리적 표현과 논리적 표현이 부합하므로 클래스 선언에 impements Serializable을 선언해 간단히 직렬화를 끝낼 수 있을 것 같다. 하지만 이것만으로는 클래스의 불변식을 보장하지 못한다.

### 이 클래스가 불변식을 보장하지 못하는 이유

이 클래스가 불변식을 보장하지 못하는 이유는 **readObject 메서드** 때문이다. 

readObject는 Serializable을 구현한 모든 타입을 생성할 수 있고, 그 타입들 안의 모든 코드도 수행할 수 있다. 따라서 readObject는 또 다른 public 생성자로 볼 수 있기 때문에 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.

보통의 생성자처럼 **인수가 유효한지 검사**해야 하고, 필요하다면 **방어적으로 복사**해야 한다. 이 작업을 제대로 수행하지 못하면 공격자는 **아주 쉽게 불변식을 깨트릴 수 있다.**

readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다. 하지만 임의로 생성한 바이트 스트림을 건내면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낸다.

```java
public class BogusPeriod {
    
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06
         .... 대충 바이트 코드들
    }
    
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    // 주어진 직렬화 형태로부터 객체를 만들어 반환한다.
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

위 프로그램을 실행하면 `Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984`를 출력한다. Period를 직렬화한 후 바이트 배열 리터럴을 조작해서 불변식을 깨트린 것이다. 

이렇듯 Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 되었다.

## 해결 방법

### 완전하지 않은 해결 방법

이 문제를 해결하려면 Period의 readObject 메서드가 defaultReadObject를 호출하도록 한다음 역직렬화된 객체가 유효한지 검사해야 한다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

이렇게 하면 허용되지 않은 Period 인스턴스를 생성하는 일을 막을 수 있다.

하지만 미묘한 문제가 하나 숨어있는데, 정상 Period 인스턴스에서 바이트 스트림 끝에 private Date 필드 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다는 것이다.

공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후, 스트림 끝에 추가된 private Date 필드를 참조를 읽어 Period 객체 내부의 정보를 얻을 수 있다. 참조로 얻은 Date 인스턴스들은 수정될 수 있으니 더 이상 불변이 아니게 된다.

```java
public class MutablePeriod {
    //Period 인스턴스
    public final Period period;

    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));

            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가

            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }

    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;

        //시간 되돌리기
        pEnd.setYear(78);
        System.out.println(p);

        //60년대로 회귀
        pEnd.setYear(60);
        System.out.println(p);
    }
}
```

이 프로그램의 실행 결과는 다음과 같다.

![](https://blog.kakaocdn.net/dn/cDfKjD/btrmirh6D6S/CKHJe9q62sxPFx5Hq8E9ck/img.png)

처음에는 불변식으로 생성됐지만 의도적으로 내부의 값을 수정할 수 있게되어 올바르지 않은 값이 생성되었다. 이와 같은 방법으로 공격자는 불변이라고 생각하는 클래스에 엄청난 보안 문제를 일으킬 수 있다.

### 진짜 해결 방법!

이 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다. **객체를 역직렬화할 때는 클라이언트가 소유하면 안 되는 객체 참조를 갖는 필드 모두를 반드시 방어적으로 복사해야 한다.**

readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareto(end) > 0) {
        throw new InvalidObjectException(start + " after " + end);
    }
}
```

방어적 복사를 유효성 검사보다 앞서 수행해야 하며, Date의 clone 메서드를 사용하지 않아야 한다. 또한, final 필드는 방어적 복사가 불가능하니 주의하자. (start와 end의 final 한정자를 제거했다.)

## 기본 readObject와 커스컴 readObject의 선택 기준

transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은지를 기준으로 생각해보면 된다.

괜찮다면 **기본 readObject**를 사용해도 되지만, 아니라면 **직렬화 프록시 패턴을 사용하거나 커스텀 readObject** 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야 한다.

final이 아닌 직렬화 가능한 클래스라면 readObject 메서드도 재정의 가능한 메서드를 호출해서는 안 된다.(생성자와의 공통점) 하위 클래스의 상태가 역직렬화되기 전에 하위 클래스에서 재정의된  메서드가 실행될 수 있고, 이는 프로그램의 오작동으로 이어질 수 있다.

## 핵심 정리

- readObject 메서드는 언제나 public 생성자를 작성하는 자세로 임하자.
- readObject 메서드는 어떤 바이트 스트림이 넘어와도 유효한 인스턴스를 만들어내야 한다.
- 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해선 안된다.
- 안전한 readObject를 메서드를 만드는 지침
    - private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하자.(불변 클래스의 가변 필드)
    - 모든 불변식을 검사하고 어긋나는 게 있다면 InvalidObjectException을 던진다.
    - 방어적 복사 후에는 반드시 불변식 검사가 뒤따라야 한다.
    - 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
    - 직접이든 간접이든, 재정의할 수 있는 메서드는 호출하지 말자.