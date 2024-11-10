---
title: '[Spring] Spring Batch 3. 실행 - 1) 배치 초기화'
date: 2024-10-29 20:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 배치 초기화 설정
1. **`JobLauncherApplicationRunner`**
    - Spring Batch 작업을 시작하는 ApplicationRunner로서 BatchAutoConfiguration에서 생성됨
    - Spring Boot에서 제공하는 ApplicationRunner의 구현체로 어플리케이션이 정상적으로 구동되자마자 실행됨
    - 기본적으로 Bean으로 등록된 모든 Job을 실행시킨다.

2. **`BatchProperties`**
    - Spring Batch의 환경 설정 클래스
    - Job 이름, 스키마 초기화 설정, 테이블 Prefix 등의 값을 설정할 수 있다.
    - application.properties or application.yml 파일에 설정함
    ```yml
    spring:
        batch:
            job:
                names: ${job.name:NONE} # batchJob1으로 작성 가능, job.name이라는 argument가 없으면 NONE 동적 실행
                enabled: true # 생략 가능
            jdbc:
                initialize-schema: never # always
                tablePrefix: SYSTEM_ # Table 접두사
    ```

3. Job 실행 옵션
    - 지정한 Batch Job만 실행하도록 할 수 있음
    - `spring.batch.job.names: ${job.name:NONE}`
    - 어플리케이션 실행시 **Program arguments**로 job 이름 입력한다.
        - `--job.name=helloJob`
        - `--job.name=helloJob,simpleJob` (하나 이상의 Job을 실행할 경우 쉼표로 구분해서 입력함)

- `git checkout Part4.1`
```java
@RequiredArgsConstructor
@Configuration
public class JobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob1() {
        return this.jobBuilderFactory.get("batchJob1")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .next(step2())
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
}
```

- Run Configuration
> Program arguments : `name=user1`
> Program arguments : `--job.name=batchJob1 name=user1` -> executeLocalJobs(JobParameters)에서내가 수행하고 싶은 job만 execute

- 참고
> `BatchAutoConfiguration`, `JobLauncherApplicationRunner`, `BatchProperties`

```java
@RequiredArgsConstructor
@Configuration
public class JobConfiguration2 {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob2() {
        return this.jobBuilderFactory.get("batchJob2")
                .incrementer(new RunIdIncrementer())
                .start(step3())
                .next(step4())
                .build();
    }

    @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("step3 has executed");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step4() {
        return stepBuilderFactory.get("step4")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("step4 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

- Run Configuration
> Program arguments : `--job.name=batchJob1,batchJob2 name=user1`