# 엔티티 매핑

### 엔티티 매핑 종류

- 객체와 테이블 매핑 : @Entity, @Table
- 필드와 컬럼 매핑 : @Column
- 기본키 매핑 : @Id
- 연관관계 매핑 : @ManyToOne, @JoinColumn


## 객체와 테이블 매핑

### @Entity
- @Entity가 붙은 클래스는 JPA가 관리, JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
- 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스 사용X, 저장할 필드에 final 사용 X
- @Entity(name = "") 와 같이 엔티티 이름을 지정할 수 있지만 기본값(클래스 이름 그대로 사용)을 사용한다.

### @Table
- 엔티티와 매핑할 테이블 지정
- 속성

@Table(name = "매핑할 테이블 이름(default=엔티티 이름)", catalog = "DB catalog 매핑", schema = "DB schema 매핑", uniqueConstraints = "DDL 생성 시에 유니크 제약 조건 생성")

<br>

___
## 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성(운영에서는 X, 로컬에서 사용하면 좋음)
- 테이블 중심 -> 객체 중심 (환경이 다를때 DB 테이블 생성 하고 이런 작업을 안해도 된)
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성(ex. varchar2, varchar)

### hibernate.hbm2ddl.auto 속성

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

- create : 기존테이블 삭제 후 다시 생성 (DROP + CREATE)
- create-drop : create와 같으나 종료시점에 테이블 DROP
- update : 변경분만 반영
- validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
- none : 사용하지 않음

** 운영에서는 validate, none만 사용

<br>

___
## 필드와 컬럼 매핑

