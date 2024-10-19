---
title: '[Spring] Spring Batch 1. 시작'
date: 2024-10-19 20:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Spring Batch 소개
- **Spring Batch Docs** <https://docs.spring.io/spring-batch/><br/>

- 대용량 데이터를 처리하기 위한 프레임워크
- 탄생 배경 : 자바 기반 표준 배치 기술 부재(JSR : IO, Network, Thread, JDBC 등), SpringSource(Pivotal)+Accenture 합작
- 배치 핵심 패턴 : Read, Process(데이터 가공), Write = DB의 ETL(Extract Transform Load)
- 배치 시나리오
  1. 배치 프로세스를 주기적으로 커밋(데이터 단위, 최소로 최대의 효율 전략)
  2. 동시 다발적인 Job의 배치 처리, 대용량 병렬 처리(멀티 Thread)
  3. 실패 후 수동 또는 스케줄링에 의한 재시작
  4. 의존관계가 있는 step 여러 개를 순차적으로 처리
  5. 조건적 Flow 구성을 통한 체계적이고 유연한 배치 모델 구성
  6. 반복, 재시도, Skip 처리

- 아키텍처(org.springframework.batch:spring-batch-*)
  1. **Application** : 개발자가 만든 모든 배치 Job 과정과 커스텀 코드
  2. **Batch Core** : Job 실행, 모니터링, 관리하는 API ex) JobLauncher, Job, Step, Flow 등
  3. **Batch Infrastructure** : Application, Core 모두 공통 Infrastructure 위에서 빌드, Job 실행의 흐름과 처리를 위한 툴 제공 ex) Reader, Processor, Writer, Skip Retry 등

## Dependency
> spring-boot-starter-batch<br/>
spring-batch-test<br/>
spring-boot-starter-test<br/>
spring-boot-configuration-processor<br/>
lombok<br/>
h2

## 프로젝트 구조
- 스프링 배치 활성화 `@EnableBatchProcessing` 추가<br/>

```java
@SpringBootApplication
@EnableBatchProcessing
public class SpringBatchLectureApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBatchLectureApplication.class, args);
    }

}
```

- 총 4개의 설정 클래스 실행, 스프링 배치의 모든 초기화 및 실행 구성이 이루어진다.
- 스프링 부트 배치의 자동 설정 클래스가 실행됨으로 빔으로 등록된 모든 Job을 검색해서 초기화와 동시에 Job을 수행하도록 구성된다.
- DB 스키마 필수

## 스프링 배치 초기화 설정 클래스
> `@EnableBatchProcessing` -> `BatchConfigurationSelector` -> `SimpleBatchConfiguration` -> `BatchConfigureConfiguration(BasicBatchConfigurer, JPABatchConfigurer)` -> `BatchAutoConfiguration(JobLauncherApplicationRunner)`

1. `BatchAutoConfiguration`
  - 스프링 배치가 초기화될 때 자동으로 실행되는 설정 클래스
  - Job을 수행하는 `JobLauncherApplicationRunner` 빈을 생성
2. `SimpleBatchConfiguration`
  - `JobBuilderFactory`와 `StepBuilderFactory` 생성
  - 스프링 배치의 주요 구성 요소 생성
  - 프록시 객체로 생성됨
3. `BatchConfigureConfiguration`
  1) `BasicBatchConfigurer`
    - `SimpleBatchConfiguration`에서 생성한 프록시 객체의 실제 대상 객체를 생성하는 설정 클래스
    - 빈으로 의존성 주입 받아서 주요 객체들을 참조해서 사용할 수 있다.
  2) `JpaBatchConfigurer`
    - JPA 관련 객체를 생성하는 설정 클래스
    - `BasicBatchConfigurer` 상속 받음
  - 사용자 정의 `BatchConfigurer` 인터페이스를 구현하여 사용할 수 있음

## Hello Spring Batch 시작
1. `@Configuration` 선언 : 하나의 배치 Job을 정의하고 빈 설정
2. `JobBuilderFactory` : Job을 생성하는 빌더 팩토리
3. `StepBuilderFactory` : Step을 생성하는 빌더 팩토리
4. Job(일, 일감) : helloJob 이름으로 Job 생성
5. Step(일의 항목, 단계) : helloStep 이름으로 Step 생성
6. tasklet(작업 내용) : Step 안에서 단일 태스크로 수행되는 로직 구현
7. Job 구동 -> Step 실행 -> Step 구동 -> Taskelt 실행

