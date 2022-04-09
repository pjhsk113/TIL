# [Java] Java Bean VS Spring Bean

---

Spring으로 개발을 하다보면 Bean이라는 개념이 자주 등장한다. 자주 사용하는 용어이기 때문에

당연히 알고있는 개념이라고 생각하지만, 막상 의미를 정의하라고 하면 헷갈리는 경우가 많다.

그래서 오늘은 Bean이라는 개념을 명확하게 정리해보고자 한다.

---

# 1. Java Bean

먼저 Java Bean에 대해 알아보자.

결론부터 말하자면 Java Bean은 특정 형태의 클래스를 가르키는 뜻으로 사용된다.

DTO 혹은 VO의 형태가 Java Bean이라고 생각하면 쉽다.

필드는 private로 구성되어 getter와 setter를 통해 접근할 수 있고, 전달 인자가 없는 생성자를

가지는 형태의 클래스이다.

- getter / setter
- public의 no-argument 생성자
- 모든 필드는 private로 getter와 setter를 통해서만 접근 가능

getter와 setter, 생성자를 가지는 클래스를 가르키는 뜻으로 사용되는 만큼

POJO(Plain Old Java Object)와 거의 동일한 개념이라고 이해하면 될 것 같다.

코드로 보면 이해가 더욱 쉽다. 아래와 같은 형태의 클래스를 Java Bean이라 부른다.

~~어디서 많이 본 클래스죠?~~

```java
public class AboutJavaBean {
		// 필드는 private로 선언
    private String bean;
    private int beanValue;

    public AboutJavaBean() {
    // no-argument 생성자
    }
		
		// getter
    public String getBean() {
        return beanName;
    }
		// setter
    public void setBean(String bean) {
        this.bean = bean;
    }

    public int getBeanValue() {
        return beanValue;
    }

    public void setBeanValue(int beanValue) {
        this.beanValue = beanValue;
    }
}
```

---

# 2. Spring Bean

Spring에서의 Bean은 스프링 IoC컨테이너가 관리하는 Java 객체를 뜻한다.

일반 Java 객체와 다른 점은 없다. 그냥 스프링 IoC컨테이너에서 관리되는 객체를 Bean이라고

부르는 것이다.

스프링 IoC가 관리하는 객체라함은 스프링에 의해 생성되고, 라이프 사이클을 수행하고, 의존성 주입이 일어나는 객체들을 말한다.

즉, 개발자가 관리하는 객체가 아닌 스프링에게 제어권을 넘긴 객체를 스프링에서 Bean이라고 부른다.

Spring Bean은 간단한 설명만으로 이해될 것이라 생각한다.

혹시 Spring에서 Bean을 등록하는 방법이나, 동작 원리가 궁금하다면 Spring 핵심 기술이라는

카테고리의 글을 참고하면 될 것 같다.

---

Spring Bean에 대한 글을 작성하다보니 한 가지 의문점이 생겼다.

바로 Bean을 등록할 때 사용되는 애노테이션들의 차이점이다.

Bean을 등록하는 방법에는

- xml에 등록하는 방법
- @Bean 애노테이션을 이용하는 방법
- @Component 애노테이션을 이용하는 방법

등 여러가지 방법이 존재하는데 여기에서 @Bean과 @Component를 언제 써야할지 헷갈렸던 경험이 있다. 다음 글은 이 애노테이션들의 차이점에 대해 공부해봐야겠다.