---
title: '[Backend] DB 및 Transaction 개념'
date: 2025-02-01 15:30:00 +0900
categories: [정리, 데이터베이스]
tags: [데이터베이스]
math: true
mermaid: true
published: false
---

## 원티드 프리온보딩 백엔드 챌린지 2월
<https://www.wanted.co.kr/events/pre_challenge_be_4>

## 데이터베이스의 원칙
1. 무결성(Integrity)
    - Accuracy, Consistency 등
    - 데이터에 오류가 없어야함
    - 사용자가 저장하고자 하는 내용들이 저장되어야 함
    - 엔티티 무결성, 참조 무결성, 도메인 무결성

2. 안정성(Reliability)
    - Resilient
    - 고장이 잘 안 나야함
    - ACID 원칙
    - 백업과 복구
    - 에러 처리

3. 확장성(Scalability)
    - Scale Up vs scale Out
    - Case Study
        - 대규모 데이터 처리 : Scale Out
        - Stateless 어플리케이션 : Scale Out
        - CPU-bound : Scale Out
        - CDN : Scale Out
        - Memory-intensive : Scale Up
        - 모노리식 서버 : Scale Up
    - 분산 시스템
    - 캐시

## DB 종류
- RDBMS vs NoSQL
1. RDBMS(Relational)
    - Row Oriented Database
        - Small READ/WRITE
        - 전체 READ 보다 특정 데이터에 접근하는 경우가 빈번할 때
        - 사전 테스트
    - Column Oriented Database
        - 데이터분석
        - 머신러닝
2. NoSQL
    - Key-Value : DynamoDB, Redis
    - Graph : neo4j
    - Document : mongodb

## DB 선택
- 서비스에 적합한 데이터베이스 선택
    - 서비스를 이해한 후 결정할 것
    - 사용자들이 이 서비스를 어떻게 사용하는가
    - 서비스를 사용하기 위해 어떤 식으로 데이터에 접근하는가
    - READ / WRITE 중 무엇이 더 빈번하게 발생하는가?
    - 서비스를 잘 운영하기 위해서 중요한 것은 무엇인가?
- ex)
    - 넷플릭스 평점 RDB
    - 인스타그램 피드 ApacheCassandra NoSQL
    - MySQL
        1. E-commerce
        2. 분산시스템
        3. 그래프 기반 데이터
        4. 대용량 실시간 분석
        5. 인메모리 데이터베이스

## CAP 이론(CAP Theorem)
- Consistency, Availability, Partition-Tolerance
- Eventual Consistency vs Strong Consistency
    - 일관성(최신 데이터 or 오래된 데이터 접근)과 가용성(Lock) 중 하나를 포기해야 하는 상황
    - Eventual Consistency(가용성 > 일관성) : 오래된 데이터를 가져올 가능성이 있음, 그러나 언젠가는 동기화가 되어 모든 클라이언트가 동일한 데이터 사용 가능
    - Strong Consistency(가용성 < 일관성) : 클라이언트가 데이터 사용할 수 없는 상황 발생, Replication Copy 필요


## Transaction
1. 작업의 완전성 보장
2. 사용자의 작업셋 모두 완벽하게 처리
3. 처리하지 못하면 원상태로 복구한다

## ACID 원칙
### Atomicity
1. 더이상 쪼개질 수 없는 가장 작은 단위
    - transaction의 단위
2. 모두 성공하거나 모두 실패, 중간이 없음
    - Write 중에 실패했을 때
    - 어디부터 실패했는지 파악해서 retry 하지 않고
    - 그냥 다시 retry 가능

### Consistency
1. Transaction 전후로 데이터가 일정해야함
    - 오류가 없어야 한다는 뜻
    - 데이터베이스가 판단하기는 어려움
    - 어플리케이션 단에서 관리해줘야함
