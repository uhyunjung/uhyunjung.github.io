---
title: '[Backend] Session 관리'
date: 2025-01-25 15:30:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## 쿠키 및 세션
- 서버는 세션 관리
- 클라이언트는 세션 ID를 쿠키에 저장
- 세션 1시간 = 60분 = 3600초
- ex) GA 쿠키

## Session 서버 관리
1. 중앙 세션 서버
- Simple
- 클라이언트 -> 서버 -> 중앙 DB
- Sticky Session X
- 최적화 X

2. 클러스터링
- 복잡
- N^2번 요청
- 최적화 파일
- Serialize 필요
- Round Robin 방식

## Session 저장소
1. 톰캣 세션 사용
- 2대 이상의 서버 사용 시 세션 클러스터링 필요

2. DB 사용
- JDBC Session
- ex) MySQL, NoSQL

3. 메모리 DB 사용
- ex) Redis, Memcached

## Redis 사용 용도
1. 캐시 시스템
2. 세션 저장소
3. Message Queue(MQ)

## Spring Session JDBC
- SPRING_SESSION 테이블 : Session ID, Created, Last Access, Max Inactive Interval, Expiry Time
- SPRING_SESSION_ATTRIBUTES 테이블 : Session ID, Attribute Name, Attribute Bytes
- application.yml or application.properties
	- `spring.session.timeout=15m`
	- `spring.session.store-type=jdbc`