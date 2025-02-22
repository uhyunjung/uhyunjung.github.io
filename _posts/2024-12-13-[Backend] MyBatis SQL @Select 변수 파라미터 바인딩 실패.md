---
title: '[Backend] MyBatis SQL @Select 변수 파라미터 바인딩 실패'
date: 2024-12-13 15:30:00 +0900
categories: [백엔드, 오늘의 에러]
tags: [Backend]
math: true
mermaid: true
---

## 문제
> 파라미터 바인딩 실패

## 해결 과정
### 1. #{item1}, #{item2} -> 불가능
- item1 = 'A', 'B'
- item2 = A", "B
- **#{}**는 SQL 파라미터 바인딩 방식으로, IN 절에서 각 값에 대해 ?를 사용하여 전달하므로 콤마로 구분된 값들을 처리하기 어렵다.
```
and ("'A', 'B'" is null or item in ("A", "B"))
```

### 2. #{item}, ${item} -> 가능하나, 보안 취약
- item = 'A', 'B'
- **${}** 문자열 처리 방식으로 해당 조건이 처리 가능하나, 인젝션 때문에 보안이 취약해진다.
```
and ("'A', 'B'" is null or item in ('A', 'B'))
```

### 3. #{item}, #{itemList}(List, String[]) -> 불가능
- item = 'A', 'B'
- itemList = ['A', 'B']
```
and "'A', 'B'" is null or item in ('** STREAM DATA **')
```

### 4. #{item}, foreach -> 가능하나, 쿼리 길어짐짐
- item = 'A', 'B'
```
and "'A', 'B'" is null or item in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
```

### 5. find_in_set() -> 가능
- items = 'A', 'B'
```
and ("'A', 'B'" is null or find_in_set(item, #{items}) > 0)
```

## 결과
- find_in_set 함수를 사용한다.