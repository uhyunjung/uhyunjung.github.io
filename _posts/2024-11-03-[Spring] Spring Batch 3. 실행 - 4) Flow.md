---
title: '[Spring] Spring Batch 3. 실행 - 4) Flow'
date: 2024-11-03 22:30:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## FlowJob
1. 기본 개념
    - Step을 순차적으로만 구성하는 것이 아닌 **특정한 상태에 따라 흐름을 전환하도록 구성**할 수 있으며 FlowJobBuilder에 의해 생성된다.
        - Step이 실패하더라도 Job은 실패로 끝나지 않도록 해야 하는 경우
        - Step이 성공했을 때 다음에 실행해야 할 STep을 구분해서 실행해야 하는 경우
        - 특정 Step은 전혀 실행되니 않게 구성해야 하는 경우
    - Flow와 Job의 흐름을 구성하는데만 관여하고 실제 비즈니스 로직은 Step에서 이루어진다.
    - 내부적으로 SimpleFlow 객체를 포함하고 있으며 Job 실행 시 호출한다.

2. SimpleJob vs FlowJob
    - SimpleJob - 순차적 흐름
        - Step A -> Step B -> Step C : Step A 먼저 실행, Step A가 실패하면 전체 Job 실패, Step B와 Step C 실행되지 않음
    - FlowJob - 조건적 흐름
        - Step A -> Success ? -> Yes -> Flow or No -> Step B : Step A 가장 먼저 실행, Step A가 성공하면 Flow 실행, Step A가 실패하면 Step B 실행, 성공/실패 모두 Job이 성공함

3. 구조
    - JobBuilderFactory -> JobBuilder -> **JobFlowBuilder** -> **FlowBuilder** -> FlowJob
    - Flow : 흐름을 정의하는 역할 ex) start, from, next
    - Transition : 조건에 따라 흐름을 전환시키는 역할 ex) on, to, stop, fail, end, stopAndRestart
    - JobBuilderFactory -> JobBuilder -> SimpleJobBuilder, **FlowJobBuilder** -> **JobFlowBuilder**
        - SimpleJobBuilder, TransitionBuilder, FlowBuilder, SplitBuilder, **TransitionBuilder, SplitBuilder**
    - **FlowBuilder** : step(start, next, from), **on(String pattern) -> TransitionBuilder(to, stop, fail, end, stopAndRestart)**, flow(start, next, from), decider(start, next, from), split(Executor)
    - 단순한 Step으로 생성하는 SimpleJob보다 다양한 Flow로 구성하는 FlowJob의 생성 구조가 더 복잡하고 많은 API를 제공한다.

    - `git checkout Part4.3.1.1`
    ```java
    public Job batchJob() {
        return jobBuilderFactory.get("batchJob")
                .start(Step) // Flow 시작하는 Step 설정
                .on(String pattern) // Step의 실행 결과로 돌려받는 종료상태(ExitStatus)를 캐치하여 매칭하는 패턴, TransitionBuilder 반환
                .to(Step) // 다음으로 이동할 Step 지정
                .stop() / fail() / end() / stopAndRestart() // Flow를 중지, 실패, 종료하도록 Flow 종료
                .from(Step) // 이전 단계에서 정의한 Step의 Flow를 추가적으로 정의함
                .next(Step) // 다음으로 이동할 Step 지정
                .end() // build() 앞에 위치하면 FlowBuilder 종료하고 SimpleFlow 객체 생성
                .build() // FlowJob 생성하고 flow 필드에 SimpleFlow 저장
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlowJobConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .on("COMPLETED").to(step2()) // FlowBuilder, FlowJobBuilder, TransitionBuilder
                    .from(step1())
                    .on("FAILED").to(step3())
                    .end() // FlowBuilder
                    .build(); // FlowJob
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("step1 has executed");
    //                        throw new RuntimeException("");
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }

        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println("step2 has executed");
                        return RepeatStatus.FINISHED;
                    }).build();
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
                    }).build();
        }
    }
    ```

    - 참고
    > JobBuilder, SimpleJobBuilder, JobFlowBuilder, FlowJobBuilder, TransitionBuuilder, FlowBuilder, FlowJob

4. start() / next()
    - start(Flow) : 처음 실행할 Flow 설정, JobFlowBuilder가 반환된다. Step이 인자로 오게 되면 SimpleJobBuilder 가 반환된다.
    - next(Step, Flow, JobExecutionDecider)
    - Flow A -> Step 1 -> Step 2 -> Step A -> Flow B -> Step 3 -> Step 4 -> Step B
        - Step을 Flow 별로 분리하여 구성하는 장점은 있지만, Step 중 하나라도 실패할 경우 전체 Job이 실패하는 규칙은 동일하다.
    
    - `git checkout Part4.3.1.2`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class StartNextConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job batchJob() {
            return jobBuilderFactory.get("batchJob")
                    .start(flowA()) // Flow 시작, Step 1, 2, 3, 4, 5, 6 실행
                    .next(step3())
                    .next(flowB())
                    .next(step6())
                    .end()
                    .build();
        }

        @Bean
        public Flow flowA() {
            FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flowA");
            flowBuilder.start(step1())
                    .next(step2())
                    .end();

            return flowBuilder.build();
        }

        @Bean
        public Flow flowB() {
            FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flowB");
            flowBuilder.start(step4())
                    .next(step5())
                    .end();

            return flowBuilder.build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(">> step1 has executed");
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }

        @Bean
        public Step step2() {
            return stepBuilderFactory.get("step2")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step2 has executed");
                        return RepeatStatus.FINISHED;
                    }).build();
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
                    }).build();
        }

        @Bean
        public Step step4() {
            return stepBuilderFactory.get("step4")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(">> step4 has executed");
    //                        throw new RuntimeException("step4 was failed"); // step 1, 2, 3 COMPLETED, 4 FAILED
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }

        @Bean
        public Step step5() {
            return stepBuilderFactory.get("step5")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(">> step5 has executed");
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }

        @Bean
        public Step step6() {
            return stepBuilderFactory.get("step6")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(">> step6 has executed");
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }
    }
    ```