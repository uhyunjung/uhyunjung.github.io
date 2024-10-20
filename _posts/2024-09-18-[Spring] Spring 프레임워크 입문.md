---
title: '[Spring] Spring 프레임워크 입문'
date: 2024-09-18 20:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

# Spring 프레임워크 입문
> 예제 코드 [**Spring PetClinic**](https://github.com/spring-projects/spring-petclinic)

## Dependency
> spring-boot-devtools<br/>
spring-boot-starter-web<br/>
spring-boot-starter-jdbc<br/>
spring-boot-starter-jpa<br/>
spring-boot-starter-data-jpa<br/>
spring-boot-starter-log4j2<br/>
slf4j-api<br/>
spring-boot-starter-test<br/>
springdoc-openapi-starter-webmvc-ui<br/>
lombok<br/>
mybatis-spring-boot-starter<br/>
mybatis-spring-boot-starter-test<br/>
mysql-connector-java<br/>
mariadb-java-client<br/>
h2<br/>
swagger-annotations

## IoC
  - 의존성에 대한 컨트롤이 바뀜
  - 일반적인 의존성에 대한 제어권 : Controller가 new로 Repository를 만들어야 함 → ControllerTest에서 Controller 생성자로 Repository 전달해야 함
  - Servlet의 제어권 역시 Servlet 자체가 아닌 컨테이너가 가지고 있음

## IoC 컨테이너
  - Application Context
      - IoC 컨테이너 내부에서 Controller와 Repository 객체(Bean) 생성
      - Bean의 의존성 관리
      - Bean Factory 및 여러 개 인터페이스 상속 받음
      - Servlet 3.5부터 Java 지원, XML X → 직접 사용X
      - `@Autowired` ApplicationContext 생성 → applicationContext.getBean(repository.class)로 사용
      - Spring Data JPA 인터페이스가 빈을 생성해 Application Context에 등록하는 라이프사이클 콜백 있음

## Bean
  - IoC 컨테이너가 관리하는 객체(Model은 Bean 아님)
  - 이름 또는 ID, 타입, 스코프
  - 등록 방법
      - Conponent Scanning : 생성자에 주입 `@SpringBootApplication`에서 같은 패키지 컴포넌트들 처리
          - `@Component` 포함하는 Annotation
              - `@Repository` `@Service` `@Controller`
      - 직접 등록
          - `@Configuration` 클래스 안에 `@Bean`으로 return
  - 사용 방법
      - `@Autowired` or `@Inject` or ApplicationContext에서 getBean()
  - Annotation : 주석, 프로세서 따로 존재
  - `@MockBean` : 가짜 객체 주입 ex) 테스트에서 만든 객체 주입 가능

## DI(의존성 주입)
  - **Autowired 없어도** 생성자가 하나만 있고 생성자의 매개변수가 빈으로 등록되어 있다면 자동 주입
  - `@Autowired` or `@Inject` 사용 방법
      - **생성자**
      - 필드
      - Setter

## AOP
  - Single Responsibility Principle 객체 지향 원칙에 적합하게 코딩하기 위함
  - 흩어져 있는 코드를 한 곳으로 모으는 코딩 기법
  - 구현 기법
    - 바이트 코드 조작(.class)
    - 프록시 패턴 사용 : A 클래스 상속 받아 A' 프록시 클래스 생성

  ```java
  //Before
  class A {
    method a () {
      AAAA
      ....
      BBBB
    }

    method b () {
      AAAA
      ....
      BBBB
    }
  }

  class B {
    method c() {
      AAAA
      ....
      BBBB
    }
  }
  ```

  ```java
  //After
  class A {
    method a () { .... }

    method b () { .... }
  }

  class B {
    method c() { .... }
  }

  class AAAABBBB {
    method aaaabbbb(JoinPoint point) {
      AAAA
      point.execute()
      BBBB
    }
  }
  ```

  - `@Transactional(readOnly = true)` : 트랜잭션 매니저에서 트랜잭션을 오토커밋을 False로 만들고 try catch로 Commit -> 문제 생기면 Rollback 
  - `@LogExecutionTime` : 수미상관 식으로 RunTime 동안 실행
  ```java
  // Annotaion 등록
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface LogExcecutionTime {}

  // Bean 등록
  @Component
  @Aspect
  public class LogAspect {
    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("@Aannotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint) throws Throwable {
      StopWatch stopWatch = new StopWatch();
      stopWatch.start();

      Object proceed = joinPoint.proceed();

      stopWatch.stop();
      logger.info(stopWatch.prettyPrint());

      return proceed;
    }
  }
  ```

## PSA(Portable Service Abstraction)
- 나의 코드 + 잘 만든 인터페이스(PSA) + 확장성이 좋지 못한 코드, 기술에 특화되어 있는 코드 : 테스트 용이, 기술 변경에 유연함
- Spring의 API 90% 전부 PSA 추상화
- Aspect에서 트랜잭션 처리, Transaction Manager의 구현체들이 바뀌어도 Aspect 코드는 바뀌지 않음, Aspect 안에서 Transaction Manager 가져다 사용, Platform Transaction Manager 구현체들은 빈으로 등록 됨
- ex) Jpa Datasource Hibernate Transaction Manager

- Spring Cache : `@EnableCaching`으로 `@Cacheable` or `@CacheEvict` 사용 가능
  - _JcacheManager, EhcacheManager_
- Spring Web MVC : Controller와 GetMapping에서 Servlet, Reactive를 쓰는지 알 수 없음
