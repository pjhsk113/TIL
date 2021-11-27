# JPA 영속성 컨텍스트

> 김영한님의 **자바 ORM 표준 JPA 프로그래밍** 강의를 들으며 공부한 내용을 정리한 글입니다.
> 

JPA의 내부 동작 방식을 이해하기 위한 중요한 개념이 있다.
바로 **영속성 컨텍스트**이다.

## 영속성 컨텍스트

영속성 컨텍스트는 JPA를 이해하는데 가장 중요한 용어로써 "엔티티를 영구 저장하는 환경"이라는 뜻이다.

우리는 아래와 같이 EntityManager를 통해 값을 영속화 시킬 수 있다. 

```java
EntityManager.persist(entity);
```

하지만 실제 EntityManager는 값을 DB에 저장하는 것이 아니라 **영속성 컨텍스트**에 저장한다. 

눈에 보이지 않는 논리적 개념으로 하나의 EntityManager는 하나의 영속성 컨텍스트를 가지고 있다.(Spring Data는 N:1)

![Untitled](JPA%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20f3730995edda44758446db29f067e79c/Untitled.png)

## 영속성 컨텍스트의 라이프 사이클

### 비영속(new/transient)

새로운 객체가 생성된 상태이다. 즉, 영속성 컨텍스트와는 관계가 없는 새로운 객체가 생성된 상태이다. 영속성 컨텍스트에 의해 관리되지 않는 객체를 비영속 상태의 객체라 부른다.

![Untitled](JPA%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20f3730995edda44758446db29f067e79c/Untitled%201.png)

```java
// 객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");
```

### 영속(managed)

생성된 객체가 영속화되어 영속성 컨텍스트에 의해 관리되는 상태를 말한다.

![Untitled](JPA%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20f3730995edda44758446db29f067e79c/Untitled%202.png)

```java
// 객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");

EntitiyManager em = entityManagerFactory.createEntityManager();
em.getTransaction().begin();

// 객체를 영속성 컨텍스트에 저장(영속)
em.persist(member);
```

### 준영속(detached)

어떤 객체가 영속성 컨텍스트에 의해 관리되다가 분리된 상태를 말한다. 준영속 상태의 객체들은 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.

```java
// member를 영속성 컨텍스트에서 분리한다.(준영속)
em.detach(member);

// 영속성 컨테스트를 초기화한다.
em.clear();

// 영속성 컨텍스트를 종료한다.
em.close();
```

### 삭제(removed)

말 그대로 영속성 컨텍스트에서 객체를 삭제한 상태를 말한다. 

```java
// member를 영속성 컨텍스트에서 삭제한다.(삭제)
em.remove(member);
```

## 영속성 컨텍스트의 장점

[JPA 알아보기](https://www.notion.so/JPA-114a026f28a648eeb1133c0f07bf6d64)에서 살펴봤던 JPA의 장점들이 사실 영속성 컨텍스트가 해준다고 볼 수 있다. 이외에 추가적인 장점으로 변경 감지를 살펴보자.

### 변경 감지(Dirty Checking)

영속성 컨텍스트는 엔티티에 수정이 일어났을 경우 자동으로 변경을 감지하는 기능이다.

영속성 컨텍스트의 내부에 스냅샷과 현재 엔티티를 비교해 변경을 감지한다.
변경이 감지된 경우 쓰기 지연 SQL 저장소에 UPDATE 쿼리가 생성되고 해당 쿼리가 DB에 날아간다.

![Untitled](JPA%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20f3730995edda44758446db29f067e79c/Untitled%203.png)

```java
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");

EntitiyManager em = entityManagerFactory.createEntityManager();
em.getTransaction().begin();

// 영속화
em.persist(member);

// 영속 엔티티 조회
Member memberA = em.find(Member.class, "member1");

// 영속 엔티티 데이터 수정(변경 감지)
memberA.setUsername("고길동");

transaction.commit(); // 커밋하면 update 쿼리가 발생
```

## 플러시(flush)란?

flush()는 영속성 컨텍스트의 쓰기 지연 SQL 저장소에 저장된 쿼리 혹은 변경 감지 내용을 데이터베이스에 동기화한다. 이때 1차 캐시에 저장되어 있는 내용은 비워지지 않는다.

다음과 같은 방법으로 영속성 컨텍스트를 플러시할 수 있다.

```java
// 직접 호출
em.flush();

// 트랜젝션 커밋 - flush가 자동으로 호출된다.
transaction.commit();

// JPQL 쿼리 - flush가 자동으로 호출된다.
query = em.createQuery("select m from Member m", Member.class);
```

JPQL 쿼리를 실행하면 플러시가 자동으로 호출된다. 
그 이유는 무엇일까? 다음 예제를 살펴보자.

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 영속화 후 JPQL 쿼리 실행
query = em.createQuery("select m from Member m", Member.class);
List<Member> mnembers = query.getResultList();
```

위 예제는 member를 영속화 시킨 후 JPQL을 통해 모든 member 목록을 조회한다.

만약 JPQL이 실행될 때 플러시를 먼저 호출하는 매커니즘이 아니었다면 위 예제는 오류가 발생했을 것이다. 

`em.persist(memberA)`와 같은 로직들은 member들을 영속성 컨텍스트에 영속화 시켰을 뿐 실제 데이터베이스에는 반영하지 않은 정보이다.

하지만 JPQL은 실제 쿼리를 DB에 날려 조회를 해오기 때문에 DB에 없는 정보를 조회하려 한다.

이러한 흐름상의 오류를 방지하기 위해 JPQL이 실행되기 전, 플러시가 자동으로 호출되어 이전 상태들을 데이터베이스와 동기화하는 것이다.

### 플러시 모드 옵션

- FlushModeType.AUTO
    - 커밋이나 쿼리를 실행할 때 플러시(기본값)
- FlushModeType.COMMIT
    - 커밋할 때만 플러시
    

```java
// 플러시 모드 옵션 적용방법
em.setFlushMode(FlushModeType.COMMIT)
```