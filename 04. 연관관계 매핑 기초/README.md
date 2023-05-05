# 연관관게 매핑 기초


객체의 참조와 테이블의 외래키를 어떻게 매핑하는가?

___
## 연관관계가 필요한 이유

* 예제 시나리오 : 회원과 팀이 있고, 회원은 하나의 팀에만 소속될 수 있다. 회원과 팀은 다대일 관계.

### 객체를 테이블에 맞추어 모델링(연관관계가 없는 객체)
![화면 캡처 2023-05-05 232635](https://user-images.githubusercontent.com/48784785/236486186-39855710-3996-455e-a089-55e49cac0c72.png)

- 참조 대신에 외래 키를 그대로 사용
```java
//Member class
@Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;  // Member class에서 Team 에 대한 외래키를 그냥 가지고 있음.
//Team class    
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;   
```

- 외래 키 식별자를 직접 다룸.
```java
            //팀 저장
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            //회원 저장
            Member member = new Member();
            member.setName("member1");
            member.setTeamId(team.getId()); // 이부분.
            em.persist(member);
```

- 식별자로 다시 조회 (연관 관계가 없어서 호출을 계속 해야함. 객체 지향 X)
```java
Member findMember = em.find(Member.class, member.getId());
Long findTeamId = findMember.getTeamId();
Team findTeam = em.find(Team.class, findTeamId);
```

** 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
테이블은 외래 키로 조인을 사용하여 연관된 테이블을 찾고, 객체는 참조를 통해서 찾는다.

## 단방향 연관관계

### 객체 지향 모델링 (객체 연관관계 사용)
![화면 캡처 2023-05-06 002209](https://user-images.githubusercontent.com/48784785/236500015-a26e441b-4b3b-4ece-82fa-8944ae514d21.png)

- 객체의 참조와 테이블의 외래키를 매
```java
//    @Column(name = "TEAM_ID")
//    private Long teamId;

@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;
```

- ORM 매핑이 된다. Team team(객체 연관관계) -> TEAM_ID(FK)(테이블 연관관계)

- 연관관계 저장, 조회(객체 그래프 탐색)
```java
//팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team);   //단방향 연관관계 설정, 참조 저장
em.persist(member);

//조회
Member findMember = em.find(Member.class, member.getId());
//참조를 사용해서 연관관계 조회 , team 이 아닌 findMember.getTeam()에서 team 객체를 불러옴.
Team findTeam = findMember.getTeam();

//연관 관계 수정
// 팀 A -> B 로 수정
Team newTeam = em.find(Team.class, 100L);
findMember.setTeam(newTeam);
```



