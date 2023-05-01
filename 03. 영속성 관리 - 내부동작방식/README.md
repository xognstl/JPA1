# 영속성 관리 - 내부동작방식

- JPA에서 가장 중요한 두가지는 객체와 관계형DB 매핑(ORM), 영속성 컨텍스트 이다.

![화면 캡처 2023-05-01 233314](https://user-images.githubusercontent.com/48784785/235471401-bb7c0e41-da68-409b-9dfe-cb98988498b2.png)

## 영속성 컨텍스트

- 엔티티를 영구 저장하는 환경이다.
- 엔티티 매니저를 통해 영속성 컨텍스트에 접근한다.
- J2SE 환경에선 EntityManager 와 PersistenceContext 1:1 , J2EE, Spring 에선 N:1 매핑된다.

## 엔티티 생명주기

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
em.remove(member); // 객체를 
```








