# 다양한 연관관계 매핑

___
## 연관관계 매핑시 고려사항

### 다중성
- 다대일 : @ManyToOne
- 일대다 : @OneToMany
- 일대일 : @OneToOne
- 다대다 : @ManyToMany (거의 사용 X)
  
### 단방향, 양방향
- 테이블
  - 외래키 하나로 양쪽 조인 가능
  
- 객체
  - 참조용 필드가 있는 쪽으로만 참조 가능
 
### 연관관계의 주인
- 객체 양방향 관계는 A -> B, B -> A 참조가 두군데, 테이블은 외래키 하나로 연관관계를 맺음
- 객체는 둘중 테이블의 외래 키를 관리할 곳을 지정해야 한다.
- 연관관계의 주인은 외래 키를 관리하는 참조, 반대편은 외래키에 영향X, 단순 조회만 가능

<br>

___
## 다대일 [N:1]

![화면 캡처 2023-05-08 204745](https://user-images.githubusercontent.com/48784785/236816100-b80ee7b3-3ddf-41da-9288-e390d282b3d0.png)

MEMBER N : TEAM 1 , N쪽에 외래키가 있어야 한다.   

```java
    //Member.class
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    //Team.class
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;
    
    // 다대일 양방향, 테이블에 연관이 없고, 조회만 가능 
//    @OneToMany(mappedBy = "team")
//    private List<Member> members = new ArrayList<>();
```

<br>

___
## 일대다 [1:N]

![화면 캡처 2023-05-08 210108](https://user-images.githubusercontent.com/48784785/236818565-5ff775bd-1ba5-40c7-ac4a-786275860ad7.png)

1이 연관관계의 주인이다. 1방향에서 외래키를 관리하겠다는 의미.

```java
//Member.class
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;
    
//Team.class
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
```

```java
            Member member = new Member();
            member.setUsername("member1");

            em.persist(member);

            Team team = new Team();
            team.setName("teamA");
            team.getMembers().add(member);
            
            em.persist(team);
```
- 단점
  - 위와 같은 코드 작성시 insert, insert, update 쿼리가 발생한다. 쿼리가 한번더 발생한다.
  - 엔티티가 관리하는 외래 키가 다른 테이블에 있다.
- 일대다 단방향 매핑보다 다대일 양방 매핑을 사용하는 편이 좋다.   
- 일대다 양방향은 @JoinColumn(insertable=false, updatable=false) 읽기전용으로 @ManyToOne 사용. 이런게 있다만 알고 사용하지말자.


<br>

___
## 일대일 [1:1]

- 주 테이블이나 대상 테이블 중 외래키 선택 가능.
- 외래 키에 DB 유니크 제약 조건 추가.
- 다대다에서 유니크 제약 조건이 추가됬다고 생각하면 된다.

![화면 캡처 2023-05-08 212444](https://user-images.githubusercontent.com/48784785/236823124-ba5e35ed-b502-4617-a9cd-3a3bffd62db9.png)

```java
//Member.class
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
//Locker.class
    @Id @GeneratedValue
    private Long id;

    private String name;
     // 일대일 양방향
//    @OneToOne(mappedBy = "locker")
//    private Member member;
```

** 외래키 설정
- 주 테이블에 외래 키(Member)
  - 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
  - 단점: 값이 없으면 외래 키에 null 허용

- 대상 테이블에 외래 키(Locker)
  - 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
  - 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨


