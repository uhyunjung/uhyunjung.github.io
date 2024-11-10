---
title: '[Spring] Spring Batch 2. 도메인 이해 - 2) Step'
date: 2024-10-27 20:10:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Step
### 1. Step
1. 기본 개념
  - Batch Job을 구성하는 독립적인 하나의 단계로서 실제 배치 처리를 정의하고 컨트롤하는 데 필요한 모든 정보를 가지고 있는 도메인 객체
  - 단순한 단일 태스크뿐 아니라 입력과 처리 그리고 출력과 관련된 복잡한 비즈니스 로직을 포함하는 모든 설정들을 담고 있다.
  - 배치 작업은 어떻게 구성하고 실행할 것인지 Job의 세부 작업을 Task 기반으로 설정하고 명세해 놓은 객체
  - **모든 Job은 하나 이상의 step으로 구성됨**
2. 기본 구현체
  - `TaskletStep` : 가장 기본이 되는 클래스로서 Tasklet 타입의 구현체들을 제어한다.
  - `PartitionStep` : 멀티 스레드 방식으로 Step을 여러 개로 분리해서 실행한다.
  - `JobStep` : Step 내에서 Job을 실행하도록 한다.
  - `FlowStep` : Step 내에서 Flow를 실행하도록 한다.
3. Step(void execute(StepExecution stepExecution)) : Step을 실행시키는 executed 메소드, 실행결과 상태는 StepExecution에 저장된다. -> AbstractStep(name, startLimit(Step 실행 제한 횟수), allowStartIfComplete(Step 실행이 완료된 후 재실행 여부), stepExecutionListener, jobRepository(Step 메타데이터 저장소)) -> JobStep(Job, JobLauncher), TaskletStep(Steps, Tasklet), FlowStep(Flow), PartitionStep(StepExecutionSplitter, PartitionHandler)
4. Job execute() -> 여러 Step execute() -> 각각 Tasklet(ItemReader, ItemProessor, ItemWriter)
  - API를 설정할 때 Tasklet과 Chunk 기반 클래스를 동시에 설정할 수 없다.
5. API 설정에 따른 각 Step 생성
  1) 직접 생성한 Tasklet 실행
  ```java
  public Step taskletStep() {
    return this.stepBuilderFactory.get("step")
                                  .tasklet(myTasklet())
                                  .build();
  }
  ```
  2) ChunkOrientedTasklet 실행
  ```java
  public Step taskletStep() {
    return this.stepBuilderFactory.get("step")
                                  .<Member, Member> chunk(100)
                                  .reader(reader())
                                  .writer(writer())
                                  .build();
  }
  ```
  3) Step에서 Job 실행
  ```java
  public Step jobStep() {
    return this.stepBuilderFactory.get("step")
                                  .job(job())
                                  .launcher(jobLauncher)
                                  .parametersExtractor(jobParametersExtractor())
                                  .build();
  }
  ```
  4) Step에서 Flow 실행
  ```java
  public Step taskletStep() {
    return this.stepBuilderFactory.get("step")
                                  .flow(myFlow())
                                  .build();
  }
  ```

- `git checkout Part3.2.1`
```java
@RequiredArgsConstructor
@Configuration
public class StepConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() { // tasklet 생성 -> step 생성 -> job 생성 -> job 실행 -> step 실행(execute -> handler) -> tasklet 실행
        return this.jobBuilderFactory.get("Job")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                // .tasklet(new CustomTasklet())
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
```java
public class CustomTasklet implements Tasklet {

  @Override
  public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
    System.out.println("step2 was executed");

