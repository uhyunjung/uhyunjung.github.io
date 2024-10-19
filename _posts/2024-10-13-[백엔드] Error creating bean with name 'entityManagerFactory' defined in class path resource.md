---
title: '[Backend] Error creating bean with name ''entityManagerFactory'' defined in class path resource'
date: 2024-10-13 09:10:00 +0900
categories: [오늘의 에러, Backend]
tags: [Backend]
math: true
mermaid: true
---

## 문제
> org.springframework.beans.factory.BeanCreationException: Error creating bean with name ''entityManagerFactory'' defined in class path resource

## 개념
- [**Dialect 정리**](https://2dongdong.tistory.com/66)
-> DB는 현재 h2인 상태

## 해결 과정
### 1. spring.datasource.url = jdbc:mysql:/localhost:3306/db_name -> 해당 없음
<https://velog.io/@jiisuniui/Spring-Boot-JPA-Gradle-에러-entityManagerFactory-defined-in-class-path-resource>

`spring.datasource.url=jdbc:mysql:://localhost:3306/db_name`을 `spring.datasource.url=jdbc:mysql:/localhost:3306/db_name`로 변경한다.

### 2. spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect -> 성공
`spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect`을 추가하여 본인 DB와 맞는 dialect를 설정한다.

## 결과
`spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect`을 추가한다.