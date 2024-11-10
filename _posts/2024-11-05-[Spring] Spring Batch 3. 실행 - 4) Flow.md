---
title: '[Spring] Spring Batch 3. 실행 - 4) Flow'
date: 2024-11-05 22:30:00 +0900
categories: [백엔드, Spring Batch]
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

## Transition
1. 배치상태 유형
    - **BatchStatus**
        - JobExecution과 StepExecution의 속성으로 Job과 Step의 종료 후 최종 결과 상태가 무엇인지 정의
        - SimpleJob
            - 마지막 Step의 BatchStatus 값을 Job의 최종 BatchStatus 값으로 반영
            - Step이 실패할 경우 해당 Step이 마지막 Step이 된다.
        - FlowJob
            - Flow 내 Step의 ExitStatus 값을 FlowExecutionStatus 값으로 저장
            - 마지막 Flow의 FlowExecutionStatus 값을 Job의 최종 BatchStatus 값으로 반영
        - **COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN**
            - enum Status 정의
            - ABANDONED : 처리를 완료했지만 성공하지 못한 단계와 재시작시 건너 뛰어야 하는 단계
    - **ExitStatus**
        - JobExecution과 StepExecution의 속성으로 Job과 Step의 실행 후 어떤 상태로 종료되었는지 정의
        - 기본적으로 ExitStatus는 BatchStatus와 동일한 값으로 설정된다.
        - SimpleJob
            - 마지막 Step의 ExitStatus 값을 Job의 최종 ExitStatus 값으로 반영
        - FlowJob
            - Flow 내 Step의 ExitStatus 값을 FlowExecutionStatus 값으로 저장
            - 마지막 Flow의 FlowExecutionStatus 값을 Job의 최종 ExitStatus 값으로 반영
        - **UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED**
            - ExitStatus 인스턴스 생성
            - String exitCode 속성으로 참조
    - **FlowExecutionStatus**
        - FlowExecution의 속성으로 Flow의 실행 후 최종 결과 상태가 무엇인지 정의
        - Flow 내 Step이 실행되고 나서 ExitStatus 값을 FlowExecutionStatus 값으로 저장
        - FlowJob의 배치 결과 상태에 관여함
        - **COMPLETED, STOPPED, FAILED, UNKNOWN**
            - FlowExecutionStatus 인스턴스 생성
            - enum Status 정의
2. 구조
    - 실패 : SimpleJob -> Step1 -> ExitStatus? -> **ExitStatus.FAILED, BatchStatus.FAILED** -> JobExecution, StepExecution -> DB
    - 성공 : SimpleJob -> Step1 -> ExitStatus? -> **ExitStatus.COMPLETED, BatchStatus.COMPLETED** -> JobExecution, StepExecution -> Step2 -> ExitStatus? -> ExitStatus.COMPLETED, BatchStatus.COMPLETED or ExitStatus.FAILED, BatchStatus.FAILED -> JobExecution, StepExecution -> DB

- `git checkout Part4.3.1.3.3`
```java
@RequiredArgsConstructor
@Configuration
public class BatchStatusExitStatusConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob1() { 
        return jobBuilderFactory.get("batchJob") // BATCH_JOB_EXECUTION, BATCH_STEP_EXECUTION(STATUS : COMPLETED, EXIT_CODE : COMPLETED)
                .start(step1()) // SimpleJob 생성
                .next(step2())
                .build();

    //    return jobBuilderFactory.get("batchJob") // BATCH_JOB_EXECUTION, BATCH_STEP_EXECUTION(STATUS : COMPLETED, EXIT_CODE : COMPLETED)
    //            .start(step1()) // FlowJob 생성
    //            .on("FAILED")
    //            .to(step2())
    //            .end()
    //            .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
    //                contribution.setExitStatus(ExitStatus.FAILED); // BATCH_JOB_EXECUTION(STATUS : COMPLETED, EXIT_CODE : COMPLETED), BATCH_STEP_EXECUTION(STEP1 = STATUS : COMPLETED, EXIT_CODE : FAILED, STEP2 = STATUS : COMPLETED, EXIT_CODE : COMPLETED)
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step2 has executed");
    //                contribution.setExitStatus(ExitStatus.FAILED); // BATCH_JOB_EXECUTION(STATUS : COMPLETED, EXIT_CODE : FAILED), BATCH_STEP_EXECUTION(STEP 1 = STATUS : COMPLETED, EXIT_CODE : COMPLETED, STEP 2 = STATUS : COMPLETED, EXIT_CODE : FAILED)
                    return RepeatStatus.FINISHED;
                }).build();
    }
}
```

- 참고
> `JobExecution의 BatchStatus(STARTING), exitStatus(UNKNOWN)`, `StepExecution의 BatchStatus(STARTING), exitStatus(EXECUTING)`, `FlowExecution의 FlowExecutionStatus`
> `ExitStatus`, `FlowExecutionStatus`
> `SimleJob의 doExecute`, `FlowJob의 doExecute`

