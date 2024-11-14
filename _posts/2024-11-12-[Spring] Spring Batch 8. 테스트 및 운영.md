---
title: '[Spring] Spring Batch 8. 테스트 및 운영'
date: 2024-11-12 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Spring Batch Test
1. Dependency(Spring Batch 4.1.x 이상, Spring Boot 2.1 기준)
> spring-batch-test

2. `@SpringBatchTest`
    - 자동으로 ApplicationContext에 테스트에 필요한 여러 유틸 Bean을 등록해 주는 어노테이션
        - JobLauncherTestUtils : launchJob(), launchStep()과 같은 스프링 배치 테스트에 필요한 유틸성 메소드 지원
            - void setJob(Job job) : 실행할 Job을 자동으로 주입 받음, 한 개의 Job만 받을 수 있음(Job ㅓㄹ정클래스를 한 개만 지정해야 함)
            - JobExecution launchJob(JobParameters jobParameters) : Job을 실행시키고 JobExecution을 반환
            - JobExecution launchStep(String stepName) : STep을 실행시키고 JobExecution을 반환
        - JobRepositoryTestUtils : JobRepository를 사용해서 JobExecution을 생성 및 삭제 기능 메소드 지원
            - `List<JobExecution>` createJobExecution(String jobNAme, String[] stepNames, int count) : JobExecution 생성(job 이름, step 이름, 생성 개수)
            - void removeJobExecutions(`Collection<JobExecution>` list) : JobExecution 삭제(JobExecution 목록)
        - StepScopeTestExecutionListener : `@StepScope` 컨텍스트를 생성해주며 해당 컨텍스트를 통해 JobParameter 등을 단위 테스트에서 DI 받을 수 있다.
        - JobScopeTestExecutionListener : `@JobScope` 컨텍스트를 생성해주며 해당 컨텍스트를 통해 JobPArameter 등을 단위 테스트에서 DI 받을 수 있다.
3. BatchJobConfigurationTest
    - `@SpringBatchTest` : JobLauncherTestUtils, JobRepositoryTestUtils 등을 제공하는 어노테이션
    - `@SpringBootTest(classes={...})` : Job 설정 클래스 지정, 통합 테스트를 위한 여러 의존성 빈들을 주입받기 위한 어노테이션
    ```java
    @RunWith(SpringRunner.class)
    @SpringBatchTest
    @SpringBootTest(classes=[BatchJobConfiguration.class, TestBatchConfig.class])
    public class BatchJobConfigurationTest {
        ...
    }
    ```

4. TestBatchConfig
    - `@EnableBatchProcessing` : 테스트 시 배치환경 및 설정 초기화를 자동 구동하기 위한 어노테이션
    - 테스트 클래스마다 선언하지 않고 공통으로 사용하기 위함
    ```java
    @Configuration
    @EnableAutoConfiguration
    @EnableBatchProcessing
    public class TestBatchConfig {
        ...
    }
    ```

- `git checkout Part10.1`
```java
@RequiredArgsConstructor
@Configuration
public class SimpleJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(100)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> customItemReader() {
        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setPageSize(100);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        reader.setQueryProvider(queryProvider);

        return reader;
    }

    @Bean
    public JdbcBatchItemWriter customItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }
}
```
```java
@Data
@AllArgsConstructor
public class Customer {

    private final long id;
    private final String firstName;
    private final String lastName;
    private final Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```

- test/java/io.spring.batch.springbatchlecture/
```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@SpringBootTest(classes={SimpleJobConfiguration.class, TestBatchConfig.class})
public class SimpleJobTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void simple_job_test() throws Exception {

        // given
        JobParameters jobParameters = new JobParametersBuilder()
                .addString("requestDate", "20020101")
                .addLong("date", new Date().getTime())
                .toJobParameters();

        // when
        // JobExecution jobExecution = jobLauncherTestUtils.launchJob(jobParameters); // Job 테스트
        JobExecution jobExecution = jobLauncherTestUtils.launchStep("step1"); // Step 테스트

        // then
        // Assert.assertEquals(jobExecution.getStatus(), BatchStatus.COMPLETED);
        // Assert.assertEquals(jobExecution.getExitStatus(), ExitStatus.COMPLETED);

        StepExecution stepExecution = (StepExecution)((List) jobExecution.getStepExecutions()).get(0);

        Assert.assertEquals(stepExecution.getCommitCount(), 11); // commit 개수 = 10 + 1(읽을 데이터 없지만 확인)
        Assert.assertEquals(stepExecution.getWriteCount(), 1000); // 데이터 개수 = 1000
        Assert.assertEquals(stepExecution.getWriteCount(), 1000);
    }

    @After
    public void clear() throws Exception {
        jdbcTemplate.execute("delete from customer2");
    }
}
```
```java
@Configuration
@EnableAutoConfiguration
@EnableBatchProcessing // 배치 환경을 자동 설정
public class TestBatchConfig {
}
```

- test/java/resources/application.yml
```yml
spring:
  profiles:
    active: mysql
  batch:
    job:
    #   enabled: false
      names: ${job.name:NONE}
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
```

- 결과
> customer2 테이블에 데이터 생성 : 1000개 데이터, 100개씩 Chunk 처리, 10개 commit<br/>
> BATCH_STEP_EXECUTION : COMMIT_COUNT(11), READ_COUNT(1000)

## JobExplorer / JobRegistry / JobOperator
1. JobExplorer
    - JobRepositoy의 readonly 버전
    - 실행 중인 Job의 실행 정보인 JobExecution 또는 Step의 실행 정보인 StepExecution을 조회할 수 있다
    - `List<JobInstance>` getJobInstances(String jobName, int start, int count) : start 인덱스부터 count만큼의 JobInstances을 얻는다.
    <br/>

    - JobExecution getJobExecution(Long executionId) : JobExecutionId를 이용하여 JobExecutions을 얻는다.
    - StepExecution getStepExecution(Long jobExecutionId, Long stepExecutionId) : JobExecutionId와 StepExecutionId를 이용하여 StepExecution을 얻는다.
    - JobInstance getJobInstance(Long instnaceId) : JobInstanceId를 이용하여 JobInstance를 얻는다.
    - `List<JobExecution>` getJobExecutions(JobInstance jobInstance) : JobInstance를 이용하여 JobExecutions들을 얻는다.
    - `Set<JobExecution>` findRunningJobExecutions(String jobName) : jobName을 이용하여 실행중인 Job의 JobExecution들을 얻는다.
    - `List<String>` getJobNames() : 실행 가능한 Job들의 이름을 얻는다.

2. JobRegistry
    - 생성된 Job을 자동으로 등록, 추적 및 관리하며 여러 곳에서 Job을 생성한 경우 ApplicationContext에서 Job을 수집해서 사용할 수 있다.
    - 기본 구현체로 Map 기반의 MapJobRegistry 클래스를 제공한다.
        - jobName을 Key로 하고 job을 값으로 하여 매핑한다.
    - Job 등록
        - JobRegistryBeanPostProcessor - BeanPostProcessor 단계에서 Bean 초기화 시 자동으로 JobRegistry에 Job을 등록시켜준다.
    <br/>

    - void register(JobFactory jobFactory) : JobFactory에 Job을 등록한다.
    - void unregister(String name) : JobFactory에 Job을 삭제한다.
    - Job getJob(String name) : Job을 얻는다.
    - `Set<String>` getJobNames() : jobName 들을 얻는다.

