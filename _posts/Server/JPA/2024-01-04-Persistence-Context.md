---
title: "[JPA 기초] 02. 영속성 컨텍스트"

categories:
  - JPA
tags:
  - [Java]

toc: true
toc_sticky: true

date: 2024-01-04
last_modified_at: 2024-01-07
---

[JPA의 영속성 컨텍스트와 엔티티 생명주기](https://siyoon210.tistory.com/138)

위 링크를 참고하여 포스팅을 작성하였습니다.

# JPA에서 가장 중요한 2가지

1. **영속성 컨텍스트**: Persistence Context
2. **엔티티 매핑**: 객체와 관계형 DB를 매핑하는 것, Obejct Relational Mapping (ORM)
   - [[JPA 기초] 03. 엔티티 매핑](https://hanqpark.github.io/jpa/entity-mapping/)에 정리해두었습니다.

# 영속성 컨텍스트

영속성 컨텍스트는 buffer라고 생각하면 이해하기 쉽다.

Entity가 변경될 때마다 commit을 날려버리면 네트워크를 타고 DB서버를 왕복해야 하는 비용이 commit을 할 때마다 발생한다.

이러한 비용을 최대한 줄이기 위해, 영속성 컨텍스트라는 buffer에 변경 사항들을 모두 저장하여 두고, 마지막에 딱 한 번의 commit으로 네트워크를 타는 작업을 최소화할 수 있다.

`EntityManager.persist(entity);` 를 통해 `entity`를 영속성 컨텍스트 안으로 귀속시킬 수 있다.

<a href="https://ibb.co/vzrf7PN"><img src="https://i.ibb.co/bdyYSB4/Untitled.png" alt="Persistence Context" border="0"></a>

영속성 컨텍스트는 `1차 캐시`와 `SQL 저장소` 2가지 영역으로 나뉜다.

- `1차 캐시`: `entity`가 저장되는 장소
- `SQL 저장소`: `entity`의 변경사항(I, D, U)을 쿼리문으로 쌓아두는 곳

위와 같이 알아두면 영속성 컨텍스트를 이해하기 편하다.

## 1. Entity 생명주기

<a href="https://ibb.co/sgfJd3p"><img src="https://i.ibb.co/dfTJwKs/2023-12-24-3-50-01.png" alt="Entity Lifecycle" border="0"></a>

```java
public class JpaMain {
    public static void main(String[] args) {

        /* Entity Manager Factory 생성 */
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        /* Entity Manager 생성 */
        EntityManager em = emf.createEntityManager();

        /* Transaction 실행 */
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {

            /* 비영속 상태: 단순히 객체를 생성한 상태 */
            Member member = new Member();
            member.setId("A123")
            member.setUsername("James");

            /* 영속 상태: 객체를 영속성 컨텍스트에 저장한 상태 */
            em.persist(member);

            /* 준영속 상태: member entity를 영속성 컨텍스트에서 분리 */
            em.detach(member);

            /* 삭제 상태: 객체를 삭제 */
            em.remove(member);

            tx.commit();

        } catch (Exception e) {
            tx.rollback();

        } finally {
            /* Entity Manager 종료 */
            em.close();
        }
        emf.close();
    }
}
```

1. 비영속
   - 말 그대로, entity가 영속성 컨텍스트와 전혀 관계가 없는 상태이다.
2. 영속
   - 1차 캐시에 entity가 등록되어 있는 상태
   - 즉, entity가 JPA의 관리를 받고 있는 상태
   - 영속되어 있는 entity를 조회할 때, DB에 select query를 날리지 않고 1차 캐시에 있는 entity를 조회한다.
3. 준영속
   - 영속되어 있다가 분리된 entity
   - 준영속 상태로 전환하는 방법
     1. `em.detach(entity)`
        - 특정 entity만 준영속 상태로 전환
     2. `em.clear()`
        - 영속성 컨텍스트를 완전히 초기화
     3. `em.close()`
        - 영속성 컨텍스트를 종료

## 2. 특장점

### 1) 1차 캐시

<a href="https://ibb.co/59C4sQ3"><img src="https://i.ibb.co/rtq05Bh/2023-12-21-5-28-44.png" alt="1st cache" border="0"></a>

```java
Member member = new Member();
member.setId("member1");
member.setUsername("James");

// 영속성 컨텍스트의 1차 캐시에 entity 저장
em.persist(member);

// DB가 아닌, 1차 캐시에 저장된 값을 불러 옴
Member findMember = em.find(Member.class, "member1");

// 1차 캐시 조회 후, 1차 캐시에 없는 값은 DB에서 불러 옴
Member findMember2 = em.find(Member.class, "member2");
```

### 2) 동일성(identity) 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b)  // true
```

- `em.find`로 조회하는 순간, ID에 해당하는 entity는 영속성 컨텍스트의 1차 캐시에 저장됨
- 같은 ID를 조회하면, 1차 캐시를 먼저 조회하여 결과 값을 반환하기 때문에 SELECT Query를 또 뿌리지 않음
- 1차 캐시를 활용하여, `REPEATABLE READ` 등급의 트랜잭션 격리 수준을 DB가 아닌 AP 차원에서 제공
- 성능적으로 크게 향상되는 부분은 없음

### 3) 트랜잭션을 지원하는 쓰기 지연: Transactional write-behind

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        /* Transaction 실행 */
        tx.begin();

        /* 1차 캐시에 memberA, memberB entity 저장 */
        /* SQL 저장소에 memberA, memberB를 DB에 저장할 INSERT SQL 저장 */
        em.persist(memberA);
        em.persist(memberB);


        /* commit하는 순간 DB에 INSERT SQL 실행 */
        tx.commit();
    }
}
```

<a href="https://ibb.co/q092bpg"><img src="https://i.ibb.co/6vbq6Ft/2023-12-21-5-44-46.png" alt="write-behind-1" border="0"></a>

- `1차 캐시`에는 `memberA`, `memberB` entity 저장
- `SQL 저장소`에는 `memberA`, `memberB`를 DB에 저장할 INSERT SQL 저장

<a href="https://ibb.co/YDvymBJ"><img src="https://i.ibb.co/pWCRcjN/2023-12-21-5-44-53.png" alt="write-behind-2" border="0"></a>

- `commit`을 실행하면 SQL 저장소에 있던 SQL문들이 DB로 `flush`

### 4) 변경 감지: Dirty Checking

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        /* SELECT from 영속 엔티티 */
        Member member = em.find(Member.class, 101L);

        /* UPDATE 영속 엔티티 */
        member.setName("ChangedName");
        member.setAge("30");

        // em.update(member);  -> JPA가 1차 캐시에서 알아서 변경사항을 비교해 줌
        // em.persist(member); -> find에서 이미 영속성 컨텍스트 안에 member를 포함시켰으므로 중복 실행이 됨

        tx.commit();
    }
}
```

<a href="https://ibb.co/k6mTngk"><img src="https://i.ibb.co/NnTqh6Q/2023-12-24-2-54-32.png" alt="dirty-checking" border="0"></a>

1. `em.find`로 entity를 1차 캐시에 저장할 때 `@Id`, `@Entity`, 그리고 불러올 당시의 entity에 대한 `snapshot`을 저장함

   | @Id | @Entity                           | snapshot                          |
   | --- | --------------------------------- | --------------------------------- |
   | 1   | memberA (name → James / age → 20) | memberA (name → James / age → 20) |

2. entity를 업데이트하면 `@Entity`의 내용이 변경됨

   | @Id | @Entity                                  | snapshot                          |
   | --- | ---------------------------------------- | --------------------------------- |
   | 1   | memberA (name → **Hans** / age → **30**) | memberA (name → James / age → 20) |

3. `commit`을 하면 JPA는 `@entity`와 `snapshot`의 내용을 비교함

   > memberA의 이름이 Hans, 나이가 30으로 변경 되었군

4. 변경된 내용에 대해 UPDATE SQL 생성 후, SQL 저장소에 저장

   ```sql
   UPDATE MEMBER
   SET Username = 'Hans', Age = 30
   WHERE ID = 1;
   ```

5. `flush`를 통해 SQL문 실행으로 entity의 변경 된 내용을 DB에 저장

## 3. Flush

### 1) 개념

- 영속성 컨텍스트의 변경내용을 DB에 동기화
- 영속성 컨텍스트를 비우지 않음
- TR 작업 단위가 중요 → commit 직전에만 동기화 하면 됨

### 2) 방법

1. 직접 호출

   - `em.flush()`

     ```java
     Member member = new Member(200L, "member200");
     em.persist(member);

     em.flush();
     em.clear();
     ```

     테스트를 할 때, em.flush로 DB 동기화 작업을 마치고, em.clear로 영속성 컨텍스트를 완전히 초기화 하는 방법도 종종 사용한다.

2. 자동 호출
   - `tx.commit()`
   - JPQL 쿼리 실행
