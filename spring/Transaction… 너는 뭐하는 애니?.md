# 트랜잭션이란?

트랜잭션은 어떤 작업의 완전성을 보장해주는 것을 의미한다.
논리적인 작업 단위를 완벽하게 처리하거나 모두 취소하여 작업의 일부만 적용되는 현상을 방지하는 기술이다. 즉, 데이터의 정합성을 보장하기 위한 기능라고 볼 수 있다.

트랜잭션의 가장 쉬운 예로 계좌 송금 시스템을 떠올릴 수 있다.
계좌 송금 시스템의 논리적인 작업 단위는 다음과 같다.

1. A가 B에게 계좌 송금 신청
2. A가 송금 가능한 상태인지 확인(신청한 송금 금액이 계좌에 들어있는지)
3. A가 신청한 금액만큼 A의 계좌 금액 차감
4. B에게 금액 송금
5. B의 계좌에 송금된 금액이 가산

이러한 논리적 작업을 수행하는 도중 4번 과정에서(B에게 금액을 송금) 장애가 발생한다면 어떻게 될까?
당연히 송금은 취소되고 A의 계좌에서 차감됐던 금액도 원상복구가 된다.
이게 가능한 이유가 바로 **트랜잭션 덕분이다.**

이런 상황에 트랜잭션이 보장되지 않았다면 A의 계좌는 송금 금액만큼 차감됐지만 B의 계좌 금액은 가산되지 않는 불상사가 일어났을 것이다.

