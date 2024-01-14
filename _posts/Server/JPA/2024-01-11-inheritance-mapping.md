---
title: "[JPA 기초] 06. 상속관계 매핑"

categories:
  - JPA
tags:
  - [Java]

toc: true
toc_sticky: true

date: 2024-01-11
last_modified_at: 2024-01-11
---

# 상속관계 매핑

이전 시간에 짰던 객체들이 왜 자꾸 테이블로 등록되나 했는데, h2 db 설정이 문제가 아니라 이전 시간에 짰던 객체들에 @Entity가 붙어있었기 때문.. JPA는 모든 package의 Entity를 등록하는 것에 주의하도록 하자

## 1. 개요

<a href="https://ibb.co/s95RkR7"><img src="https://i.ibb.co/zV8xjxd/2024-01-10-6-08-58.png" alt="2024-01-10-6-08-58" border="0"></a>

RDBMS는 상속 관계가 존재하지 않지만, 슈퍼타입 ~ 서브타입 모델링 기법은 객체 상속과 유사하다.

이러한 점을 활용하여, 객체의 상속 구조와 DB의 슈퍼타입 ~ 서브타입 관계를 매핑하는 것을 **상속관계 매핑**이라고 한다.

<a href="https://ibb.co/tbrkfNH"><img src="https://i.ibb.co/WcJq9dH/2024-01-10-6-31-38.png" alt="2024-01-10-6-31-38" border="0"></a>

객체의 관계가 다음과 같을 때, 테이블 관계를 구현하는 방법은 3가지가 존재한다.

1. 조인 전략
2. 단일 테이블 전략
3. 구현 클래스마다 테이블 전략

## 2. Annotation

### 1) `@Inheritance(strategy=InheritanceType.XXX)`

1. `JOINED` : 조인 전략

   <a href="https://ibb.co/prdGjXf"><img src="https://i.ibb.co/Hpq1Tgt/2024-01-10-6-32-18.png" alt="2024-01-10-6-32-18" border="0"></a>

   - 부모, 자식 모두 별도로 존재
   - 부모 ID로 JOIN

2. `SINGLE_TABLE` : 단일 테이블 전략

   <a href="https://imgbb.com/"><img src="https://i.ibb.co/x3VDy1f/2024-01-10-6-32-49.png" alt="2024-01-10-6-32-49" border="0"></a>

   - 부모만 존재
   - 모든 자식의 컬럼을 부모에 다 때려박음
   - 어떠한 노티도 주지 않으면, JPA는 알아서 단일 테이블 전략을 선택한다.

     ```bash
     Hibernate:

         create table Item (
            DTYPE varchar(31) not null,
             id bigint not null,
             name varchar(255),
             price integer not null,
             actor varchar(255),
             director varchar(255),
             author varchar(255),
             isbn varchar(255),
             artist varchar(255),
             primary key (id)
         )
     ```

3. `TABLE_PER_CLASS` : 구현 클래스마다 테이블 전략

   <a href="https://ibb.co/k57Ttww"><img src="https://i.ibb.co/vVFWSTT/2024-01-10-6-33-28.png" alt="2024-01-10-6-33-28" border="0"></a>

   - 자식들만 존재
   - 부모의 컬럼을 각 자식마다 때려박음

### 2) `@DiscriminatorColumn(name=”DTYPE”)`

DTYPE column을 생성해주는 annotation.

주로 부모 클래스에 붙여준다.

조인 전략에서, 데이터의 출처를 확인하기 위해 DTYPE column을 만들어 준다.

<a href="https://ibb.co/mJfV7p9"><img src="https://i.ibb.co/XXcrTH7/2024-01-10-7-36-25.png" alt="2024-01-10-7-36-25" border="0"></a>

### 3) `@DiscriminatorValue(“XXX”)`

DTYPE의 value를 원하는 값으로 넣을 수 있게 설정해주는 annotation.

주로 자식 클래스에 붙여준다.

<a href="https://ibb.co/7gGy5Wr"><img src="https://i.ibb.co/f8C05pr/2024-01-10-7-39-04.png" alt="2024-01-10-7-39-04" border="0"></a>

## 3. 전략

### 1) 조인 전략

INSERT의 경우 다음과 같다.

```java
/* Item.java */

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

}
```

```java
/* Movie.java */

@Entity
public class Movie extends Item{

    private String director;
    private String actor;

}
```

```java
/* Main.java */

Movie movie = new Movie();
movie.setDirector("Bong");
movie.setActor("Song");
movie.setName("Parasite");
movie.setPrice(10000);

em.persist(movie);

tx.commit();
```

`Main.java`를 실행하면 ITEM, MOVIE 테이블이 생성된 후, 두 번의 INSERT 쿼리가 나가면서 테이블에 데이터가 저장된다.

