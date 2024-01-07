---
title: "[JPA 기초] 03. 엔티티 매핑"

categories:
  - JPA
tags:
  - [Java]

toc: true
toc_sticky: true

date: 2024-01-07
last_modified_at: 2024-01-07
---

# Entity Mapping

Java Object와 DB Table을 서로 연결한다고 이해하면 된다.

```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;
    private Integer age;

    /*  Getter, Setter...  */
}
```

| **Java Entity**     | **DB Table** |
| ------------------- | ------------ |
| Member              | Table        |
| username, age, etc. | column       |
| id                  | pk           |

---

1. **객체와 테이블 매핑**: `@Entity`, `@Table`
   - Member 객체와 Member 테이블을 매핑한다.
2. **필드와 컬럼 매핑**: `@Column`, `@Temporal`, `@Enumerated`, `@Lob`, `@Transient`
   - Member 객체의 `private String name` 항목과 Member 테이블의 `name` 컬럼을 매핑한다.
3. **기본 키 매핑**: `@Id`, `@GeneratedValue`
   - Member 객체의 `private Long id` 항목과 Member 테이블의 `pk`를 매핑한다.
4. **연관관계 매핑**: `@ManyToOne`, `@OneToMany`, `@JoinColumn`
   - 추후 포스팅 예정

## 1. 객체와 테이블 매핑

### 1) `@Entity`

- `@Entity`가 붙은 class는 JPA가 관리, Table과 매핑한다.
- 기본 생성자가 필수이다.

  ```java
  @Entity
  public class Member {
      @Id
      private Long id;

      @Column(name = "name")
      private String username;

      /* 기본 생성자 */
      public Member { }
  }
  ```

- `final`, `enum`, `interface`, `inner` class 사용 금지
- `name` 속성
  - JPA에서 사용할 Entity 이름을 지정한다.
  - 기본 값은 class 이름을 그대로 사용한다.
  - 가급적 기본 값을 사용하는 것이 좋다.

### 2) `@Table`

<a href="https://ibb.co/Wxyzkc0"><img src="https://i.ibb.co/98T4bqh/2024-01-04-9-39-21.png" alt="2024-01-04-9-39-21" border="0"></a>

## 2. 필드와 컬럼 매핑

```java
package hellojpa;

import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;

@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)    // EnumType.ORDINAL -> ENUM 순서가 바뀌면 뒤죽박죽 섞이는 대참사 발생
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    private LocalDate testLocalDate;
    private LocalDateTime testLocalDateTime;

    @Lob
    private String description;

    @Transient
    private int temp;

    /*  Getter & Setter  */
}
```

### 1) `@Column`: column 매핑

<a href="https://ibb.co/D42tJG1"><img src="https://i.ibb.co/PxHNLTr/2024-01-04-9-45-08.png" alt="2024-01-04-9-45-08" border="0"></a>

- `unique`
  - ID가 랜덤으로 할당되어 운영에서는 이슈 트래킹이 불가하므로 잘 사용하지 않는다.
  - `@Table(uniqueConstraints = )` 방식을 많이 사용한다.

### 2) `@Enumerated`: enum 타입 매핑

<a href="https://ibb.co/TPdK17L"><img src="https://i.ibb.co/mhgv028/2024-01-04-9-46-54.png" alt="2024-01-04-9-46-54" border="0"></a>

- `EnumType.ORDINAL` 사용하면 안된다!!

  1. 처음에 Member 객체의 RoleType enum을 아래와 같이 저장하고,

     ```java
     public enum RoleType { GUEST, ADMIN }
     ```

  2. DB에 RoleType을 저장하면 `GUEST=1`, `ADMIN=2`로 저장된다.
  3. 그런데, RoleType의 수정이 필요해 다음과 같이 변경한 후,

     ```java
     public enum RoleType { GUEST, USER, ADMIN }
     ```

  4. DB에 RoleType을 저장하면 `GUEST=1`, `USER=2`, `ADMIN=3` 으로 저장된다.
  5. 기존에 DB에 저장 된 RoleType=2는 ADMIN을 의미했는데, RoleType의 수정이 발생한 후로는 USER를 의미하게 되는 불상사 발생
  6. RoleType의 변경이 발생한다고 해도, JPA에서 이를 자동으로 인식하여 DB UPDATE를 해주는 기능은 없으니 `EnumType.ORDINAL`은 사용하지 말 것!