2. 기본 개념
    - Flow 내의 조건부 전환(전이)을 정의함
    - Job의 API 설정에서 on(String pattern) 메소드를 호출하면 TransitionBuilder가 반환되어 Transition Flow를 구성할 수 있음
    - Step의 종료상태(ExitStatus)가 어떤 pattern과도 매칭되지 않으면 스프링 배치에서 예외를 발생하고 Job은 실패
    - Transition은 구체적인 것부터 그렇지 않은 순서로 적용된다.

3. API
    - **on(String pattern)**
        - TransitionBuilder 반환
        - Step의 실행 결과로 돌려받는 종료상태(ExitStatus)와 매칭하는 패턴 스키마, BatchStatus와 매칭하는 것이 아님
        - pattern과 ExitStatus와 매칭이 되면 다음으로 실행할 Step을 지정할 수 있다.
        - 특수문자는 두 가지만 허용
            - "*" : 0개 이상의 문자와 매칭, 모든 ExitStatus와 매칭된다.
            - "?" : 정확히 1개의 문자와 매칭
            - ex) "C*T"는 "cat"과 "count"에 매칭되고, "c?t"는 "cat"에는 매칭되지만 "count"
    - **to(Step or Flow or JobExecutionDecider)**
        - 다음으로 실행할 단계를 지정
    - **from()**
        - 이전 단계에서 정의한 Transition을 새롭게 추가 정의함
    - **stop(), fail(), end(), stopAndRestart()**
        - Job을 중단하거나 종료하는 Transition API
        - Flow가 실행되면 FlowExecutionStatus에 상태값이 저장되고 최종적으로 Job의 BatchStatus와 ExitStatus에 반영된다.
        - Step의 BatchStatus 및 ExitStatus에는 아무런 영향을 주지 않고 Job의 상태만을 변경한다.<br/>

        - **stop()**
            - FlowExecutionStatus가 STOPPED 상태로 종료되는 Transition
            - Job의 BatchStatus와 ExitStatus가 STOPPED로 종료된
        - **fail()**
            - FlowExecutionStatus가 FAILED 상태로 종료되는 Transition
            - Job의 BatchStatus와 ExitStatus가 FAILED로 종료됨
        - **end()**
            - FlowExeucution가 COMPLETED 상태로 종료되는 Transition
            - Job의 BatchStatus와 ExitStatus가 COMPLETED으로 종료됨
            - Step의 ExitStatus가 FAILED더라도 Job의 BatchStatus가 COMPLETED로 종료하도록 가능하며 이 때 Job의 재시작은 불가능함
        - **stopAndRestart(Step or Flow or JobExecutionDecider)**
            - stop() Transition과 기본 흐름은 동일
            - 특정 Step에서 작업을 중단하도록 설정하면 중단 이전의 STep만 COMPLETED 저장되고, 이후의 Step은 실행되지 않고 STOPPED 상태로 Job 종료
            - Job이 다시 실행됐을 때 실행해야 할 Step을 Restart 인자로 넘기면 이전에 COMPLETED로 저장된 STep은 건너뛰지 중단 이후 Step부터 시작한다.

4. 작업 정의
    1. 단계 1을 실행해서 "A"가 나오면 단계 2를 실행하라 : Step1 -> A(FAILED) -> Step2
    2. 단계 2가 성공하면 Step의 결과에 상관없이 작업을 중단하라 : Step2 -> * -> Stop
    3. 단계 1을 실행해서 A가 아니면 Step의 결과에 상관없이 단계 3을 실행하라 : Step1 -> * Step3
    4. 단계 3이 성공하면 단계 4를 실행하라 : Step3 -> Step4
    5. 단계 4를 실행해서 "B"가 나오면 작업을 종료하라 : Step 4 -> B -> End

5. 작업 결과
    - FlowJob
        - Flow(A, * : Transition 4개)
            - Step1 -> `FAILED` -> Step2 -> `*` -> Stop
                - `.start(step1())`
                    - `.on("FAILED")`
                    - `.to(step2())`
                    - `.on("*")`
                    - `.stop()`
            - Step1 -> `*` -> Step3 -> Step4 -> `*` -> End
                - `.from(step1())`
                    - `.on("*")`
                    - `.to(step3())`
                    - `.next(step4())`
                    - `.on("FAILED")`
                    - `.end()`
                - `.end()`

- `git checkout Part4.3.1.3.3`
```java
@RequiredArgsConstructor
@Configuration
public class TransitionAPIConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() { 
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                    .on("FAILED")
                    .to(step2())
                    .on("*")
                    .stop()
                .from(step1())
                    .on("*") // Step1은 위에서 Transition을 이미 정의했는데 새롭게 Transition을 정의할 때 fron()을 사용
                    .to(step3())
                    .next(step4())
                    .on("FAILED")
                    .end()
                .end()
                .build();
    }
}
```

