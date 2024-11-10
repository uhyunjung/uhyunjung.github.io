---
title: '[Spring] Spring Batch 3. 실행 - 3) Step'
date: 2024-11-01 20:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## StepBuilderFactory
1. StepBuilderFactory
    - StepBuilder를 생성하는 팩토리 클래스로서 get(STring name) 메소드 제공
    - StepBuilderFactory.get("stepName")
        - "stepName"으로 Step을 생성

2. StepBuilder
    - Step을 구성하는 설정 조건에 따라 다섯 개의 하위 빌더 클래스를 생성하고 실제 Step 생성을 위임한다.
    - TaskletStepBuilder : TaskletStep을 생성하는 기본 빌더 클래스
    - SimpleStepBuilder : TaskletSTep을 생성하며 내부적으로 청크기반의 작업을 처리하는 ChunkOrientedTasklet 클래스를 생성한다.
    - PartitionStepBuilder : PartitionStp을 생성하며 멀티 쓰레드 방식으로 Job을 실행한다.
    - JobStepBuilder : JobStep을 생성하여 Step 안에서 Job을 실행한다.
    - FlowStepBuilder : FlowStep을 생성하여 STep 안에서 Flow을 실행한다.

3. 구조
    - StepBuilderFactory(get(stepName)) -> StepBuilder
    - API의 파라미터 타입과 구분에 따라 적절한 5가지 하위 빌더가 생성된다.
        -> tasklet(tasklet()) -> TaskletStepBuilder
        -> chunk(chunkSize), chunk(completionPolicy) -> SimpleStepBuilder
        -> partitioner(stepName, partitioner), partitioner(step) -> PartitionStepBuilder
        -> job(job) -> JobStepBuilder
        -> flow(flow) -> FlowStepBuilder
    - StepBuilderFactory -> StepBuilderHelper(-> CommonStepProperties, AtomicReference`<JobRepository>` -> SimpleJobRepository) <- StepBuilder, AbstractTaskletStepBuilder(<- **SimpleStepBuilder**, **TaskletStepBuilder** -> TaskletStep), **PartitionStepBuilder**(-> PartitionStep), **JobStepBuilder**(-> JobStep), **FlowStepBuilder**(-> FlowStep)
    - JobRepository은 빌더 클래스를 통해 Step 객체에 전달되어 메타데이터를 기록하는데 사용된다.

- `git checkout Part4.2.3`
```java
@RequiredArgsConstructor
@Configuration
public class StepBuilderConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() {
        return this.jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .next(step2())
                .next(step4())
                .next(step5())
//                .next(step3())
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
                .<String, String>chunk(3)
                .reader(() -> null)
                .writer(list -> {})
                .build();
    }
   /* @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .partitioner(step1())
                .gridSize(2)
                .build();
    }
*/
    @Bean
    public Step step4() {
        return stepBuilderFactory.get("step4")
                .job(job())
                .build();
    }

    @Bean
    public Step step5() {
        return stepBuilderFactory.get("step5")
                .flow(flow())
                .build();
    }
    @Bean
    public Job job() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
//                .next(step3())
                .build();
    }
    @Bean
    public Flow flow() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");
        flowBuilder.start(step2()).end();
        return flowBuilder.build();
    }
}
```

- 참고
> StepBuilder, TaskletStepBuilder(AbstractTaskletStepBuilder), SimpleStepBuilder, PartitionStepBuilder, JobStepBilder, FlowStepBuilder

## TaskletStep
1. 기본 개념
    - 스프링 배치에서 제공하는 Step의 구현체로서 Tasklet을 실행시키는 도메인 객체
    - RepeatTemplate를 사용해서 TAsklet의 구문을 트랜잭션 경계 내에서 반복해서 실행함
    - Task 기반과 Chunk 기반으로 나누어서 Tasklet을 실행함

