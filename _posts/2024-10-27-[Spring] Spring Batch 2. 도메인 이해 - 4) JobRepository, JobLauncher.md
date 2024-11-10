---
title: '[Spring] Spring Batch 2. 도메인 이해 - 4) JobRepository, JobLauncher'
date: 2024-10-27 20:10:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## JobRepository / JobLauncher
### JobRepository
1. 기본 개념
    - 배치 작업 중의 정보를 저장하는 저장소 역할
    - Job이 언제 수행되었고, 언제 끝났으며, 몇 번이 실행되었고 실행에 대한 결과 등의 배치 작업의 수행과 관련된 모든 meta data를 저장함
        - JobLauncher, Job, Step 구현체 내부에서 CRUD 기능을 처리함
    - DB -> JobRepository <-> (JobLauncher -> Job -> Step(ItemReader, ItemProcessor, ItemWriter))
2. JobRepository 메소드
    - boolean isJobInstanceExsits(jobName, JobParameters)
    - jobExecution createJobExecution(jobName, JobParameters)
    - jobExecution getLastJobExecution(jobName, JobParameters)
    - update(JobExecution) (StepExecution)
    - add(StepExecution)
    - updateExecutionContext(JobExecution) (StepExecution)
    StepExecution getLastStepExecution(JobInstance, stepName)
3. JobRepository 설정
    - `@EnableBatchProcessing` 어노테이션만 선언하면 JobRepository가 자동으로 빈으로 생성됨
    - BatchConfigurer 인터페이스를 구현하거나 BasicBatchConfigurer를 상속해서 JobRepository 설정을 커스터마이징 할 수 있다.
        - **JDBC 방식으로 설정 - JobRepositoryFactoryBean**
            - 내부적으로 AOP 기술을 통해 트랜잭션 처리를 해주고 있음
            - 트랜잭션 isolation의 기본값은 SERIALIZEBLE로 최고 수준, 다른 레벨(READ_COMMITED, REPEATABLE_READ)로 지정 가능
            - 메타테이블의 Table Prefix를 변경할 수 있음, 기본 값은 "BATCH_"임
        - **In Memory 방식으로 설정 - MapJobRepositoryFactoryBean**
            - 성능 등의 이유로 도메인 오브젝트를 굳이 데이터베이스에 저장하고 싶지 않을 경우
            - 보통 Test나 프로토타입의 빠른 개발이 필요할 때 사용

    1) JDBC
    ```java
    @Override
    protected JobRepository createJobRepository() throws Exception {
        JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTransactionManager(transactionManager);
        factory.setIsolationLevelForCreate("ISOLATION SERIALIZABLE"); // isolation 수준, 기본값은 "ISOLATION_SERIALIZABLE"
        factory.setTablePrefix("SYSTEM_"); // 테이블 Prefix, 기본값은 "BATCH_", BATCH_JOB_EXECUTION가 SYSTEM_JOB_EXECUTION으로 변경됨
        factory.setMaxVarCharLength(1000); // varchar 최대 길이(기본값 2500)
        return factory.getObject(); // Proxy 객체가 생성됨 (트랜잭션 Advice 적용 등을 위해 AOP 기술 적용)
    }
    ```

    2) In Memory
    ```java
    @Override
    protected JobRepository createJobRepository() throws Exception {
        MapJobRepositoryFactoryBean factory = new MapJobRepositoryFactoryBean();
        factory.setTransactionManager(transactionManager); // ResourcelessTransactionManager 사용
        return factory.getObject();
    }
    ```

