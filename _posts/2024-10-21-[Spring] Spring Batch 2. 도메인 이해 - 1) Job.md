---
title: '[Spring] Spring Batch 2. 도메인 이해 - 1) Job'
date: 2024-10-27 18:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 1) Job
### 1. Job
1. 기본 개념
  - 배치 계층 구조에서 가장 상위에 있는 개념으로서 하나의 배치 작업 자체를 의미함.
    - 'API 서버의 접속 로그 데이터를 통계 서버로 옮기는 배치'인 Job 자체를 의미한다.
  - Job Configuration을 통해 생성되는 객체 단위로서 배치 작업을 어떻게 구성하고 실행할 것인지 전체적으로 설정하고 명세해 놓은 객체
  - 배치 Job을 구성하기 위한 최상위 인터페이스이며 스프링 배치가 기본 구현체를 제공한다.
  - 여러 Step을 포함하고 있는 컨테이너로서 반드시 한 개 이상의 Step으로 구성해야 한다.
2. 기본 구현체
  - ****SimpleJob**
    - 순차적으로 Step을 실행시키는 Job
    - 모든 Job에서 유용하게 사용할 수 있는 표준 기능을 갖고 있음
  - **FlowJob**
    - 특정한 조건과 흐름에 따라 Step을 구성하여 실행시키는 Job
    - Flow 객체를 실행시켜서 작업을 진행함
3. 구조
  - Job -> Step -> Tasklet
  - JobParameters -> `JobLauncher(run(job, parameters))` -> Job(execute()) -> Steps(List 변수)
  - `Job(void execute(JobExecution))` <- AbstractJob(name, restartable(기본값 true), JobRepository(메타 데이터 저장소), JobExecutionListener, JobParametersIncrementer, JobParametersValidator, SimpleStepHandler) <- `SimpleJob(steps 실행)`, `FlowJob(Flow 실행)`

```java
@RequiredArgsConstructor
@Configuration
public class JobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return this.jobBuilderFactory.get("Job")
                .start(step1()) // SimpleJobBuilder에서 step 상태 저장
                .next(step2())
                .build(); // SimpleJob 생성(컨테이너 역할), steps 리스트는 2개의 step 가짐
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
                    return RepeatStatus.FINISHED; // 1번 실행
                })
                .build();
    }
}
```
### 2. JobInstance
1. 기본 개념
  - **Job이 실행될 때** 생성되는 Job의 논리적 실행 단위 객체로서 **고유하게 식별 가능한 작업 실행**을 나타냄
  - Job의 설정과 구성은 동일하지만 Job이 실행되는 시점에 처리하는 내용은 다르기 때문에 Job의 실행을 구분해야 함
    - 예를 들어 하루에 한 번씩 배치 Job이 실행된다면 매일 실행되는 각각의 Job을 **JobInstance**로 표현한다.
  - JobInstance 생성 및 실행
    - **처음 시작**하는 Job + JobParameter일 경우 **새로운 JobInstance 생성**
    - **이전과 동일**한 Job + JobParameter으로 실행할 경우 **이미 존재하는 JobInstance 리턴**
      - 내부적으로 JobName + JobKey(JobParameters의 해시값)을 가지고 JobInstance 객체를 얻음
  - Job과는 1:M 관계
2. **BATCH_JOB_INSTANCE** 테이블과 매핑
  - JOB_NAME(Job)과 JOB_KEY(JobParameter 해시값)가 동일한 데이터는 중복해서 저장할 수 없음
3. **BATCH_JOB_EXECUTIONS_PARAMS** : JobParameters 저장
4. JobLauncher -> Job(JobParameters) -> JobRepository -> DB -> JobInstance(JobName, JobKey)
5. Job(일별 정산) -> JobInstance -> Job : 일별 정산 + JobParameters : 2024.01.01 != Job : 일별 정산 + JobParameters : 2024.01.02
6. `a job instance already exists and is complete for parameters` : Job + JobParameters 중복 X

- `git checkout Part3.1.2`
```yml
spring:
  profiles:
    active: local

---
spring:
  config:
    activate:
      on-profile: local
  datasource:
    hikari:
      jdbc-url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
      username: sa
      password:
      driver-class-name: org.h2.Driver
  batch:
    jdbc:
      initialize-schema: embedded

---
spring:
  config:
    activate:
      on-profile: mysql
  datasource:
    hikari:
      jdbc-url: jdbc:mysql://localhost:3306/springbatch?useUnicode=true&characterEncoding=utf8
      username: root
      password: pass
      driver-class-name: com.mysql.jdbc.Driver
  batch:
    jdbc:
      initialize-schema: always
    job:
      enabled: false # 자동 실행 X
```
```java
@RequiredArgsConstructor
@Configuration
public class JobInstanceConfiguration {

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
                        JobInstance jobInstance = contribution.getStepExecution().getJobExecution().getJobInstance();
                        System.out.println("jobInstance.getId() : " + jobInstance.getId());
                        System.out.println("jobInstance.getInstanceId() : " + jobInstance.getInstanceId());
                        System.out.println("jobInstance.getJobName() : " + jobInstance.getJobName());
                        System.out.println("jobInstance.getJobVersion : " + jobInstance.getVersion());
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
@Component
public class JobRunner implements ApplicationRunner {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addString("name", "user1")
//                .addDate("reqDate", new Date())
                .toJobParameters();

        jobLauncher.run(job, jobParameters); // Job + JobParameters
    }
}
```

### 3. JobParameter
1. 기본 개념
  - Job을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 도메인 객체
  - 하나의 Job에 존재할 수 있는 여러 개의 JobInstance를 구분하기 위한 용도
  - JobParameters와 JobInstance는 1:1 관계
