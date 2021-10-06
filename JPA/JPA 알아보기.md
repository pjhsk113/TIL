# JPA 알아보기

# 1. JPA란?

JPA(Java Persistence API)는 자바 진영의 ORM 기술 표준이다. 

> ORM(Object Realational Mapping)이란?
> 

객체와 RDBMS 사이를 중간에서 매핑해주는 객체 관계 매핑 기술이다.

JPA는 애플리케이션과 JDBC 사이에서 동작하며 Java의 객체와 RDBMS 사이를 매핑하는 역할을 한다. 프로그래머가 Entity를 작성하면 JPA는 이를 분석해서 자동으로 필요한 SQL을 생성한다. 따라서 프로그래머는 조금 더 객체 지향적인 관점에서 프로그래밍을 할 수 있게된다.

![Untitled](JPA%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%206932b67bda26472b96ec965e4d628a0c/Untitled.png)

# 2. JPA가 왜 필요할까?

기존의 SQL 중심의 개발에서는 객체에 필드를 추가하면 그와 관련된 CRUD SQL을 모두 수정해야했다. 즉, 객체답게 모델링 할수록 매핑 작업이 늘어나기 때문에 생산성이 매우 떨어졌다. 

```java
// 필드 추가 전 Member 클래스
public class Member {
   private String memberId;
   private String name;
}

// SQL
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES (?, ?)
SELECT MEMBER_ID, NAME FROM MEMBER WHERE MEMBER_ID = ? 
UPDATE MEMBER SET .....
```

```java
// 필드 추가 후 Member 클래스
public class Member {
   private String memberId;
   private String name;
   private String age;
   private String tel;
}

// SQL을 추가된 필드만큼 모두 수정이 일어난다.
INSERT INTO MEMBER(MEMBER_ID, NAME, AGE, TEL) VALUES (?, ?, ?, ?)
SELECT MEMBER_ID, NAME, AGE, TEL FROM MEMBER WHERE MEMBER_ID = ? 
UPDATE MEMBER SET .....
```

또한, 객체 지향 언어의 상속이나 연관관계, 데이터 타입 등을 RDBMS로 표현하기 위해서는 조인 SQL이나 각각의 객체를 생성하는 등 상당히 복잡한 작업들이 병행되어야 한다. 이는 객체 지향과 관계형 데이터베이스의 패러다임 불일치를 나타낸다.

JPA는 이러한 패러다임 불일치를 해결하고 SQL 중심적인 개발에서 객체 중심의 개발을 가능해게 해준다. 이에 따라 얻어지는 이점도 굉장히 많아진다.

# 3. JPA의 장점

### 1. 유지보수와 생상성 향상

프로그래머는 객체의 필드를 작성하고 SQL은 JPA가 처리해준다. 따라서 필드의 변경이나 수정이 일어나도 별도의 SQL 수정 과정이 사라지게 된다. 또한, JPA에서 제공하는 메서드를 통해 CRUD를 수행하기 때문에 생산성도 향상된다.

```java
저장 - jpa.persist(member)

조회 - Member member = jpa.find(memberId)

수정 - member.setName("수정")

삭제 - jpa.remove(member)
```

### 2. 패러다임의 불일치 해결

RDBMS에서 객체의 상속 관계를 나타내고 저장하는 것은 꽤 많은 리소스가 필요하다. 

예를 들어, 다음과 같은 상속 구조가 있다.

![Untitled](JPA%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%206932b67bda26472b96ec965e4d628a0c/Untitled%201.png)

이때 Album을 조회하면 일반적인 SQL은 **ALBUM 테이블과** **ITEM 테이블**을 **Join해서 데이터를 조회**해야 한다. 간단한 예제이기 때문에 쉬워보일 수 있지만, 복잡한 애플리케이션이라면 데이터 하나를 조회하는데 엄청나게 긴 SQL문이 작성되어야 할 수도 있다.

하지만 JPA에서는 `jpa.find(Album.class, albumId);` 와 같이 간단한 프로그래밍으로 데이터 조회를 구현할 수 있다. 복잡한 Join문은 JPA가 처리해주기 때문이다. 따라서 프로그래머는 마치 컬렉션에서 데이터를 조회하고 저장하듯 객체를 다룰 수 있어 객체와 RDBMS의 패러다임 불일치 문제를 해결해준다.

### 3. 1차 캐시와 동일성 보장

JPA는 같은 트랜잭션 안에서 1차 캐시와 동일성을 보장한다. 즉, 연속 조회 시 같은 Entity를 반환하는 것을 보장하고 1차 캐시를 활용한 약간의 조회 성능 향상을 기대할 수 있다. (같은 트랜잭션에서만 보장하므로 드라마틱한 성능 향상은 기대하기 어렵다.)

```java
long memberId = 100L;
Member m1 = jpa.find(Member.class, memberId); // SQL 
Member m2 = jpa.find(Member.class, memberId); // 1차 캐시

System.out.println(m1 == m2) // true -> 동일성 보장
```

### 4. 쓰기 지연

성능 최적화 기능으로 쓰기 지연을 지원한다. 쓰기 지연이란 트랜잭션을 커밋할 때까지 SQL을 모아서 Commit 하는 순간 SQL을 한번에 보내는 기능을 말한다.

```java
transaction.begin(); // 트랜잭션 시작

em.persist(memberA);
em.persist(memberA);
em.persist(memberA);
// Insert SQL을 DB로 보내지 않고 모은다. 

transaction.commit(); // 트랜젝션 커밋 - 모아놓은 Insert SQL을 한번에 보낸다. (JDBC Batch 기능 활용)
```

SQL은 트랜잭션이 Commit 되기 전에만 수행되면 된다. 따라서 DB 커넥션을 최소화하고 Commit 시점에 모아 놓은 SQL을 Batch 처리로 DB에 보냄으로써 성능을 최적화할 수있다.

### 5. 지연 로딩과 즉시 로딩

**지연 로딩**은 객체가 실제 사용될 때 로딩된다.

```java
// 지연 로딩
Member member = memberDao.find(memberId); // SELECT * FROM Member;

Team team = member.getTeam();
String teamName = team.getName(); // SELECT * FROM Team; -> 실제 사용될 때 SQL이 로딩된다!
```

만약 Member 정보만 사용하는 로직이라면 Team까지 조회해올 필요가 없다. 우선 Member만 조회하고 나중에 Team이 필요해질 때(Team이 사용되는 시점) 다시 Team을 조회하면 된다. 이런 경우에 **지연 로딩**을 사용하면 불필요한 정보를 조회하지 않으므로 성능을 더 끌어올릴 수 있다.

반면에, **즉시 로딩**은 연관된 객체까지 미리 로딩한다.

```java
// 즉시 로딩
Member member = memberDao.find(memberId); // SELECT M.*, T.* FROM Member Join Team ...
Team team = member.getTeam();
String teamName = team.getName();
```

만약 Member와 Team 모두 자주 사용되는 로직에 지연 로딩을 사용하면 쿼리가 두 번씩 나가게 되므로 성능상의 손해를 보게된다. 이런 경우 **즉시 로딩**을 사용해 한 번에 Member와 Team의 정보를 얻어온다면 성능 상 이점을 가질 수 있게 된다.

이와 같은 기법을 통해 성능 최적화를 노려볼 수 있다.