### 3) `@Temporal`: 날짜 타입 매핑

<a href="https://ibb.co/M62vhTv"><img src="https://i.ibb.co/16vgnVg/2024-01-07-11-04-07.png" alt="2024-01-07-11-04-07" border="0"></a>

- `java.util.Date;` 타입일 때 사용한다.
- `java.time.LocalDate, LocalDateTime`을 사용하면 생략 가능하다.

### 4) `@Lob`: BLOB, CLOB 매핑

[[Oracle] BLOB, CLOB 차이점](https://javaoop.tistory.com/73)

- 텍스트, 그래픽, 이미지, 비디오, 사운드 등 구조화 되지 않은 대형 데이터를 저장하는 목적.
- `CLOB`: 필드 타입이 문자일 때 사용한다.
- `BLOB`: 그 외 형태일 때 사용한다.

### 5) `@Transient`: 매핑 무시

## 3. 기본 키 매핑

- DB Table의 `pk`를 Entity 클래스의 `field`와 매핑하는 방법을 나타낸다.
- Entity 클래스의 식별자를 정의하고, DB Table과의 연관성을 확립하는데 중요한 역할을 한다.
- 매핑 방법
  - 직접 할당 → `@Id`
  - 자동 생성 → `@GeneratedValue`
    1. IDENTITY 전략
    2. SEQUENCE 전략
    3. TABLE 전략
    4. AUTO 전략

### 1) **IDENTITY 전략**

```java
/* Member class -> IDENTITY 전략 */
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;  // 기본 키

    private String name;
    private int age;

    // getters, setters, constructors 등...
}
```

- 기본 키 생성을 **DB에 위임**하는 방식이다.
  - DB에 위임하였기 때문에, ID에 값을 넣어주면 안된다.
- 대부분의 RDBMS에서 사용된다.
  - MySQL: AUTO\_ INCREMENT

```java
/* main 함수 */
try {
    Member member1 = new Member();
    member1.setID("1L")
    member1.setUsername("A");

    /*  @GeneratedValue(strategy=GenerationType.IDENTITY) 일 경우,
        JPA가 INSERT SQL 날리는 분기  */
    em.persist(member1);

    /*  일반적으로, JPA가 INSERT SQL 날리는 분기  */
    tx.commit();

} catch (Exception e) {
    tx.rollback();

} finally {
    /* Entity Manager 종료 */
    em.close();
}
```

- JPA는 보통 `tx.commit` 시점에 INSERT SQL을 실행하지만, IDENTITY 전략은 DB에 INSERT SQL를 날리기 전에는 ID를 알 수 없는 문제가 발생한다.
- 따라서, IDENTITY 전략일 경우, JPA는 `em.persist` 시점에 INSERT SQL을 실행하고, 이후에 `object.getID()`를 통해 ID 호출 가능하도록 문제를 해결하였다.

### 2) **SEQUENCE 전략**

```java
/* Member class -> Sequence 전략 */
@Entity
@SequenceGenerator(
    name = "member_seq_generator",
    sequenceName = "member_seq"
)
public class Member {

    @Id
    @GeneratedValue(
        strategy = GenerationType.SEQUENCE,
        generator = "member_seq_generator"
    )
    private Long id;

    /* Constructor, Getter & Setter ... */
}
```

<a href="https://ibb.co/X7YTs9M"><img src="https://i.ibb.co/2vZBtwH/2023-12-25-12-08-59.png" alt="2023-12-25-12-08-59" border="0"></a>

- 개념
  - DB `Sequence` 객체를 사용하여 pk를 생성하는 전략이다.
  - `Sequence`는 특정 순서에 따라 일련번호를 생성하는 객체이다.
    - Oracle Sequence
    - Object의 ID를 관리하기 위한 전용 DB 같은 개념이다.
- 작동 원리

  1. `setID()`는 사용하지 않는다.
  2. `em.persist()` 단계에서 DB Sequence에서 `getID()` 진행한다.

     ```bash
     # em.persist() 진행 후 log
     Hibernate:
         call next value for MEM_SEQ
     member.getId() = 2
     ```

  3. 영속성 컨텍스트에 쌓아둔 후 `tx.commit()`을 통해 INSERT SQL 날린다.

- 문제점
  - DB에 여러 번 접근하면서 네트워크를 여러 번 타게 되어 성능 저하 이슈가 발생할 수 있다.
    1. `em.persist()` : ID를 얻기 위해 DB에 접근 1번
    2. `tx.commit()` : INSERT SQL을 날리기 위해 DB에 접근 1번
- 해결 방안
  - `allocationSize = 50`으로 설정하여 ID를 얻기 위해 DB에 접근하는 횟수를 줄인다.
    1. DB Sequence에는 51을 저장, 나머지 50회는 미리 AP단 메모리에 저장한다.
    2. ID를 얻기 위해 DB가 아닌 AP단 메모리에 접근하여 ID Increment를 실행한다.
    3. 50회를 다 채우면 새로이 DB Sequence에 접근하여 101을 저장, 메모리에 추가로 50회를 저장한다.
  - 여러 WAS가 있어도 동시성 이슈가 없다.

### 3) **TABLE 전략**

```sql
/* 키 생성 전용 테이블 생성 */
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```

```java
/* Member class -> TABLE 전략 */
@Entity
@TableGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "MEMBER_SEQ",
		allocationSize = 1
)
public class Member {
    @Id
    @GeneratedValue(
        strategy = GenerationType.TABLE,
        generator = "MEMBER_SEQ_GENERATOR"
    )
    private Long id;

    /* Constructor, Getter & Setter ... */
}
```

<a href="https://ibb.co/qdG4xYV"><img src="https://i.ibb.co/R9nM7Td/2023-12-25-1-12-29.png" alt="2023-12-25-1-12-29" border="0"></a>

- 키 생성 전용 테이블을 하나 만들어서 Sequence 전략을 흉내내는 전략
- Sequence 전략은 Oracle 등 일부 DB에서만 사용 가능하지만, TABLE 전략은 모든 DB에 적용 가능하다.
- 하지만, 성능이 구리다..

### 4) 정리: 권장하는 식별자 전략

- 기본 키 제약 조건
  1. NOT NULL
  2. 유일성
  3. **불변성** → 이 조건 만족하는 것이 어렵다.
- 미래까지 기본 키 제약 조건을 만족하는 자연키는 찾기 어렵다.
  - 국가에서 갑자기 주민등록번호를 사용하지 말라고 했던 사례가 있다..
- **Long형 + 대체 키 + 키 생성전략**을 사용하자!
  1. long형
     - 범위가 약 -9경부터 9경에 이른다.
     - ID 값이 경에 이르기는 쉽지 않으니 long형을 사용하는 것이 적절하다.
  2. 대체 키
     - DB의 pk를 대체할 수 있는 entity 전용 키를 생성하는 것.
     - 외부에 노출되지 않고, DB 내부적으로만 사용되어 보안 상 이점이 있다.
  3. 키 생성전략
     - IDENTITY 전략의 AUTO_INCREMENT 방식
     - SEQUENCE 전략

## 4. DB Schema 자동 생성

```xml
<!-- persistence.xml의 일부 -->
<properties>
	<property name="hibernate.hbm2ddl.auto" value="create" />
</properties>
```

- 개요
  - DDL(CREATE, ALTER, DROP 등)을 AP 실행 시점에 자동 생성한다.
  - DB 방언을 활용해서 DB에 맞는 적절한 DDL 생성한다.
  - DDL은 개발 서버에서만 사용하고, 운영 서버에서는 사용하지 않거나 적절히 다듬은 후 사용해야 한다.
- 속성
  - `create`: DROP → CREATE
  - `create-drop`: DROP → CREATE → DROP
  - `update`
    - 변경 분 반영
    - 삭제 불가능
    - 운영DB 사용 금지
  - `validate`: Entity와 Table이 정상 매핑 되었는 지 확인.
  - `none`: 사용 안함
- 장점
  - 개발할 때, DB console 가서 schema 조작하는 것이 귀찮다.
    - DDL문을 JPA로 해결 가능하여 편리함.
- 주의
  - 혼자 사용하는 프로젝트면 사용해도 무관하다.
  - 남들이랑 사용할 때 → 웬만하면 사용하지 않는 것이 좋음
    - 개발 초기: create, update
    - 테스트 서버: update, validate
    - 운영 서버: **사용하지 마라**
