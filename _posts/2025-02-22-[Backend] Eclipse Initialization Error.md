---
title: '[Backend] Eclipse Initialization Error'
date: 2025-02-22 11:10:00 +0900
categories: [백엔드, 오늘의 에러]
tags: [Backend]
math: true
mermaid: true
---

## 문제
**Eclipse에서 git clone하여 import한 프로젝트 실행 불가능**

```
Error occurred during initialization of boot layer
java.lang.LayerInstantiationException: Package jdk.internal.jimage in both module jrt.fs and module java.base
```

## 해결 과정
### Run Configuration 수정
- main 클래스 등록

## 결과
**Run Configuration 수정**