- 참고
    - **Step1의 ExitStatus = FAILED**
        - BATCH_JOB_EXECUTION(STATUS : STOPPED, EXIT_CODE : STOPPED)
        - BATCH_STEP_EXECUTION(STEP1 = STATUS : **ABANDONED**, EXIT_CODE : **FAILED**, STEP2 = STATUS : COMPLETED, EXIT_CODE : COMPLETED)
    - **Step1의 ExitStatus = `*`**
        - BATCH_JOB_EXECUTION(STATUS : COMPLETED, EXIT_CODE : COMPLETED)
        - BATCH_STEP_EXECUTION(STEP1 = STATUS : COMPLETED, EXIT_CODE : COMPLETED, STEP3 = STATUS : COMPLETED, EXIT_CODE : COMPLETED, STEP4 = STATUS : **FAILED**, EXIT_CODE : **FAILED**)

- `git checkout Part4.3.1.3.1`
```java
@RequiredArgsConstructor
@Configuration
public class TransitionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() {
        return jobBuilderFactory.get("batchJob")
                .start(flow())
                .next(step3())
                .end()
                .build();

    //    return jobBuilderFactory.get("batchJob") // Flow 3개, step1 -> step3 -> step4 모두 COMPLETED
    //            .start(step1())
    //                .on("FAILED")
    //                .to(step2())
    //                .on("FAILED")
    //                .stop()
    //            .from(step1())
    //                .on("*")
    //                .to(step3())
    //                .next(step4())
    //            .from(step2())
    //                .on("*")
    //                .to(step5())
    //                .end()
    //            .build();
    }

    @Bean
    public Flow flow() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flowA");
        flowBuilder.start(step1())
                .next(step2())
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
    //                    contribution.setExitStatus(ExitStatus.FAILED); // step1(FAILED) -> step2(COMPLETED) -> step5(COMPLETED)
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
}
```

- 참고
> `JobBuilder`, `SimpleJobBuilder`, `FlowBuilder`

## 사용자 정의 ExitStatus
1. 기본 개념
    - ExitStatus에 존재하지 않는 exitCode를 새롭게 정의해서 설정
    - StepExecutionListener의 afterStep() 메소드에서 Custom exitCode 생성 후 새로운 ExitStatus 반환
    - Step 실행 후 완료 시점에서 현재 exitCode를 사용자 정의 exitCode로 수정할 수 있음

2. 구조
    - Job -> Step1 -> ExitStatus.COMPLETED
    - Job -> Step1 -> on("DO_PASS") -> **No : Step2 or Yes : Step3**
    - Job -> Step1 -> **StepExecutionListener** -> afterStep() -> new ExitStatus("PASS") -> finally : ExitStatus.and(StepListener.afterStep()) -> ExitStatus.PASS -> on("DO_PASS") -> **Yes : Step3**

- `git checkout Part4.3.1.3.3`
```java
@RequiredArgsConstructor
@Configuration
public class CustomExitStatusConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() {
        return this.jobBuilderFactory.get("batchJob")
                .start(step1()) // step1(FAILED) -> step2(COMPLETED) -> Job(COMPLETED)
                    .on("FAILED")
                    .to(step2())
                    .on("PASS")
                    .stop()
    //            .from(step())
    //                .on(step2())
    //                .to(step1())
                .end()
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    contribution.setExitStatus(ExitStatus.FAILED);
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
                .listener(new PassCheckingListener()) // 없으면 step2(FAILED) -> Job(FAILED), 있으면 step2(PASS) -> Job(STOPPED)
                .build();
    }

    static class PassCheckingListener extends StepExecutionListenerSupport {

        public ExitStatus afterStep(StepExecution stepExecution) {

            String exitCode = stepExecution.getExitStatus().getExitCode();
            if (!exitCode.equals(ExitStatus.FAILED.getExitCode())) {
                return new ExitStatus("PASS ");
            } else {
                return null;
            }
        }
    }
}
```

- 참고
> `FlowJob`, `PassCheckingListener`, `CompositeStepExecutionListener`, `TaskletStep`, `AbstractStep`

## JobExecutionDecider
1. 기본 개념
    - E**xitStatus를 조작하거나 StepExecutionListener를 등록할 필요 없이** Transition 처리를 위한 전용 클래스
    - Step과 Transition 역할을 명확히 분리해서 설정할 수 있음
    - Step의 ExitStatus가 아닌 **JobExecutionDecider의 FlowExecutionStatus** 상태값을 새롭게 설정해서 반환함

2. 구조
    - JobExecutionDecider
        - FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution)

