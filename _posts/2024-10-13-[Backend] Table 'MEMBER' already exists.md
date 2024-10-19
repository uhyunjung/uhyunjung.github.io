---
title: '[Backend] Table ''MEMBER'' already exists'
date: 2024-10-13 08:10:00 +0900
categories: [오늘의 에러, Backend]
tags: [Backend]
math: true
mermaid: true
---

## 문제
> Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Table "MEMBER" already exists; SQL statement:

## 개념
- [**DDL auto 옵션 정리**](https://colabear754.tistory.com/136)
-> DDL auto를 update로 설정한 상태

## 해결 과정
### 1. DATABASE_TO_UPPER=false 삭제 -> 해당 없음
<https://developer-minji.tistory.com/m/533>

1. schema.sql을 실행하면 대문자로 테이블을 생성된다.
2. data.sql을 실행한다. h2 database의 DATABASE_TO_UPPER=false 옵션으로 소문자로 테이블을 찾으므로 찾지 못한다.
3. 다시 대문자로 테이블 생성을 시도한다.
4. Table already exists; SQL statement 에러가 발생한다.

### 2. spring.jpa.defer-datasource-initialization = true, spring.sql.init.mode= always -> 해당 없음
<https://wildeveloperetrain.tistory.com/m/228>

<https://cl8d.tistory.com/59>

```bash
spring.jpa.defer-datasource-initialization = true
spring.sql.init.mode= always
```

EntityManagerFactory이 먼저 생성되고 초기화가 되었기 때문에 @Entity가 붙은 클래스의 정보에 따라서 테이블을 생성했을 것이다.
그 다음 schema.sql에 있는 내용으로 테이블을 초기화하려고 했기 때문에 이미 테이블이 존재한다면 오류가 발생한다.

### 3. spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults = true -> 성공
```bash
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults = true
```

**schema.sql가 resources 폴더 안에 있는 상태에서 위의 값이 false이면 Table already exists 에러가 발생한다.**
초기화한 jdbc metadata를 사용하지 않게 되기 때문에 ddl-auto update로 다시 table을 생성하려고 시도하고 테이블이 이미 존재한다는 로그를 보여준다.
**위의 값을 false에서 true로 바꾸면 에러가 발생하지 않는다.**
`spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults`의 기본값은 `true`이기 때문에 생략해도 된다.

schema.sql이 없으면 true이든 false이든 테이블을 schema로 새로 생성하지 않기 때문에 에러가 발생하지 않는다.

### 결과
`Spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults = false`를 `true`로 변경한다.
**ddl auto가 update일 때, entity의 column명을 변경하면 기존의 column은 그대로 존재하고 새로운 column이 DB에 생성된다.
ddl auto가 update일 때, entity의 column명을 삭제해도 DB에는 그대로 column이 존재한다.**