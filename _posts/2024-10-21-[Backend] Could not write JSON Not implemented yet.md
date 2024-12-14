---
title: '[Backend] Could not write JSON Not implemented yet'
date: 2024-10-21 18:10:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## 문제
ResultHandler를 이용해 엑셀 다운로드 API 구현
> org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Not implemented yet at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.writeInternal(AbstractJackson2HttpMessageConverter.java:492)
> com.fasterxml.jackson.databind.jsonmappingexception not implemented

## 개념
- [**@Controller와 @RestController 비교**](https://mangkyu.tistory.com/49)

## 해결 과정
### `@RestController` -> `@Controller` 변경
1. `@Controller`
- View를 반환
- 기본 웹 MVC의 컨트롤러

2. `@RestController`
- Json 형태로 객체 데이터를 반환
- @Controller에 @ReponseBody를 붙인 것과 완벽히 동일

## 결과
- Controller의 어노테이션을 잘 확인해야 한다.