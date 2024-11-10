---
title: '[Spring] Spring Batch 3. 실행 - 2) Job'
date: 2024-10-30 20:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## JobBuilderFactory
1. JobBuilderFactory
    - JobBuilder를 생성하는 팩토리 클래스로서 get(String name) 메소드 제공
    - jobBuilderFactory.get("jobName")
        - "jobName"은 스프링 배치가 Job을 실행시킬 때 참조하는 Job의 이름
    - Bean으로 생성
    
2. JobBuilder
    - Job을 구성하는 설정 조건에 따라 두 개의 하위 빌더 클래스를 생성하고 실제 Job 생성을 위임한다.
    - **SimpleJobBuilder**
        - SimpleJob을 생성하는 Builder 클래스
        - Job 실행과 관련된 여러 설정 API를 제공한다.
        - start(step), next(step), on(String pattern) -> TransitionBuilder, start(decider), next(decider), split(taskExecutor) -> SplitBuilder
    - **FlowJobBuilder**
        - FlowJob을 생성하는 Builder 클래스
        - 내부적으로 FlowBuilder를 반환함으로써 Flow 실행과 관련된 여러 설정 API를 제공한다.
        - start(step or flow), next(step or flow), on(String pattern) -> TransitionBuilder, from(step or flow), start(decider), next(decider), from(decider), 

3. 아키텍처
    - JobBuilderFactory(get(jobName)) -> JobBuilder(start(step)) -> **SimpleJobBuilder** -> SimpleJob 생성
    - JobBuilderFactory(get(jobName)) -> JobBuilder(start(flow), flow(step)) -> **FlowJobBuilder** -> **FlowJob 생성**  -> **JobFlowBuilder** -> **FlowBuilder** -> **Flow 생성**

