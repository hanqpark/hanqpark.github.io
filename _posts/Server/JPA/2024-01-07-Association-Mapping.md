---
title: "[JPA 기초] 04. 연관관계 매핑"

categories:
  - JPA
tags:
  - [Java]

toc: true
toc_sticky: true

date: 2024-01-07
last_modified_at: 2024-01-07
---

# 연관관계 매핑

단순히 Entity Mapping만 진행했을 때, 객체지향적인 설계가 불가능해지는 것을 보완하기 위함.

- Table: FK로 JOIN을 사용하여 연관 테이블을 찾음
- Entity: 참조를 사용하여 연관 객체를 찾음

<a href="https://ibb.co/Gn1ZQzF"><img src="https://i.ibb.co/S51kwFJ/2024-01-07-2-30-23.png" alt="2024-01-07-2-30-23" border="0"></a>

```java
/* Member.java & Team.java */
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    /* Team entity의 id를 참조 X
       FK 그대로 사용 */
    @Column(name = "TEAM_ID")
    private Long teamId;
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    private String name;
}
```

```java
/* Main.java */

/* 팀 저장 */
Team team = new Team();
team.setName("TeamA");
em.persist(team);

/* 회원 저장 */
Member member = new Member();
member.setName("member1");
member.setTeamId(team.getId());  // FK 직접 사용
em.persist(member);

/* 조회 */
Member findMember = em.find(Member.class, member.getId());

/* 연관관계가 없음 -> 객체 지향적인 방법 X */
Team findTeam = em.find(Team.class, team.getId());
```

## 1. 단방향 연관관계

<a href="https://ibb.co/s5j9ZQW"><img src="https://i.ibb.co/B4L29jn/2024-01-07-2-51-48.png" alt="2024-01-07-2-51-48" border="0"></a>

```java
/* Member.java */
package jpa_relation;

import javax.persistence.*;

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;
    private int age;

//    @Column(name = "TEAM_ID")
//    private Long teamId;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    /* Getter & Setter */
    public Long   getId()              { return id; }
    public void   setId(Long id)       { this.id = id; }
    public String getName()            { return name; }
    public void   setName(String name) { this.name = name; }
    public int    getAge()             { return age; }
    public void   setAge(int age)      { this.age = age; }
    public Team   getTeam()            { return team; }
    public void   setTeam(Team team)   { this.team = team; }
}
```

기존의 `Long teamId`를 `Team team`으로 변경하여 연관관계 매핑을 설정해주었다.

```java
/* Main.java */

/* Team 저장 */
Team team = new Team();
team.setName("Team A");
em.persist(team);

/* Member 저장 */
Member member = new Member();
member.setName("member1");
member.setTeam(team);
em.persist(member);

/* SELECT */
Member findMember = em.find(Member.class, member.getId());
Team findTeam = findMember.getTeam();
System.out.println("findTeam = " + findTeam.getName());

/* UPDATE */
Team newTeam = new Team();
newTeam.setName("Team B");
em.persist(newTeam);
findMember.setTeam(newTeam);
```

기존에는 Team의 Id를 구하기 위해 FK를 사용하여 객체 지향적이지 못했지만,

지금은 getter를 활용하여 Team의 Id를 구하고, setter를 활용하여 member의 Team을 갱신할 수 있도록 객체 지향적인 코드를 완성하였다.

단방향 매핑의 한계는 다음과 같다.

- Member → Team 참조 가능 (인수은은 대한민국 사람이다 → O)
- Team → Member 참조 불가능 (대한민국 사람 중에 인수은이 있다 → X)

## 2. 양방향 연관관계

### 1) 양방향 매핑

<a href="https://ibb.co/ygw3wSt"><img src="https://i.ibb.co/bNSfSJT/2023-12-27-8-05-45.png" alt="2023-12-27-8-05-45" border="0"></a>

Team의 PK랑, Member의 FK를 JOIN하면 알아서 양방향으로 연관관계가 생기기 때문에,
테이블 연관관계는 단방향 or 양방향 개념이 없다.

