---
title: '[Spring] Java ORM 표준 JPA 프로그래밍 1장 JPA 소개'
date: 2024-10-20 20:10:00 +0900
categories: [백엔드, ORM]
tags: [ORM]
math: true
mermaid: true
---

## 자바 ORM 표준 JPA 프로그래밍 도서
> [**자바 ORM 표준 JPA 프로그래밍**](https://product.kyobobook.co.kr/detail/S000000935744)

## 1.1 SQL을 직접 다룰 때 발생하는 문제점
- 반복이 많다.
- 진정한 의미의 계층 분할이 어렵다.
- 엔티티를 신뢰할 수 없다.
- SQL에 의존적인 개발을 피하기 어렵다.

## 1.2 패러다임의 불일치
- 애플리케이션은 자바라는 객체지향 언어로 개발(추상화, 캡슐화, 정보은닉, 상속, 다형성 등), 데이터는 관계형 데이터베이스에 저장해야 하므로 객체와 관계형 데이터베이스의 패러다임 불일치 문제가 발생한다.
  1. 상속(상속 vs 슈퍼타입, 서브타입) : JPA가 데이블을 조인해 필요 데이터를 조회하고 결과 반환
  2. 연관관계(참조 vs 외래 키) : 객체는 참조가 있는 방향으로만 조회 가능하나, 테이블은 아니다. JPA는 참조를 외래 키로 변환해서 데이터베이스에 전달하고, 객체를 조회할 때 외래 키를 참조로 변환한다.
  3. 객체 그래프 탐색 : JPA는 즉시 로딩과 지연 로딩을 사용해 연관된 객체를 함께 조회할 수 있다.
  4. 비교 : 객체는 동일성(identity, 객체 인스턴스의 주소 값 비교)과 동등성(equality, 객체 내부의 값 비교)을 사용하나, 테이블은 기본 키의 값으로 각 로우(row)를 구분한다. JPA는 데이터베이스에서 조회한 데이터들 간의 동일성 비교를 가능하게 해준다.
- JPA를 사용해야 하는 이유
  1. 생산성
  2. 유지보수
  3. 패러다임의 불일치 해결
  4. 성능
  5. 데이터 접근 추상화와 벤더 독립성
  6. 표준

## 1.3 JPA란 무엇인가
1) **JDBC(Java DataBase Connectivity)**  
2) **Persistence framework**
  1. **SQL Mapper**  
    - **MyBatis** : CRUD 중 R(DB 조회)에 사용, 최적화가 필요하기 때문에 복잡한 조회 쿼리에 주로 사용(.xml, @Select 등으로 구현)
    - Spring JdbcTemplate
  2. **ORM(Object Relational Mapping)** : 객체와 관계형 데이터베이스를 매핑해 패러다임의 불일치 문제를 해결   
    - **JPA(Java Persistant API)** : 자바 진영의 ORM 기술 표준. CRUD 중 CUD(DB 수정)에 사용
    - **Spring Data JDBC** : JPA처럼 ORM 기술을 사용하지만 JPA의 기술적 복잡도를 낮춘 기술  
    - **Spring Data JPA** : JPA를 한 단계 더 추상화시킨 Repository로 인터페이스 제공  
    - **Hibernate** : JPA 기반 구현체  

3) **Query**
  - **JPQL** : SQL을 추상화하여 특정 데이터베이스 SQL에 의존적이지 않은 객체지향 쿼리 언어(@Query 등)
  - **QueryDSL** : JPQL 빌더로 동적 쿼리를 메소드로 구조화하여 관리할 수 있도록 돕는 쿼리 빌더 

## Q&A
- JPA는 실시간 처리용 쿼리에 최적화
- <https://www.jboss.org/>