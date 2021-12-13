# JPA Entity 매핑

> 김영한님의 **자바 ORM 표준 JPA 프로그래밍** 강의를 들으며 공부한 내용을 정리한 글입니다.
> 

## 객체와 테이블 매핑

JPA가 관리하는 클래스라는 것을 명시하기 위해 **@Entity** 애너테이션을 사용한다. 
따라서 JPA를 사용해 어떤 테이블과 매핑할 클래스는 **@Entity**를 필수적으로 명시해줘야 한다.

```java
@Entity
public class Member {
    .....
    .....
}
```

Entity 클래스를 만들 때 몇가지 주의할 점이 있는데, 주의 사항은 다음과 같다.

### 주의사항

- public 혹은 protected 레벨의 파라미터가 없는 기본 생성자는 필수이다.
- final 클래스, enum, interface, inner 클래스는 Entity로 사용할 수 없다.
- 값을 담는 필드에 final을 사용하면 안 된다.

### @Entity의 속성

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | JPA에서 사용할 Entity의 이름을 지정 | 클래스의 이름을 그대로 사용 |

### @Table의 속성

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | 매핑할 테이블 이름 | 엔티티 클래스 이름을 그대로 사용 |
| catalog | 데이터베이스 catalog 매핑 |  |
| schema | 데이터베이스 schema 매핑 |  |
| uniqueConstraints | DDL 생성 시에 유니크 제약 조건 생성 |  |
| indexes | 데이터베이스 index를 지정 |  |

## 데이터베이스 스키마 자동 생성

JPA를 활용해 데이터베이스와 매핑할 때, 스키마 생성에 대한 속성을 줄 수 있다.
이러한 속성은 일반적으로 application.properties 혹은 application.yml에 명시한다.

```java
// application.yml
jpa:
    hibernate:
      ddl-auto: update

// application.properties
jpa.hibernate.ddl-auto: update
```

### 스키마 자동 생성 속성

| 속성 | 기능 |
| --- | --- |
| create | 기존 테이블 삭제 후 다시 생성(drop + create) |
| create-drop | create와 같지만 종료시점에 테이블을 drop 시킴 |
| update | 변경된 부분만 반영 |
| validate | 엔티티와 테이블이 정상 매핑되었는지 확인 |
| none | 사용하지 않음 |

### 주의사항

- 운영 서버에는 절대 create, create-drop, update를 사용하면 안된다.
- 스테이징과 운영 서버는 validate 또는 none 속성을 사용한다.
- 테스트 서버에서는 update 또는 validate 속성을 사용한다.

## 기본 키 매핑

객체와 테이블을 매핑 했다면, 객체와 테이블의 기본 키에 대한 매핑이 필요하다.

기본 키 매핑에 사용되는 애너테이션은 다음과 같다.

- @Id
- @GeneratedValue

```java
@Entity
public class Member {
    @Id
    @GenratedValue(strategy = GenerationType.AUTO)
    private Long id;
}
```

@Id만 명시할 경우 Id 값을 직접 할당하겠다는 의미이며, 일반적으로는 @GeneratedValue(자동생성) 전략을 선택해 Id를 생성한다.

### @GeneratedValue 전략

**IDENTITY 전략**

기본 키 생성을 데이터베이스에 위임한다. 
MySQL의 AUTO_INCREMENT와 비슷한 기능이라고 생각하면 된다.

AUTO_INCREAMENT는 데이터베이스에 insert 쿼리가 실행 된 후 ID 값을 알 수 있기 때문에
IDENTITY 전략은 persist() 시점, 영속화되는 시점에 insert 쿼리를 실행하고 식별자를 조회한다.

```java
@Entity
public class Member {
    @Id
    @GenratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

**SEQUENCE 전략**

유일한 값을 순서대로 생성하는 데이터베이스 시퀀스 오브젝트를 사용한다.
데이터베이스에서 시퀀스 기능을 제공해야만 사용 가능한 DB 종속적인 전략이다.

```java
@Entity
@SequenceGenerator(name = "MEMBER_SEQ_GENERATOR",
                   sequenceName = "MEMBER_SEQ" // 매핑할 데이터베이스 시퀀스 이름
                   initialValue = 1, allocationSize = 1)
