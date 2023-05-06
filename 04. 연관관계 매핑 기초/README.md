# 연관관계 매핑 기초


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

- 객체의 참조와 테이블의 외래키를 매핑
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

<br>

___
## 양방향 연관관계와 연관관계의 주인

### 양방향 매핑

```java
Team findTeam = findMember.getTeam();
```
Member 객체에서 Team 을 조회 할 수 있지만 Team 에서 Member 를 볼 수 없다.

![화면 캡처 2023-05-06 135327](https://user-images.githubusercontent.com/48784785/236600701-d959ec27-7490-4af0-9e14-ba436141ca77.png)

- Team class에 members 추가
```java
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
```

### mappedBy
객체는 양방향이라기보단 회원-> 팀 , 팀 -> 회원 형태의 단방향 2개 이고, 테이블은 회원 <-> 팀 처럼 양방향이다.   
객체는 둘 중 하나로 외래 키를 관리 해야하는데 두 관계중 하나를 연관관계의 주인으로 지정한다.   
연관관계 주인만이 외래 키를 관리(등록, 수정) 할 수 있고, 나머지는 읽기만 가능하다.   
주인은 mappedBy 속성을 사용 하지않고 주인이 아닐때만 mappedBy 속성으로 주인을 지정한다.   

### 주의점

```java
            //회원 저장
            Member member = new Member();
            member.setName("member1");
            em.persist(member);

            //팀 저장
            Team team = new Team();
            team.setName("TeamA");
            team.getMembers().add(member);  //역방향(주인이 아닌 방향)만 연관관계 설정
            em.persist(team);
```
위와 같이 하면 member 테이블에 TEAM_ID 가 null 이 된다. 주인이 아닌 TEAM 에서 변경을 하면 반영되지 않는다.   
```java
            //팀 저장
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            //회원 저장
            Member member = new Member();
            member.setName("member1");
            member.setTeam(team);
            em.persist(member);
```
위와 같이 주인이 되는 곳에서 변경을 하면 테이블에 반영이 된다.

매 번 getMembers().add(team) 이런 코드를 쓰지말고 연관관계 편 메소드를 만들어준다.
```java
public void setTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
```



