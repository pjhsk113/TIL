# [이펙티브 자바] Item6- 불필요한 객체 생성을 피하라

---

# 1. 불필요한 객체 생성이란?

어떤 객체의 생성이 잦고 그 객체의 생성 비용이 높다면, 하나의 객체를 재사용하는 편이 좋다.

**불필요한 객체 생성을 하지 않고 자원을 아낄 수 있기 때문이다.**

특히, 불변 객체는 언제든 안심하고 재사용할 수 있다. 

> **불변 객체**는 인스턴스가 생성되면 그 상태를 변경할 수 없는 객체를 말한다. 객체의 상태가 변하지 않기 때문에 일관된 인스턴스를 공유해 재사용할 수 있다.

**불변 객체**에서 같은 값을 가지는 인스턴스가 한 개 이상 생성된다면, 그것은 **불필요한 객체 생성**이라 할 수 있다.

# 2. String을 통한 예시

대표적인 **불변 클래스**인 String을 통해 불필요한 객체 생성에 대한 예를 더욱 쉽게 이해할 수 있다.

```java
String a = new String("Effective");
```

위와 같은 코드는 String 인스턴스를 실행할때마다 새로만든다. 완전히 불필요한 객체 생성이라고 할 수 있다.

String은 같은 문자열 리터럴이라면 같은 레퍼런스를 가지게 된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvLvVP%2FbtqPTCXtcXJ%2FFCk6JdpwqMSgeiHkAKIwW0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxKeQo%2FbtqPRAr0wHC%2FPsBcQEx36yLkKZyCzvnhJ0%2Fimg.png)

위에서 생성한 **a와 b**는 같은 레퍼런스를 가지고 있다. 객체 공유(재사용)를 통해 불필요한 객체 생성을 하지 않는다는 뜻이다. 

이러한 재사용이 가능한 이유는 String Pool 덕분이다.

## 2.1 String Pool

Java의 String은 String Pool이라는 공간을 가지는데, 문자열 리터럴로 생성된 String은 String Pool에 객체를 생성하고 String Pool을 가르키게된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbephjf%2FbtqPYkvacoY%2FnIEobl27K2W70zEKRe6HVk%2Fimg.png)

## 2.2 String이 불변 클래스로 만들어진 이유

만약 String이 **가변(mutable)** 이라면 **같은 참조**를 가지는 **객체의 값을 변경**할 수 있게된다. 이는 **같은 참조 & 다른 값**이라는 상황을 만들 수 있기 때문에 재사용이 불가능하다. 따라서 객체 공유를 통한 재사용이 목적인 String Pool을 사용할 수 없다.

또한, 가변 상태의 객체는 같은 문자열 리터럴을 가지더라도 객체를 매번 생성해야한다. String이 빈번하게 사용되는 메서드나 반복문에서 쓸데없는 인스턴스가 수백만 개 만들어 질 수도 있다는 것이다. 이는 애플리케이션의 성능에 안좋은 영향을 미칠 수 있다.

ex)

```java
// 새로운 String 인스턴스를 10000000개 생성함...
// 사용 후 바로 가비지 컬렉션의 대상이 된다.
for (int i = 0; i < 10000000; i++) {
	String a = new String("hi");
}
```

따라서 String을 불변으로 만들어 위와 같은 단점들을 제거하고 성능을 개선할 수 있게 된다.

가변 객체라도 사용 중에 변경이 되지 않음을 신뢰할 수 있다면 재사용할 수 있다.

# 3. 생성 비용이 비싼 경우

만약 생성 비용이 비싼 객체라면 **캐싱**을 통해 재사용을 해야한다.

가장 간단하게 정규표현식을 예로 들 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FE21ho%2FbtqPYkBXOof%2FzUuudut4Ls2KIhheenS7B1%2Fimg.png)

위처럼 String.matches를 사용하는 것보다

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcfmomo%2FbtqPRAevRcF%2FwWURUFktTHQ5yRU4bca8Ck%2Fimg.png)

이렇게 Pattern 인스턴스를 클래스 초기화 과정에서 캐싱하여 사용하는 것이 성능을 훨씬 끌어올릴 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc9n7YX%2FbtqP0tZdK2b%2FpibtbKkqQ8NPbbw6r1Eptk%2Fimg.png)

Parrern은 입력받은 정규표현식의 유한 상태 머신을 만들기 때문에 생성 비용이 높다. 하지만 String.matches 내부의 정규표현식용 Pattern 인스턴스는 한번 쓰고 버려져 바로 가비지 컬렉션 대상이 된다. 이는 비싼 칫솔을 사서 한번 쓰고 버리는 것과 같은 행위이다.

 위 두 방법은 빈번하게 호출되는 상황에서 상당한 성능차이를 보인다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWRQ2E%2FbtqPYTc1i22%2FcK0OkgELuCJzTZrpjlVp60%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrpX8V%2FbtqPYkvaiG5%2FNtTXoAQ7jYrSy9Ow4grLBk%2Fimg.png)

캐싱된 인스턴스를 사용한 경우 약 10배 정도의 성능 향상 뿐만 아니라, 이름을 지어주어 코드의 의미가 훨씬 잘 드러난다는 장점도 있다.

# 4. 어댑터 패턴 사용시 불필요한 객체 생성

어댑터는 실제작업은 뒷단 객체에 위임하고, 자신은 제 2인터페이스 역할을 해주는 객체를 말한다.

Map의 keySet을 대표적인 예로 들수있다.

keySet 메서드는 Map 객체 안의 키를 전부 담은 Set 인스턴스를 반환한다. 하지만 같은 Map에서 호출한 keySet 메서드는 같은 Set 인스턴스를 반환하기 때문에 결국 동일한 Map을 대변한다.

따라서 이를 매번 새로 만들어 사용해도 상관없지만, 이는 불필요한 객체 생성이라 할 수 있다.

반환된 Set 인스턴스를 수정하면 다른 모든 객체(Set과 Map)가 따라서 바뀌기 때문에 값에 대한 신뢰할 수 없게 된다.

이러한 문제점을 해결하기위해 매번 복사해서 새로운 객체를 반환하도록 하는 방어적 복사 방식(이번 아이템 주제와 대조적이지만)을 사용할 수 있다. 

# 5. 오토박싱

오토박싱은 기본 타입과 박싱된 기본 타입을 자동으로 변환해주는 기술이다.

하지만 이를 잘 못 사용하게 되면 불필요한 메모리 할당, 재할당을 반복하여 성능이 느려질 수 있다. 

```java
private static long sum(){
  Long sum = 0L;	
  for(long i =0; i <= Integer.MAX_VALUE; i++){
    sum += i;
  }
  return sum;
}
```

위의 코드에서는 불필요한 인스턴스가 2^31개 만들어진다. 박싱된 기본 타입인 Long을 명시했기 때문이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1cQ1z%2FbtqPQsnuTMc%2FeQcIBdkNqTg9aL7URI5Zx0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fo67yU%2FbtqP4myvSGY%2FheXgFLc729GIkfUPekbQD0%2Fimg.png)

Long(박싱 기본 타입)을 long(기본 타입)으로 변경해주는 것 만으로 엄청난 성능 차이가 나는 것을 확인 할 수있다.

꼭 박싱 타입이 필요한 경우가 아니라면 기본타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않게 주의하자.