2. ex)
    - 마지막 한 개 남은 상품을 동시에 2명의 고객이 주문할 수 없도록
    - 카드 한도 초과되려고 하는데 동시에 2가지 아이템을 주문할 수 없도록

### Isolation
1. 동시에 발생하는 transaction이 서로 독립적이어야함
2. 독립되지 않으면 발생할 수 있는 문제

### Durability
1. Commit 된 transaction은 손실되지 않는다
2. 단일 시스템
    - `write-ahead log`
    - 디스크가 corrupt되면 회복할 수 있게 함
3. 복제 시스템
    - 데이터가 충분한 수의 node로 replicate 되었는지 확인

## MySQL 스토리지 엔진
- 요즘은 다 InnoDB
- InnoDB 장점
    - 버퍼링
    - Foreign key
    - transaction
- 스토리지 vs (스토리지) 엔진
- ex)
    1. 트랜잭션을 지원하는 스토리지 엔진(InnoDB)
    2. 트랜잭션을 지원하지 않는 스토리지 엔진(MyISAM)

## Transaction 예제
- `SHOW CREATE TABLE transaction;`
- `INSERT INTO no_transaction (id) VALUES (1), (2), (3);`
    - 3

    - Disk(3) -> Undo Log : 일시저장(X -> 3)
    - Buffer Pool(1, 2, 3 -> X) -> Disk : Write 시작(3 -> 1, 2, 3)
    - Undo Log(3) -> Disk : Undo Log에서 데이터 복구(1, 2, 3 -> 3)
- `INSERT INTO transaction (id) VALUES (1), (2), (3);`
    - 1,2,3

## Transaction States
- Activate state -> (R/W operations) -> Partially Commited State -> (Permanent Store) -> Committed State -> Terminated State
- Activate State or Partially Committed State -> (Failure) -> Failed State -> (Roll Back) -> Aborted State -> Teminated State


## DB Lock
1. 단어 그대로 "잠금"
2. 하나의 데이터를 동시에 여러명이 조작할 수 없도록
3. 동시성(concurrency)를 보장함
4. MySQL엔진락 vs InnoDB 락

### Global Lock
1. 말그대로 글로벌. 전역 LOCK
2. 특징
    - READ만 가능하고 수정이 불가능
    - 요즘은 TRANSACTION 적용돼서 잘 사용하지 않음

### Table Lock
1. 테이블을 잠그는 것
2. 특징
    - READ LOCK : READ 할거니까 아무도 수정 금지
    - WRITE LOCK : WRITE 할거니까 아무도 READ 금지
    - 동시성을 보장한다고 생각하면 됨
        - 수정하려고 하는데 누군가 읽어버린다면?
        - 읽으려고 하는데 누군가 수정해버린다면?
3. MyISAM이나 MEMORY에서 자동으로 사용되는 LOCK
    - InnoDB의 경우에는 트랜잭션을 활용하기 때문에 불필요

### Named Lock
1. 그냥 내가 임의로 LOCK을 잡는것
2. 다양한 로직 처리에 유리할 수 있음

### Metadata Lock
1. 테이블 정보를 수정할 때 자동으로 획득
    - 테이블 이름
    - 컬럼 이름이나 컬럼 정보 등을 수정할 때 자동으로

### Record Lock
- 스토리지 엔진단에서 락을 잡는 것
    - ROW 에 lock을 획득한다고 보면 됨
    - 정확히는 index에 lock을 잡음

### Auto Increment Lock
- 여러 클라이언트 동시에 데이터를 추가하려고 할 때를 대비함
    - Auto increment 설정 해놨는데 동시에 데이터를 추가하면
    - 같은 PK를 가진 row들이 여러개가 될 수도 있음

### 비관적 Lock vs 낙관적 Lock
- 비관적 Lock : "모든 것이 나쁠 수 있다"를 가정하고, Lock을 미리 걸어 다른 트랜잭션의 접근을 차단
- 낙관적 Lock : "모든 것이 좋을 수 있다"를 가정하고, Lock 없이 트랜잭션 실행 후 충돌 발생하면 Roll Back