- `git checkout Part4.3.1.3.4`
```java
@RequiredArgsConstructor
@Configuration
public class JobExecutionDeciderConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
    //            .incrementer(new RunIdIncrementer()) // Job 여러 번 실행 가능
                .start(startStep()) // FlowJob 구성
                .next(decider())
                .from(decider()).on("ODD").to(oddStep()) // decider()의 실행 후 상태가 ODD이면 oddStep() 실행
                .from(decider()).on("EVEN").to(evenStep()) // decider()의 실행 후 상태가 EVEN이면 evenStep() 실행
                .end()
                .build();
    }

    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("This is the start tasklet");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public Step evenStep() {
        return stepBuilderFactory.get("evenStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">>EvenStep has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public Step oddStep() {
        return stepBuilderFactory.get("oddStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">>OddStep has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public JobExecutionDecider decider() {
        return new CustomDecider();
    }

    public static class CustomDecider implements JobExecutionDecider {

        private int count = 0;

        @Override
        public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
            count++;

            if(count % 2 == 0) {
                return new FlowExecutionStatus("EVEN");
            }
            else {
                return new FlowExecutionStatus("ODD");
            }
        }
    }
}
```

- 참고
> `SimpleFlow`, `DecisionState`, `StateTransition`

## FlowJob 아키텍처
1. 구조
    - JobLauncher -> JobParameters, JobInstance, JobExecution(ExecutionContext(updateStatus(execution, **BatchStatus.STARTED**))) -> **FlowJob** -> JobListener(Composite**JobExecutionListener**.beforeJob()) -> FlowExecutor -> **FlowExecution** -> **SimpleFlow** -> State(Step, Flow, Decider) -> FlowExecutionStatus -> SimpleFlow -> FlowExecution(**FlowExecution** result = new FlowExecution(stateName, status);) -> FlowExecutor -> JobListener(Composite**JobExecutionListener**.AfterJob()) -> FlowJob -> JobExecution(**jobExecution**.setStatus(FlowExecution.getStatus()); exitStatus = exitStatus.and(new ExitStatus(floxExecution.getStatus().getName())); jobExecution.setExitStatus(exitStatus);)

2. 클래스 상속 관계도
    - JobBuilderFactory -> JobBuilderHelper -> CommonJobProperties, AtomicReference`<JobRepository>`
    - JobBuilderHelper -> SimpleJobBuilder(-> SimpleJob), JobBuilder, FlowJobBuilder(-> FlowJob)

- `git checkout Part4.3.1.3.5`
```java
@RequiredArgsConstructor
@Configuration
public class FlowJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob") // JobBuilder
                .start(flow()) // JobFlowBuilder
                .next(step3()) // FlowBuilder<FlowJobBuilder> (Flow의 SimpleFlow)
                .end() // FlowJobBuilder (FlowJob의 SimpleFlow)
                .build();
    }

    @Bean
    public Flow flow() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");

        flowBuilder.start(step1())
                .next(step2())
                .end();

        return flowBuilder.build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    return RepeatStatus.FINISHED;
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
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }
}
```

- 참고
> `JobBuilder의 FlowJobBuilder의 SimpleFlow`, `FlowBuilder의 SimpleFlow`, `FlowExecution`

## SimpleFlow
1. 기본 개념
     - 스프링 배치에서 제공하는 Flow의 구현체로서 각 요소(Step, Flow, JobExecutionDecider)들을 담고 있는 State를 실행시키는 도메인 객체
     - FlowBuilder를 사용해서 생성하며 Transition과 조합하여 여러 개의 Flow 및 중첩 Flow를 만들어 Job을 구성할 수 있다.

2. 구성 요소
    - Flow
        - getName() : Flow 이름 조회
        - State getState(String stateName) : State 명으로 State 타입 반환
        - FlowExecution start(FlowExecutor executor) : Flow를 실행시키는 start 메소드, 인자로 FlowExecutor를 넘겨주어 실행을 위임함, 실행 후 FlowExecution을 반환
        - FlowExecution resume(String stateName, FlowExecutor executor) : 다음에 실행할 State를 구해서 FlowExecutor에게 실행을 위임함
        - Collection`<State>` getStates() : Flow가 가지고 있는 모든 State를 Collection 타입을 반환
    - SimpleFlow
        - String name : Flow 이름
        - State startState : State들 중에서 처음 실행할 State
        - Map`<String, Set<StateTransition>>` transitionMap : State 명으로 매핑되어 있는 Set<StateTransition>
        - Map`<String, State>` stateMap : State 명으로 매핑되어 있는 State 객체
        - List`<StateTransition>` stateTransitions : State 명으로 매핑되어 있는 State 객체
        - Comparator`<StateTransition>` stateTransitionComparator : State와 Transition 정보를 가지고 있는 StateTransition 리스트