- `git checkout Part3.4`
```java
@RequiredArgsConstructor
@Configuration
public class JobRepositoryConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final JobRepositoryListener jobRepositoryListener;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("Job")
                .start(step1())
                .next(step2())
                .listener(jobRepositoryListener) // Listener 등록
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> null)
                .build();
    }
}
```
```java
@Component
public class JobRepositoryListener implements JobExecutionListener {

    @Autowired
    private JobRepository jobRepository;

    @Override
    public void beforeJob(JobExecution jobExecution) {

    }

    @Override
    public void afterJob(JobExecution jobExecution) {

        String jobName = jobExecution.getJobInstance().getJobName();
        JobParameters jobParameters = new JobParametersBuilder().addString("requestDate", "20210102").toJobParameters();

        JobExecution lastExecution = jobRepository.getLastJobExecution(jobName, jobParameters);

        if(lastExecution != null) {
            for (StepExecution execution : lastExecution.getStepExecutions()) {
                BatchStatus status = execution.getStatus();
                System.out.println("BatchStatus = " + status.isRunning());
                System.out.println("BatchStatus = " + status.name());
            }
        }
    }
}
```
```java
@Configuration
public class CustomBatchConfigurer extends BasicBatchConfigurer { // 

    private final DataSource dataSource;

    protected CustomBatchConfigurer(BatchProperties properties, DataSource dataSource, TransactionManagerCustomizers transactionManagerCustomizers) {
        super(properties, dataSource, transactionManagerCustomizers);
        this.dataSource = dataSource;
    }

    @Override
    protected JobRepository createJobRepository() throws Exception {

        JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTransactionManager(getTransactionManager());
        factory.setIsolationLevelForCreate("ISOLATION_SERIALIZABLE"); // isolation 수준, 기본값은 “ISOLATION_SERIALIZABLE”
        factory.setTablePrefix("BATCH_");  // 테이블 Prefix, 기본값은 “BATCH_”,
                                            // BATCH_JOB_EXECUTION 가 SYSTEM_JOB_EXECUTION 으로 변경됨
                                            // 실제 테이블명이 변경되는 것은 아니다
        factory.setMaxVarCharLength(1000); // varchar 최대 길이(기본값 2500)

        return factory.getObject(); // Proxy 객체가 생성됨 (트랜잭션 Advice 적용 등을 위해 AOP 기술 적용)

    }
}
```

- Run Configuration
> > Program arguments : --name=batchJob requestDate=20240101

### JobLauncher
1. 기본 개념
    - 배치 Job을 실행시키는 역할을 한다.
    - Job과 Job Parameters를 인자로 받으며 요청된 배치 작업을 수행한 후 최종 client에게 JobExecution을 반환함
    - 스프링 부트 배치가 구동이 되면 JobLauncher 빈이 자동 생성된다.
    - Job 실행
        - JobLauncher.run(Job, JobParameters)
        - 스프링 부트 배치에서는 JobLauncherApplicationRunner가 자동적으로 JobLauncher을 실행시킨다.
        - **동기적 실행**
            - taskExecutor를 SyncTaskExecutor로 설정할 경우(기본값은 SyncTaskExecutor)
            - JobExecution을 획득하고 배치 처리를 최종 완료한 이후 Client에게 JobExecution을 반환
            - **스케줄러에 의한 배치처리에 적합함** - 배치처리시간이 길어도 상관없는 경우
        - **비동기적 실행**
            - taskExecutor가 SimpleAsyncTaskExecutor로 설정할 경우
            - JobExecution을 획득한 후 Client에게 바로 JobExecution을 반환하고 배치처치를 완료한다.
            - **HTTP 요청에 의한 배치처리에 적합함** - 배치처리 시간이 길 경우 응답이 늦어지지 않도록 함
2. 구조
    - 동기적 실행
        - Client(run()) -> JobLauncher(execute()) -> Job -> Business -> Job(ExitStatus) -> JobLauncher(**JobExecution**) -> Client
        - `ExistStatus.FINISHED` or `FAILED` 최종 완료 후 응답 값 반환
    - 비동기적 실행
        - Client(run()) -> JobLauncher(**JobExecution**) -> Client
        - `ExistStatus.UNKNOWN` 즉시 응답값 반환
        - JobLauncher(executed()) -> Job -> Business -> Job(ExitStatus) -> JobLauncher

- Dependency 추가
> spring-boot-start-web

- `git checkout Part3.5`
```java
@RequiredArgsConstructor
@Configuration
public class JobLauncherConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("Job")
                .start(step1())
                .next(step2())
                .incrementer(new RunIdIncrementer())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        Thread.sleep(3000);
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> null)
                .build();
    }
}
```
```java
@RestController
public class JobLaunchingController {

	@Autowired
	private Job job;

	@Autowired
	private JobLauncher simpleLauncher; // Bean 생성 X -> BasiBatchConfigurer 필요

	@Autowired
	private BasicBatchConfigurer basicBatchConfigurer;

	@PostMapping(value = "/batch")
	public String launch(@RequestBody Member member) throws Exception {

		JobParameters jobParameters = new JobParametersBuilder()
						.addString("id", member.getId())
						.addDate("date", new Date())
						.toJobParameters();

//		SimpleJobLauncher jobLauncher = (SimpleJobLauncher)simpleLauncher; // 프록시 객체로 생성되기 때문에 불가능
		SimpleJobLauncher jobLauncher = (SimpleJobLauncher)basicBatchConfigurer.getJobLauncher();
		jobLauncher.setTaskExecutor(new SimpleAsyncTaskExecutor()); // 동기(3초 걸림), 비동기(즉시 실행)
		jobLauncher.run(job, jobParameters);

		System.out.println("Job is completed");

		return "batch completed";
	}
}
```
```java
@Data
public class Member {
    private String id;

}
```
```yml
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