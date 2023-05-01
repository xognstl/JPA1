# JPA 시작

## 프로젝트 생성

### 라이브러리 추가 - pom.xml
```xml
<dependencies>
    <!-- JPA 하이버네이트 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.3.10.Final</version>
    </dependency>
    <!-- H2 데이터베이스 -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.199</version>
    </dependency>
</dependencies>
```

### JPA 설정 - persistence.xml
resource/META-INF/persistence.xml 생성
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.jdbc.batch_size" value="10"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```


## JPA

### 구동방식
![화면 캡처 2023-05-02 002529](https://user-images.githubusercontent.com/48784785/235477001-ada79aba-05ee-4eed-a32b-22587b76be02.png)

### 엔티티 생성
```java
package hellojpa;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Member {

    @Id // DB의 PK와 매핑.
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

### JPA 코드
```java
public static void main(String[] args) {
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
  EntityManager em = emf.createEntityManager();
  
  EntityTransaction tx = em.getTransaction(); //트랜젝션 가져오기
}
```
EntityManager 생성.

```java
        tx.begin();

        try{
            Member member = new Member();
            // 저장.
              member.setId(2L);
              member.setName("helloB");
              em.persist(member);

            // 조회
            Member findMember = em.find(Member.class, 1L);
            System.out.println("findMember.getId() = " + findMember.getId());
            System.out.println("findMember.getName() = " + findMember.getName());

            // 삭제
            em.remove(findMember);

            // 수정
            findMember.setName("helloJPA");

            // JPQL 전체 회원 검색
            // Member 객체를 대상으로 쿼리를 한다.
            List<Member> result = em.createQuery("select m from Member as m", Member.class)
//                    .setFirstResult(1).setMaxResults(10)    //페이징
                    .getResultList();
            for (Member member : result) {
                System.out.println("member = " + member.getName());
            }*

            tx.commit();    // 트랜젝션 커밋을 해야 영속성 콘텍스트에서 db 쿼리가 날라감.
        }catch (Exception e){
            tx.rollback();
        }finally {
            em.close();
        }

        emf.close();
    }
```