3. 구조
    - JobBuilderFactory -> FlowJobBuilder -> FlowBuilder -> SimpleFlow(SimpleFlow1, SimpleFlow2)
        - **FlowJob 안의 SimpleFlow, SimpleFlow 안의 SimpleFlow, SimpleFlow 안의 Step**
    - Flow 안에 Step을 구성하거나, Flow를 중첩되게 구성할 수 있다.
    ```java
    public Job batchJob() {
        return jobBuilderFactory.get("flowJob")
                .start(flow1()) // Flow를 정의하여 설정함
                .on("COMPLETED").to(flow2()) // Flow를 Transition과 함께 구성
                .end() // SimpleFlow 객체 생성
                .build(); // FlowJob 객체 생성
    }
    ```

- `git checkout Part4.3.2.1`
```java
@RequiredArgsConstructor
@Configuration
public class SimpleFlowConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(flow())
                .next(step3())
                .end()
                .build();
    }

    @Bean
    public Flow flow() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");

        flowBuilder.start(step1())
                .next(step2())
                .end();

        return flowBuilder.build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    return RepeatStatus.FINISHED;
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
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }
}
```

- 참고
> `SimpleFlow`, `State`, `StepState`


4. 예제
- 다양한 step을 조합해 flow 구성 가능
- Job의 BATCH STATUS는 COMPLETED 되도록 구성, EXIT_CODE는 FAILED 가능
- Job(**Flow**(**Flow1**(Step1, Step2), **Flow2**(**Flow3**(Step3, Step4), Step5, Step6)))
- Job -> Flow1 -> Step1 -> Step2 -> Flow2 -> Flow3 -> Step3 -> Step4 -> Step5 -> Step6
- `git checkout Part4.3.2.2`
```java
@RequiredArgsConstructor
@Configuration
public class SimpleFlowConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob") // BATCH_JOB_EXECUTION(STATUS : COMPLETED, EXIT_CODE : FAILED)
                .start(flow1())
                    .on("COMPLETED")
                    .to(flow2())
                .from(flow1())
                    .on("FAILED")
                    .to(flow3())
                .end() // 최초 SimpleFlow 생성
                .build();
    }

    @Bean
    public Flow flow1() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow1");

        flowBuilder.start(step1())
                .next(step2())
                .end();

        return flowBuilder.build();
    }

    @Bean
    public Flow flow2() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow2");

        flowBuilder.start(flow3())
                .next(step5())
                .next(step6())
                .end();

        return flowBuilder.build();
    }

    @Bean
    public Flow flow3() {
        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow3");

        flowBuilder.start(step3())
                .next(step4())
                .end();

        return flowBuilder.build();
    }
    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
    //                throw new RuntimeException("step1 was failed"); // flow1 -> step1 -> step2(FAILED) -> flow3 -> step3 -> step4
                    return RepeatStatus.FINISHED;
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
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public Step step4() {
        return stepBuilderFactory.get("step4")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step4 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public Step step5() {
        return stepBuilderFactory.get("step5")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step5 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }

    @Bean
    public Step step6() {
        return stepBuilderFactory.get("step6")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step6 has executed");
                    return RepeatStatus.FINISHED;
                }).build();
    }
}
```

