---
title: '[Backend] Maven Force Update 실패'
date: 2024-10-13 11:10:00 +0900
categories: [오늘의 에러, Backend]
tags: [Backend]
math: true
mermaid: true
---

## 문제
**Maven의 Dependency 인식 X**

## 개념
- [**Maven 명령어 정리**](https://thalals.tistory.com/345)
- [**Maven 명령어 비교**](https://jinlunalee.tistory.com/12)

## 해결 과정
### 1. 사용자 폴더/본인 폴더/.m2 내 Dependency 관련 폴더 삭제 -> 실패
사용자 폴더/본인 폴더/.m2 내 Dependency 관련 폴더 삭제 후
1. Maven의 Download sources and Documents : Dependency 다운로드
2. Update Project : Dependency 인식
3. Maven clean
4. Maven install : Build 실행

### 2. IntelliJ Invalidate Caches -> 실패
1. File > Reload all from disk
2. File > Invalidate Caches

### 3. IntelliJ .idea 폴더 삭제 -> 성공
IntelliJ가 가진 폴더 내의 IDE 설정을 삭제한 후 다시 IntteliJ를 실행하고 Maven Update 과정을 수행한다.

## 결과
**IntelliJ .idea 폴더를 삭제한 후 재실행한다.**