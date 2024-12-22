---
title: '[Spring] Java ORM 표준 JPA 프로그래밍 4장 엔티티 매핑'
date: 2024-11-30 15:10:00 +0900
categories: [백엔드, ORM]
tags: [ORM]
math: true
mermaid: true
---

## 자바 ORM 표준 JPA 프로그래밍 도서
> [**자바 ORM 표준 JPA 프로그래밍**](https://product.kyobobook.co.kr/detail/S000000935744)
> [**Github**](https://github.com/holyeye/jpabook)

## 매핑 어노테이션
1. 객체와 테이블 매핑 : @Entity, @Table
2. 기본 키 매핑 : @Id
3. 필드와 컬럼 매핑 : @Column
4. 연관관계 매핑 : @ManyToOne, @JoinColumn

## 4.1 @Entity
1. JPA를 사용해 테이블과 매핑할 클래스는 `@Entity` 어노테이션 필수, JPA가 관리하며 엔티티라 부른다.
2. 속성
  - name : 엔티티 이름, 보통 기본값인 클래스 이름 사용, 설정하지 않으면 클래스 이름 그대로 사용, 다른 패키지에 이름이 같은 엔티티 클래스가 있다면 이름을 짖어해서 충돌하지 않도록 해야 한다.
3. 주의사항
  - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
  - Java는 생성자가 하나도 없으면 다음과 같은 기본 생성자 자동 생성
  ```java
  public Member() { }
  ```
  - Java는 생성자를 하나 이상 만들면 기본 생성자를 자동으로 만들지 않으므로 직접 만들어야 한다.
  ```java
  public Member() { }
  public Member(String name) { this.name = name; }
  ```
  - final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
  - 저장할 필드에 final을 사용하면 안 된다.

## 4.2 @Table
1. 엔티티와 매핑할 테이블을 지정한다. 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다.
2. 속성
  - name : 매핑할 테이블 이름, 기본값으로 엔티티 이름을 사용
  - catalog : catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.
  - schema : schema 기능이 있는 데이터베이스에서 schema를 매핑한다.
  - DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다.

## 4.3 다양한 매핑 사용
1. 회원 관리 프로그램에 요구사항 추가
  - 회원은 일반 회원과 관리자로 구분해야 한다.
  - 회원 가입일과 수정일이 있어야 한다.
  - 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java
package jpabook.start;

import javax.persistence.*;  //**
import java.util.Date;

/**
 * User: HolyEyE
 * Date: 13. 5. 24. Time: 오후 7:43
 */
@Entity
@Table(name="MEMBER", uniqueConstraints = {@UniqueConstraint( //추가 //**
        name = "NAME_AGE_UNIQUE",
        columnNames = {"NAME", "AGE"} )})
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME", nullable = false, length = 10) //추가 //**
//    @Column(name = "NAME") //추가 //**
    private String username;

    private Integer age;

    //=== 추가
    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;

    @Transient
    private String temp;


    //Getter, Setter

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public RoleType getRoleType() {
        return roleType;
    }

    public void setRoleType(RoleType roleType) {
        this.roleType = roleType;
    }

    public Date getCreatedDate() {
        return createdDate;
    }

    public void setCreatedDate(Date createdDate) {
        this.createdDate = createdDate;
    }

    public Date getLastModifiedDate() {
        return lastModifiedDate;
    }

    public void setLastModifiedDate(Date lastModifiedDate) {
        this.lastModifiedDate = lastModifiedDate;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getTemp() {
        return temp;
    }

    public void setTemp(String temp) {
        this.temp = temp;
    }
}
```
```java
package jpabook.start;

/**
* Created by 1001218 on 15. 3. 27..
*/
public enum RoleType {
    ADMIN, USER
}
```
```java
package jpabook.start;

import javax.persistence.*;
import java.util.List;

/**
 * @author holyeye
 */
public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {


            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }

    public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        //등록
        em.persist(member);

        //수정
        member.setAge(20);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

    }
}
```

## 4.4 데이터베이스 스키마 자동 생성
1. 클래스의 매핑 정보와 데이터베이스 방언을 사용해 데이터베이스 스키마를 생성한다.
  - persistence.xml

  ```xml
  <property name="hibernate.hbm2ddl.auto" value="create" /> <!--애플리케이션 실행 시점에 데이터베이스 테이블 자동 생성-->
  <property name="hibernate.show_sql" value="true" /> <!--테이블 생성 DDL 출력-->
  ```

  - DDL 콘솔 출력

  ```bash
  Hibernate:
    drop table MEMBER if exists
  Hibernate:
    create table MEMBER {
      ID varchar(255) not null,
      NAME varchar(255),
      age integer, # 오라클 데이터베이스용 방언 : number 타입 생성
      roleType varchar(255), # 오라클 데이터베이스용 방언 : varchar2 타입 생성
      createdDate timestamp,
      lastModifiedDate timestamp,
      description clob,
      primary key (ID)
    }
  ```

  - 오라클 데이터베이스용 방언

  ```sql
    create table MEMBER {
    ID varchar2(255 char) not null,
    NAME varchar2(255 char),
    age number(10, 0),
    roleType varchar2(255 char),
    createdDate timestamp,
    lastModifiedDate timestamp,
    description clob,
    primary key (ID)
  }
  ```

2. 스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지는 않으므로 개발 환경에서 사용하거나 매핑을 어떻게 해야 하는지 참고하는 정도로만 사용하는 것이 좋다.
3. `hibernate.hbm2ddl.auto` 속성
  - create : 기존 테이블을 삭제하고 새로 생성한다. DROP + CREATE
  - create-drop : create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다. DROP + CREATE + DROP
  - update : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다.
  - validate : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. 이 설정은 DDL을 수정하지 않는다.
  - none : 자동 생성 기능을 사용하지 않으려면 hibernate.hbm2ddl.auto 속성 자체를 삭제하거나 유효하지 않은 옵션 값을 주면 된다(참고로 none은 유효하지 않은 옵션 값이다).
  <br/>
  - 운영 서버에서 create, create-drop, update처럼 DDL을 수정하는 옵션은 절대 사용하면 안 된다. 오직 개발 서버나 개발 단계에서만 사용해야 한다. 이 옵션들은 운영 중인 데이터베이스의 테이블이나 컬럼을 삭제할 수 있다.
    - 개발 초기 단계는 create 또는 update
    - 초기화 상태로 자동화된 테스트를 진행하는 개발자 환경과 CI 서버는 create 또는 create-drop
    - 테스트 서버는 update 또는 validate
    - 스테이징과 운영 서버는 validate 또는 none
  - JPA는 2.1부터 스키마 자동 생성 기능을 표준으로 지원한다. 하지만 하이버네이트의 hibernate.hbm2ddl.auto 속성이 지원하는 update, validate 옵션을 지원하지 않는다. (지원 옵션 : none, create, drop-and-create, drop)

  ```xml
  <property name="javax.persistence.schema-generation.database.action" value="drop-and-create" />
  ```
  - Java 언어는 카멜 표기법, 데이터베이스는 스네이크 표기법을 주로 사용

  ```java
  @Column(name="role_type") // 스네이크 표기법
  String roleType; // 카멜 표기법
  ```
  - hibernate.ejb.naming_strategy 속성을 사용하면 이름 매핑 전략을 변경할 수 있다. 직접 이름 매핑 전략을 구현해서 변경해도 되지만, 하이버네이트는 org.hibernate.cfg.ImprovedNamingStrategy 클래스를 제공한다. 이 클래스느느 테이블 명이나 컬럼 명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑한다.
  
  ```xml
  <property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy" />
  ```
  ```sql
    create table MEMBER {
    ID varchar2(255 char) not null,
    NAME varchar2(255 char),
    age number(10, 0),
    role_type varchar2(255 char),
    created_date timestamp,
    lastModified_date timestamp,
    description clob,
    primary key (ID)
  }
  ```

## 4.5 DDL 생성 기능
1. `@Column` 매핑정보의 nullable 속성 값을 false로 지정하면 자동 생성되는 DDL에 not null 제약조건을 추가할 수 있다.
2. length 속성 값으로 자동 생성되는 DDL에 문자의 크기를 지정할 수 있다.
```java
@Entity
@Table(name="MEMBER")
public class Member {
  @Id
  @Column(name="ID")
  private String id;

  @Column(name="NAME", nullable=false, length=10) // 추가
  private String username;
}
```
```sql

```