객체 연관관계는 Member entity와 Team entity 양 쪽 필드가 서로를 참조할 수 있도록 구현해야 한다.

```java
/* Member.java */
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    private int age;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

Member entity는 단방향 연관관계 설정해주었을 때와 다른 점이 없다.

```java
/* Team.java */
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<Member>();
}
```

Team entity에, member list를 구현하여, 양방향으로 참조할 수 있도록 설정해주었다.

```java
/* Main.java */
Team findTeam = em.find(Team.class, team.getId());
int memberSize = findTeam.getMembers().size();  // 역방향 조회
```

### 2) mappedBy

- 테이블 연관관계 → 양방향 1개
  - Member ↔ Team
- 객체 연관관계 → 양방향 1개인듯 보이나 사실 단뱡향 2개
  - Member → Team
  - Team → Member

<a href="https://ibb.co/L9CXWjt"><img src="https://i.ibb.co/9yqkLKT/2023-12-27-8-20-03.png" alt="2023-12-27-8-20-03" border="0"></a>

둘 중 하나의 객체로 외래 키를 관리해야 한다. 그 하나의 객체를 고르는 방법이 **연관관계 주인**을 선정하는 방법이다.

## 3. 연관관계의 주인(Owner)

### 1) 설정 방법

<a href="https://ibb.co/d5DGn13"><img src="https://i.ibb.co/YLhpvYK/2023-12-27-8-31-18.png" alt="2023-12-27-8-31-18" border="0"></a>

Member, Team 중 하나를 연관관계 주인으로 지정해야 하는데, **외래키가 있는 곳을 Owner로 설정한다.**

| 주인                    | 하인                        |
| ----------------------- | --------------------------- |
| 외래키 관리(등록, 수정) | 읽기만 가능                 |
| mappedBy 속성 사용 X    | mappedBy 속성으로 주인 지정 |
| ManyToOne               | OneToMany                   |
| N, 多 쪽이 주인         | 1                           |

### 2) 주의

1. 양쪽 다 값을 입력해야 한다.

   ```java
   /* Main.java */
   Team team = new Team();
   team.setName("TeamA");
   em.persist(team);

   Member member = new Member();
   member.setName("member1");

   /* 역방향만 연관관계 설정됨 */
   team.getMembers().add(member);  // member.team_id = null

   /* 연관관계의 주인에 값 설정 */
   member.setTeam(team);           // member.team_id = 2

   em.persist(member);
   ```

2. 연관관계 편의 메소드를 생성한다.

   ```java
   /* Member.java */
   @Entity
   public class Member {
      public void changeTeam(Team team) {
         this.team = team;
         team.getMembers().add(this);
      }
   }
   ```

   setter와 기능이 헷갈릴 수 있으니, 이름은 setTeam 말고 다른 것으로 설정하는 것을 추천한다.

3. 무한 루프 조심
   - `toString()`
   - `lombok`
   - JSON 생성 라이브러리
     - Controller에는 entity 절대 반환하지 말 것
     - entity는 DTO로 변환해서 반환할 것
       [[Spring] DTO란?](https://velog.io/@modsiw/Spring-DTO란)

## 4. 정리

단방향 매핑만으로도, 이미 연관관계 매핑은 완료되었다.

양방향 매핑은 단순히 반대 방향으로 조회하는 기능이 추가된 것이라,
실무에서 JPQL 등을 활용하여 역방향으로 탐색할 때 추가로 설정해주면 된다. (통계 산출 등)

<a href="https://ibb.co/zHTCR25"><img src="https://i.ibb.co/VpcbSCN/2024-01-01-9-44-01.png" alt="2024-01-01-9-44-01" border="0"></a>

실무(실시간 애플리케이션) 기준으로 생각해 보았을 때,
주문이 아이템 테이블을 참조할 일은 많지만, 아이템으로 주문 테이블을 참조할 일은 거의 없다.

따라서, 연관관계 주인은 비즈니스 로직을 기준이 아닌, 외래 키의 위치를 기준으로 선택해야 한다.