5. **SimpleFlow 아키텍처**
    - Flow의 API 설정 및 Transition에 따라 State 객체를 생성하고 StateTransition의 속성에 저장한다.
    - Transition은 구체적인 것부터 그렇지 않은 순서로 적용된다. ex) FAILEd -> *
    - SimpleFlow를 구성하고 있는 모든 STep들이 Transition에 따라 분기되어 실행된다.
    - SimpleFlow 내 SimpleFlow를 2중, 3중으로 중첩해서 복잡하게 구성할 수 있다.<br/>

    - **FlowBuilder**
        - start, next, from(step) : Step 저장, StepState
        - start, next, from(flow) : Flow 저장, FlowState
        - start, next, from(decider) : JobExecutionDecider 저장, DecisionState
        - on(String pattern) : ExitStatus 값 설정
        - split(Executor) : 여러 Flow 저장, SplitState
    - **StateTrnasition**
        - State state : 현재 State
        - String pattern : on() Transition
        - String next : 다음 State
    - **SimpleFlow** : StateTransition 객체들을 담은 List`<StateTransition>`을 가지며, List`<StateTransition>`을 사용해 다른 속성의 값을 초기화한다.
        - Map`<String, State>` stateMap : state별로 Grouping
        - State startState
        - Map`<String, Set<StateTransition>>` transitionMap
        - List`<StateTransition>` stateTransition

    - State: String getName(), FlowExecutionStatus handle(FlowExecutor executor), boolean isEndState()
        - Step, Flow, JobExecutionDecider의 각 요소들을 저장함
        - Flow를 구성하면 내부적으로 생성되어 Transition과 연동
        - handle() 메소드 실행 후 FlowExecutionStatus 반환
            - on(pattern)의 pattern 값과 매칭여부 판단
            - 마지막 실행 상태가 FlowJob의 최종 상태가 됨

    - SimpleFlow는 StateMap에 저장되어 있는 모든 State들의 handle 메소드를 호출해서 모든 Step들이 실행되도록 한다.
    - SimpleFlow는 현재 호출되는 State가 어떤 타입인지 알지 못하고 관심도 없다. 단지 handle 메소드를 실행하고 상태값을 얻어올 뿐이다.(상태 패턴)

    - StateTrnasition(CurrentState, NextState) <- State <- AbstractState <- StepState step, FlowState flow, DecisionState decider, EndState, SplitState flows<br/>

    - FlowJob -> FlowExecutor -> SimpleFlow -> **stateMap(StepState, FlowState, DecisionState)** -> start(), resume(), nextState()
        - StateMap은 StateTransition 객체로부터 State 객체의 정보를 참조해서 매핑함
        - start() : 처음 실행될 StartState 지정 후 실행
        - resume() : 현재 State 실행이 완료되면 다음 State 실행함, 실행 순서는 순차적 개념이 아닌 설정 및 조건에 따라 결정된다. State가 null이거나 실행불가능한 상태이면 종료, flowState일 경우 다시 재귀적 호출
            - resume() -> StepState -> Step, resume() -> FlowState -> Flow(), resume() -> DecisionState -> Decider

        - FlowJob(flow.start(executor)) -> SimpleFlow(state.handle(executor)) -> **StepState**(executor.executeStep()) -> JobFlowExecutor -> stepHandler -> Step
        - FlowJob(flow.start(executor)) -> SimpleFlow(state.handle(executor)) -> **FlowState**(flow(start(executor))) -> SimpleFlow -> FlowState, StepState, DecisionState, SplitState -> SimpleFlow(재귀적 호출)
        - FlowJob(flow.start(executor)) -> SimpleFlow(state.handle(executor)) -> **DecisionState**(decider.decide(jobExecution, stepExecution)) -> JobExecutionDecider
        - FlowJob(flow.start(executor)) -> SimpleFlow(state.handle(executor)) -> **SplitState** -> SimpleFlow -> FlowState, StepState<br/>
        - return FlowExecutionStatus -> SimpleFlow

    - `git checkout Part4.3.2.3`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class SimpleFlowConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job job() {
            return jobBuilderFactory.get("batchJob")
                    .start(step1())
                    .on("COMPLETED").to(step2())
                    .from(step1())
                    .on("FAILED").to(flow())
                    .end()
                    .build();
        }
        @Bean
        public Flow flow() {
            FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");

            flowBuilder.start(step2())
                    .on("*")
                    .to(step3())
                    .end();

            return flowBuilder.build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step1 has executed");
                        contribution.setExitStatus(ExitStatus.FAILED);
                        return RepeatStatus.FINISHED;
        //                throw nes RuntimeException("step1 was failed");
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
                    .tasklet((contribution, chunkContext) -> {
                        System.out.println(">> step3 has executed");
                        return RepeatStatus.FINISHED;
                    }).build();
        }
    }
    ```

    - 참고
    > SimpleFlow, SimpleJobBuilder, JobBuilder, FlowBuilder, JobFlowBuilder, StepState

## FlowStep
1. 기본 개념
    - Step 내에서 Flow를 할당하여 실행시키는 도메인 객체
    - FlowStep의 BatchStatus와 ExitStatus은 Flow의 최종 상태값에 따라 결정된다.

2. API 소개
    - StepBuilderFactory -> StepBuilder -> FlowStepBuilder -> FlowStep
    - Job -> FlowStep -> Flow -> Step2 -> Tasklet -> Status -> COMLETEDD: Step3 or FAILED : Job
    - StepBuilderFactory -> StepBuilder -> Flow(flow) -> FlowStepBuilder

    - `git checkout Part4.4.1`
    ```java
    public Step flowStep() {
        return stepBuilderFactory.get("flowStep")
                .flow(flow()) // Step 내에서 실행될 Flow 설정, FlowStepBuilder 반환
                .build(); // FlowStep 객체를 생성
    }
    ```
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlowStepConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job job() {
            return jobBuilderFactory.get("batchJob")
                    .start(flowStep())
                    .next(step2())
                    .build();
        }

        public Step flowStep() {
            return stepBuilderFactory.get("flowStep") // BATCH_JOB_EXECUTION(FAILED, FAILED), BATCH_STEP_EXECUTION(flowStep(FAILED, FAILED) step1(FAILED, FAILED))
                    .flow(flow())
                    .build();
        }

        @Bean
        public Flow flow() {
            FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("flow");
            flowBuilder.start(step1())
                    .end();
            return flowBuilder.build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("step1 was executed");
                            throw new RuntimeException("step1 was failed");
    //                        return RepeatStatus.FINISHED;
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
                            System.out.println("step2 was executed");
                            return RepeatStatus.FINISHED;
                        }
                    })
                    .build();
        }
    }
    ```

    - 참고
    > FlowStep

