---
title: '[Spring] Test 가이드'
date: 2024-12-17 08:19:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## 테스트 코드 작성법
- Given-When-Then 패턴 : TDD에서 파생된 BDD(Behavior-Driven-Development)를 통해 탄생한 테스트 접근 방식
- 좋은 테스트를 작성하는 5가지 속성(F.I.R.S.T)

## TDD(Test Driven Development)
- TDD란 Test Driven Development의 준말로, 테스트 주도 개발이란 뜻을 가진다.
- 애자일 방법론 중 하나인 익스트림 프로그래밍(eXtream programming)의 Test-First를 기반
- 테스트 코드를 먼저 작성하고 성공하면 그때 해당 코드를 작성하는 과정을 반복

## TDD 이점
- 디버깅 시간 단축
- 생산성 향상
- 재설계 시간 단축
- 기능 추가와 같은 추가 구현이 용이

## 통합 테스트
### 라이브러리
- **JUnit 5** : 자바 애플리케이션 단위 테스트 지원
- **Spring Test, Spring Boot Test** : 스프링 부트 애플리케이션에 대한 유틸리티, 통합 테스트 지원
- **AssertJ** : 단정문(assert) 지원의 라이브러리
- **Mockito** : 자바 Mock 객체 지원의 프레임워크

- `@Test`
- `@SpringBootTest` : Spring Boot, IntegrationTest
- `@MockBean` : Spring Boot

## 단위 테스트
- 서버를 시작하지 않고 아래 계층들 테스트
- Spring Application Context 없이 테스트

### Controller Test
- MVC 테스트
- 테스트 우선순위 낮음
- `@WebMvcTest` : MockApiTest
- `@MockMvc`: SpringTest, mockMvc.perform()
- `@AutoConfigureMockMvc` : `@Autowired`로 MockMvc 주입

### Service Test
- Mock 테스트
- `@Mock` : Mockito.mock(RepositoryClass)
- `@InjectMocks` : Mockito.mock(ServiceClass)

### Repository Test
- MyBatis 테스트 : 테스트 우선순위 낮음
- JPA 테스트
- `@DataJpaTest` : `@Transactional` 포함, 기본값 임베디드 데이터베이스 사용

### POJO Test
- Domain 테스트

## 세부 모듈
- JUnit5
    - JUnit Platform
    - JUnit Jupiter
    - JUnit Vintage

## Jacoco 테스트 커버리지

### 참고
- <https://medium.com/simform-engineering/testing-spring-boot-applications-best-practices-and-frameworks-6294e1068516>
- <https://spring.io/guides/gs/testing-web>
- <https://github.com/spring-guides/gs-testing-web>
- <https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/html/boot-features-testing.html>
- <https://cheese10yun.github.io/spring-guide-test-1/>
- <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md>
- <https://mangkyu.tistory.com/184>
- <https://github.com/rahul-baghaniya/spring-unit-tests/tree/master/src/test/java/com/baghaniya/springunittests>