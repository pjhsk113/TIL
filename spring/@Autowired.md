# [스프링 프레임워크 핵심기술] - @Autowired

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

오늘은 @Autowired Annotation의 사용 방법, 동작 원리까지 알아본다.

# 1. @Autowired란?

- @Autowired는 의존성 주입을 할 때 사용하는 Annotation으로  의존 객체의 **타입**에 해당하는 bean을 찾아 주입하는 역할을 한다.

# 2. @Autowired를 사용할 수 있는 위치

@Autowired는 기본적으로 아래의 위치에서 사용할 수 있다.

- 생성자 (스프링 4.3부터는 생략 가능)
- Setter
- 필드

# 3. 사용 시 주의점

> 해당 타입의 bean이 없거나 한 개인 경우
>

## 3-1. 생성자에 @Autowired 명시 (스프링 4.3부터는 생략 가능)

```java
@Service
public class TestService {

    TestRepository testRepository;
		
		@Autowired
    public TestService(TestRepository testRepository) {
        this.testRepository = testRepository;
    }
}
```

```java
public class TestRepository {
	....
}
```

위의 코드에서 TestRepository의 의존성 주입이 작동할까?

당연히 작동하지 않는다. @Autowired는 의존 객체의 **타입**에 해당하는 bean을 찾아 주입하기 때문이다.

즉, TestRepository는 bean으로 등록되어있지 않기 때문에 스프링이 bean을 찾지 못해 의존성을 주입할 수 없다.

이를 작동하게 하기위해서는 @Repository 혹은 @Component Annotation을 이용해

TestRepository를 bean으로 등록해주면 된다.

```java
@Repository
public class TestRepository {
	....
}
```

~~참쉽쥬?~~

## 3-2. Setter에 @Autowired 명시

```java
@Service
public class TestService {

    TestRepository testRepository;
		
		@Autowired
		//@Autowired(required = false)
    public void setTestRepository(TestRepository testRepository) {
        this.testRepository = testRepository;
    }
}
```

```java
public class TestRepository {
	....
}
```

이번엔 Setter를 이용해 의존성 주입을 시도해보자. 하지만 이 역시 작동하지 않는다.

setter로 TestService 자체의 인스턴스를 만들었는데 왜 작동을 안할까?

그 이유는 @Autowired Annotation 때문이다.

@Autowired가 명시되어 있기 때문에 스프링은 의존성을 주입하려고 시도한다.

setter를 이용해 인스턴스를 생성할 수 있지만, 의존성 주입에는 실패하는 것이다.

**@Autowired(required = false)** 옵션을 주면 의존성 주입을 받지 않고 인스턴스를 만들어서 bean으로 등록할 수 있다. 즉, TestRepository는 의존성 주입이 되지 않은 상태로 bean이 등록된다.

## 3-3. 필드에 @Autowired 명시

```java
@Service
public class TestService {

		@Autowired
		//@Autowired(required = false)
    TestRepository testRepository;
}
```

```java
public class TestRepository {
	....
}
```

필드 위에 @Autowired 명시

> **setter**나 **필드** Injection을 사용할 때는 Optional 설정(required = false)을 통해 TestService가 해당하는 의존성 없이도 bean으로 등록할 수 있다.
>

---

> 해당하는 타입의 bean이 여러개인 경우
>

```java
@Service
public class TestService {

		@Autowired
    TestRepository testRepository;
}
```

```java
public interface TestRepository {
	....
}

@Repository
public class MyTestRepository implements TestRepository {
  // TestRepository의 구현체
}

@Repository
public class ExampleRepository implements TestRepository {
  // TestRepository의 구현체
}
```

TestService는 TestRepository를 사용하는데, TestRepository는 인터페이스이고 이를 구현하는

Repository bean이 2개 있다고 생각해보자.

스프링은 TestService에는 어떤 Repository를 주입해줄까?

스프링은 개발자가 원하는 의존성을 알지 못하기 때문에 주입을 못해준다.

따라서 같은 타입의 bean을 2개 발견했다는 에러 메시지를 뱉고, 다음과 같은 액션을 추천해준다.

