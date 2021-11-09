# [이펙티브 자바] item88 - readObject 메서드는 방어적으로 작성하라

### 가변 필드를 사용하는 불변 클래스의 Serializable 구현

[item50](https://www.notion.so/Item50-98048da652474d1a9a21ca16aa30c88c)에서는 불변 날짜 범위 클래스를 만드는데 가변 Date 필드를 이용했다. 따라서 불변식을 지키기 위해 생성자와 접근자에 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어졌다.

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

이 클래스가 불변식을 보장하지 못하는 이유는 readObject 메서드 때문이다. 

readObject는 Serializable을 구현한 모든 타입을 생성할 수 있고, 그 타입들 안의 모든 코드도 수행할 수 있다. 따라서 readObject는 또 다른 public 생성자로 볼 수 있기 때문에 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.

보통의 생성자처럼 **인수가 유효한지 검사**해야 하고, 필요하다면 **방어적으로 복사**해야 한다. 이 작업을 제대로 수행하지 못하면 공격자는 **아주 쉽게 불변식을 깨트릴 수 있다.**

readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다. 하지만 임의로 생성한 바이트 스트림을 건내면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낸다.

```java
public class BogusPeriod {
    
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06....
    }
    
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```