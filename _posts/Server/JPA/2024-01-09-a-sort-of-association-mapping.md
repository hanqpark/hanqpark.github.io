---
title: "[JPA 기초] 05. 다양한 연관관계 매핑"

categories:
  - JPA
tags:
  - [Java]

toc: true
toc_sticky: true

date: 2024-01-09
last_modified_at: 2024-01-09
---

# 고려사항

1. 다중성
   - @ManyToOne → N:1
   - @OneToMany → 1:N
   - @OneToOne → 1:1
   - @ManyToMany → N:M
     - 실무에서 사용하면 안됨
2. 단방향 or 양방향

   Table은 FK 하나로 양쪽이 JOIN 가능해서, 사실 방향의 개념이 존재하지 않는다

   Entity는 참조용 필드가 있는 쪽으로만 참조 가능해서, 한 쪽만 참조하면 단방향, 양쪽이 서로 참조하면 양방향이다.

3. 연관관계 주인

   Entity의 양방향 관계는 A → B, B → A와 같이 참조가 2군데 존재하여, 테이블의 FK를 관리할 곳을 한 곳 지정해야 한다.

   연관관계의 주인은 FK를 관리하는 참조이고, 그 반대편은 FK에 영향을 주지 않고 단순 조회만 가능하다.

# 다중성

## 1. @ManyToOne

### 1) 단방향

<a href="https://ibb.co/khBNk0C"><img src="https://i.ibb.co/9qgQz25/2024-01-07-7-15-04.png" alt="2024-01-07-7-15-04" border="0"></a>

가장 많이 사용하는 연관관계이다.

```java
/* Member.java */

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    /* Getter & Setter */
}
```

FK인 team에 ManyToOne annotation을 삽입하였다.

```java
/* Team.java */

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    /* Getter & Setter */
}
```

단방향이므로 어떠한 연관관계 annotation도 삽입하지 않았다.

### 2) 양방향

<a href="https://ibb.co/K91QhrK"><img src="https://i.ibb.co/HrJM4Hh/2024-01-07-7-23-40.png" alt="2024-01-07-7-23-40" border="0"></a>

```java
/* Team.java */

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

	@OneToMany(mappedBy = "team")  // Member에 있는 필드 명 team을 참조
    private String name;
	private List<Member> members = new Member();

    /* Getter & Setter */
}
```

Team에서도 Member를 참조할 수 있도록 코드를 추가해주면 된다.

## 2. @OneToMany

Entity와 Table의 차이 때문에, Team Entity가 Member Table의 FK를 관리하게 되는 특이한 구조이다.

### 1) 단방향

<a href="https://ibb.co/MpRn27c"><img src="https://i.ibb.co/sQFjmqK/2024-01-08-6-13-36.png" alt="2024-01-08-6-13-36" border="0"></a>

표준 스펙이지만, 실무에서 사용하지 않기를 권장한다.

Entity에서 FK의 관리 주체가 Member에서 Team으로 넘어온다고 하더라도,

결국, Table에서는 Member가 FK의 관리 주체가 될 수 밖에 없는 구조이기 때문이다.

굳이 꼬아서 처리할 바에는 FK의 관리 주체를 제대로 설정하는 것이 좋다.

```java
/* Main.java */

Member member = new Member();
member.setName("member1");
em.persist(member);

Team team = new Team();
team.setName("teamA");
team.getMembers().add(member); // 주석 1)
em.persist(team);

tx.commit();
```

주석 1) member의 관리 주체는 Member table이기 때문에 바람직하지 않은 구성이 된다.

```java
/* Team.java */

@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

    /* Getter & Setter */
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public List<Member> getMembers() { return members; }
    public void setMembers(List<Member> members) { this.members = members; }
}
```

`@JoinColumn`을 사용하지 않으면 Join Table 방식을 사용하여, Table이 하나 더 추가된다.

```bash
Hibernate:
    create table Team_Member (
       Team_TEAM_ID bigint not null,
        members_MEMBER_ID bigint not null
    )
```

```java
/* Member.java */

@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    /* Getter & Setter */
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

}
```

`Main.java`를 실행하면 아래와 같은 JPA에서 SQL문이 실행된다.

```bash
Hibernate:
    /* insert jpa_relation.Member
        */ insert
        into
            Member
            (USERNAME, MEMBER_ID)
        values
            (?, ?)
Hibernate:
    /* insert jpa_relation.Team
        */ insert
        into
            Team
            (name, TEAM_ID)
        values
            (?, ?)
Hibernate:
    /* create one-to-many row jpa_relation.Team.members */ update
        Member
    set
        TEAM_ID=?
    where
        MEMBER_ID=?
```