public class Member {
    @Id
    @GenratedValue(strategy = GenerationType.SEQUENCE,
                   generator = "MEMBER_SEQ_GENBERATOR")
    private Long id;
}
```

- @SequenceGenerator 속성
    
    
    | 속성 | 기능 | 기본 값 |
    | --- | --- | --- |
    | name | 시퀀스  이름 | 필수 |
    | sequenceName | 데이터베이스에 등록되어 있는 시퀀스 이름  | hibername_sequence |
    | initialValue | 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다. DDL 생성 시에만 사용됨 | 1 |
    | allocationSize | 시퀀스 한 번 호출에 증가하는 크기.
    성능 최적화에 사용되며, DB 시퀀스 값이 하나씩 증가하도록 설정되어 있다면 반드시 1로 설정해야 한다. | 50 |
    | catalog | 데이터베이스 catalog 이름 |  |
    | schema | 데이터베이스 schema 이름 |  |
    

**TABLE 전략**

키 생성 전용 테이블을 만들어서 데이터베이스 시퀀스 기능을 흉내내는 전략이다.
시퀀스 기능을 제공하지 않는 데이터베이스에서도 적용 가능하지만 성능이 느리다는 단점이 있다.

```sql
// 키 생성 전용 테이블을 생성
CREATE TABLE MY_SEQUENCE {
	sequence_name varchar(255) not null,
  next_val bigint,
  primary key ( sequence_name )
}
```

```java
@Entity
@TableGenerator(name = "MEMBER_SEQ_GENERATOR",
                table = "MY_SQUENCE"
                pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
public class Member {
    @Id
    @GenratedValue(strategy = GenerationType.TABLE,
                   generator = "MEMBER_SEQ_GENBERATOR")
    private Long id;
}
```

- @TableGenerator 속성
    
    
    | 속성 | 기능 | 기본 값 |
    | --- | --- | --- |
    | name | 식별자 생성기 이름 | 필수 |
    | table | 키 생성 테이블 명  | hibername_sequence |
    | pkColumnName | 시퀀스 컬럼 명 | sequence_name |
    | valueColumnNa | 시퀀스 값 컬럼명 | next_val |
    | pkColumnValue | 키로 사용할 값 이름 | 엔티티 이름 |
    | initialValue | 초기 값, 마지막으로 생성된 값이 기준 | 0 |
    | allocationSize | 시퀀스 한 번 호출에 증가하는 크기.
    성능 최적화에 사용되며, DB 시퀀스 값이 하나씩 증가하도록 설정되어 있다면 반드시 1로 설정해야 한다. | 50 |
    | catalog | 데이터베이스 catalog 이름 |  |
    | schema | 데이터베이스 schema 이름 |  |
    | uniqueConstraints |  |  |
    |  |  |  |
    

**AUTO 전략**

@GeneratedValue의 기본 값으로 데이터베이스 별 방언에 따라 ID 값을 자동 지정한다.

```java
@Entity
public class Member {
    @Id
    @GenratedValue(strategy = GenerationType.AUTO)
    private Long id;
}

```

## 필드와 컬럼 매핑

이제 객체의 필드와 테이블의 컬럼을 매핑할 때 사용하는 매핑 애너테이션들과 그 속성을 알아보자.

먼저, 컬럼 매핑에 사용되는 애너테이션은 다음과 같다.

- @Column - 컬럼 매핑
- @Temporal - 날짜 타입 매핑
- @Enumerated - enum 타입 매핑
- @Lob - BLOB, CLOB 매핑
- @Transient - 특정 필드를 컬러에 매핑하지 않음(매핑 무시)

```java
@Entity
public class Member {
    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;
}
```

### @Column의 속성

@Column은 필드와 테이블 컬럼을 매핑할 때 사용한다.  \

DDL이 붙은 속성들은 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 주지 않는다.
@Column 애너테이션은 아래와 같은 속성들을 가지고 있다. 용된다.

| 속성 | 기능 | 기본 값 |
| --- | --- | --- |
| name | 필드와 매핑할 컬럼 명 | 객체 필드의 이름 |
| insertable | 등록 가능 여부 | true |
| updatable | 변경 가능 여부 | true |
| nullable(DDL) | null 허용 여부 |  |
| unique(DDL) | @Table의 uniqueConstraints와 같지만 하나의 컬럼에 간단히 유니크 조건을 걸 때 사용한다. |  |
| columnDefinition(DDL) | DB 컬럼 정보를 직접 줄수 있다. |  |
| length(DDL) | 문자 길이 제약 조건 | 255 |
| precision | BIgDecimal 혹은 BigInteger 처럼 큰 숫자나 정밀한 소수를 다루어야 할 때 사용한다. 
소숫점을 포함한 전체 자릿수를 나타냄 | persision = 19 |
| scale(DDL) | BIgDecimal 혹은 BigInteger 처럼 큰 숫자나 정밀한 소수를 다루어야 할 때 사용한다. 
소수의 자릿수를 나타냄 | scale = 2 |

### @Enumerated 속성

@ㄷEnumerated 속성은 enum 타입을 매핑할 때 사용한다.

절대 ORDINAL 타입을 사용하면 enum의 순서인 Integer 타입의 값이 들어가기 때문에 프로그램이 깨지기 쉽다.