3. JobOperator
    - JobExplorer, JobRepository, JobRegistry, JobLauncher를 포함하고 있으며 배치의 중단, 재시작, Job 요약 등의 모니터링이 가능하다.
    - 기본 구현체로 SimpleJobOperator 클래스를 제공한다.
    <br/>

    - `Set<String>` getJobNames() : 실행가능한 Job들의 이름을 얻는다.
    - int getJobInstanceCount(String jobName) : JobInstance 개수를 얻는다.
    - `List<JobInstances>` getJobInstances(String jobName, int start, int count) : start 인덱스부터 count만큼의 JobInstances의 id들을 얻는다.
    - `List<Long>` getRunningExecutions(String jobName) : jobName을 이용하여 실행중인 Job의 JobExecution의 id를 얻는다.
    - Properties getParameters(long execuionId) : Job의 Execution id를 이용하여 Parameters를 얻는다.
    - start(String jobName, Properties jobParameters) : Job 이름, Job Parameter를 이용하여 Job을 시작한다.
    - restart(long executionId, Properties restartParameters) : JobExecutionId를 이용하여, 정지되었거나 이미 종료된 Job 중 재실행 가능한 Job을 재시작한다.
    - Long startNextInstance(String jobName) : 항상 새로운 Job을 실행시킨다. Job에 문제가 있거나 처음부터 재시작할 경우에 적합하다.
    - stop(long executionId) : JobExecutionId를 이용하여, 실행 중인 Job을 정지시킨다. Stop은 graceful하게 동작한다. 즉 Stop이 즉시 이뤄지지 않으며 현재 실행 중이던 step은 끝까지 다 실행된 후 job이 stop 된다.
    - JobInstance getJobInstance(long executionId) : JobExecutionId를 이용하여 JobInstance를 얻는다.
    - `List<JobExecution>` getJobExecutions(JobInstance instance) : JobInstance를 이용하여 JobExecution들을 얻는다.
    - JobExecution getJobExecution(long executionId) : JobExecutionId를 이용하여 JobExecution을 얻는다.
    - `List<StepExecution>` getStepExecutions(long jobExecutionId) : JobExecutionId를 이용하여 StepExecution들을 얻는다.

- Dependency
> spring-boot-starter-web

- `git checkout Part10.2`
```java
@RequiredArgsConstructor
@Configuration
public class JobOperationConfiguration {

    public final JobBuilderFactory jobBuilderFactory;
    public final StepBuilderFactory stepBuilderFactory;
    public final JobRegistry jobRegistry;

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) ->
                        {
                            System.out.println("step1 was executed");
                            Thread.sleep(5000);
                            return RepeatStatus.FINISHED; 
                        }
                )
                .build();
    }

    @Bean
    public Step step2() throws Exception {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) ->
                        {
                            System.out.println("step2 was executed");
                            Thread.sleep(5000);
                            return RepeatStatus.FINISHED;
                        }
                )
                .build();
    }

    @Bean
    public Job job1() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public BeanPostProcessor jobRegistryBeanPostProcessor() throws Exception {
        JobRegistryBeanPostProcessor postProcessor = new JobRegistryBeanPostProcessor();
        postProcessor.setJobRegistry(jobRegistry);
        return postProcessor;
    }
}
```
```java
@RestController
public class JobController {

	@Autowired
	private JobRegistry jobRegistry;

	@Autowired
	private JobOperator jobOperator;

	@Autowired
	private JobExplorer jobExplorer;

	@PostMapping(value = "/batch/start")
	public String start(@RequestBody JobInfo jobInfo) throws Exception { // NoSuchJobException, NoSuchJobExecutionException, JobExecutionNotRunningException

		for(Iterator<String> iterator = jobRegistry.getJobNames().iterator(); iterator.hasNext();){

			SimpleJob job = (SimpleJob)jobRegistry.getJob(iterator.next());
			System.out.println("job name: " + job.getName());

			jobOperator.start(job.getName(), "id=" + jobInfo.getId());
		}

		return "batch is started";
	}

	@PostMapping(value = "/batch/restart")
	public String restart() throws Exception {

		for(Iterator<String> iterator = jobRegistry.getJobNames().iterator(); iterator.hasNext();){

			SimpleJob job = (SimpleJob)jobRegistry.getJob(iterator.next());
			System.out.println("job name: " + job.getName());

			JobInstance lastJobInstance = jobExplorer.getLastJobInstance(job.getName());
			JobExecution lastJobExecution = jobExplorer.getLastJobExecution(lastJobInstance);
			jobOperator.restart(lastJobExecution.getId()); // 마지막 execution id

		}

		return "batch is restarted";
	}

	@PostMapping(value = "/batch/stop")
	public String stop() throws Exception {

		for(Iterator<String> iterator = jobRegistry.getJobNames().iterator(); iterator.hasNext();){

			SimpleJob job = (SimpleJob)jobRegistry.getJob(iterator.next());
			System.out.println("job name: " + job.getName());

			Set<JobExecution> runningJobExecutions = jobExplorer.findRunningJobExecutions(job.getName());
			JobExecution jobExecution = runningJobExecutions.iterator().next();

			jobOperator.stop(jobExecution.getId());
		}

		return "batch is stopped";
	}
}
```
```java
@Data
public class JobInfo {
    private String id;
}
```
```java
@Data
@AllArgsConstructor
public class Customer {

    private final long id;
    private final String firstName;
    private final String lastName;
    private final Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```