2. Task vs Chunk 기반 비교
    - 스프링 배치에서 Step의 실행 단위는 크게 2가지로 나누어짐
        - Task 기반
            - ItemReader와 ITemWriter와 같은 청크 기반의 작업보다 **단일 작업 기반**으로 처리되는 것이 더 효율적인 경우
            - 주로 Tasklet 구현체를 만들어 사용
            - 대량 처리를 하는 경우 chunk 기반에 비해 더 복잡한 구현 필요
            - Job -> TaskletStep -> RepeatTemplate -> Tasklet -> Business Logic
        - Chunk 기반
            - 하나의 큰 덩어리를 n개씩 나누어서 실행한다는 의미로 **대량 처리**를 하는 경우 효과적으로 설계됨
            - ItemReader, ItemProcessor, ItemWriter를 사용하며 chunk 기반 전용 TAsklet인 ChunkOrientedTasklet 구현체가 제공된다
            - Job -> TaskletStep -> RepeatTemplate -> ChunkOrientedTasklet -> ItemReader, ItemProcessor, ItemWriter

3. 구조
    - StepBuilderFactory -> StepBuilder -> TaskletStepBuilder -> TaskletStep
    
    - `git checkout Part4.2.4.1`
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep") // StepBuilder 생성하는 팩토리, STep 이름을 매개변수로 받음
                .tasklet(Tasklet) // Task 기반 TaskletStep 생성, TaskletStepBuilder 반환
    //            .<String, String> chunk(100) // Chunk 기반 TaskletStep 생성
                .startLimit(10) // Step의 실행 횟수를 설정, 설정한 만큼 실행되고 초과시 오류 발생, 기본값은 INTEGER.MAX_VALUE
                .allowStartIfComplete(true) // Step의 성공, 싪패와 상관없이 항상 Step을 실행하기 위한 설정
                .listener(StepExecutionListener) // Step 라이프 사이클의 특정 시점에 콜백 제공받도록 StepExecutionListener 설정
                .build(); // TaskletStep 생성
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class TaskletStepConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(taskStep())
                    .next(chunkStep())
                    .build();
        }

        @Bean
        public Step taskStep() {
            return stepBuilderFactory.get("taskStep")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
        @Bean
        public Step chunkStep() {
            return stepBuilderFactory.get("chunkStep")
                    .<String, String>chunk(3)
                    .reader(new ListItemReader(Arrays.asList("item1","item2","item3")))
                    .processor(new ItemProcessor<String, String>() {
                        @Override
                        public String process(String item) throws Exception {
                            return item.toUpperCase();
                        }
                    })
                    .writer(list -> {
                        list.forEach(item -> System.out.println(item));
                    })
                    .build();
        }
    }
    ```

    - 참고
    > StepBuilder, StepBuilderHelper, TaskletStep, AbstractTaskStepBuilder, AbstractStep, TransactionTemplate, RepeatTemplate
    > ChunkOrientedTasklet, ListItemReader, SimpleChunkProcessor

4. tasklet()
    - Tasklet 타입의 클래스를 설정한다.
    - Step 내에서 구성되고 실행되는 도메인 객체로서 주로 단일 태스크를 수행하기 위한 것
    - TaskletStep에 의해 반복적으로 수행되며 반환값에 따라 계속 수행 혹은 종료한다.
    - RepeatStatus - Tasklet의 반복 여부 상태 값
        - RepeatStatus.FINISHED - Tasklet 종료, RepeatStatus를 null로 반환하면 RepeatStatus.FINISHED로 해석됨
        - RepeatStatus.CONTINUABLE - Tasklet 반복
        - RepeatStatus.FINISHED가 리턴되거나 실패 예외가 던져지기 전까지 TaskletStep에 의해 while 문 안에서 반복적으로 호출됨(무한루프 주의)
    - 익명 클래스 혹은 구현 클래스를 만들어서 사용한다.
    - 이 메소드를 실행하게 되면 TaskletStepBuilder가 반환되어 관련 API를 설정할 수 있다.
    - Step에 오직 하나의 Tasklet 설정이 가능하며 두 개 이상을 설정했을 경우 마지막에 설정한 객체가 실행된다.
    - Tasklet RepeatStatus execute(StepContribution, ChunkContext);

    - `git checkout Part4.2.4.2`
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        return RepeatStatus.FINISHED;
                    }
                })
                .startLimit(10)
                .allowStartIfComplete(true)
                .listener(StepExecutionListener)
                .build();
    }
    ```
    ```java
    @Component
    public class CustomTasklet implements Tasklet {

        @Override
        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
            String stepName = contribution.getStepExecution().getStepName();
            String jobName = chunkContext.getStepContext().getJobName();

            System.out.println("stepName = " + stepName);
            System.out.println("jobName = " + stepName);
            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);

            return RepeatStatus.FINISHED;
        }
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class TaskletConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;
        private final CustomTasklet customTasklet;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .next(step2())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);
                            return RepeatStatus.FINISHED;
                        }
                    })
                    .build();
        }
        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet(customTasklet)
                    .build();
        }
    }
    ```

    - 참고
    > TaskletStep

