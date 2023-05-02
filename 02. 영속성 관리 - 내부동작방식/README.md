# 영속성 관리 - 내부동작방식

- JPA에서 가장 중요한 두가지는 객체와 관계형DB 매핑(ORM), 영속성 컨텍스트 이다.

![화면 캡처 2023-05-01 233314](https://user-images.githubusercontent.com/48784785/235471401-bb7c0e41-da68-409b-9dfe-cb98988498b2.png)

___
## 영속성 컨텍스트

- 엔티티를 영구 저장하는 환경이다.
- 엔티티 매니저를 통해 영속성 컨텍스트에 접근한다.
- J2SE 환경에선 EntityManager 와 PersistenceContext 1:1 , J2EE, Spring 에선 N:1 매핑된다.

### 엔티티 생명주기

- 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태.
```java
Member member = new Member();
member.setId(101L);
member.setName("HelloJPA");
```

- 영속(managed) : 영속성 컨텍스트에 관리되는 상태
 ```java
 em.persist(member);    // db에 저장 되지않고 영속성 컨텍스트에 올라감.
 ```
 
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
```java
em.detach(member);  // 영속성 컨텍스트에서 분리
```

- 삭제(removed) : 삭제된 상태
```java
em.remove(member); // 객체를 삭제
```

### 영속성 컨텍스트의 이점
- 1차 캐시
- 동일성(identity) 보장
- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
- 변경 감지(Dirty Checking)
- 지연 로딩(Lazy Loading)

#### 1차 캐시
```java
Member member = new Member();
member.setId(101L);
member.setName("HelloJPA");

em.persist(member);
Member findMember1 = em.find(Member.class, 101L);
```
DB가 아닌 persist(member)를 한후 find(member,101L)을 조회를 해도 select 쿼리가 날라가지 않고 1차캐시에서 find 한다.

단, 동일한 트랜잭션 안에서

```java
Member findMember1 = em.find(Member.class, 101L);
Member findMember2 = em.find(Member.class, 101L);
```
같은것을 두번 조회하면 select 문을 쓰지않고, 1차캐시에서 조회해서 select문이 안찍힌다.

#### 동일성 보장
```java
Member findMember1 = em.find(Member.class, 101L);
Member findMember2 = em.find(Member.class, 101L);
System.out.println("result = " + (findMember1 == findMember2)); // true 
```

#### 쓰기 지연
```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

transaction.commit(); // [트랜잭션] 커밋
```
JDBC 배치처럼 모아뒀다가 commit 되는 순간 한번에 insert SQL을 보낸다.
persistence.xml 에 <property name="hibernate.jdbc.batch_size" value="10"/> 에 batch size 수정 가능.

#### 엔티티 수정(변경 감지)
```java
Member member = em.find(Member.class, 150L);
member.setName("ZZZZZ");
```
em.persist나 update 같은 코드를 따로 적어 주지 않아도 update가 된다.

![화면 캡처 2023-05-02 001546](https://user-images.githubusercontent.com/48784785/235475248-394d0797-f830-4773-b77e-f55a4ea61c63.png)

엔티티와 스냅샷을 비교하여 변경이 되었으면 알아서 update를 해준다.


<br>

___

## 플러시

플러시란 영속성 컨텍스트의 변경내용을 DB에 반영하는 것.
DB 트랜잭션이 commit 되면 자동으로 플러시가 발생

변경 감지 -> 수정된 엔티티 쓰기 지연 SQL 저장소에 등록 -> 쿼리를 DB에 전송

### flush 방법
- em.flush() - 직접호출(커밋 전 쿼리를 볼 수 없다. 미리 DB에 반영하거나 쿼리를 미              리보고 싶을떄 사용.)
- 트랜잭션 커밋 - 플러시 자동 호출,
- JPQL 쿼리 실행 - 자동 호출

### JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
// 이때 까지 실제 DB에는 쿼리가 날라가지 않는다.
//중간에 JPQL 실행
query = em.createQuery("select m from Member m", Member.class); 
List<Member> members= query.getResultList();
```

기본모드가 JPQL 쿼리가 싱행시 플러시가 자동으로 호출된다. 그래서 members 에서는 A,B,C 가 조회가 된다.

### 플러시 옵션

- FlushModeType.AUTO : 커밋이나 쿼리를 실행할 때 플러시(기본값)

- FlushModeType.COMMIT : 커밋할 때만 플러시


### 플러시는
- 영속성 컨텍스트를 비우지 않음

- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화

- 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화하면 됨


___

## 준영속 상태

### 준영속 상태란

- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)

- 영속성 컨텍스트가 제공하는 기능을 사용 못함(ex. Dirty Checking)

### 준영속 상태로 만드는 방법

• em.detach(entity) : 특정 엔티티만 준영속 상태로 전환

• em.clear() : 영속성 컨텍스트를 완전히 초기화

• em.close() : 영속성 컨텍스트를 종료

```java
Member member = em.find(Member.class, 150L);
member.setName("AAA");
em.detach(member);
// commit을 해도 update가 되지 않는다.
tx.commit();
```