`Main.java`의 `team.getMembers().add(member);`라인이 실행됐을 때, 결국 Member Table이 UPDATE되는 것을 확인할 수 있다.

<a href="https://imgbb.com/"><img src="https://i.ibb.co/djNfQdm/Untitled.png" alt="Untitled" border="0"></a>

그렇다고 실행이 안되는 것은 아니다.

결과적으로 MEMBER와 TEAM Table에 데이터가 적절하게 들어간 것을 확인할 수 있다.

하지만 Team Entity를 통해 Member Table을 UPDATE 치는 비효율성을 야기하게 되기 때문에 적절하지 않은 구성인 것이다.

실무에서는 테이블이 수십 개가 연결되어 돌아가는데, SQL 로그를 따라가다가 저렇게 구성이 되어있는 것을 보면 이슈 트래킹이 매우 힘들어진다.

> 따라서, `ManyToOne` 단방향에서 역방향으로의 추적이 필요할 때는, `OneToMany`이 아닌, `ManyToOne` 양방향으로 구조를 변경하는 것이 좋다.

### 2) 양방향

<a href="https://ibb.co/rH4XVRP"><img src="https://i.ibb.co/sqyXZdS/2024-01-08-6-44-38.png" alt="2024-01-08-6-44-38" border="0"></a>

Team, Member Entity 모두 Member Table에 Mapping이 되어 있는 괴이한 구조

따라서, Member Entity에는 읽기 전용 모드를 설정해주면 에러가 발생하지는 않는다..

```java
/* Member.java */

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

    /* Getter & Setter */

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }

    public void setName(String name) { this.name = name; }

}
```

Member Class에도 `team`을 선언해주고, `@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)` annotation을 달면 된다.

## 3. @OneToOne

FK에 DB 유니크 제약 조건 추가

- 중복을 허용하지 않는 유일한 값을 갖는 제약 조건
- `UNIQUE + NOT NULL`

### 1) 주 테이블에 FK 단방향

ManyToOne 단방향 매핑과 유사하다.

<a href="https://ibb.co/LgDCLKg"><img src="https://i.ibb.co/SmDKhSm/2024-01-08-7-22-26.png" alt="2024-01-08-7-22-26" border="0"></a>

```java
/* Member.java */

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

    /* Getter & Setter */
}
```

`locker`에 `@OneToOne` annotation 달아주면 된다.

```java
/* Locker.java */

@Entity
public class Locker {

    @Id @GeneratedValue
    private Long id;

    private String name;

    /* Getter & Setter ... */
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

### 2) 주 테이블에 FK 양방향

ManyToOne 양방향 매핑과 유사하다.

<a href="https://ibb.co/Df4BkF1"><img src="https://i.ibb.co/4RWbMLF/2024-01-08-7-29-10.png" alt="2024-01-08-7-29-10" border="0"></a>

```java
/* Locker.java */

@Entity
public class Locker {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;

    /* Getter & Setter ... */
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

```

`member`를 선언해주고,`@OneToOne(mappedBy = "locker")` annotation을 달아준다.

`locker`는 `Member.java`에 선언되어 있는 필드이다.

### 3) 대상 테이블에 FK 단방향

<a href="https://ibb.co/Kr379KJ"><img src="https://i.ibb.co/rw1MZdV/2024-01-08-7-36-45.png" alt="2024-01-08-7-36-45" border="0"></a>

Member Entity에서 Locker를 관리하겠다고 했는데, Locker Table에서 Member ID를 관리하고 있는 기이한 현상이 벌어진다.

따라서 이런 말도 안되는 구성은 JPA에서 지원하지도 않는다.

### 4) 대상 테이블에 FK 양방향

<a href="https://ibb.co/CvqtVFL"><img src="https://i.ibb.co/2Pw6cBJ/2024-01-08-7-38-56.png" alt="2024-01-08-7-38-56" border="0"></a>

2번 그림을 그대로 뒤집은 모양

Member Entity는 Member Table을, Locker Entity는 Locker Table을 관리하도록 설계하면 된다.

### 5) 생각해 볼 점

1. 하나의 Member가 여러 개의 Locker를 사용할 수 있도록 Rule이 변경되면? → 4 유리
2. 하나의 Locker를 여러 명의 Member가 사용할 수 있도록 Rule이 변경되면? → 1 유리
3. 개발자의 입장 → 1 유리
   - 보통은 Member를 조회하여 처리하는 비즈니스 로직이 대부분이기 때문.
   - 굳이 Locker를 조회하지 않아도 locker에 대한 정보가 이미 존재.

### 6) 정리

- 주 테이블에 FK
  - 객체지향 개발자 선호
  - 장점
    1. JPA 매핑이 편리
    2. 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
  - 단점
    1. 값이 없으면 FK에 null 허용
- 대상 테이블에 FK
  - DBA 선호
  - 장점
    1. OneToOne → OneToMany로 변경하여도 테이블 구조 유지
  - 단점
    1. 프록시 기능 한계 → 지연 로딩으로 설정하여도, 항상 즉시 로딩 된다.
       - 프록시에 대한 설명은 추후 포스팅 예정

## 4. @ManyToMany

### 1) 개념

<a href="https://ibb.co/QMnM8f3"><img src="https://i.ibb.co/cQtQb1S/Untitled-1.png" alt="Untitled-1" border="0"></a>

**실무에서 사용하면 안되는 구조!**

정규화된 테이블 2개로 ManyToMany 관계를 표현하는 것이 불가능하기 때문.

굳이 구현하고 싶으면 OneToMany, ManyToOne 관계로 풀어내는 것이 좋다.

<a href="https://ibb.co/tQWcV1T"><img src="https://i.ibb.co/TYFR9Nx/2024-01-08-8-13-10.png" alt="2024-01-08-8-13-10" border="0"></a>

객체는 collection을 사용하여 ManyToMany 관계를 표현할 수 있다.

- Member에 `List<Product>`, Product에 `List<Member>` 필드를 구현하면 된다.

따라서, JPA는 연결 테이블을 하나 생성하는 방식으로 ManyToMany 관계를 지원한다.

```java
/* Member.java */

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();