## Oracle Lock
- READ할 때 Lock 사용 X -> 타 DB에 비해 성능 좋음


## Isolation level(격리수준)
1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE

- 사전세팅
    1. `SET SESSION autocommit = 0;`
    2. `SHOW VARIABLES LIKE 'autocommit';`
    3. `SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;`
    4. `SHOW GLOBAL VARIABLES LIKE 'transaction_isolation';`

### READ UNCOMMITTED - DIRTY READ
- `insert into emails (recipient_id, body, unread_flag) values(2, 'Hello', true)`
- `select body, unread_flag from emails where recipient_id = 2 limit 50`
- `select unread from mailboxes where recipient_id = 2`
- `update mailboxes set unread = unread + 1 where recipient_id = 2`

- **Dirty Read**
    - 일관성 보장 X
    - Commit 되지 않은 데이터를 읽음

- Commit 전에 read
    - 읽지 않은 이메일의 내용은 확인 가능하지만
    - 읽지 않은 이메일의 개수는 0
    - 만약 2번째 query가 commit되지 않는다면 에러가 계속됨
- Need for Atomicity
    1. 2번째가 실패하면 1번도 날아감
    2. transaction의 단위를 결정해야함
        - 일반적으로 TCP connection
        - BEGIN TRANSACTION; ~ COMMIT;

- `START TRANSACTION;`
- `INSERT INTO iso (name) VALUES (‘kim’);`
- `SELECT * FROM iso;`
- `COMMIT;`

### READ COMMITTED - NON-REPEATABLE READ
- **NON-REPEATABLE READ (READ SKEW)**
    - 트랜잭션이 데이터를 읽었을 때 그 데이터가 다른 트랜잭션에 의해 변경되는 경로, 동일한 데이터를 여러 번 읽을 때 값이 달라질 수 있다.
    - READ COMMITTED의 한계
    - 같은 transaction인데 읽어오는 값이 다름
    - READ가 repeatable 하지 않다

1. COMMIT된 record들만 READ 할 수 있는 것
    - Commit 되지 않은 사항들을 읽을 수 있는 dirty read를 방지함
    - 여러 row를 동시에 수정한다고 할 때, commit되지 않은 사항들을 read하면 아까 이메일 같은 상황 발생함
    - 에러가 발생해서 abort 후 rollback 된 데이터를 read할 위험이 있음
2. COMMIT된 record들만 WRITE 할 수 있는 것
    - Commit 되지 않은 사항들을 수정할 수 있는 dirty write를 방지함
    - 하지만 여러명이 동시에 같은 record를 수정하는 것을 방지할 수는 없음
3. Row-level lock을 사용해서 dirty write를 예방함
    - MySQL 활용 데모
4. write를 시도할 때, lock을 획득해야 함
    - 해당 lock은 transaction이 commit; / abort; 될 때까지 가지고 있음
    - 다른 transaction이 해당 row를 수정하고자 할 때는, 해당 lock을 기다려야 함
5. 하지만 lock을 가지고있는 transaction이 오래걸리면 client가 오래 기다려야 함

- `START TRANSACTION;`
- `UPDATE iso SET name=’lee’ WHERE name=’kim’;`
- `SELECT * FROM iso;`
- **Undo Log 기록 : 'kim'**
- `COMMIT;`

- 문제점
    1. consistency가 깨진다
    2. backup 할 때
        - backup하는 중에 데이터가 계속 변경된다면?
        - backup 중에 변경된 데이터가 rollback 된다면?
        - backup은 온전한 backup이 아님
    3. Analytic queries and integrity checks
        - 데이터 분석할 때 데이터를 불러오는데
        - 일부는 수정된 데이터고, 일부는 기존 데이터라면?
        - read하는 중에 에러가 발생해서 갑자기 데이터가 변경된다면?