5. startLimit() / allowStartIfComplete()
    - startLimit()
        - Step의 실행 횟수를 조정할 수 있다.
        - Step마다 설정할 수 있다.
        - 설정 값을 초과해서 다시 실행하려고 하면 StartLimitExceededException이 발생
        - start-limit의 디폴트 값은 Integer.MAX_VALUE

    - allowStartIfComplete()
        - **재시작 가능한 job에서 step의 이전 성공 여부와 상관없이 항상 step을 실행하기 위한 설정**
        - 실행마다 유효성을 검증하는 step이나 사전 작업이 꼭 필요한 step 등
        - **기본적으로 COMPLETED 상태를 가진 step은 job 재시작 시 실행하지 않고 스킵한다.**
        - **allow-start-if-complete가 "true"로 설정된 step은 항상 실행한다.**
        - Job -> Step -> BatchStatus.COMPLETED -> allowStartIfComplete? -> No -> NextStep -> BatchStatus.COMPLETED -> allowStartIfCompleted? -> Yes -> Tasklet -> Business

    - `git checkout Part4.2.4.3`
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep")
                .tasklet(Tasklet))
                .startLimit(10)
                .allowStartIfComplete(true)
                .listener(StepExecutionListener)
                .build();
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class Limit_AllowConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .next(step2())
                    .build();
        }


        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);
                            return RepeatStatus.FINISHED;
                        }
                    })
                    .allowStartIfComplete(true) // step1 항상 실행
                    .build();
        }

        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);
                            throw new RuntimeException("");
    //                        return RepeatStatus.FINISHED;
                        }
                    })
                    .startLimit(3) // step2 최대 3번 실행
                    .build();
        }
    }
    ```

6. TaskletStep 아키텍처
    - Job -> Step -> RepeatTemplate -> Tasklet -> Business Logic -> exception? -> Yes : Step(반복문 및 Step 종료) or No : RepeatStatus(FINISHED : 반복문 및 Step 종료, CONTINUABLE : RepeatTemplate로 Tasklet 반복)
    - Job(StepExecution -> ExecutionContext : BatchStatus.STARTED, ExitStatus.EXECUTING) -> TaskletStep(StepListener : CompositeStepExecutionListener.beforeStep()) -> RepeatTemplate(RepeatStatus.CONTINUABLE, RepeatStatus.FINISHED) -> Tasklet -> RepeatTemplate(StepListener : CompositeStepExecutionListener.afterStep(), StepExecution : BatchStatus.COMPLETED, ExitStatus.COMPLETED, StepExecutionListener 호출 후 추가적인 exitStatus 상태 업데이트 가능 : exitStatus.and(getCompositeListener().afterStep(stepExecution))) -> TaskletStep -> Job
    - StepBuilderFactory -> StepBuilderHelper(-> CommonStepProperties) <- StepBuilder, AbstractTaskletStepBuilder <- SimpleStepBuilder, **TaskletStepBuilder <- TaskletStep**, PartitionStepBuilder, JobStepBuilder, FlowStepBuilder

    - `git checkout Part4.2.4.4`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class TaskletStepArchitectureConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return this.jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .next(step2())
                    .build();
        }


        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);
                            return RepeatStatus.FINISHED;
                        }
                    })
                    .allowStartIfComplete(true)
                    .build();
        }

        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("stepContribution = " + stepContribution + ", chunkContext = " + chunkContext);
                            throw new RuntimeException("");
    //                        return RepeatStatus.FINISHED;
                        }
                    })
                    .startLimit(3)
                    .build();
        }
    }
    ```

    - 참고
    > SimpleJob, SimpleStepHandler, JobExecution, AbstractStep, TaskletStep

