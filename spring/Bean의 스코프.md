# [스프링 프레임워크 핵심기술] - Bean의 스코프

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

오늘 기록할 내용은 Bean의 스코프에 대한 전반적인 이해와 활용 방법, 사용시 주의할 사항 등을 다룬다.

# 1. Bean의 스코프

Bean의 스코프로는 크게 싱글톤과 프로토타입으로 나눌 수 있다.  (더 다양한 스코프가 있지만 이번시간에는 이 두가지를 주제로 다룹니다.)

먼저 싱글톤과 프로토타입의 개념을 설명하자면 싱글톤 스코프는 애플리케이션 전반에 걸쳐 해당 Bean의 인스턴스가 오직 한번 생성되는 것이다. 반면에 프로토타입은 매번 새로운 인스턴스를 생성한다.

- 싱글톤
    - 해당 Bean의 인스턴스를 오직 한번만 생성
    - Bean을 생성할 때 별도의 설정을 해주지 않으면 default는 싱글톤
- 프로토타입
    - 해당 Bean의 인스턴스를 매번 생성

싱글톤 스코프는 해당 Bean의 인스턴스를 한번만 생성한다. 이를 확인하기 위해

간단한 테스트를 해보자.

```java
@Component
public class Proto {

}
```

먼저 Proto라는 클래스를 만들어 Bean으로 등록해준다.

```java
@Component
public class Single {

    @Autowired
    private Proto proto;

    public Proto getProto() {
        return proto;
    }
}
```

그 후, Proto를 주입받는 Single이라는 클래스를 Bean으로 등록하고 Single의 proto 값을 전달받을 수 있도록 getter를 만들어 준다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Single single;

    @Autowired
    Proto proto;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(proto);
        System.out.println(single.getProto());
    }
}
```

AppRunner를 통해 두개의 Bean을 주입받고 Proto와 Single의 proto를 호출해본다.

![](https://blog.kakaocdn.net/dn/c0YsAi/btqEuYXdkkH/NOlwudvkd1ro1a9z97gqL1/img.png)

결과를 보면 각 Proto가 같은 레퍼런스를 가지고있는 것을 확인할 수 있다.

Single과 AppRunner에서 Proto라는 타입을 각각 주입받고있는데, Proto라는 Bean의 인스턴스가

한번만 생성된 것이다.

그렇다면 이번엔 프로토타입 스코프에 대해 테스트를 해보자.

```java
@Component @Scope("prototype")
public class Proto {

}
```

Proto 클래스에 @Scope("prototype") 라는 애노테이션을 통해 프로토타입의 스코프를 설정해준다.

그 후, ApplicationContext를 이용해 Bean의 레퍼런스를 직접 출력해보자.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("proto");
        System.out.println(ctx.getBean(Proto.class));
        System.out.println(ctx.getBean(Proto.class));
        System.out.println(ctx.getBean(Proto.class));

        System.out.println("single");
        System.out.println(ctx.getBean(Single.class));
        System.out.println(ctx.getBean(Single.class));
        System.out.println(ctx.getBean(Single.class));
    }
}
```

![](https://blog.kakaocdn.net/dn/bzT3r3/btqEttqw0UF/pfkjkbnukjCHZeAd4wVIq0/img.png)

프로토타입으로 스코프를 준 Proto라는 Bean은 매번 새로운 인스턴스를 생성한 것이 확인된다.

이렇게 간단하게 Bean의 스코프를 관리할 수 있는 것도 스프링의 장점중 하나이다.

---

# 2. 복합 사용시의 문제점

프로토타입과 싱글톤이 섞여 쓰이게 되면 굉장히 복잡한 문제가 발생한다.

예를 들면, 프로토타입의 Bean이 싱글톤 Bean을 참조하여 사용하는 경우

혹은 싱글톤 Bean이 프로토타입 빈을 참조하는 경우 등이다.

사실 전자의 경우 별 문제가 없다.

```java
// 프로토타입 스코프를 가지는 Proto 클래스
@Component @Scope("prototype")
public class Proto {

	// 싱글톤 스코프를 가지는 Single을 주입
	@Autowired
	Single single;

}
```

프로토타입의 Bean에 싱글톤 Bean을 주입한 경우,

프로토 타입의 Bean은 항상 새로운 인스턴스를 생성하지만 그 새로운 인스턴스가 참조하고있는

싱글톤 Bean은 항상 같은 레퍼런스를 가지고 있기 때문에 사용하는데 아무 지장이 없다.

하지만 그 반대의 경우 문제가 발생한다.

```java
// 싱글톤 스코프를 가지는 Single클래스
@Component
public class Single{

	// 프로토타입 스코프를 가지는 Proto을 주입
	@Autowired
	Proto proto;

}
```

싱글톤 Bean에서 프로토타입 Bean을 주입하여 사용할 경우,

이 코드의 **원래 의도대로**라면 Single이 사용될 때 Proto의 인스턴스는 매번 생성되어야 한다.

하지만 싱글톤 Bean의 인스턴스가 생성될때 주입받은 프로토타입의 Bean의 프로퍼티도 함께 생성되고 싱글톤 스코프는 인스턴스가 한번만 생성되기 때문에 이 싱글톤 Bean을 사용할때 프로토타입의 프로퍼티가 변경되지 않는다.

즉, 원래 의도대로 사용할 수 없는 것이다.

---

# 3. 해결 방법

싱글톤 Bean이 프로토타입 Bean을 의도대로 참조하기 위해서 Class 기반의 Proxy를 사용한다.

CGLIB를 이용하여 프로토타입 Bean을 Class 기반의 Proxy 객체로 감싸서 Bean으로 사용할 수있다.

![프토로타입을 감싼 Proxy Bean](https://blog.kakaocdn.net/dn/MPaEl/btqEvhaSX7h/CQp0IwkNwf4BU5sRKvhaK1/img.png)

프토로타입을 감싼 Proxy Bean

왜 Proxy로 감싸는 걸까?

그 이유는 싱글톤 스코프의 Bean이 프로토타입 스코프의 Bean을 직접 참조하지 못하게 하기 위해서이다. Proxy를 거침으로써 프로토타입 Bean이 새로운 인스턴스로 바뀔 수 있도록 해주는 것이다.

```java
@Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class Proto {

}
```

다음과 같이 Class기반 proxyMode를 정의해서 감싸줄 수 있다.