    return RepeatStatus.FINISHED;
  }
}
```

### 2. StepExecution
1. 기본 개념
  - Step에 대한 한 번의 시도를 의미하는 객체로서 Step 실행 중에 발생한 정보들을 저장하고 있는 객체
    - 시작시간, 종료시간, 상태(시작됨, 완료, 실패), commit count, rollback count 등의 속성을 가짐
  - Step이 매번 시도될 때마다 생성되며 각 Step 별로 생성된다.
  - Job이 재시작하더라도 이미 성공적으로 완료된 Step은 재실행되지 않고 실패한 Step만 실행된다.
  - 이전 단계 Step이 실패해서 현재 Step을 실행하지 않았다면 StepExecution을 생성하지 않는다. Step이 실제로 시작됐을 때만 StepExecution을 생성한다.
  - JobExecution과의 관계
    - **Step의 StepExecution이 모두 정상적으로 완료되어야 JobExecution이 정상적으로 완료된다.**
    - Step의 StepExecution 중 하나라도 실패하면 JobExecution은 실패한다.
2. `BATCH_STEP_EXECUTION` 테이블과 매핑
  - JobExecution와 StepExecution은 1:M의 관계
  - 하나의 Job에 여러 개의 Step으로 구성했을 경우 각 StepExecution은 하나의 JobExecution을 부모로 가진다.
3. Job -> JobExecution -> 여러 Step -> 여러 StepExecution -> `COMPLETED` -> **Job 최종 `COMPLETED`**
  - Step마다 StepExecution 생성
4. Job -> JobExecution -> 여러 Step -> 여러 StepExecution -> `COMPLETED` or `FAILED` -> **Job 최종 `FAILED`**
  - **이미 `COMPLETED` 성공한 Step은 재실행되지 않음**
  - `BATCH_STEP_EXECUTION`와 `BATCH_JOB_EXECUTION` 저장
5. Entity(id, version) -> StepExecution(JobExecution, stepName, BatchStatus, readCount, writeCount, commitCount, rollbackCount, readSkipCount, processSkipCount, writeSkipCount, filterCount, startTime, endTime, lastUpdated, ExecutionContext, ExitStatus(UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED), failureExceptions)
6. Job(JobParameter) -> JobInstance -> JobExecution(Status) -> StepExecution(Status)

- `git checkout Part3.2.2`
```java
@RequiredArgsConstructor
@Configuration
public class StepExecutionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("Job")
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
                        // throw new RuntimeException("step2 has failed");
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

   /* @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(executionContextTasklet1)
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(executionContextTasklet2)
                .build();
    }
    @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet(executionContextTasklet3)
                .build();
    }*/
}
```

### 3. StepContribution
1. 기본 개념
  - 청크 프로세스의 변경 사항을 버퍼링한 후 StepExecution 상태를 업데이트하는 도메인 객체
  - 청크 커밋 직전에 StepExecution의 apply 메소드를 호출하여 상태를 업데이트 함
  - ExitStatus의 기본 종료코드 외 **사용자 정의 종료코드**를 생성해서 적용할 수 있음
2. 구조
  - StepContribution(readCount, writeCount, filterCount, parentSkipCount, readSkipCount, writeSkipCount, processSkipCount, ExitStatus(UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED), stepExecution)
3. TaskletStep(create()) -> StepExecution -> StepContribution
4. TaskletStep(execute(contribution, chunkContext)) -> ChunkOrientedTasklet(ItemReader, ItemProcessor, ItemWriter) -> StepContribution(청크 프로세스의 변경 사항을 버퍼링, apply(contribution) StepExecution이 완료되는 시점에 속성들 상태 최종 업데이트) -> StepExecution

- `git checkout Part3.2.3`
```java
@RequiredArgsConstructor
@Configuration
public class StepContributionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("Job")
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
                        System.out.println("contribution.getExitStatus(): " + contribution.getExitStatus());
                        System.out.println("contribution.getStepExecution().getStepName(): " + contribution.getStepExecution().getStepName());
                        System.out.println("contribution.getStepExecution().getJobExecution().getJobInstance().getJobName(): " + contribution.getStepExecution().getJobExecution().getJobInstance().getJobName());
                        contribution.setExitStatus(ExitStatus.STOPPED);
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
//                    throw new RuntimeException("step has failed");
                    System.out.println("step2 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```