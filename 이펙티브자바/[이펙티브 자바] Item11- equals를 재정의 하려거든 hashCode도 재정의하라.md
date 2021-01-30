# [이펙티브 자바] Item11- equals를 재정의 하려거든 hashCode도 재정의하라

---

## 1. equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

hashCode를 재정의하지 않으면 hashCode 일반 규약을 어기게 되어 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생한다. 

### hashCode 재정의 조건

1. equals 비교에 사용되는 정보가 변경되지 않는다면, hashCode는 항상 같은 값을 반환해야 한다.
2. equals가 두 객체가 같다고 판단하면, hashCode는 같은 값을 반환해야 한다.
3. equals가 두 객체가 다르다고 판단해도, hashCode가 꼭 다를 필요는 없다. 다만, 다른 객체에서는 다른 값을 반환해야 성능이 좋아진다.

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 2번이다. 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수는 있다. 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여 서로 다른 값을 반환한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdYUdFP%2FbtqS4LRk1mG%2FxTHJsurhhAyXi6ERlhkeV0%2Fimg.png)

위의 코드를 살펴보자. 결과로 "제니"가 나와야 할 것 같지만 실제로는 null을 반환한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdRGEry%2FbtqTfDdFnPm%2F9vpl4r6sbn1J7RldOJKUv0%2Fimg.png)

여기에는 2개의 인스턴스가 사용되었다. 하나는 put할때, 다른 하나는 get할 때 사용되었다. PhoneNumber 클래스는 hashCode를 재정의 하지 않았기 떄문에 논리적으로 동치인 두 객체가 다른 hashCode를 반환하여 2번 항목을 지키지 못한다.

이러한 문제는 PhoneNumber 클래스에 equals와 hashCode를 재정의 해주기만 하면 해결된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb0G5Jf%2FbtqS3I8qwaH%2FKS18ruePMe6kBIeWcfhQJ0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnXB0d%2FbtqThZ1OkWt%2F3JhS7KcYFqO5WG3DaiwKA0%2Fimg.png)

### 항상 같은 hashCode를 반환하면 2번 항목을 만족시키는 것 아닌가?

```java
@Override
public int hashCode() {
	return 42;
}
```

절대 사용하면 안된다. 항상 같은 hashCode는 링크드리스트(LinkedList)처럼 동작하기 떄문에 O(1)이던 시간복잡도가 O(N)으로 느려진다. 따라서 객체가 많으면 도저히 쓸 수 없는 방법이다.

### 올바른 hashCode를 만드는 방법?

```java
// 전형적인 hashCode 메서드
@Override
public int hashCode() {
	int result = Integer.hashCode(areaCode);
  result = 31 * result + Integer.hashCode(prefix);
  result = 31 * result + Integer.hashCode(lineNum);
  return result;
}
```

```java
// 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉬움
// IntellIJ의 자동완성 hashCode는 해당 메서드를 사용한다.
@Override
public int hashCode() {
	return Ovjects.hash(lineNum, prefix, areaCode);
}
```

**주의 사항**

- equals에서 사용되지 않는 필드는 **반드시** 제외해야 한다. (두 번째 항목 위배)
- 참조 타입 필드가 null일 경우 0을 사용
- 31을 곱함으로써 큰 수로 만들어 해시 효과를 증가시킨다.(더하는 경우 같은 값이 나올 확률이 높기 때문에)

## 2. hashCode 지연 초기화

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 캐싱을 고려해야한다.

- 해시의 키로 자주 사용될 것 같으면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.
- 해시의 키로 사용되지 않는다면 지연 초기화가 좋다.

```java
private int hashCode;

@Override
public int hashCode() {
      	int result = hashCode;
        if(result == 0) {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
        }
        return result;
}
```

**지연 초기화**는 쓰레드의 안정성을 고려해야 한다. 여러개의 쓰레드가 동시에 hashCode를 호출하는 경우 여러번의 hashCode 연산이 발생할 수 있다.  지연 초기화를 사용하기 위해서는 쓰레드 **동기화 시점**을 신경써야 한다.

### 성능을 높인다고 hashCode를 계산할 때 핵심 필드를 생략하면 안된다.

속도는 빨리질 수 있어도 해시 테이블의 성능(품질)은 나빠진다. 

### hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 공표하지 마라.

API를 공표해버리면 해시 기능을 개선할 여지가 없어진다. 반대로 API를 공표하지 않으면 클라이언트가 값에 의존하지 않게 되고, 추후 계산 방식을 바꿀 수 있게 된다.