![](https://blog.kakaocdn.net/dn/tj1Cj/btrMBscrtg9/s4eDpYgKfJCuno7zmGeOLk/img.png)

트랜잭션이란 개념은 우리가 사용하는 모든 서비스에 적용되어있다. 트랜잭션 덕분에 데이터의 정합성이 보장되고그렇기에 우리는 안심하고 서비스를 사용할 수 있게된다.

그렇다면 우리는 Java + Spring 기반의 애플리케이션 개발을 할 때 트랜잭션을 어떻게 적용할 수 있을까?

# Spring의 트랜잭션

스프링에서 트랜잭션을 사용하는 방법은 크게 두 가지다.
**트랜잭션 서비스 추상화 계층**을 이용하는 방법과 **선언적 트랜잭션**을 이용하는 방법이다.

## 트랜잭션 서비스 추상화 계층

스프링은 트랜잭션 추상화 기술을 제공하고 있다.
이를 이용하면 애플리케이션에서 트랜잭션 API를 이용하지 않고 트랜잭션 경계설정 작업이 가능해진다.

트랜잭션 추상화 계층의 상위 인터페이스인 **PlatformTransactionManager**를 통해 일관된 방식으로 트랜잭션을 제어하는 경계설정 작업이 가능해지는 것이다.

우리는 각자의 환경에 맞는 TransactionManager 클래스를 주입해서 트랜잭션 추상화 기술을 사용할 수 있다. JDBC를 이용하는 경우는 DataSourceTransactionManager를 주입해 사용하면 되고, JPA를 이용하는 경우JpaTransactionManager를 주입하면 된다.

![](https://blog.kakaocdn.net/dn/cqtXaS/btrMDgiio0B/81d798jIGWobgkhqk0eOwk/img.png)

이제 트랜잭션 추상화 API의 사용 방법을 간단히 살펴보자.

```java
@Service
@RequiredArgsConstructor
public class SomeService { 
    // 환경에 맞는 트랜잭션 매니저 주입   
    private final PlatformTransactionManager transactionManager;

    public void remittance() {
        TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
        // 트랜잭션 시작
        TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);
        
        try {
            // 송금에 대한 비즈니스 로직 실행
            businessLogic();
            transactionManager.commit(transactionStatus);
        } catch (Exception e) {
            transactionManager.rollback(transactionStatus);
        }
    }
}
```

### DefaultTransactionDefinition

DefaultTransactionDefinition은 트랜잭션에 대한 네 가지 속성(propagation, isolationLevel, timeout, readOnly)을 담고 있다.

- propagation(트랜잭션 전파 옵션)
  - 트랜잭션의 경계에서 이미 선행되는 트랜잭션이 있거나 없는 경우 트랜잭션을 어떻게 동작시킬 것인가에 대한 설정
  - 이 속성에 대한 설명은 이후 아래에서 더 자세히 살펴보자.
- isolation(격리 수준)
  - 트랜잭션 격리 수준에 대한 설정
  - 스프링 트랜잭션의 기본값은 *`Isolation.DEFAULT`*로 현재 사용중인 DB 격리 수준의 기본값을 따른다.
  - 격리 수준에 대한 자세한 설명은 다음을 참고하자.
- timeout(제한 시간)
  - 트랜잭션의 수행시간을 제한
- readOnly(읽기 전용)
  - 읽기 전용 트랜잭션 설정
  - 해당 옵션을 명시하면 트랜잭션에서 시도되는 데이터 조작을 방지할 수 있다.

### TransactionStatus

TransactionStatus는 시작된 **트랜잭션에 대한 구분 정보**를 담고 있으며, 트랜잭션에 대한 조작이 필요할 때(커밋이나 롤백) PlatformTransactionManager 메소드의 파라미터로 전달해 사용한다.

## 선언적 트랜잭션 - @Transactional

선언적 트랜잭션이라고도 불리는 @Transactional 애너테이션은 트랜잭션을 단순하고 직관적으로 사용할 수 있게 해준다. 따라서 일반적으로 가장 많이 사용되는 방식이다.

사용 방법은 아주 간단하다.
설정 파일에 @EnableTransactionManagement를 선언한 후, 트랜잭션을 적용하고 싶은 **타입 혹은 메서드에** **@Transactional 애너테이션**을 붙여주면 된다. 스프링 부트에서는 AutoConfiguration에 의해 @EnableTransactionManagement가 자동으로 설정되므로 별도의 설정 자체도 필요 없다.

선언적 트랜잭션을 사용한 코드의 예시를 살펴보면 다음과 같다.

```java
@Service
@RequiredArgsConstructor
public class SomeService { 
    
    @Transactinal 
    public void remittance() {
        businessLogic();
    }
}
```

트랜잭션에 대한 코드를 작성하지 않고 @Transactional 애너테이션을 명시하는 것만으로 트랜잭션 기능을 사용할 수 있게 되었고 트랜잭션 코드와 비즈니스 로직이 분리되어 훨씬 명확한 코드가 됐다. 또한, @Transactional은 속성 정보를 메서드마다 다르게 설정할 수 있어 세밀한 트랜잭션 속성의 제어가 필요한 경우 아주 유연하게 사용할 수도 있다.

어떻게 애너테이션 하나만으로 이런 멋진 일들이 가능한 것일까?
비밀은 Spring의 핵심 기술 중 하나인 **AOP**에 숨겨져 있다.

![](https://blog.kakaocdn.net/dn/SXkZ7/btrMBsKka7C/RvpbiiMMCZk2L1X8l3hs3K/img.png)

AOP란 애플리케이션에서 사용되는 부가 기능들을 모듈화해 재사용할 수 있도록하는 기술이다. 예를 들면 트랜잭션이나 로깅, 실행 시간 측정 등 비즈니스 로직과 함께 수행되는 부가 기능들을 모듈화하고, 특정 시점에 끼워 넣어 재사용함으로써 중복을 제거하고 비즈니스 로직과 부가 기능을 명확히 분리하는 것이다.

Spring의 AOP는 **런타임 시점에 별도의 코드 조작없이** 자동으로 ****프록시를 생성하고 이를 이용해 Target Object에 부가 기능을 적용시킨다. 위의 예시 코드를 풀어보면 개략적으로 다음과 같은 형태가 된다.

```java
// 프록시 생성
public class TargetObjectProxy { 
    private SomeService target;
    
    public void remittance() {
        TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
        TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);
        
        try {
            // target 메서드 호출
            target.businessLogic();
            transactionManager.commit(transactionStatus);
        } catch (Exception e) {
            transactionManager.rollback(transactionStatus);
        }
    }
}
```

### Spring AOP의 프록시 자동생성 기법

Spring은 반복적인 위임 코드가 필요한 **프록시 클래스 코드의 중복 문제**를 **런타임 코드 자동생성 기법**을 활용해 풀어냈다. 런타임 코드 자동생성 기법에는 다음과 같은 두 가지 방식이 있다.

- JDK Dynamic Proxy
- CGLIB

![](https://blog.kakaocdn.net/dn/XtXyQ/btrME3PyL0W/tesQYQpWlkjCRMPHZJfqz1/img.png)

**JDK Dynamic Proxy는 인터페이스를 구현한 오브젝트에 대해** 프록시 클래스를 런타임에 동적으로 생성해준다. 타겟 오브젝트의 **인터페이스를 구현한 프록시 객체를 생성하므로 구체 클래스에 대한 타입 캐스팅이 불가능**하다. 따라서 프록시 빈을 정상적으로 사용하려면 의존 주입시 **반드시 인터페이스의 타입을 명시**해야한다.

```java
@Controller
public class SomeController { 
    @Autowired 
    private SomeServiceImpl someService; // 런타임 오류 발생 -> 구체 클래스 타입 캐스팅 불가능
}

@Service
public class SomeServiceImpl implements SomeService {
	 .....
}
```

![](https://blog.kakaocdn.net/dn/k6IM4/btrMDYA9XJs/FNKYd9Ydg9tcbRiv9vkxL0/img.png)

**CGLIB는** 타겟 클래스의 바이트 코드를 조작해 프록시 객체를 생성한다. 타겟 오브젝트를 상속해 프록시 객체를 생성하므로 JDK Dynamic Proxy와는 다르게 구체 클래스에 대해서도 프록시 생성이 가능하다. 리플랙션을 이용하는 JDK Dynamic Proxy에 비해 속도가 빠르며 구체 클래스가 AOP를 사용할 수 있다는 장점이 있다.

타겟 오브젝트를 상속해 프록시를 구현하므로 상속이 불가능한 final 클래스나 final 메서드, private 메서드는 AOP의 대상이 되지 않으며 public 메서드만 프록시를 생성할 수 있다.

## @Transactional 속성

@Transactional 애너테이션은 클래스 레벨 또는 메서드 레벨에 부착할 수 있으며 다양한 속성을 설정할 수 있어 아주 간편하게 원하는 옵션을 트랜잭션에 끼워넣을 수 있는 기능을 제공하고 있다.

스프링 트랜잭션이 제공하고 있는 속성 정보는 다음과 같다.

![](https://blog.kakaocdn.net/dn/brdvDc/btrMDnVSk0S/NIiUOio3ZgmzOmbK11ryfK/img.png)

| 속성(옵션) | 설명 |
| --- | --- |
| value | transactionManager의 별칭을 설정 |
| transactionManager | 지정된 transactionManager의 식별 값(qualifier value) 또는 빈 이름 설정 |
| label | 트랜잭션 설명 목적으로 사용되는 label을 정의 |
| propagation | 트랜잭션이 어떻게 동작할 것인가에 대한 전파 옵션을 설정 |
| isolation | 트랜잭션의 격리 수준을 설정 |
| timeout | 트랜잭션 타임아웃 설정(int) |
| timeoutString | 트랜잭션 타임아웃 설정(String) |
| readOnly | 읽기 전용(데이터 조작 방지) 트랜잭션 설정, true인 경우 활성화 |
| rollbackFor | 검사 예외(Checked Exception)발생 시 롤백을 수행할 예외 지정 속성 |
| rollbackForClassName | rollbackFor과 동일하지만 클래스명을 문자열로 지정하는 속성 |
| noRollbackFor | 비검사 예외, 즉 RuntimeException과 그 하위 예외 발생 시 롤백 처리를 수행하지 않을 예외를 지정하는 속성 |
| noRollbackForClassName | noRollbackFor와 동일하지만 클래스명을 문자열로 명시하는 속성 |

이 중 가장 많이 사용되는 몇 가지 옵션들을 더 자세히 살펴보자.

### Propagation

트랜잭션 전파 옵션(propagation)은 트랜잭션의 경계에서 이미 선행되는 트랜잭션이 있거나 없는 경우 트랜잭션을 어떻게 동작시킬 것인가에 대한 설정이다.

다양하고 복잡한 현실 세계의 문제를 해결하기 위해서는 트랜잭션을 단순하게만 사용할 수는 없다. 비즈니스 요구사항에 따라 트랜잭션도 복잡해질 수 있다. 예를들어, 선행 트랜잭션 내부에서 독립적으로 실행되어야 하는 트랜잭션이 존재한다거나, 커밋이나 롤백 시점을 어떤 트랜잭션에 의존적으로 동작하게 만들지 제어해야 하는 경우도 있다.

이처럼 트랜잭션 전파 옵션(propagation)은 트랜잭션 동작 방식을 애플리케이션단에서 조금 더 쉽게 설정하고 사용할 수 있도록하는 옵션이다.

| 속성 | 설명 |
| --- | --- |
| REQUIRED | - 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 생성 <br/> - 이미 트랜잭션이 존재하는 경우 새로운 트랜잭션을 생성하지 않고 기존 트랜잭션을 그대로 사용 |
| REQUIRES_NEW | - 항상 새로운 독립 트랜잭션을 생성 <br/> - 생성된 트랜잭션은 별도의 커밋 & 롤백 시점을 가짐 |
| MANDATORY | - 트랜잭션이 존재하지 않는 경우 예외 발생 <br/> - 이미 트랜잭션이 존재하는 경우 새로운 트랜잭션을 생성하지 않고 기존 트랜잭션을 그대로 사용 |
| NESTED | - 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 생성 <br/> - 이미 트랜잭션이 존재하는 경우 새로운 트랜잭션을 생성 <br/> - 중첩 트랜잭션 롤백 시 선행 트랜잭션에 전파되지 않음 <br/> - 선행 트랜잭션 롤백 시 중첩 트랜잭션도 함께 롤백(전파)되고 커밋될 때 중첩 트랜잭션도 함께 커밋 |
| SUPPORTS | - 트랜잭션이 존재하지 않는다면 트랜잭션을 사용하지 않음 <br/> - 이미 트랜잭션이 존재하는 경우 새로운 트랜잭션을 생성하지 않고 기존 트랜잭션을 그대로 사용 |
| NOT_SUPPORTED | - 트랜잭션을 사용하지 않음 <br/> - 트랜잭션을 무시하고 로직을 수행 |
| NEVER | - 트랜잭션이 존재하는 경우 예외 발생 <br/> - 트랜잭션을 허용하지 않는 경우에 사용 |

```java
@Service
@RequiredArgsConstructor
public class SomeService { 
    // 기본값인 REQUIRED 전파 옵션 사용
    @Transactional 
    public void remittance1() {
        businessLogic();
    }
    
    // REQUIRES_NEW 전파 옵션 사용
    @Transactional(propagation = Propagation.REQUIRES_NEW) 
    public void remittance2() {
        businessLogic();
    }
}
```

### RollbackFor & NoRollbackFor

**스프링 트랜잭션은 확인된 예외(Checked Exception)에 대해서는 롤백을 수행하지 않는다.** 스프링 트랜잭션의 **롤백 대상은 RuntimeException과 Error이며** RuntimeException을 상속한 NullPointerException이나 IllegalArgumentException과 같은 **비검사 예외(Unchecked Exception)가 롤백의 대상**이라고 보면 된다.

rollbackFor 속성은 Checked Exception 발생 시 롤백을 수행할 예외를 설정할 때 사용된다. 해당 속성 값으로는 Throwable의 하위 클래스 범위 내에 N개의 예외 클래스를 지정할 수 있다.

![](https://blog.kakaocdn.net/dn/bk9Dek/btrMBsXQH0a/A1K82b3bC9U2efa6hhN1NK/img.png)

@Transactional 애너테이션의 rollbackFor 속성의 기본 값은 RuntimeException과 Error이다. rollbackFor 속성을 명시하지 않은 경우 롤백의 대상은 RuntimeException과 Error가 된다.

```java
// rollbackFor 옵션 생략
@Transactional
public void remittance() {
    businessLogic();


// 위 메서드와 동일한 동작
@Transactional(rollbackFor = { RuntimeException.class, Error.class })
public void remittance() {
    businessLogic();
}
```

rollbackFor 옵션에는 Throwable의 하위 클래스를 모두 지정할 수 있으므로 Exception 클래스 자체를 지정할 수도 있다. 이렇게 하면 모든 예외에 대해 롤백을 수행할 수 있게 되지만, Exception은 거의 모든 예외를 포함하므로 다른 규칙들을 잡아 먹어 버린다. 필수 사항은 아니지만 rollbackFor 옵션에는 좀 더 구체적인 예외 정보를 명시해주는 것이 좋은 선택이 될 수 있다.

```java
// 모든 예외에 대해 롤백 수행
@Transactional(rollbackFor = { Exception.class })
public void remittance() {
    businessLogic();
}

// 구체적인 CheckedException을 명시해 특정 예외 발생 시 롤백을 수행
@Transactional(rollbackFor = { FileNotFoundException.class, RuntimeException.class, Error.class })
public void remittance() {
    businessLogic();
}
```

noRollbackFor 옵션은 rollbackFor 옵션과 반대로 Error나 RuntimeException 예외 발생 시 롤백을 수행하지 않을 예외를 지정하는 속성이다.

```java
// 특정 비즈니스 로직에서 발생하는 예외(RuntimeException 상속)에 대해서만 롤백 수행
@Transactional(noRollbackFor = { SomeBusinessException.class })
public void remittance() {
    businessLogic();
}
```

## @Transactional 사용시 주의사항

스프링에서는 메서드나 클래스 레벨에 @Transactional 애너테이션을 부착하는 것만으로 손쉽게 트랜잭션 기능을 사용할 수 있다. 하지만 동작 방식에 대한 이해없이 마구 사용하다보면 왜 트랜잭션이 제대로 동작을 안하는지 어디서 문제가 발생하지 찾기가 쉽지않다.(내 얘기..)

따라서 선언적 트랜잭션을 사용할 때 자주 실수할 수 있는 몇 가지 주의사항을 알아보자.

### private 메서드와 final 키워드

트랜잭션 AOP는 대상에 대한 프록시 객체를 생성하고 트랜잭션 기능을 끼워넣는 식으로 동작한다. 이때 타겟 오브젝트를 상속(extends)하거나 구현(implements)해서 프록시를 생성하는데 트랜잭션을 적용해야 할 대상 메서드의 접근 제어자가 private인 경우 재정의가 불가능하고 호출 자체도 불가능하다.

이와 같은 맥락으로 메서드에 final 키워드를 명시하면 재정의를 금지시키므로 AOP를 적용할 대상이 되지 못한다.

![](https://blog.kakaocdn.net/dn/tIVHg/btrMBsQ5HdX/Ck4I4KS9R3ycz2A3uGMybK/img.png)

![](https://blog.kakaocdn.net/dn/m8bXZ/btrMBsDvbFT/pIVw0HK3F10T9H7b1CseyK/img.png)

### 프록시 내부 호출

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class SomeService {

    private final SomeRepository someRepository;

    public void targetMethod() {
        innerMethod();
    }

    @Transactional
    public void innerMethod() {
        someRepository.save(new Some("something!"));
    }
}
```

위 코드의 트랜잭션은 정상적으로 동작하지 않는다.
앞서 살펴봤듯 트랜잭션 AOP를 적용할 때 타겟 오브젝트의 **프록시 객체를 생성하**고 트랜잭션 기능을 끼워넣는다. 따라서 클라이언트는 **프록시 빈을 호출**하고, 트랜잭션을 시작한 후 **프록시의 타겟 메서드를 호출**한다. 타겟 메서드 실행 후에는 커밋이나 롤백을 수행한다. 이 말은 클라이언트가 프록시로 감싸진 타겟 메서드를 호출했을 때 정상적으로 트랜잭션 AOP가 적용될 수 있다는 걸 나타낸다.

하지만 위 예제에서는 `targetMethod()`가 내부 메서드 `innerMethod()`를 호출하는 형태를 가지고 있다.

`targetMethod()`는 클라이언트에게 호출될 때 프록시를 통해 호출되지만 실제 트랜잭션이 필요한 `innerMethod()`는 프록시에게 호출되는 것이 아닌 자기 자신(targetMethod)에게 호출당하는 것이기 때문에 트랜잭션이 동작하지 않는 것이다.

이 문제는 innerMethod()를 분리해서 프록시로 감싸지게 만들거나 self injection을 통해 해결할 수 있다.

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class SomeService {

    private final InnerService innerService;

    public void targetMethod() {
        // 프록시로 감싸진 internalMethod 호출
        innerService.internalMethod();
    }
}

// innerMethod 분리!
@Slf4j
@Component
@RequiredArgsConstructor
public class InnerService {

    private final SomeRepository someRepository;

    @Transactional
    public void innerMethod() {
        someRepository.save(new Some("something!"));
    }
}
```

```java
// self injection을 통해 innerMethod를 프록시로 감싸기
@Slf4j
@Service
@RequiredArgsConstructor
public class SomeService {
    @Autowired
    private SomeService someService;
    private final SomeRepository someRepository;

    public void targetMethod() {
        someService.internalMethod();
    }

    @Transactional
    public void innerMethod() {
        someRepository.save(new Some("something!"));
    }
}
```

### 트랜잭션의 범위 최소화

외부 API에 의존적인 비즈니스 로직이나 수행 시간이 너무 긴 로직의 경우 트랜잭션 AOP를 사용하는 것이 비효율적일 수 있다. **@Transactional은 메서드 단위로 경계 설정되므로 트랜잭션 범위를 제어하기 어렵기 때문이다**. 트랜잭션의 범위가 커질수록 DB 커넥션이나 잠금에 대한 유지 시간이 길어져 다양한 문제를 일으킬 수 있다.

예를들어, 다음과 같은 흐름을 가진 메서드가 있다고 생각해보자.

```java
1) 처리 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
9) 알림 메일 발송 이력을 DBMS에 저장
10) 처리 완료
```

@Transactional을 이용했다면 처리 로직의 트랜잭션의 범위는 다음과 같이 잡히게 된다.

```java
1) 처리 시작
  => 데이터베이스 커넥션 생성
  => 트랜잭션 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
9) 알림 메일 발송 이력을 DBMS에 저장
  <= 트랜잭션 종료(COMMIT)
  <= 데이터베이스 커넥션 반납
10) 처리 완료
```

비즈니스 로직 흐름에 따라 메서드 수행 로직 전체가 트랜잭션 범위로 설정된다.

하지만 2~4번 작업은 단순 조회 작업이므로 트랜잭션에 포함될 필요는 없다. 또한, 8번의 **메일 발송 작업은 외부 메일 서버에서 수행하므로 불필요한 트랜잭션 범위**이고 만약 네트워크 장애 등의 이유로 외부 메일 서버가 통신 불가의 상태에 빠진다면 웹 서버뿐아니라 DB 서버 장애로까지 이어질 수 있기 때문에 트랜잭션 범위에서 제외시키는 것이 좋다.

실제 DB에 데이터 저장이 일어나는 시점은 5번과 6번, 9번이기 때문에 다음과 같이 트랜잭션 범위를 최소화하고 나눠주면 위험도를 낮출 수 있다.

```java
1) 처리 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 발생 여부확인 
4) 첨부로업로드된 파일 확인 및 저장
  => 데이터베이스 커넥션 생성(또는 커넥션 풀에서 가져오기)
  => 트랜잭션 시작
5) 사용자의 입력 내용을 DBMS에 저장 
6) 청부 파일 정보를 DBMS에 저장
  <= 트랜잭션 종료(COMMIT)
7) 저장된내용 또는 기타 정보를 DBMS에서 조회 
8) 게시물등록에 대한 알림 메일 발송
  => 트랜잭션 시작
9) 알림 메일 발송 이력을 DBMS에 저장
  <= 트랜잭션 종료(COMMIT)
  <= 데이터베이스 커넥션 종료(또는 커넥션 풀에 반납) 
10) 처리 완료
```

위의 예시처럼 트랜잭션 범위를 최소화하기 위해서는 개발자가 직접 트랜잭션의 경계를 설정해줘야 한다. 트랜잭션의 경계를 수동으로 설정하는 방법은 앞서 살펴본 **트랜잭션 서비스 추상화 계층을 사용하는 방법**과 **TransactionTemplate를 사용하는 방법**이 있다.

**트랜잭션 서비스 추상화 계층**

```java
/**
* 트랜잭션 서비스 추상화 계층을 이용한 트랜잭션 범위 최소화
*/
@Service
@RequiredArgsConstructor
public class SomeService { 
    // 환경에 맞는 트랜잭션 매니저 주입
    private final PlatformTransactionManager transactionManager;

    public void businessLogic() {
        사용자의 로그인 여부 확인();
        사용자의 글쓰기 내용의 오류 여부 확인();
        첨부로 업로드된 파일 확인 및 저장();
        doSaveTransaction();
        저장된 내용 또는 기타 정보를 DBMS에서 조회();
        게시물 등록에 대한 알림 메일 발송(); 
        saveEmailHistoryTransaction();
    }
  
    public void doSaveTransaction() {
        TransactionStatus transactionStatus = transactionManager.getTransaction(new DefaultTransactionDefinition());
          
        try {
            사용자의 입력 내용을 DBMS에 저장();
            첨부 파일 정보를 DBMS에 저장();
            transactionManager.commit(transactionStatus);
        } catch (Exception e) {
            transactionManager.rollback(transactionStatus);
        }
    }
    
    public void saveEmailHistoryTransaction() {
        TransactionStatus transactionStatus = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try {
            알림 메일 발송 이력을 DBMS에 저장(); 
            transactionManager.commit(transactionStatus);
        } catch (Exception e) {
            transactionManager.rollback(transactionStatus);
        }
    }
}
```

**TransactionTemplate**

```java
/**
* TransactionTemplate을 이용한 트랜잭션 범위 최소화
*/
@Service
@RequiredArgsConstructor
public class SomeService { 
    private final TransactionTemplate transactionTemplate;
    
    public void businessLogic() {
        사용자의 로그인 여부 확인(); 
        사용자의 글쓰기 내용의 오류 여부 확인(); 
        첨부로 업로드된 파일 확인 및 저장(); 
        doSaveTransaction();
        저장된 내용 또는 기타 정보를 DBMS에서 조회();
        게시물 등록에 대한 알림 메일 발송(); 
        saveEmailHistoryTransaction();
    }

    public void doSaveTransaction() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override 
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                사용자의 입력 내용을 DBMS에 저장();
                첨부 파일 정보를 DBMS에 저장();
            }
        });
    }

    public void saveEmailHistoryTransaction() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override 
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                알림 메일 발송 이력을 DBMS에 저장();
            }
        });
    }
}
```

TransactionTemplate의 경우 내부 execute 메서드에 try-catch문이 정의되어 있어 커밋과 롤백에 대한 설정을 따로 해줄 필요가 없다.

![](https://blog.kakaocdn.net/dn/cHuovV/btrMzDMbplt/P98PTmYOPTYmPmwKsdJNNk/img.png)

만약 특정 예외에 대한 롤백을 수행하고 싶다면 로직에 try-catch문을 추가해주기만 하면된다.

```java
/**
 * TransactionTemplate을 이용한 트랜잭션 범위 최소화
 */
@Service
@RequiredArgsConstructor
public class SomeService {
  private final TransactionTemplate transactionTemplate;

  public void businessLogic() {
    사용자의 로그인 여부 확인();
    사용자의 글쓰기 내용의 오류 여부 확인();
    첨부로 업로드된 파일 확인 및 저장();
    doSaveTransaction();
    저장된 내용 또는 기타 정보를 DBMS에서 조회();
    게시물 등록에 대한 알림 메일 발송();
    saveEmailHistoryTransaction();
  }

  public void doSaveTransaction() {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
          사용자의 입력 내용을 DBMS에 저장();
          첨부 파일 정보를 DBMS에 저장();
        } catch (Exception e) {
          status.setRollbackOnly();
        }
      }
    });
  }
}
```