## @JobScope / @StepScope
1. 기본 개념
    - Scope
        - 스프링 컨테이너에서 빈이 관리되는 범위
        - singleton, prototype, request, session, application 있으며 기본은 **singleton**으로 생성됨(객체가 메모리에 하나만 존재)
    - 스프링 배치 스코프
        - Job과 Step의 빈 생성과 실행에 관여하는 스코프 ex) `@JobScope`, `@StepScope`
        - 프록시 모드를 기본값으로 하는 스코프 : `@Scope(value = "job", proxyMode = ScopedProxyMode.TARGET_CLASS)`
        - 해당 스코프가 선언되면 빈이 생성이 어플리케이션 구동시점이 아닌 **빈의 실행시점**에 이루어진다.
            - @Values를 주입해서 빈의 실행 시점에 값을 참조할 수 있으며 **일종의 Lazy Binding**이 가능해진다.
            - `@Value("#{jobParameters[파라미터명]}")`, `@Value("#{jobExecutionContext[파라미터명]"})`, `@Value("#{stepExecutionContext[파라미터명]"})`
            - @Values를 사용할 경우 빈 선언문에 @JobScope, @StepScope를 정의하지 않으면 오류를 발생하므로 반드시 선언해야 함
        - 프록시 모드로 빈인 선언하기 때문에 어플리케이션 구동시점에는 **빈의 프록시 객체가 생성**되어 **실행 시점에 실제 빈을 호출**해준다.(AOP)
        - **병렬처리 시 각 스레드마다 생성된 스코프 빈이 할당**되기 때문에 스레드에 안전하게 실행이 가능하다.
        - 동적인 시간에 참조 가능
    - 스프링 배치 스코프 종류
        - `@JobScope`
            - Step 선언문에 정의한다.
            - @Value : jobParameter, jobExecutionContext만 사용가능
        - `@StepScope`
            - Tasklet이나 ItemReader, ItemWriter, ItemProcessor 선언문에 정의한다.
            - @Value : jobParameter, jobExecutionContext, stepExecutionContext 사용가능


2. 파라미터 설정
    - `@JobScope`를 선언하지 않으면 `Property of file 'jobParameters' cannot be found on object of type 'org.spring framework.beans.factory.config.BeanExpressionContext'` 발생
    - Step 구문에는 `@JobScope`를 선언해야 하며 `@Value` 사용 시 필수적으로 선언해야 한다.
    - Tasklet 구문에는 `@StepScope`를 선언해야 하며 `@Value` 사용 시 필수적으로 선언해야 한다.
    - `@Value("#{jobParameters['name']}")`, `@Value("#{jobParameters['requestDate']}")`

    ```java
    @Component
    @StepScope
    public class MyJobParameter implements Tasklet {
        @Value("#{jobParameters[name]}")
        private String name;

        @Override
        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) {
            return RepeatStatus.FINISHED;
        }

        public void requestDate(String requestDate) {
            System.out.println("name = " + name);
            System.out.println("requestDate = " + requestDate);
        }
    }
    ```

- `git checkout Part4.4.2`
```java
@RequiredArgsConstructor
@Configuration
public class JobScope_StepScope_Configuration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1(null)) // 컴파일 오류가 나지 않기 위해 null 값 설정
                .next(step2())
                .listener(new CustomJobListener())
                .build();
    }

    @Bean
    @JobScope
    public Step step1(@Value("#{jobParameters['message']}") String message) { // 파라미터 설정
        System.out.println("jobParameters['message'] : " + message);
        return stepBuilderFactory.get("step1")
                .tasklet(tasklet1(null))
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
    @StepScope
    public Tasklet tasklet1(@Value("#{jobExecutionContext['name']}") String name){ // 파라미터 설정
        return (stepContribution, chunkContext) -> {
            System.out.println("jobExecutionContext['name'] : " + name);
            return RepeatStatus.FINISHED;
        };
    }
}
```
```java
@RequiredArgsConstructor
@Configuration
public class JobScope_StepScope_Configuration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1(null)) // 컴파일 오류가 나지 않기 위해 null 값 설정
                .next(step2())
                .listener(new CustomJobListener())
                .build();
    }

    @Bean
    @JobScope
    public Step step1(@Value("#{jobParameters['message']}") String message) { // 파라미터 설정
        System.out.println("jobParameters['message'] : " + message);
        return stepBuilderFactory.get("step1")
                .tasklet(tasklet1(null))
                .build();
    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(tasklet2(null))
                .listener(new CustomStepListener())
                .build();
    }

    @Bean
    @StepScope // 생략 시 오류
    public Tasklet tasklet1(@Value("#{jobExecutionContext['name1']}") String name){ // 파라미터 설정
        return (stepContribution, chunkContext) -> {
            System.out.println("jobExecutionContext['name1'] : " + name);
            return RepeatStatus.FINISHED;
        };
    }

        @Bean
    @StepScope
    public Tasklet tasklet2(@Value("#{jobExecutionContext['name2']}") String name){ // 파라미터 설정
        return (stepContribution, chunkContext) -> {
            System.out.println("jobExecutionContext['name2'] : " + name);
            return RepeatStatus.FINISHED;
        };
    }
}
```
```java
public class CustomJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
        jobExecution.getExecutionContext().putString("name1", "user1");
    }

    @Override
    public void afterJob(JobExecution jobExecution) {

    }
}
```
```java
public class CustomStepListener implements StepExecutionListener {

    @Override
    public void beforeJob(StepExecution stepExecution) {
        stepExecution.getExecutionContext().putString("name2", "user2");
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        return null;
    }
}
```