2. 생성 및 바인딩
  - 어플리케이션 시랭 시 주입
    - `Java -jar LogBatch.jar requestDate=20210101`
  - 코드로 생성
    - `JobParameterBuilder, DefaultJobParametersConverter`
  - SpEL 이용
    - `@Value("#{jobParameter[requestDate]}")`, `@JobScope`, `@StepScope` 선언 필수
3. `BATCH_JOB_EXECUTION_PARAM` 테이블과 매핑
  - JOB_EXECUTION과 1:M의 관계
4. `JobParameters` -> `JobParameter`(Object parameter, ParameterType parameterType, Boolean identifying) -> `ParameterType`(String, Date, Long, Double)

- `git checkout Part3.1.3`
```java
@Component
public class JobParameterTest implements ApplicationRunner {

    @Autowired
    JobLauncher jobLauncher;

    @Autowired
    Job job;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        JobParameters jobParameters = new JobParametersBuilder()
                .addString("name", "user1")
                .addLong("seq", 1L)
                .addDate("date", new Date())
                .toJobParameters();

        jobLauncher.run(job, jobParameters);
    }
}
```
```java
@RequiredArgsConstructor
@Configuration
public class JobParameterConfiguration {

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

                        JobParameters jobParameters = contribution.getStepExecution().getJobParameters(); // 1. contribution 이용
                        String name = jobParameters.getString("name");
                        long seq = jobParameters.getLong("seq");
                        Date date = jobParameters.getDate("date");

                        System.out.println("===========================");
                        System.out.println("name:" + name);
                        System.out.println("seq: " + seq);
                        System.out.println("date: " + date);
                        System.out.println("===========================");

                        Map<String, Object> jobParameters2 = chunkContext.getStepContext().getJobParameters(); // 2. chunkContext 이용(Map 반환)
                        String name2 = (String)jobParameters2.get("name");
                        long seq2 = (long)jobParameters2.get("seq");

                        System.out.println("step1 has executed");
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
                        System.out.println("step1 has executed");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
}
```
- 터미널 실행 : 타입 지정 필수
> mvn package<br/>
cd ./target<br/>
java -jar springbatchlecture-0.0.1-SNAPSHOT.jar name=user1 seq(long)=2L date(date)=2024/01/01 age(double)=16.5

### 4. JobExecution
1. 기본 개념
  - JobInstance에 대한 한 번의 시도를 의미하는 객체로서 Job 실행 중에 발생한 정보들을 저장하고 있는 객체
    - 시작시간, 종료시간, 상태(시작됨, 완료, 싪), 종료 상태의 속성을 가짐
  - JobInstance와의 관계
    - JobExecution은 `FAILED` 또는 `COMPLETED` 등의 Job의 실행 결과 상태를 가지고 있음
    - JobExecution의 실행 상태 결과가 `COMPLETED`면 JobInstance 실행이 완료된 것으로 간주해서 **재실행이 불가함**
    - JobExecution의 실행 상태 결과가 `FAILED`면 JobInstance 실행이 완료되지 않은 것으로 간주해서 **재실행이 가능함**
      - JobParameter가 동일한 값으로 Job을 실행할지라도 JobInstance를 계속 실행할 수 있음
    - JobExecution의 실행 상태 결과가 `COMPLETED` 될 때까지 하나의 JobInstance 내에서 여러 번의 시도가 생길 수 있음
2. `BATCH_JOB_EXECUTION` 테이블과 매핑
  - JobInstace와 JobExecution은 1:M의 관계로서 JobInstance에 대한 성공/실패의 내역을 가지고 있음
3. Entity(id, version) -> JobExecution(...)
  - JobParameters
  - JobInstance
  - ExecutionContext : 실행하는 동안 유지해야 하는 데이터를 담고 있음
  - BatchStatus : 실행 상태를 나타내는 Enum 클래스(COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN)
  - ExitStatus : 실행결과를 나타내는 클래스로서 종료코드를 포함(UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED)
  - failureExceptions : Job 실행 중 발생한 예외 리스트
  - startTime : Job을 실행할 때의 시스템 시간
  - createTime : JobExecution이 처음 저장될 때의 시스템 시간
  - endTime : 성공 여부와 상관없이 실행이 종료되는 시간
  - lastUpdated : JobExecution이 마지막 저장될 때의 시스템 시간
4. JobLauncher -> Job, JobParameters -> JobRepository, DB -> oldJobInstance or newJobInstance -> new JobExecution
  - oldJobInstance -> BatshStatus? `COMPLETED` -> **JobInstanceAlreadyCompleteException**, BATCH_JOB_INSTANCE, BATCH_JOB_EXECUTION 테이블들에 Row 하나씩 저장됨
  - oldJobInstance -> BatshStatus? `FAILED` -> **new JobExecution**, BATCH_JOB_INSTANCE 테이블에 Row 하나 저장, BATCH_JOB_EXECUTION 테이블에 `FAILED` 될 때마다 `COMPLETED` 될 때까지 **하나씩 추가됨**

- `git checkout Part3.1.4`
```java
@RequiredArgsConstructor
@Configuration
public class JobExecutionConfiguration {

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

                        JobExecution jobExecution = contribution.getStepExecution().getJobExecution();
                        System.out.println("jobExecution = " + jobExecution);

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
//                    throw new RuntimeException("JobExecution has failed"); // FAILED
                    System.out.println("step2 has executed"); // COMPLETED
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```
- Run Configuration
> Program arguments : name=user1