    /* Getter & Setter */
}
```

`@JoinTable` annotation을 통해 연결 테이블에 대한 정보를 설정해준다.

```java
/* Product.java */

@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();

    /* Getter & Setter */
}
```

참고로, ManyToMany 관계에서는 연관관계 주인을 어떤 쪽으로 잡아도 상관 없다.

[안녕하세요. 다대다 관계에서 질문 있습니다. - 인프런](https://www.inflearn.com/questions/91992/안녕하세요-다대다-관계에서-질문-있습니다)에서 내용 확인 가능하다.

### 2) 한계

<a href="https://ibb.co/4gVMf3y"><img src="https://i.ibb.co/6JWbZLh/2024-01-08-8-27-11.png" alt="2024-01-08-8-27-11" border="0"></a>

Member_Product 테이블은 Member ID와 Product ID만 관리하는데, 나중에 주문시간, 수량 등의 데이터가 추가된다면 이에 대한 관리가 어려워지게 된다.

또한 쿼리를 날릴 때에도, 연결 테이블이 숨겨져 있기 때문에 디버깅을 할 때에도 꽤나 복잡해지게 된다.

### 3) 극복

연결 테이블 Member_Product를 Entity로 승격한다.

그리고, `ManyToMany`를 `OneToMany` + `ManyToOne` 구조로 변경한다.

<a href="https://ibb.co/Njmf3y8"><img src="https://i.ibb.co/2tsR6kD/2024-01-08-8-37-37.png" alt="2024-01-08-8-37-37" border="0"></a>

```java
/* Order.java */

@Entity
public class Order {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

		/* 필드 추가가 용이해짐 */
		private int count;
    private int price;
    private LocalDateTime orderDateTime;
}
```

PK는 괜히 member_id, product_id 섞어서 사용하지 말고, `GeneratedValue`로 의미 없는 값을 집어 넣는 것이 최고다. 그래야 나중에 유연성이 생긴다.

```java
/* Member.java */

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

}
```

```java
/* Product.java */

@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "product")
    private List<Order> orders = new ArrayList<>();
}
```

# Annotation

## 1. @JoinColumn

<a href="https://ibb.co/tMNDN5C"><img src="https://i.ibb.co/qM4p4w1/2024-01-09-5-56-54.png" alt="2024-01-09-5-56-54" border="0"></a>

외래 키를 매핑할 때 사용하는 annotation이다.

주로, `@ManyToOne`, `@OneToOne`의 연관관계 주인 쪽에 함께 붙어 사용된다.

## 2. @ManyToOne

<a href="https://ibb.co/1Xc3Gjb"><img src="https://i.ibb.co/6tQ20LN/2024-01-09-5-58-09.png" alt="2024-01-09-5-58-09" border="0"></a>

## 3. @OneToMany

<a href="https://ibb.co/6BNtwjp"><img src="https://i.ibb.co/rsvp0DL/2024-01-09-5-58-32.png" alt="2024-01-09-5-58-32" border="0"></a>

속성에 `mappedBy`가 있는 것을 확인할 수 있다.

즉, 연관관계 주인이 아닌 그 반대편에 붙어서 주로 사용된다.