4. 클래스 상속 구조
    - JobBuilderFactory -> **JobBuilderHelper** -> CommonJobProperties
    - **JobBuilderHelper** -> SimpleJobRepository -> JobInstanceDao, JobExecutionDao, StepExecutionDao, ExecutionContextDao
    - **JobBuilderHelper** <- JobBuilder -> SimpleJobBuilder, FlowJobBuilder -> SimpleJob, FlowJob <- SimpleJobRepository
    - Job Builder가 생성되는 클래스 간의 계층 및 구조를 명확하게 이해하면 Job Configuration을 구성할 때 많은 도움이 된다.
    - JobRepository은 빌더 클래스를 통해 Job 객체에 전달되어 메타데이터를 기록하는데 사용된다.

    - `git checkout Part4.1`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class JobBuilderConfiguration {

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
        public Job batchJob2() {
            return this.jobBuilderFactory.get("batchJob1")
                    .incrementer(new RunIdIncrementer())
                    .start(flow())
                    .next(step2())
                    .end()
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step1 has executed");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step2 has executed");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }

        @Bean
        public Flow flow() {
            FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");
            flowBuilder.start(step3())
                    .next(step4())
                    .end();
            return flowBuilder.build();
        }

        @Bean
        public Step step3() {
            return stepBuilderFactory.get("step3")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(">> step3 has executed");
                            return RepeatStatus.FINISHED;
                        }
                    })
                    .build();
        }
        @Bean
        public Step step4() {
            return stepBuilderFactory.get("step4")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step4 has executed");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    }
    ```

    - 참고
    > `SimpleBatchConfiguration.java`

## SimpleJob
1. 기본 개념
    - SimpleJob은 STep을 실행시키는 Job 구현체로서 SimpleJobBuilder에 의해 생성된다.
    - 여러 단계의 Step으로 구성할 수 있으며 Step을 순차적으로 실행시킨다.
    - 모든 Step의 실행이 성공적으로 완료되어야 Job이 성공적으로 완료된다.
    - 맨 마지막에 실행한 Step의 BatchStatus가 Job의 최종 BatchStatus가 된다.

2. 흐름
    - SimpleJob -> COMPLETED -> Step -> COMPLETED -> Step -> COMPLETED -> SimpleJob
    - SimpleJob -> FAILED -> Step -> FAILED -> SimpleJob (다음 Step 실행되지 않음)
    - JobBuilderFactory > JobBuilder > SimpleJobBuilder > SimpleJob

    ```java
    public Job batchJob() {
        return jobBuilderFactory.get("batchJob") // JobBuilder를 생성하는 팩토리
                .start(step) // SimpleJobBuilder 반환
                .next(step) // 횟수 제한 없음
                .incrementer(JobParametersIncrementer) // JobParameter 자동 증가
                .preventRestart(true) // Job 재시작 가능 여부 설정, 기본값 true
                .validator(JobParameterValidator) // JobParameter(key, value)를 실행하기 전에 올바른 구성이 되었는지 검증하는 JobParametersValidator 설정
                .listener(JobExecutionListener) // Job 라이프 사이클의 특정 시점에 콜백 제공받도록 JobExecutionListener 설정
                .build(); // SimpleJob 설정
    }
    ```

    - `git checkout Part4.2.2.1`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class SimpleJobConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .incrementer(new RunIdIncrementer())
                    .validator(new JobParametersValidator() {
                        @Override
                        public void validate(JobParameters parameters) throws JobParametersInvalidException {

                        }
                    })
                    .preventRestart()
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .build();
        }
        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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
    //                    throw new RuntimeException("failed");
                        chunkContext.getStepContext().getStepExecution().setStatus(BatchStatus.FAILED);
                        contribution.setExitStatus(ExitStatus.STOPPED); // 배치상태 FAILED, 종료코드 STOPPED
                        System.out.println("step3 has executed");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    }
    ```

    - 참고
    > `SimpleJobBuilder` -> `JobBuilderHelper` -> `SimpleJobLauncher`

3. start()/next()
    - 여러 번 설정 가능하며 모든 next()의 step이 종료되면 job이 종료된다.
    - SimpleJobBuilder, FlowJobBuilder, JobFlowBuilder 호출

    - `git checkout Part4.2.2.2`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class StartNextConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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

4. validator()
    - Job 실행에 꼭 필요한 파라미터를 검증하는 용도
    - DefaultJobParametersValidator 구현체를 지원하며, 좀 더 복잡한 제약 조건이 있다면 인터페이스를 직접 구현할 수도 있음
    - JobParametersValidator(void validate(@Nullable JobParameters parameters)) : JobParameters 값을 매개변수로 받아 검증함
    - `DefaultJobParametersValidator`
        - SimpleJob -> validate() -> JobParametersValidator(**requiredKeys: {"date", "name"}**), optionalKeys: {"count"} -> JobParameters({"date=20240101", "name=name", "count=1"}) -> `Validation Success`(필수 key 모두 존재, 옵션 key 없어도 됨) or `JobParametersInvalidException`(필수 key인 date or name 없음)

    - `git checkout Part4.2.2.3`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class ValidatorConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .validator(new CustomJobParametersValidator())
    //                .validator(new DefaultJobParametersValidator(new String[]{"name"},new String[]{"year"}))
    //                .validator(new DefaultJobParametersValidator(new String[]{"date","name"},new String[]{"count","year"}))
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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
    ```java
    public class CustomJobParametersValidator implements JobParametersValidator {

        @Override
        public void validate(JobParameters jobParameters) throws JobParametersInvalidException {

            if (jobParameters.getString("name") == null) {

                throw new JobParametersInvalidException("name parameter is not found.");

            }
        }
    }
    ```

    - Run Configuration
    > Program arguments : `--job.name=batchJob name=user1`

    - 참고
    > `SimpleJobLauncher(getJobParametersValidator())` -> `DefaultJobParametersValidator` or `CustomJobParametersValidator` -> `SimpleJob` -> `AbstractJob(setJobParametersValidator())`

5. preventRestart()
    - Job의 재시작 여부 설정
    - preventRestart() 자체가 재시작 여부 금지 의미
    - 기본값(**restartable**)은 true이며 false로 설정 시 "이 Job은 재시작을 지원하지 않는다"라는 의미
    - Job이 실패해도 재시작이 안 되며 Job을 재시작하려고 하면 JobRestartExeption 발생
    - 재시작과 관련 있는 기능으로 Job을 처음 실행하는 것과는 아무런 상관 없음
    - Job -> exist JobExecution? -> JobInstance -> JobExecution -> Business -> preventRestart? -> JobRestartException
    - Job의 실행이 처음이 아닌 경우는 Job의 성공/실패와 상관없이 오직 preventRestart 설정 값에 따라서 실행 여부를 판단한다.

    - `git checkout Part4.2.2.4`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class PreventRestartConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .preventRestart() // 재시작 금지
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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
                        throw new RuntimeException("step3 has failed"); // 재시작 금지
    //                    return RepeatStatus.FINISHED; // 재시작 가능
                    })
                    .build();
        }
    }
    ```

    - 참고
    > `JobBuilderHelper(CommonJobProperties)` -> `SimpleJob` -> `AbstractJob(restartable = true)` -> `SimpleJobLauncher(lastExecution != null)`

6. incrementer()
    - JobParameters에서 필요한 값을 증가시켜 다음에 사용될 JobParameters 오브젝트를 리턴
    - 기존의 JobParameter 변경없이 Job을 여러 번 시작하고자 할 때
    - RunIdIncrementer 구현체를 지원하며 인터페이스를 직접 구현할 수 있음
    - JobParametersIncrementer(getNext(@Nullable JobParameters parameters))

    - `git checkout Part4.2.2.5`
    ```java
    @Override
    public JobParameters getNext(@Nullable JobParameters parameters) {
        JobParameters params = (parameters == null)? new JobParameters() : parameters;

        long id = params.getLong(key, new Long(0)) + 1;
        return new JobParametersBuilder(params).addLong(key, id).toJobParameters();
    }
    ```
    ```java
    public class CustomJobParametersIncrementer implements JobParametersIncrementer {

        static final SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd-hhmmss");

        public JobParameters getNext(JobParameters parameters) {

            String id = format.format(new Date()); // next()할 때마다 항상 변함

            return new JobParametersBuilder().addString("run.id", id).toJobParameters(); // date, name, run.id Parameter와 BATCH_JOB_EXECUTION_PARAMS DB에 저장
        }
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class IncrementerConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
    //                .incrementer(new RunIdIncrementer()) // 테스트를 위해 자주 사용
                    .incrementer(new CustomJobParametersIncrementer())
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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

    - Run Configuration
    > Program arguments : `--job.name=batchJob name=user1 date=20240101`
    > DB 초기화 X

    - 참고
    > `JobBuilderHelper(CommonJobProperties)` -> `SimpleJob` -> `JobParametersBuilder`

7. SimpleJob 아키텍처
    - Job -> Step -> Tasklet : 각각 실행할 때 메타데이터 DB에 저장
    - JobLauncher -> **JobParameters** + SimpleJob -> Batch 실행(Step -> Tasklet 반복) -> SimpleJob -> JobLauncher
    - JobLunahcer -> **JobInstance** : **JobExecution(ExecutionContext(updateStatus(execution, BatchStatus.STARTED)))** = 1대다 관계-> SimpleJob(**JobExecutionListener.beforeJob()**) -> StepExecution(ExecutionContext) -> SimpleJob(**JobExecutionListener.afterJob()**) -> **JobExecution(stepExecution.getStatus(), stepExecution.getExitStatus())** -> JobLauncher
    - 클래스 상속 관계도
        - JobBuilderFactory -> JobBuilderHelper(->CommonJobProperties, AtomicReference`<JobRepository>`) <- SimpleJobBuilder -> SimpleJob, JobBuilder, FlowJobBuilder -> FlowJobs
    
    - `git checkout Part4.2.2.6`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class SimpleJobArchitectureConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .incrementer(new RunIdIncrementer())
                    .start(step1())
                    .next(step2())
                    .listener(new CustomJobListener())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
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
    public class CustomJobListener implements JobExecutionListener {

        @Override
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("jobExecution = " + jobExecution);
        }

        @Override
        public void afterJob(JobExecution jobExecution) {
            System.out.println("jobExecution = " + jobExecution);
        }
    }
    ```

    - 참고
    > SimpleJobLauncher(SimpleJobRepository(getLastJobExecution, getJobInstance), Parameters(DefaultJobParametersValidators), **createJobExecution, createJobInstance, ExecutionContext**)
    > AbstractJob(updateStatus, listener.before(), doExecute) -> SimpleJob(**updateStatus(step)**) -> AbstractJob(listener.after(), updateStatus)