## JobStep
1. 기본 개념
    - Job에 속하는 Step 중 외부의 Job을 포함하고 있는 STep
    - 외부의 Jbo이 실패하면 해당 Step이 실패하므로 결국 최종 기본 Job도 실패한다.
    - 모든 메타데이터는 기본 Job과 외부 Job 별로 각각 저장된다.
    - 커다란 시스템을 작은 모듈로 쪼개고 Job의 흐름을 관리하고자 할 때 사용할 수 있다.

2. 기본 구조
    - StepBuilderFactory -> StepBuilder -> JobStepBuilder -> JobStep
    - ParentJob -> JobStep -> JobLauncher -> ChildJob -> Step1 -> Tasklet -> Status? -> COMPLETED -> Step2 or FAILED -> ParentJob
    - StepBuilderFactory -> StepBuilder -> Job(job) -> **JobStepBuilder**
    - StepBuilderFactory -> StepBuilderHelper(-> CommonStepProperties) <- StepBuilder, AbstractTaskletStepBuilder <- SimpleStepBuilder, TaskletStepBuilder, PartitionStepBuilder, **JobStepBuilder**, FlowStepBuilder

    ```java
    public Step jobStep() {
        return stepBuilderFactory.get("jobStep") // StepBuilder를 생성하는 팩토리, Step의 이름을 매개변수로 받음
                .job(Job) // JobStep 내에서 실행될 Job 설정, JobStepBuilder 반환
                .launcher(JobLauncher) // Job을 실행할 JobLauncher 설정
                .parameterExtractor(JobParametersExtractor) // Step의 ExecutionContext를 Job이 실행되는 데 필요한 JobPArameters로 변환
                .build(); // JobStep 생성
    }
    ```

- `git checkout Part4.2.5`
```java
@RequiredArgsConstructor
@Configuration
public class JobStepConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job parentJob() { // JobInstance(parentJob, childJob), jobStep -> step1 -> step2 실행
        return this.jobBuilderFactory.get("parentJob")
                .start(jobStep(null))
                .next(step2())
                .build();
    }
    @Bean
    public Step jobStep(JobLauncher jobLauncher) {
        return this.stepBuilderFactory.get("jobStep")
                .job(childJob())
                .launcher(jobLauncher)
                .listener(new StepExecutionListener() {
                    @Override
                    public void beforeStep(StepExecution stepExecution) {
                        stepExecution.getExecutionContext().putString("name", "user1");
                    }

                    @Override
                    public ExitStatus afterStep(StepExecution stepExecution) {
                        return null;
                    }
                })
                .parametersExtractor(jobParametersExtractor())
                .build();
    }
    @Bean
    public Job childJob() {
        return this.jobBuilderFactory.get("childJob")
                .start(step1())
                .build();
    }
    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
//                        throw new RuntimeException("step1 was failed"); // BATCH_STEP_EXECUTION jobStep, step1 FAILED, 연쇄적 실패
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        throw new RuntimeException("step2 was failed"); // BATCH_JOB_EXECUTION ParentJob FAILED, childJob COMPLETED
//                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public DefaultJobParametersExtractor jobParametersExtractor() {
        DefaultJobParametersExtractor extractor = new DefaultJobParametersExtractor();
        extractor.setKeys(new String[]{"name"}); // String
        return extractor;
    }
}
```

- Run Configuration
> Program arguments : --job.name=parentJob date=20240101

- 참고
> JobStep, DefaultJobParametersExtractor