```bash
Hibernate:

    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
Hibernate:

    create table Movie (
       actor varchar(255),
        director varchar(255),
        id bigint not null,
        primary key (id)
    )

Hibernate:
    /* insert jpa_relation.Movie
        */ insert
        into
            Item
            (name, price, id)
        values
            (?, ?, ?)
Hibernate:
    /* insert jpa_relation.Movie
        */ insert
        into
            Movie
            (actor, director, id)
        values
            (?, ?, ?)
```

<a href="https://ibb.co/0hVwWTJ"><img src="https://i.ibb.co/NZn4k5Y/2024-01-10-7-24-12.png" alt="2024-01-10-7-24-12" border="0"></a>

SELECT의 경우는 다음과 같다.

```java
/* Main.java */

Movie movie = new Movie();
movie.setDirector("Bong");
movie.setActor("Song");
movie.setName("Parasite");
movie.setPrice(10000);

em.persist(movie);

em.flush();
em.clear();

Movie findMovie = em.find(Movie.class, movie.getId());
System.out.println("findMovie = " + findMovie);

tx.commit();
```

`Main.java`를 실행하면, `em.find`를 할 때 테이블 조인이 이루어지는 것을 확인할 수 있다.

```bash
Hibernate:
    select
        movie0_.id as id1_2_0_,
        movie0_1_.name as name2_2_0_,
        movie0_1_.price as price3_2_0_,
        movie0_.actor as actor1_5_0_,
        movie0_.director as director2_5_0_
    from
        Movie movie0_
    inner join
        Item movie0_1_
            on movie0_.id=movie0_1_.id
    where
        movie0_.id=?
findMovie = jpa_relation.Movie@2938127d
```

### 2) 단일 테이블 전략

```java
/* Item.java */

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
// @DiscriminatorColumn
public class abstract Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

    /* Getter & Setter */

}
```

`@DiscriminatorColumn`이 없어도 자동으로 DTYPE column이 생긴다. 테이블이 합쳐지기 때문에 데이터의 출처를 확인하기 위해 어쩔 수 없기 때문.

`Main.java`를 실행하면 ITEM 테이블만 생성된 후, 한 번의 INSERT문으로 데이터가 저장되는 것을 확인할 수 있다.

```bash
Hibernate:

    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        author varchar(255),
        isbn varchar(255),
        artist varchar(255),
        primary key (id)
    )

Hibernate:
    /* insert jpa_relation.Movie
        */ insert
        into
            Item
            (name, price, actor, director, DTYPE, id)
        values
            (?, ?, ?, ?, 'M', ?)
```

<a href="https://ibb.co/8Nk5XGc"><img src="https://i.ibb.co/pbqznNy/2024-01-10-7-47-30.png" alt="2024-01-10-7-47-30" border="0"></a>

데이터 조회를 할 때도, 테이블 조인 없이 간단한 SELECT 쿼리를 날리는 것을 확인할 수 있다.

```bash
Hibernate:
select
        movie0_.id as id2_0_0_,
        movie0_.name as name3_0_0_,
        movie0_.price as price4_0_0_,
        movie0_.actor as actor5_0_0_,
        movie0_.director as director6_0_0_
    from
        Item movie0_
    where
        movie0_.id=?
        and movie0_.DTYPE='M'
```

### 3) 구현 클래스마다 테이블 전략

```java
/* Item.java */

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
// @DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

    /* Getter & Setter */

}
```

```bash
Hibernate:

    create table Movie (
       id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )

Hibernate:
    /* insert jpa_relation.Movie
        */ insert
        into
            Movie
            (name, price, actor, director, id)
        values
            (?, ?, ?, ?, ?)
```

<a href="https://imgbb.com/"><img src="https://i.ibb.co/f9kfzQt/2024-01-10-8-10-10.png" alt="2024-01-10-8-10-10" border="0"></a>

```bash
Hibernate:
    select
        movie0_.id as id1_2_0_,
        movie0_.name as name2_2_0_,
        movie0_.price as price3_2_0_,
        movie0_.actor as actor1_5_0_,
        movie0_.director as director2_5_0_
    from
        Movie movie0_
    where
        movie0_.id=?
```

## 4. 장단점

조인 전략을 기본으로 생각하되, 비즈니스 로직이 정말 단순하면 DBA와 논의하여 단일 테이블 전략을 가져가는 것이 좋다.

### 1) 조인 전략

**모든 전략 중, 가장 기본이 되는 전략이다.**

- 장점

  1. 테이블 정규화 → 저장공간 효율화
  2. 외래 키 참조 무결성 제약조건 활용 가능

     → 다른 외부 테이블 (ORDER)가 ITEM을 참조해야 할 때, 자식 테이블인 ALBUM, MOVIE, BOOK 등을 굳이 참조하지 않고서도 데이터 조회가 가능하다.

