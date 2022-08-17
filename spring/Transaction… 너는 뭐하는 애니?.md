# [Spring] Transaction… 너는 뭐하는 애니?

# 트랜잭션이란?

트랜잭션은 어떤 작업의 완정성을 보장해주는 것을 의미한다.
논리적인 작업 단위를 완벽하게 처리하거나 모두 취소하여 작업의 일부만 적용되는 현상을 방지하는 기술이다. 즉, 데이터의 정합성을 보장하기 위한 기능라고 볼 수 있다.

트랜잭션의 가장 쉬운 예로 계좌 송금 시스템을 떠올릴 수 있다.
계좌 송금 시스템의 논리적인 작업 단위는 다음과 같다.

1. A가 B에게 계좌 송금 신청
2. A가 송금 가능한 상태인지 확인(신청한 송금 금액이 계좌에 들어있는지)
3. A가 신청한 금액만큼 A의 계좌 금액 차감
4. B에게 금액 송금
5. B의 계좌에 송금된 금액이 가산

이러한 논리적 작업을 수행하는 도중 만약 4번 과정에서(B에게 금액을 송금) 장애가 발생한다면 어떻게 될까?
당연히 송금은 취소되고 A의 계좌에서 차감됐던 금액도 원상복구(롤백)가 된다.
이게 가능한 이유가 바로 **트랜잭션 덕분이다.**

이런 상황에 트랜잭션이 보장되지 않았다면 A의 계좌는 송금 금액만큼 차감됐지만 B의 계좌 금액은 가산되지 않는 불상사가 일어났을 것이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c2ad5fd-92aa-4c56-bd16-957179c8b73b/Untitled.png)

트랜잭션이란 개념은 우리가 사용하는 모든 서비스에 적용되어있다. 트랜잭션 덕분에 데이터의 정합성이 보장되고그렇기에 우리는 안심하고 서비스를 사용할 수 있게된다.

그렇다면 우리는 Java + Spring 기반의 애플리케이션 개발을 할 때 트랜잭션을 어떻게 적용할 수 있을까?

# Spring의 트랜잭션

스프링에서 트랜잭션을 사용하는 방법은 크게 두 가지다.
**트랜잭션 서비스 추상화 계층**을 이용하는 방법과 **선언적 트랜잭션**을 이용하는 방법이다.

## 트랜잭션 서비스 추상화 계층

스프링은 트랜잭션 추상화 기술을 제공하고 있다.
이를 이용하면 애플리케이션에서 트랜잭션 API를 이용하지 않고 트랜잭션 경계설정 작업이 가능해진다.

트랜잭션 추상화 계층의 상위 인터페이스인 PlatformTransactionManager를 통해 일관된 방식으로 트랜잭션을 제어하는 경계설정 작업이 가능해지는 것이다.

우리는 각자의 환경에 맞는 TransactionManager 클래스를 주입해서 트랜잭션 추상화 기술을 사용할 수 있다. JDBC를 이용하는 경우는 DataSourceTransactionManager를 주입해 사용하면 되고, JPA를 이용하는 경우JpaTransactionManager를 주입하면 된다.

이제 트랜잭션 추상화 API의 사용 방법을 간단히 살펴보자.

```java
public void remittance() {
		// 환경에 맞는 트랜잭션 매니저 생성
    PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
    
    TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
		// 트랜잭션 시작
    TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);
    
    try {
        // ....
        // 송금에 대한 작업 진행..
        // ...
        transactionManager.commit(transactionStatus);
    } catch (Exception e) {
        transactionManager.rollback(transactionStatus);
    }
}
```

### DefaultTransactionDefinition

DefaultTransactionDefinition은 트랜잭션에 대한 네 가지 속성(propagation, isolationLevel, timeout, readOnly)을 담고 있다. 이 속성에 대한 설명은 이후 아래에서 더 자세히 살펴본다.

### TransactionStatus

TransactionStatus는 시작된 트랜잭션에 대한 구분 정보를 담고 있으며, 트랜잭션에 대한 조작이 필요할 때(커밋이나 롤백) PlatformTransactionManager 메소드의 파라미터로 전달해 사용한다.

## 선언적 트랜잭션 - @Transactional

선언적 트랜잭션이라고도 불리는 @Transactional 애너테이션은 트랜잭션을 단순하고 직관적으로 사용할 수 있게 해준다. 따라서 일반적으로 가장 많이 사용되는 방식이다.

사용 방법은 아주 간단하다.
설정 파일에 @EnableTransactionManagement를 선언한 후, 트랜잭션을 적용하고 싶은 **타입 혹은 메서드에** **@Transactional 애너테이션**을 붙여주면 된다. 스프링 부트에서는 AutoConfiguration에 의해 @EnableTransactionManagement가 자동으로 설정되므로 별도의 설정 자체도 필요 없다.

```java
@Transactional
public void remittance() {
      // ....
      // 송금에 대한 작업 진행..
      // ...
}
```

@Transactional은 속성 정보를 메서드마다 다르게 설정할 수 있어 세밀한 트랜잭션 속성의 제어가 필요한 경우 아주 유연하고 유용하게 사용할 수 있다.

### @Transactional 동작 원리

- @Transactional 옵션
- @Transactional 사용법 및 주의사항