```java
@RequiredArgsConstructor
@Configuration
public class HelloJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job helloJob() { // 4. Job 객체 생성
        return this.jobBuilderFactory.get("helloJob")
                .start(helloStep1()) // Step 인자 필수
                .next(helloStep2())
                .build();
    }

    @Bean
    public Step helloStep1() { // 5. Step 객체 생성
        return stepBuilderFactory.get("helloStep1")
                .tasklet(new Tasklet() { // 6. Tasklet 객체 생성(람다식)
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        System.out.println(" ============================");
                        System.out.println(" >> Hello Spring Batch");
                        System.out.println(" ============================");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    public Step helloStep2() {
        return stepBuilderFactory.get("helloStep2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(" ============================");
                    System.out.println(" >> Step2 has executed");
                    System.out.println(" ============================");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

## DB 스키마 생성 및 이해
1. 스프링 배치 메타 데이터
  - 스프링 배치의 실행 및 관리를 위한 목적으로 여러 도메인들(Job, Step, JobParameters...)의 정보들을 저장, 업데이트, 조회할 수 있는 스키마 제공
  - 과거, 현재의 실행에 대한 세세한 정보, 실행에 대한 성공과 실패 여부 등을 일목요연하게 관리함으로서 배치운용에 있어 리스크 발생시 빠른 대처 가능
  - DB와 연동할 경우 필수적으로 메타 테이블이 생성 되어야 함

2. DB 스키마 제공
  - `/org/springframework/batch/core/schema-*.sql`
  - **DB Schema** <https://docs.spring.io/spring-batch/docs/3.0.x/reference/html/metaDataSchema.html>

3. 스키마 생성 설정(applicaion.yml or application.properties)
  - `spring.batch.jdbc.initialize-schema` 설정
    - `ALWAYS` : 스크립트 항상 실행, RDBMS 설정이 되어 있을 경우 내장 DB보다 우선적으로 실행, 개발할 때 사용
    - `EMBEDDED` : 내장 DB일 때만 실행되며 스키마가 자동 생성됨, 기본값
    - `NEVER` : 스크립트 항상 실행X, 내장 DB일 경우, 스크립트가 생성이 안 되기 때문에 오류 발생, 운영에서 수동으로 스크립트 생성 후 설정하는 것을 권장
4. `DBJobConfiguration` 생성
```java
@RequiredArgsConstructor
@Configuration
public class DBJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job helloJob() {
        return this.jobBuilderFactory.get("Job") // job_name
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("step2 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("step3 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

- `never`인 상태에서 Table이 존재하지 않으면 `Table does not exists` 에러 발생

## DB 스키마
1. Job 관련 테이블
  - **BATCH_JOB_INSTANCE** : Job이 실행될 때 JobInstance 정보가 저장되며 job_name과 job_key(job_name과 job_Parameter 해싱)를 키로 하여 하나의 데이터가 저장, 동일한 job_name과 job_key로 중복 저장될 수 없다.
  - **BATCH_JOB_EXECUTION** : Job의 실행정보가 저장되며 Job 생성, 시작, 종료 시간(오류로 중단되면 NULL), 실행 상태, 메시지 등 관리, Job Instance와 Job Excuetion이 같은 실행은 DB에 하나만 저장
  - **BATCH_JOB_EXECUTION_PARAMS** : Job과 함께 실행되는 JobParameter 정보 저장
  - **BATCH_JOB_EXECUTION_CONTEXT** : Job의 실행 동안 여러 가지 상태 정보, 공유 데이터를 직렬화(Json 형식) 해서 저장, Step 간 서로 공유 가능
  
2. Step 관련 테이블
  - **BATCH_STEP_EXECUTION** : Step의 실행 정보가 저장되며, 생성, 시작, 종료 시간, 실행 상태, 메시지 등 관리
  - **BATCH_STEP_EXECUTION_CONTEXT** : Step의 실행 동안 여러 가지 상태 정보, 공유 데이터를 직렬화(Json 형식) 해서 저장, Step별로 저장되며 Step간 서로 공유할 수 없음

- BATCH_JOB_EXECUTION과 BATCH_STEP_EXECUTION은 일대다 관계