- @Primary
- 해당 타입의 bean 모두 주입 받기
- @Qulifier(bean id)

### 1. @Primary

@Primary Annotation은 같은 타입의 bean이 여러개 주입될 경우 @Primary가 붙은 bean을 주입하겠다는 의미를 가지고 있다.

```java
public interface TestRepository {
	....
}

@Repository @Primary
public class MyTestRepository implements TestRepository {
  // TestRepository의 구현체
}

@Repository
public class ExampleRepository implements TestRepository {
  // TestRepository의 구현체
}
```

- @Repository 옆에 명시
- 같은 타입의 TestRepository가 주입되었을 때 MyTestRepository가 주입된다.

### 2. @Qulifier

```java
@Service
public class TestService {

		@Autowired @Qulifier("myTestRepository")   // bean의 Id는 lower Camel Case를 사용한다.
    TestRepository testRepository;
}
```

- @Autowired 옆에 명시
- @Primary와 마찬가지로 MyTestRepository가 주입된다.

> @Primary가 @Qulifier보다 Type safe하기 때문에 @Primary를 사용하는 것을 추천
>

### 3. 해당 타입의 bean 모두 주입

```java
@Service
public class TestService {

		@Autowired
    List<TestRepository> testRepositoies;
}
```

- List로 같은 타입의 모든 bean을 주입 받을 수 있다.

# 4. 동작 원리

### 주요 개념

- **BeanPostProcessor**
    - 초기화 라이프 사이클 **이전**과 **이후**에 필요한 부가 작업을 할 수 있는 라이프 사이클 콜백
    - IoC 컨테이너에 등록되어 있음
- **AutowiredAnnotationBeanPostProcessor**
    - **BeanPostProcessor**의 구현체
    - **BeanPostProcessor**의 구현체이므로 IoC 컨테이너에 Bean으로 등록되어있음
- InitializingBean
    - 초기화 라이프 사이클
    - Bean 인스턴스가 생성되는 시점

---

@Autowired는 **BeanPostProcessor**라는 라이프 사이클 인터페이스의 구현체인 **AutowiredAnnotationBeanPostProcessor**에 의해 의존성 주입이 이루어진다.

**BeanPostProcessor**는 초기화 라이프 사이클 이전과 이후에 필요한 부가 작업을 할 수 있는 라이프 사이클 콜백이다.

즉, bean이 만들어지는 시점 이전 혹은 이후에 추가적인 작업을 하고싶을 때 사용된다.

**AutowiredAnnotationBeanPostProcessor**가 bean 초기화 라이프 사이클 이전(bean 인스턴스 생성 이전)에 @Autowired가 붙어 있는 bean을 찾아 주입해주는 작업을 한다.

대략적인 라이프 사이클을 보자면 다음과 같다.

BeanPostProcess (before) → InitializingBean → BeanPostProcess (after)

다시 한번 동작 순서를 정리하자면,

1. BeanFactory( ApplicationContext ) 가 BeanPostProcessor 타입의 Bean을 찾는다.
2. IoC 컨테이너에 등록되어있는 다른 일반적인 Bean에게 BeanPostProcessor를 적용한다.
3. 다른 Bean에 @Autowired Annotation을 처리하는 **AutowiredAnnotationBeanPostProcessor**의 로직이 적용된다.
4. 의존성 주입이 일어난다.

---

지금까지 @Autowired를 사용하면서도 이런 동작 원리를 가지고 있는지 몰랐다. 사실 지금은 깊게 알지 못해도 무방한 부분이지만, 이런 동작 원리를 알고 사용하는 것과 모르고 그냥 사용하는 것은 큰 차이가 있다고 생각한다. 시간이 지나고 경험이 쌓이면 이런 부분이 더 중요해지지 않을까?

좋은 개발자로 성장하기 위해서 이러한 부분을 탄탄히 다져야겠다는 생각이 들었다. 지금은 25분짜리 강의를 이해하고 정리하는데 하루나 걸리고 학습 진행 속도가 더디지만, 앞으로 더 좋아지겠지?