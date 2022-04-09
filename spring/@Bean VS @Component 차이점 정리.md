# [Spring] @Bean VS @Component 차이점 정리

---

@Bean과 @Component는 어떤 객체를 Bean으로 등록하고 싶을 때 사용되는 애노테이션들이다.

그렇다면 이 둘의 차이점이 뭘까? 두개의 애노테이션 모두 Bean으로 등록하겠다는 목적을 가지는

애노테이션인데 왜 둘로 나누어져있을까?

항상 헷갈렸던 부분이기도하고 명확한 이해가 없는 것 같아 이번 글에서 확실히 정리해보고자 한다.

---

# 1. @Bean

@Bean 같은 경우에는 메소드 위에 선언 가능하고 외부 라이브러리를 Bean으로 등록할 때 사용된다.

이해를 돕기위해 먼저 @Bean 애노테이션을 살펴보자.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/35b02c7e-48cf-47be-ae20-4d396e30eb04/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/35b02c7e-48cf-47be-ae20-4d396e30eb04/Untitled.png)

@Target이 METHOD로 지정되어있다.  이는 메소드 위에 선언되어야 한다는 의미이다.

이건 알겠는데.. 왜 외부 라이브러리를 Bean으로 등록할 때 사용되는걸까?

외부 라이브러리는 ReadOnly File로 @Component를 선언해 줄 수 없다.

즉, 개발자가 직접 컨트롤 할 수 없다.

따라서 개발자가 외부 라이브러리를 Bean으로 등록해야 한다면,

인스턴스를 생성하는 메소드를 만든 후, 그 메소드에 @Bean을 선언해 Bean으로 등록 할 수 있다.



![@Bean 사용 예시](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f65d350-01bb-45c6-a300-34ae2c12d9b9/Untitled.png)

@Bean 사용 예시

---

# 2. @Component

@Component는 클래스 위에 선언 가능하고, 직접 컨트롤이 가능한 객체에서 사용된다.

마찬가지로 @Component 애노테이션을 살펴보자.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b853e21-1f58-44b9-a702-04e07b496f85/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b853e21-1f58-44b9-a702-04e07b496f85/Untitled.png)

@Target이 TYPE로 설정되어있다.

즉, Class Type 혹은 Interface Type, Enum Type 등에 선언 할 수 있다.

아래 사진 처럼 개발자가 생성한 클래스 같이 직접 컨트롤 할 수 있는 클래스들에는

@Component 애노테이션을 사용한다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6bf31c3e-3680-47d2-bb43-34a8ebc7f1b2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6bf31c3e-3680-47d2-bb43-34a8ebc7f1b2/Untitled.png)

---

ElementType의 의미가 궁금하다면 ElementType을 살펴보면 된다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e1738b5-dc6e-4b67-816a-c80487847588/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e1738b5-dc6e-4b67-816a-c80487847588/Untitled.png)

열거형 타입(enum)의 ElementType을 보면 앞서 살펴봤던 METHOD뿐 아니라, TYPE의 의미를

명확히 알 수 있다.