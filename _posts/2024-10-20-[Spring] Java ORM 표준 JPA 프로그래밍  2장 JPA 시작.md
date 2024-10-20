---
title: '[Spring] Java ORM 표준 JPA 프로그래밍 2장 JPA 시작'
date: 2024-10-20 20:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## 자바 ORM 표준 JPA 프로그래밍 도서
> [**자바 ORM 표준 JPA 프로그래밍**](https://product.kyobobook.co.kr/detail/S000000935744)

## 2.1 이클립스 설치와 프로젝트 불러오기
- JDK 1.6 이상
- Maven Update

## 2.2 H2 데이터베이스 설치
- H2 Version 1.4.187
- 서버 모드 접근 : jdbc:h2:tcp://localhost/~/test
- <https://www.h2database.com>

## 2.3 라이브러리와 프로젝트 구조
- Dependency
> hibernate-entitymanager

```
src/main
│
├─java
│  ├─jpabook/start
│  │  ├─JpaMain.java
│  │  └─Memeber.java
│  │
│  └─resources
│      └─META-INF
│          └─persistence.xml
pom.xml
```

## 2.4 객체 매핑 시작
**매핑 어노테이션**
- `@Entity` : 엔티티 클래스를 테이블과 매핑
- `@Table(name = "테이블명")` : 엔티티 클래스에 매핑할 테이블 정보를 알려준다.
- `@Id` : 엔티티 클래스의 필드를 테이블의 기본 키에 매핑
- `@Column(name = "컬럼명")` : 필드를 컬럼에 매핑, 생략 가능하나 필드명과 컬럼명이 같아야 한다.

## 2.5 persistence.xml
> resources/META-INF/persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             version="2.1">
    <persistence-unit name="jpabook">
        <!-- 탐색할 엔티티 클래스 생략 가능 -->
        <properties>
            <!-- 필수 속성 -->
            <!-- 연결할 데이터베이스당 하나의 고유 영속성 유닛 등록 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <!-- 특정 데이터베이스만의 고유한 기능, JPA에 표준화되어 있지 않음 -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <!-- 콘솔에 하이버네이트가 실행하는 SQL문 출력 -->
            <property name="hibernate.show_sql" value="true"/>
            <!-- SQL 출력 시 보기 쉽게 정렬 -->
            <property name="hibernate.format_sql" value="true"/>
            <!-- 쿼리 출력 시 주석(comments)도 함께 출력 -->
            <property name="hibernate.use_sql_comments" value="true"/>
            <!-- JPA 표준에 맞춘 새로운 키 생성 전략 사용 -->
            <property name="hibernate.id.new_generator_mappings" value="true"/>
            <!-- 애플리케이션 실행 시점에 데이터베이스 테이블 자동 생성 -->
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            <!-- 이름 매핑 전략 설정(자바의 카멜 표기법을 테이블의 스네이크 표기법으로 매핑) ex) lastModifiedDate -> last_modified_date -->
            <property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy"/>
        </properties>
    </persistence-unit>
</persistence>
```

## 2.6 애플리케이션 개발
1. 엔티티 매니저 설정
  - `EntityManagerFactory` : 애플리케이션 전체에서 한 번만 생성하고 공유해서 사용
  - `EntityManager` : 엔티티를 데이터베이스에 등록, 수정, 삭제, 조회할 수 있다. 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안 된다.
  - 종료

2. 트랜잭션 관리
  - 엔티티 매니저에서 트랜잭션 API를 받아와야 한다.

3. 비즈니스 로직

```java
package jpabook.start;

import javax.persistence.*;
import java.util..List;

public class JpaMain {
    public static void main(String[] args) {
      EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook"); // 엔티티 매니저 설정
      EntityManager em = emf.createEntityManager();
      EntityTransaction tx = em.getTransaction();

      try { // 트랜잭션 관리
        tx.begin();
        logic(em);
        tx.commit();
      } catch (Exception e) {
        tx.roolback();
      } finally () {
        em.close();
      }
      emf.close();
    }

    private static void logic(EntityManager em) { // 비즈니스 로직
      String id = "id1";
      Memeber member = new Member();
      member.setId(id);
      member.setUsername("지한");
      member.setAge(2);

      em.persist(member); // 등록

      member.setAge(20); // 수정

      Member findMember = em.find(Member.class, id); // 한 건 조회
      
      List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList(); // 목록 조회(TypedQuery(<Member>)

      em.remove(member); // 삭제
    }
}
```

- 검색 쿼리에서 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서 엔티티 객첼 변경한 다음 검색해야 하는데, 사실상 불가능하다.
- 애플리케이션에서 필요한 데이터만 데이터베이스에서 불러오려면 검색 조건이 포함된 SQL을 사용해야 한다.
- JPA는 JPQL을 분석해 적절한 SQL을 만들어내 데이터베이스에서 데이터를 조회하므로 이러한 문제를 해결할 수 있다.
- **JPQL** : 엔티티 객체(클래스와 필드)를 대상으로 쿼리한다.