- Run Configuration
> --job.name=batchJob message=20240101

3. 아키텍처
    - Proxy 객체 생성
        - `@JobScope`, `@StepScope` 어노테이션이 붙은 빈 선언은 내부적으로 빈의 Proxy 객체가 생성된다.
            - `@JobScope` : `@Scope(value = "job", proxyMode = ScopedProxyMode.TARGET_CLASS)`
            - `@StepScope` : `@Scope(value = "step", proxyMode = ScopedProxyMode.TARGET_CLASS)`
        - Job 실행 시 Proxy 객체가 실제 빈을 호출해서 해당 메소드를 실행시키는 구조
    - **JobScope, StepScope**
        - Proxy 객체의 실제 대상이 되는 Bean을 등록, 해제하는 역할
        - 실제 빈을 저장하고 있는 JobContext, StepContext를 가지고 있다.
    - **JobContext, StepContext**
        - 스프링 컨테이너에서 생성된 빈을 저장하는 컨텍스트 역할
        - Job의 실행 시점에서 프록시 객체가 **실제 빈을 참조할 때** 사용됨
    - ApplicationContext -> createBean() : 초기 시점에 Bean의 프록시 객체 생성 -> @JobScope? -> **YES** : Proxy(스프링 초기화 완료 및 Job 실행) -> JobLauncher -> Job(Job은 현재 Step의 프록시 객체를 저장하고 있다.) -> Proxy(참조할 Bean을 구한다. 프록시 객체는 Step 메소드 호출 시 실제 빈 객체를 참조하는데 이 시점에 Step 빈이 생성된다.) -> JobScope(프록시의 실제 Bean 등록 및 관리) -> JobContext -> exsit Bean? -> **YES** : Step(JobContext에서 Step 빈을 꺼낸다. 실제 Step 실행) -> Tasklet
    - ApplicationContext -> @JobScope? -> **NO** : Singleton Bean(기본 싱글톤 빈 생성, @Value 사용시엔 오류 발생)
    - JobContext -> exsit Bean? -> **NO** : BeanFactory -> createBean() : Step 메소드 소출해서 빈 생성, @Value 바인딩 처리, 런타임 실행 시점, **정확히는 프록시가 Step의 메소드를 실행시키는 시점에 STep 빈이 생성되고 @Value 바인딩 처리가 이루어진다.** -> step

- `git checkout Part4.4.2`
```java
@RequiredArgsConstructor
@Configuration
public class JobScope_StepScope_Configuration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1(null)) // 프록시 객체 생성
                // .next(step2())
                // .listener(new CustomJobListener())
                .build();
    }

    @Bean
    @JobScope
    public Step step1(@Value("#{jobParameters['message']}") String message) { // 파라미터 설정
        System.out.println("jobParameters['message'] : " + message);
        return stepBuilderFactory.get("step1")
                .tasklet(tasklet1(null))
                .build();
    }

    // @Bean
    // public Step step2() {
    //     return stepBuilderFactory.get("step2")
    //             .tasklet((contribution, chunkContext) -> {
    //                 System.out.println("step2 has executed");
    //                 return RepeatStatus.FINISHED;
    //             })
    //             .build();
    // }

    @Bean
    @StepScope
    public Tasklet tasklet1(@Value("#{jobExecutionContext['name']}") String name){ // 파라미터 설정
        return (stepContribution, chunkContext) -> {
            System.out.println("jobExecutionContext['name'] : " + name);
            return RepeatStatus.FINISHED;
        };
    }
}
```

- 참고
> AbstractBatchConfiguration, JobScope, JobBuilder, JdkDynamicAopProxy, AbstractStep, SimpleStepHandler