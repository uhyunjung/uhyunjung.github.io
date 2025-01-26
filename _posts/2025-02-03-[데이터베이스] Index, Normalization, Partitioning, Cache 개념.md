---
title: '[Backend] Index, Normalization, Partitioning, Cache 개녀'
date: 2025-02-03 15:30:00 +0900
categories: [정리, 데이터베이스]
tags: [데이터베이스]
math: true
mermaid: true
---

## 원티드 프리온보딩 백엔드 챌린지 2월
<https://www.wanted.co.kr/events/pre_challenge_be_4>

## 순차I/O vs 랜덤I/O
- 순차 I/O가 훨씬 빠르나, MySQL 특성상 현실적으로 어려움
    - DELETE 후 INSERT하면 연속적인 데이터들로 인해 효율성 저하됨
- 랜덤I/O를 하되 접근하는 row의 개수를 줄이는 것

## Index
### index의 의미
1. READ 효율을 개선하는 도구
2. 별도의 테이블로 정렬을 관리

- `SHOW INDEX FROM Track;` : Primary Key

### 효율적으로 Index 사용하는 방법
1. Cardinality
```sql
CREATE TABLE reservation (
    id INT(7) AUTO_INCREMENT PRIMARY KEY,
    requested_at DATE,
    num_customer INT(2),
    created_at TIMESTAMP DEFAUKT CURRENT_TIMESTAMP,
    name VARCHAR(16),
    phone VARCHAR(16),
    time TIME
);
```
```sql
DELIMITER $$
CREATE PROCEDURE generate_reservation()
BEGIN
    DECLARE i INT DEFAULT 0;
    WHILE i < 100000000 DO
        INSERT INTO 'reservation'
            ('requested_at', 'num_customer', 'name', 'phone', 'time')
        VALUES (
            FROM_UNIXTIME(ROUND(RAND() * (1651319994-1648814392) + 1648814394))
            FLOOR(RAND()*(10))+1,
            SUBSTRING(MD5(RAND()) FROM 1 FOR 3),
            SUBSTRING(MD5(RAND()) FROM 1 FOR 7),
            SEC_TO_TIME(
                FLOOR(
                    TIME_TO_SEC('15:00:00') + RAND() * (TIME_TO_SEC(TIMEDIFF('22:00:00', '15:00:00')))
                )
            )
        );
        SET i = i + 1;
    END WHILE;
END$$
```
- `CALL generate_reservation();`

2. Update Frequency
    - Index 걸어둔 컬럼이 수정되면 index를 관리하는 테이블도 수정필요
    - 하나씩 밀어낸다고 생각해보면?

3. Size
    - InnoDB : 16KB 단위로 데이터 저장
    - Pointer : 실제는 사이즈의 주소
    - index key size + pointer size

### Index를 효율적으로 사용하는 쿼리
1. LIKE
2. BETWEEN
3. IN
4. SUBSTRING()
5. LOWER()

- Using index condition
    - 'index condition pushdown' 최적화를 사용하는 경우
    - 기존에는 일단 row를 불러오고 나서 WHERE를 확인했는데, WHERE를 기반으로 row를 불러옴
- MRR
    - Multi-Range Read
    - 기존 MySQL의 join
        - Driving table에서 read 실시
        - Driven table에서 해당 row에 해당하는 값을 읽어서 join
    - 발전된 MySQL
        - Driven table 중 하나로 레코드를 읽어와서 join buffer에 버퍼링
        - 조인을 바로 하지 않고 일단 버퍼링 하는 것
- Range : 인덱스 레인지 스캔
    - 가장 이상적인 쿼리
    - 인덱스를 제일 효율적으로 사용한 경우
- Ref : const
    - 조건에 상수가 사용되는 경우

### Cluster Index
1. InnoDB에서만 사용되는 개념
2. 유사한 Primary Key를 가진 row들 끼리 묶어서 저장
    - Primary Key가 변경되면?
    - 물리적인 위치가 변경되어버림
    - Index가 걸려있다면 pointer도 변경됨

## Normalization
- 데이터 정합성
- 중복 데이터 감소

- `DESC orders;`

## Partitioning
- 데이터 분산
- 대용량 테이블을 소규모 테이블로 나누어 저장하는 것
- 하지만 사용자는 하나의 테이블로 read/write를 한다고 생각함

- Range Partitioning : 범위로 나누는 것, 날짜가 대표적
```sql
CREATE TABLE reservation_by_month (
    id INT(7) AUTO_INCREMENT PRIMARY KEY,
    requested_at DATE,
    num_customer INT(2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    name VARCHAR(16),
    phone VARCHAR(16),
    time TIME
)   PARTITION BY RANGE (MONTH (requested)) {
    PARTITION p0 VALUES LESS THAN 3,
    PARTITION p1 VALUES LESS THAN 5,
    PARTITION p2 VALUES LESS THAN 7,
    PARTITION p3 VALUES LESS THAN 9,
    PARTITION p4 VALUES LESS THAN MAXVALUE
};
```
- List Partitioning
    - parameter의 값들을 리스트로 나열함
    - 파티션 키가 고정값인 경우
        - 카테고리
        - 연속되지 않은 값
```sql
CREATE TABLE reservation_by_category (
    requested_at DATE,
    num_customer INT(2),
    category_id INT(5),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    name VARCHAR(16),
    phone VARCHAR(16),
    time TIME
)   PARTITION BY list (category_id) {
    PARTITION p_western VALUES in ('파스타', '스테이크'),
    PARTITION p_korean VALUES in ('불고기', '제육'),
    PARTITION p_japanese VALUES in ('초밥', '회')
};
```

- Hash Partition : Range, list로 균등하게 데이터를 나누기 어려운 경우
```sql
CREATE TABLE reservation (
    id INT(7) AUTO_INCREMENT PRIMARY KEY,
    requested_at DATE,
    num_customer INT(2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    name VARCHAR(16),
    phone VARCHAR(16),
    time TIME
)   PARTITION BY HASH(id) PARTITIONS 4;
```

## Sharding
- 여러 서버에 데이터 분산 저장

## Cache
- 쿼리 효율 개선

## Procedure
- SQL단에서 사용할 수 있는 함수
- 데이터 넣을 때 사용하면 좋음

## Trigger
- 특정 이벤트 발생 시 조건 검사