- main/batch.http
```
### Send POST request with json body
POST http://localhost:8080/batch/start
Content-Type: application/json
{
  "id": "leaven"
}
### Send POST request with json body
POST http://localhost:8080/batch/stop
Content-Type: application/json
### Send POST request with json body
POST http://localhost:8080/batch/restart
Content-Type: application/json
```

- Run Configuration
> --job.name=batchJob message=20240101

- 결과1 (Run 서버 실행)
> jobName : batchJob<br/>
> step1 was executed<br/>
> step2 was executed<br/>
> BATCH_JOB_INSTANCE 테이블 job instance 1개<br/>
> BATCH_JOB_EXECUTION 테이블 job execution 1개<br/>
> BATCH_STEP_EXECUTION 테이블 step execution 2개<br/>
> BATCH_JOB_PARAMS 테이블 run.id 1개, message 1개

- 결과2 (/batch/start 호출 -> /batch/start 재호출)
> batch is started<br/>
> jobName : batchJob<br/>
> step1 was executed<br/>
> step2 was executed<br/>
> BATCH_JOB_INSTANCE 테이블 job instance 2개 (1개 추가됨)<br/>
> BATCH_JOB_EXECUTION 테이블 job execution 2개 (1개 추가됨)<br/>
> BATCH_STEP_EXECUTION 테이블 step execution 4개 (2개 추가됨)<br/>
> A job instances already exists and is complete for parameters={id=leaven}. (똑같은 Job Parameters로는 실행 X, 동일한 Job 호출하므로 다시 실행 X)

- 결과3 (step1의 Thread.sleep(3000) -> /batch/start -> /batch/stop -> /batch/restart 호출)
> batch is started<br/>
> jobName : batchJob<br/>
> step1 was executed<br/>
> batch is stopped<br/>
> BATCH_JOB_INSTANCE 테이블 job instance 2개<br/>
> BATCH_JOB_EXECUTION 테이블 job execution 2개(COMPLETED, STOPPED)<br/>
> BATCH_STEP_EXECUTION 테이블 step execution 3개(step1 = COMPLETED, step2 = COMPLETED, step1 = STOPPED)<br/>
> batch is restarted<br/>
> BATCH_JOB_INSTANCE 테이블 job instance 2개 (1개 추가되지 않음)<br/>
> BATCH_JOB_EXECUTION 테이블 job execution 3개(COMPLETED, STOPPED, COMPLETED)<br/>
> BATCH_STEP_EXECUTION 테이블 step execution 3개(step1 = COMPLETED, step2 = COMPLETED, step1 = STOPPED, step1 = COMPLETED, step2 = COMPLETED)

- 참고
> JobOperator, JobExplorer, JobRegistryBeanPostProcessor, MapJobRegistry