### REPEATABLE READ
- **PHANTOM READ (WRITE SKEW)**
    - 다른 트랜잭션이 데이터를 추가하거나 삭제하여, 트랜잭션이 실행 중에 쿼리 결과가 달라지는 현상

1. Snapshot isolation
    - transaction이 발생하는 시점에 `snapshot`을 떠두고,
    - 해당 `snapshot`을 가지고 read/write를 하는 것
    - Multi Version Concurrency Control
        - transaction 별로 auto-increment id를 부여하고
        - 해당 transaction id보다 낮은 숫자의 snapshot은 제거함
2. Write lock을 사용해서 dirty write를 방지
    - 하지만 read는 snapshot 뜨기 때문에 별도의 lock을 사용하지 않음
    - read는 write를 block 하지 않고
    - write는 read를 block 하지 않음
    - 트랜잭션이 데이터를 한 번 읽은 후에는 다른 트랜잭션이 그 데이터를 변경할 수 없다. 한 트랜잭션 내에서 읽은 데이터는 동일하게 유지된다.

- `START TRANSACTION;(trx-id=6)`
- `START TRANSACTION;(trx-id=13)`
- `SELECT * FROM iso FOR UPDATE;`
- `UPDATE iso SET name=’lee’ WHERE name=’kim’;`
- **Undo Log 기록 : 'kim', trx-id = 6**
- `COMMIT;`
- `SELECT * FROM iso;(trx-id=13)`

- Write Skew
    1. `SELECT`를 통해 현재 상태를 확인하고
    2. 현재 상태를 기점으로 어플리케이션 코드에서 다음 쿼리를 결정함
    3. 어플리케이션 코드의 결정에 따라 `INSERT`, `UPDATE`, `DELETE` 실행
- Phantoms causing write skew
    1. Write skew는 아래와 같은 패턴을 갖는다
        - `SELECT`를 통해 현재 상태를 확인하고
        - 현재 상태를 기점으로 어플리케이션 코드에서 다음 쿼리를 결정함
        - 어플리케이션 코드의 결정에 따라 `INSERT`, `UPDATE`, `DELETE` 실행
    2. `phantom`은 하나의 Transaction의 결과가 다른 Transaction에 영향을 미치는
    것
        - 1-a는 다른 transaction의 쿼리 결과에 따라 다른 결과를 리턴할 수 있음
        - 1-a의 결과가 달라지면 1-b는 기존과 다른 결정을 내릴 수 있음
    3. Snapshot isolation은 `read`에서는 Phantom을 차단할 수 있으나, write에서는 불가능
- Write skew 고려사항
    1. transaction들이 다른 record를 업데이트 함
    a. Dirty write도 아니고
    b. Lost update도 아님
    2. 두개의 transaction이 같은 record들을 read하고 겹치지 않게 일부만 update
    a. single-object에서 봤던 atomic write operation과 무관함
    b. Isolation level에서 자동으로 detect 불가능
    3. 해당 transaction과 관련있는 row들에 lock을 걸어야함
- Write skew 예제
    ex) 회의실 예약 : 특정 시간대의 `count(*)`를 확인하고 비어있으면 예약
    - `BEGIN TRANSACTION;`
    - `SELECT COUNT(*) FROM bookings WHERE room_id = 123 AND ent_time > '2015-01-01 12:00' AND start_time < '2015-01-01 13:00';`
    - `INSERT INTO bookings (room_id, start_time, end_time, user_id) VALUES (123, '2015-01-01 12:00', '2015-01-01 13:00', 666);`
    - `COMMIT;`

### SERIALIZABLE
- **DEAD LOCK**
    - 다른 트랜잭션이 데이터를 읽거나 수정할 수 없도록 모든 트랜잭션을 직렬화, 순차적 실행
- 하나의 트랜잭션에서 락을 가지고 있는 레코드에 다른 트랜잭션이 접근할 수 없음
- 무조건 LOCK을 획득해야함
- InnoDB에서는 필요없음