- 단점
  1. 조회 시 조인을 많이 사용 → 성능 저하, 쿼리 복잡
  2. 저장 시 INSERT SQL 2회 호출

### 2) 단일 테이블 전략

- 장점
  1. 조인이 없음 → 조회 성능이 빠름, 조회 쿼리가 단순하다.
- 단점
  1. 자식 엔티티가 매핑한 컬럼은 모두 `NULL` 허용 → 데이터 무결성 측면에서 안좋다.
  2. 단일 테이블에 모든 것을 저장하므로 테이블이 커진다.
  3. 상황에 따라 조회 성능이 오히려 느려질 수 있다.

### 3) 구현 클래스마다 테이블 전략

**쓰면 안되는 전략이다. DBA와 ORM 개발자 둘 다 싫어하는 전략이다.**

- 장점
  1. 서브 타입을 명확하게 구분해서 처리할 때 효과적이다.
  2. `NOT NULL` 제약조건을 사용할 수 있다.
- 단점

  1. 여러 자식 테이블을 함께 조회할 때 성능이 느려진다 → `UNION`

     `Item`을 기준으로 조회를 하면 어마무시한 SELECT SQL이 나가게 된다.

     ```java
     Item findItem = em.find(Item.class, movie.getId());
     ```

     ```bash
     Hibernate:
         select
             item0_.id as id1_2_0_,
             item0_.name as name2_2_0_,
             item0_.price as price3_2_0_,
             item0_.actor as actor1_5_0_,
             item0_.director as director2_5_0_,
             item0_.author as author1_1_0_,
             item0_.isbn as isbn2_1_0_,
             item0_.artist as artist1_0_0_,
             item0_.clazz_ as clazz_0_
         from
             ( select
                 id,
                 name,
                 price,
                 actor,
                 director,
                 null as author,
                 null as isbn,
                 null as artist,
                 1 as clazz_
             from
                 Movie
             union
             all select
                 id,
                 name,
                 price,
                 null as actor,
                 null as director,
                 author,
                 isbn,
                 null as artist,
                 2 as clazz_
             from
                 Book
             union
             all select
                 id,
                 name,
                 price,
                 null as actor,
                 null as director,
                 null as author,
                 null as isbn,
                 artist,
                 3 as clazz_
             from
                 Album
         ) item0_
     where
         item0_.id=?
     ```

  2. 자식 테이블을 통합해서 쿼리하기 어렵다.

# @MappedSuperclass

공통 매핑 정보가 필요할 때 사용하는 annotation.

- 등록일, 등록자, 수정일, 수정자 같은 디버깅 정보를 넣어줄 때 사용하면 좋다.

상속관계 매핑, Entity와는 결이 다르고, 테이블과 매핑되지 않는다.

자식 클래스에 매핑 정보만 제공하기 때문에, 추상 클래스로 만들어서 사용하면 된다.

실제 Entity가 아니기 때문에, `em.find`를 사용할 수 없다.

<a href="https://ibb.co/PGqswY5"><img src="https://i.ibb.co/9HQmsvq/2024-01-11-6-10-32.png" alt="2024-01-11-6-10-32" border="0"></a>

Member 객체와 Seller 객체에 id, name 필드가 공통으로 존재한다면, 하나의 가상의 부모 객체를 만들어 Member와 Seller가 이로부터 상속받는 형태로 변환하여 준다.

DB의 구조에는 영향을 미치지 않는다.

```java
/* BaseEntity.java */

@MappedSuperclass
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private String modifiedBy;
    private LocalDateTime modifiedDate;

    /* Getter & Setter */

}
```

모든 객체에 공통 정보를 제공해주기 위한 하나의 추상 클래스를 생성해주고,

```java
/* Member.java */

@Entity
public class Member extends BaseEntity{

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    /* Getter & Setter */

}
```

자식 클래스에서 상속받아서 사용하면 끝!

```java
/* Main.java */

Member member = new Member();
member.setName("SON");
member.setCreatedBy("KIM");
member.setCreatedDate(LocalDateTime.now());

tx.commit();
```

```bash
Hibernate:

    create table Member (
       MEMBER_ID bigint not null,
        createdBy varchar(255),
        createdDate timestamp,
        modifiedBy varchar(255),
        modifiedDate timestamp,
        USERNAME varchar(255),
        LOCKER_ID bigint,
        TEAM_ID bigint,
        primary key (MEMBER_ID)
    )
```

SQL문을 확인해보면, Member가 BaseEntity를 잘 상속받은 것을 확인할 수 있다.