## Preventing Lost Updates
1. 지금까지는 READ에서 에러가 발생하는 경우
2. 이번에는 여러개의 transaction들이 동시에 write를 시도하는 경우
    - 두개의 transaction이 동시에 Read -> modify -> write 할 때
    - 특정 transaction의 write가 무시될 수 있음
3. 코드로 처리할지, 사용자가 결정할지

### Atomic write operations
1. UPDATE counters SET value = value + 1 WHERE key = ‘foo’;
2. 가능하다면 제일 safe
3. ORM에서 지원하지 않는 경우가 많음

### Automatically detecting lost updates
1. lock을 걸지 않고, update를 동시에 진행하게 함
2. Transaction manager가 이를 파악하고
    - Concurrent transaction들을 모두 abort하고
    - retry하도록 강제함
3. Postgre, Oracle 가능함

### CAS(Compare-and-set)
1. update전에 값이 변경된 이력이 있는지 확인하고 Lock 없이 update를 진행함
2. 같은 transaction이라도
    - 처음 read했을 때 값과
    - update시도했을 때 값이 다르다면
    - 업데이트를 반영하지 않음

### Conflict resolution and replication
1. 분산시스템의 경우 여러곳에 퍼져있는 복제 데이터가 동시에 업데이트 될 수 있음
2. 특히 리더가 여러 개인 경우 발생
3. Concurrent update가 발생하면
    - conflict를 발생하게 두고
    - 어플리케이션 단에서 어떤 것이 옳은 update인지 결정하도록 함
4. Last Write Wins라는 항상 최종 업데이트를 승인하는 방향은 lost update 쉽게 발생함

### Single-object writes - 20KB JSON
1. 고려할 케이스
    - 10KB 전송 후 네트워크 에러
    - 업데이트 중간에 power fail
    - write중에 누군가가 read를 시도한다면?
2. Atomicity + Isolation
    - Crash recovery를 위한 log 사용
    - lock을 사용한 isolation
3. Compare-and-set
    - 누군가 update중이 아닐때만 작동하도록

### Multi-object transactions
1. 많은 분산 데이터 시스템은 multi-object transaction을 지원하지 않음
    - 파티션 전체를 아우르면서 구현이 어렵고
    - HA가 필요한 경우가 많기 때문
2. 하지만 그래도 구현하고 싶다면
    - 관계형 데이터 모델
        - 1-n relationship에서는 foreign key 유효성을 검증할 수 있음
        - update할 때 foreign key 상태를 확인하는 방식
    - document형 데이터 모델
        - 업데이트할 record들이 일반적으로 같은 document에 존재
        - 하지만 denormalization으로 인해 데이터가 흩어져있다면 모두 찾아서 업데이트 해야함
            - 위에서 언급한 이메일 unread_count 같은 것
        - Secondary index도 항상 같이 업데이트 되어야 함

### Handling errors and aborts
1. transaction에서 에러가 발생하면 모두 abort하고 retry해야함
2. abort할 케이스를 어떻게 정할지 고민이 필요함
    - transaction은 다 성공했는데 network fail이 발생한다면?
        - 클라이언트에게 알림을 줄 수 없는데 이걸 어떻게 처리해야 하나
    - overload로 인해 에러가 발생한다면 retry해도 똑같이 에러 발생함
3. 이번 write가 실패했다는 것이 명확한 경우
    - Deadlock - 서로 돈을 송금하는 경우
    - isolation violation - dirty reads, non-repeatable reads, phantom reads
    - temporary network failure등에서만 retry를 해야 의미있음

## 참고
- <https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation>
- <https://lealea.tistory.com/>
- <https://github.com/ParyJane/wanted-pre-onboarding-challenge-be-task-February/blob/main/answer.md>
- <https://velog.io/@mingeloper/Redis를-활용한-동시성-이슈-해결>