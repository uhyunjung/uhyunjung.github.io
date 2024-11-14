---
title: '[Spring] Spring Batch 9. 어플리케이션 개발'
date: 2024-11-13 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 어플리케이션 예제
1. Job-1
    - 기능
        - 파일로부터 데이터를 읽어서 DB에 적재한다.
    - 세부내용
        - 파일은 매일 새롭게 생성된다.
        - 매일 정해진 시간에 파일을 로드하고 데이터를 DB에 업데이트한다.

2. Job-2
    - 기능
        - DB로부터 데이터를 읽어서 API 서버와 통신한다.
    - 내용
        - Partitioning 기능을 통한 멀티 스레드 구조로 Chunk 기반 프로세스를 구현한다.
        - 제품의 유형에 따라서 서로 다른 API 통신을 하도록 구성한다(ClassifierCompositeItemWriter)
        - API 서버는 3개로 구성하여 요청을 처리한다.
        - 제품내용과 API 통신 결과를 각 파일로 저장한다(FlatFileWriter 상속)

3. Scheduler
    - 기능
        - 시간을 설정하면 프로그램을 가동시킨다.
    - 내용
        - 정해진 시간에 주기적으로 Job-1과 Job-2를 실행시킨다.
        - Quatz 오픈 소스를 활용한다.
4. 구조
    - Scheduler
        - Job-1 : File <- ItemReader -> ItemWriter -> DB
        - Job-2 : DB <- ItemReader -> ItemProcessor -> ItemWriter (멀티 스레드 Worker)
            - ItemWriter -> API, File

5. `git checkout Part11.1`
- Dependency
> spring-boot-start-quartz<br/>
> org.apache.httpcomponents.httpclient

6. 구성
> 1, 2, 3, 4 : Product 테이블 및 데이터 생성(FileJob, Domain, Entity)<br/>
> 5, 6, 7, 8, 9 : Job & Step 구성<br/>
> 10, 11 : Partitioner<br/>
> 12, 13 : Processor Classifier<br/>
> 14, 15, 16 : ItemProcessor<br/>
> 17, 18, 19, 20 : Writer Classifier<br/>
> 21, 22, 23, 24, 25, 26 : Service<br/>
> 27, 28, 29, 30 : Scheduler<br/>
> 30 : JobDetail, JobDetailImpl, Trigger 추가<br/>
> 18, 19, 20 : Api Writer 수정<br/>

7. 패키지 구조
    ```
    java/io/springbatch/springbatchlecture/
        batch/
            chunk/
                processor/
                    ApiItemProcessor1 - 14 // 1, 2, 3 type
                    ApiItemProcessor2 - 15
                    ApiItemProcessor3 - 16
                    FileItemProcessor - 3
                    ProcessorClassifier - 13
                writer/
                    ApiItemWriter1 - 18
                    ApiItemWriter2 - 19
                    ApiItemWriter3 - 20
                    WriterClassifier - 17
                domain/
                    ApiInfo - 22
                    ApiRequestVO - 12
                    ApiResponseVO - 21
                    Product - 4
                    ProductVO - 2
                job/
                    api/
                        ApiStepConfiguration - 9 (apiMasterStep, apiSlaveStep, partitioner, taskExecutor, ItemReader, ItemProcessor, ItemWriter)
                        QueryGenerator - 11
                        SendChildJobConfiguration - 8 (jobStep, childJob)
                        SendJobConfiguration - 5 (apiJob, apiStep)
                    file/
                        FileJobConfiguration - 1 (FileItemReader, FileItemProcessor, FileItemWriter)
            listener/
                JobListener
            partition/
                ProductPartitioner - 10
            rowmapper/
                ProductRowMapper - 12
            tasklet/
                ApiStartTasklet - 6
                ApiEndTasklet - 7
        scheduler/
            ApiJobRunner
            ApiSchJob - 28
            FileJobRunner - 29
            FileSchJob - 30
            JobRunner - 27
        service/
            AbstractApiService - 23
            ApiService1 - 24
            ApiService2 - 25
            ApiService3 - 26
    resources/
         application.yml
        data-mysql.sql
        product_20210101.csv
        product_20210102.csv
    ```

## java/io/springbatch/springbatchlecture/
### batch/
### chunk/
### processor/

### ApiItemProcessor1
```java
@Component
public class ApiItemProcessor1 implements ItemProcessor<ProductVO, ApiRequestVO> {

    @Override
    public ApiRequestVO process(ProductVO productVO) throws Exception {

        return ApiRequestVO.builder()
                .id(productVO.getId())
                .productVO(productVO)
                .build();
    }
}
```

### ApiItemProcessor2
```java
@Component
public class ApiItemProcessor2 implements ItemProcessor<ProductVO, ApiRequestVO> {

    @Override
    public ApiRequestVO process(ProductVO productVO) throws Exception {

        return ApiRequestVO.builder()
                .id(productVO.getId())
                .productVO(productVO)
                .build();
    }
}
```

### ApiItemProcessor3
```java
@Component
public class ApiItemProcessor3 implements ItemProcessor<ProductVO, ApiRequestVO> {

    @Override
    public ApiRequestVO process(ProductVO productVO) throws Exception {

        return ApiRequestVO.builder()
                .id(productVO.getId())
                .productVO(productVO)
                .build();
    }
}
```

### FileItemProcessr
```java
public class FileItemProcessor implements ItemProcessor<ProductVO, Product> {

    @Override
    public Product process(ProductVO item) throws Exception {

        ModelMapper modelMapper = new ModelMapper(); // DTO -> Entity
        Product product = modelMapper.map(item, Product.class);

        return product;
    }
}
```

### ProcessorClassifier
```java
public class ProcessorClassifier<C,T> implements Classifier<C, T> {

    private Map<String, ItemProcessor<ProductVO, ApiRequestVO>> processorMap = new HashMap<>();

    @Override
    public T classify(C classifiable) {
        return (T)processorMap.get(((ProductVO)classifiable).getType());
    }

    public void setProcessorMap(Map<String, ItemProcessor<ProductVO, ApiRequestVO>> processorMap) {
        this.processorMap = processorMap;
    }
}
```

### writer/
### ApiItemWriter1
```java
@Slf4j
public class ApiItemWriter1 extends FlatFileItemWriter<ApiRequestVO> {

    private final AbstractApiService apiService;

    public ApiItemWriter1(AbstractApiService apiService) {
        this.apiService = apiService;
    }

    @Override
    public void write(List<? extends ApiRequestVO> items) throws Exception {

        System.out.println("----------------------------------");
        items.forEach(item -> System.out.println("items = " + item));
        System.out.println("----------------------------------");

        ApiResponseVO response = apiService.service(items);
        System.out.println("response = " + response);

        items.forEach(item -> item.setApiResponseVO(response));

        super.setResource(new FileSystemResource("C:\\jsw\\inflearn\\spring-batch-lecture\\src\\main\\resources\\product1.txt"));
        super.open(new ExecutionContext());
        super.setLineAggregator(new DelimitedLineAggregator<>());
        super.setAppendAllowed(true);
        super.write(items);
    }
}
```

### ApiItemWriter2
```java
@Slf4j
public class ApiItemWriter2 extends FlatFileItemWriter<ApiRequestVO> {

    private AbstractApiService apiService;

    public ApiItemWriter2(AbstractApiService apiService) {
        this.apiService = apiService;
    }

    @Override
    public void write(List<? extends ApiRequestVO> items) throws Exception {

        System.out.println("----------------------------------");
        items.forEach(item -> System.out.println("items = " + item));
        System.out.println("----------------------------------");

        ApiResponseVO response = apiService.service(items);
        System.out.println("response = " + response);

        items.forEach(item -> item.setApiResponseVO(response));

        super.setResource(new FileSystemResource("C:\\jsw\\inflearn\\spring-batch-lecture\\src\\main\\resources\\product2.txt"));
        super.open(new ExecutionContext());
        super.setLineAggregator(new DelimitedLineAggregator<>());
        super.setAppendAllowed(true);
        super.write(items);
    }
}
```

### ApiItemWriter3
```java
@Slf4j
public class ApiItemWriter3 extends FlatFileItemWriter<ApiRequestVO> {

    private AbstractApiService apiService;

    public ApiItemWriter3(AbstractApiService apiService) {
        this.apiService = apiService;
    }

    @Override
    public void write(List<? extends ApiRequestVO> items) throws Exception {

        System.out.println("----------------------------------");
        items.forEach(item -> System.out.println("items = " + item));
        System.out.println("----------------------------------");

        ApiResponseVO response = apiService.service(items);
        System.out.println("response = " + response);

        items.forEach(item -> item.setApiResponseVO(response));

        super.setResource(new FileSystemResource("C:\\jsw\\inflearn\\spring-batch-lecture\\src\\main\\resources\\product3.txt"));
        super.open(new ExecutionContext());
        super.setLineAggregator(new DelimitedLineAggregator<>());
        super.setAppendAllowed(true);
        super.write(items);
    }
}
```

### WriterClassifier
```java
public class WriterClassifier<C,T> implements Classifier<C, T> {

    private Map<String, ItemWriter<ApiRequestVO>> writerMap = new HashMap<>();

    @Override
    public T classify(C classifiable) {
        return (T)writerMap.get(((ApiRequestVO)classifiable).getProductVO().getType());
    }

    public void setWriterMap(Map<String, ItemWriter<ApiRequestVO>> writerMap) {
        this.writerMap = writerMap;
    }
}
```

### domain/
### ApiInfo
```java
@Data
@Builder
public class ApiInfo {

    private String url;
    private List<? extends ApiRequestVO> apiRequestList;
}
```

### ApiRequestVO
```java
@Data
@Builder
public class ApiRequestVO{

    private long id;
    private ProductVO productVO;
    private ApiResponseVO apiResponseVO;

}
```

### ApiResponseVO
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ApiResponseVO{
    private String status;
    private String msg;
}
```

### Product
```java
@Data
@Entity
public class Product {

    @Id
    private Long id;
    private String name;
    private int price;
    private String type;
}
```

### ProductVO
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ProductVO {

    private Long id;
    private String name;
    private int price;
    private String type;
}
```

### job/
### api/
### ApiStepConfiguration
```java
@Configuration
@RequiredArgsConstructor
public class ApiStepConfiguration {

    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    private int chunkSize = 10;

    @Bean
    public Step apiMasterStep() throws Exception {

        ProductVO[] productList = QueryGenerator.getProductList(dataSource);

        return stepBuilderFactory.get("apiMasterStep")
                .partitioner(apiSlaveStep().getName(), partitioner())
                .step(apiSlaveStep())
                .gridSize(productList.length)
                .taskExecutor(taskExecutor())
                .build();
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(3); // thread 3개
        taskExecutor.setMaxPoolSize(6);
        taskExecutor.setThreadNamePrefix("api-thread-");

        return taskExecutor;
    }

    @Bean
    public Step apiSlaveStep() throws Exception {

        return stepBuilderFactory.get("apiSlaveStep")
                .<ProductVO, ProductVO>chunk(chunkSize)
                .reader(itemReader(null))
                .processor(itemProcessor())
                .writer(itemWriter())
                .build();
    }

    @Bean
    public ProductPartitioner partitioner() {
        ProductPartitioner productPartitioner = new ProductPartitioner();
        productPartitioner.setDataSource(dataSource);
        return productPartitioner;
    }

    @Bean
    @StepScope
    public ItemReader<ProductVO> itemReader(@Value("#{stepExecutionContext['product']}") ProductVO productVO) throws Exception {

        JdbcPagingItemReader<ProductVO> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(dataSource);
        reader.setPageSize(chunkSize);
        reader.setRowMapper(new BeanPropertyRowMapper(ProductVO.class));

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, name, price, type");
        queryProvider.setFromClause("from product");
        queryProvider.setWhereClause("where type = :type");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.DESCENDING);
        queryProvider.setSortKeys(sortKeys);

        reader.setParameterValues(QueryGenerator.getParameterForQuery("type", productVO.getType()));
        reader.setQueryProvider(queryProvider);
        reader.afterPropertiesSet();

        return reader;
    }

    @Bean
    public ItemProcessor itemProcessor() {

        ClassifierCompositeItemProcessor<ProductVO, ApiRequestVO> processor = new ClassifierCompositeItemProcessor<>();

        ProcessorClassifier<ProductVO, ItemProcessor<?, ? extends ApiRequestVO>> classifier = new ProcessorClassifier();

        Map<String, ItemProcessor<ProductVO, ApiRequestVO>> processorMap = new HashMap<>();
        processorMap.put("1", new ApiItemProcessor1());
        processorMap.put("2", new ApiItemProcessor2());
        processorMap.put("3", new ApiItemProcessor3());

        classifier.setProcessorMap(processorMap);

        processor.setClassifier(classifier);

        return processor;
    }

    @Bean
    public ItemWriter itemWriter() {

        ClassifierCompositeItemWriter<ApiRequestVO> writer = new ClassifierCompositeItemWriter<>();

        WriterClassifier<ApiRequestVO, ItemWriter<? super ApiRequestVO>> classifier = new WriterClassifier();

        Map<String, ItemWriter<ApiRequestVO>> writerMap = new HashMap<>();
        writerMap.put("1", new ApiItemWriter1(new ApiService1()));
        writerMap.put("2", new ApiItemWriter2(new ApiService2()));
        writerMap.put("3", new ApiItemWriter3(new ApiService3()));

        classifier.setWriterMap(writerMap);

        writer.setClassifier(classifier);

        return writer;
    }
}
```

### QueryGenerator
```java
public class QueryGenerator {

    public static ProductVO[] getProductList(DataSource dataSource) {

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        List<ProductVO> productList = jdbcTemplate.query("select type as type from product group by type", new ProductRowMapper() {
            @Override
            public ProductVO mapRow(ResultSet rs, int i) throws SQLException {
                return ProductVO.builder().type(rs.getString("type")).build();
            }
        });

        return productList.toArray(new ProductVO[]{});
    }

    public static Map<String, Object> getParameterForQuery(String parameter, String value) {

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put(parameter, value);
        return parameters;
    }
}
```

### SendChildJobConfiguration
```java
@Configuration
@RequiredArgsConstructor
public class SendChildJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final Step apiMasterStep;
    private final JobLauncher jobLauncher;

    @Bean
    public Step jobStep() throws Exception {
        return stepBuilderFactory.get("jobStep")
                .job(childJob())
                .launcher(jobLauncher)
                .build();
    }

    @Bean
    public Job childJob() throws Exception {
        return jobBuilderFactory.get("childJob")
                .start(apiMasterStep)
                .build();
    }
}
```

### SendJobConfiguration
```java
@Configuration
@RequiredArgsConstructor
public class SendJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final ApiStartTasklet apiStartTasklet;
    private final ApiEndTasklet apiEndTasklet;
    private final Step jobStep;

    @Bean
    public Job apiJob() throws Exception {

        return jobBuilderFactory.get("apiJob")
                .incrementer(new RunIdIncrementer())
                .listener(new JobListener())
                .start(apiStep1())
                .next(jobStep)
                .next(apiStep2())
                .build();
    }

    @Bean
    public Step apiStep1() throws Exception {
        return stepBuilderFactory.get("apiStep")
                .tasklet(apiStartTasklet)
                .build();
    }

    @Bean
    public Step apiStep2() throws Exception {
        return stepBuilderFactory.get("apiStep2")
                .tasklet(apiEndTasklet)
                .build();
    }
}
```

### file/
### FileJobConfiguration
```java
@Configuration
@RequiredArgsConstructor
public class FileJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final EntityManagerFactory entityManagerFactory;

    @Bean
    public Job fileJob() {
        return jobBuilderFactory.get("fileJob")
                .start(fileStep1())
                .build();
    }

    @Bean
    public Step fileStep1() {
        return stepBuilderFactory.get("fileStep1")
                .<ProductVO, Product>chunk(10)
                .reader(fileItemReader(null))
                .processor(fileItemProcessor())
                .writer(fileItemWriter())
                .build();
    }

    @Bean
    @StepScope
    public FlatFileItemReader<ProductVO> fileItemReader(@Value("#{jobParameters['requestDate']}") String requestDate) {
        return new FlatFileItemReaderBuilder<ProductVO>()
                .name("flatFile")
                .resource(new ClassPathResource("product_" + requestDate +".csv"))
                .fieldSetMapper(new BeanWrapperFieldSetMapper<>())
                .targetType(ProductVO.class)
                .linesToSkip(1)
                .delimited().delimiter(",")
                .names("id","name","price","type")
                .build();
    }

    @Bean
    public ItemProcessor<ProductVO, Product> fileItemProcessor() {
        return new FileItemProcessor();
    }

    @Bean
    public JpaItemWriter<Product> fileItemWriter() {
        return new JpaItemWriterBuilder<Product>()
                .entityManagerFactory(entityManagerFactory)
                .usePersist(true)
                .build();
    }
}
```

### listener/
### JobListener
```java
/**
 * <pre>
 * io.anymobi.core.batch.listener.job
 * ㄴ DataSendJobListener.java
 * </pre>
 * 배치 Job 이 실행되면 호출되는 JobExecutionListener
 *
 * @author : soowon.jung
 * @version : 1.0.0
 * @date : 2021-07-22 오후 1:36
 * @see :
 **/

public class JobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long time = jobExecution.getEndTime().getTime() - jobExecution.getStartTime().getTime();
        System.out.println("총 소요시간 : " + time);
    }
}
```

### partition/
### ProductPartitioner
```java
public class ProductPartitioner implements Partitioner {

	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	@Override
	public Map<String, ExecutionContext> partition(int gridSize) { // execution context 생성

		ProductVO[] productList = QueryGenerator.getProductList(dataSource);
		Map<String, ExecutionContext> result = new HashMap<>();
		int number = 0;

		for (int i = 0; i < productList.length; i++) {

			ExecutionContext value = new ExecutionContext();

			result.put("partition" + number, value);
			value.put("product", productList[i]);

			number++;
		}

		return result;
	}}
```

### rowmapper/
### ProductRowMapper
```java
public class ProductRowMapper implements RowMapper<ProductVO> {
    @Override
    public ProductVO mapRow(ResultSet rs, int i) throws SQLException {
        return ProductVO.builder()
                .id(rs.getLong("id"))
                .name(rs.getString("name"))
                .price(rs.getInt("price"))
                .type(rs.getString("type"))
                .build();
    }
}
```

### tasklet/
### ApiStartTasklet
```java
@Component // Bean 등록
public class ApiStartTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

        System.out.println("");
        System.out.println(">> ApiStartTasklet is started");
        System.out.println("");

        return RepeatStatus.FINISHED;
    }
}
```

### ApiEndTasklet
```java
@Component // Bean 등록
public class ApiEndTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

        System.out.println("");
        System.out.println(">> ApiEndTasklet is started");
        System.out.println("");
        System.out.println("******************************************************************************************************************************************************");
        System.out.println("*                                                               Spring Batch is completed                                                            *");
        System.out.println("******************************************************************************************************************************************************");
        System.out.println("");

        return RepeatStatus.FINISHED;
    }
}
```

### scheduler/
### ApiJobRunner
```java
@Component
public class ApiJobRunner extends JobRunner {

    @Autowired
    private Scheduler scheduler;

    @Override
    protected void doRun(ApplicationArguments args) {

        JobDetail jobDetail = buildJobDetail(ApiSchJob.class, "apiJob", "batch", new HashMap());
        Trigger trigger = buildJobTrigger("0/30 * * * * ?"); // 시간 30초

        try {
            scheduler.scheduleJob(jobDetail, trigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
    }
}
```

### ApiSchJob
```java
@Component
@Slf4j
public class ApiSchJob extends QuartzJobBean{

	@Autowired
	private Job apiJob;

	@Autowired
	private JobLauncher jobLauncher;

	@SneakyThrows
	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {

		JobParameters jobParameters = new JobParametersBuilder()
									.addLong("id", new Date().getTime())
									.toJobParameters();
		jobLauncher.run(apiJob, jobParameters);
	}
}
```

### FileJobRunner
```java
@Component
public class FileJobRunner extends JobRunner {

    @Autowired
    private Scheduler scheduler;

    @Override
    protected void doRun(ApplicationArguments args) {

        String[] sourceArgs = args.getSourceArgs();
        JobDetail jobDetail = buildJobDetail(FileSchJob.class, "fileJob", "batch", new HashMap());
        Trigger trigger = buildJobTrigger("0/50 * * * * ?");
        jobDetail.getJobDataMap().put("requestDate", sourceArgs[0]);

        try {
            scheduler.scheduleJob(jobDetail, trigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
    }
}
```

### FileSchJob
```java
@Component
@Slf4j
public class FileSchJob extends QuartzJobBean{

	@Autowired
	private Job fileJob;

	@Autowired
	private JobLauncher jobLauncher;

	@Autowired
	private JobExplorer jobExplorer;

	@SneakyThrows
	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {

		String requestDate = (String)context.getJobDetail().getJobDataMap().get("requestDate");

		JobParameters jobParameters = new JobParametersBuilder()
									.addLong("id", new Date().getTime())
									.addString("requestDate", requestDate)
									.toJobParameters();

		int jobInstanceCount = jobExplorer.getJobInstanceCount(fileJob.getName());
		List<JobInstance> jobInstances = jobExplorer.getJobInstances(fileJob.getName(), 0, jobInstanceCount);

		if(jobInstances.size() > 0) {
			for(JobInstance jobInstance : jobInstances){
				List<JobExecution> jobExecutions = jobExplorer.getJobExecutions(jobInstance);
				List<JobExecution> jobExecutionList = jobExecutions.stream().filter(jobExecution ->
								jobExecution.getJobParameters().getString("requestDate").equals(requestDate))
						.collect(Collectors.toList());
				if (jobExecutionList.size() > 0) {
					throw new JobExecutionException(requestDate + " already exists");
				}
			}
		}

		jobLauncher.run(fileJob, jobParameters);
	}
}
```

### JobRunner
```java
public abstract class JobRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        doRun(args);
    }

    protected abstract void doRun(ApplicationArguments args);

    public Trigger buildJobTrigger(String scheduleExp) {
        return TriggerBuilder.newTrigger()
                .withSchedule(CronScheduleBuilder.cronSchedule(scheduleExp)).build();
    }

    public JobDetail buildJobDetail(Class job, String name, String group, Map params) {
        JobDataMap jobDataMap = new JobDataMap();
        jobDataMap.putAll(params);

        return newJob(job).withIdentity(name, group)
                .usingJobData(jobDataMap)
                .build();
    }
}
```

### service/
### AbstractApiService
```java
@Service
public abstract class AbstractApiService {

    public ApiResponseVO service(List<? extends ApiRequestVO> apiRequest) {

        // 중계사업자와 API 연동 작업
        RestTemplateBuilder restTemplateBuilder = new RestTemplateBuilder();
        RestTemplate restTemplate = restTemplateBuilder.errorHandler(new ResponseErrorHandler() {
            @Override
            public boolean hasError(ClientHttpResponse clientHttpResponse) throws IOException {
                return false;
            }

            @Override
            public void handleError(ClientHttpResponse clientHttpResponse) throws IOException {

            }
        }).build();

        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        ApiInfo apiInfo = ApiInfo.builder().apiRequestList(apiRequest).build();
        HttpEntity<ApiInfo> reqEntity = new HttpEntity<>(apiInfo, headers);

        return doApiService(restTemplate, apiInfo);

    }

    protected abstract ApiResponseVO doApiService(RestTemplate restTemplate, ApiInfo apiInfo);
}
```

### ApiService1
```java
@Service
public class ApiService1 extends AbstractApiService{

    @Override
    public ApiResponseVO doApiService(RestTemplate restTemplate, ApiInfo apiInfo){

        ResponseEntity<String> response = restTemplate.postForEntity("http://localhost:8081/api/product/1", apiInfo, String.class);

        int statusCodeValue = response.getStatusCodeValue();
        ApiResponseVO apiResponseVO = new ApiResponseVO(statusCodeValue + "", response.getBody());

        return apiResponseVO;
    }
}
```

### ApiService2
```java
@Service
public class ApiService2 extends AbstractApiService{

    @Override
    public ApiResponseVO doApiService(RestTemplate restTemplate, ApiInfo apiInfo){

        ResponseEntity<String> response = restTemplate.postForEntity("http://localhost:8081/api/product/2", apiInfo, String.class);

        int statusCodeValue = response.getStatusCodeValue();
        ApiResponseVO apiResponseVO = new ApiResponseVO(statusCodeValue + "", response.getBody());

        return apiResponseVO;
    }
}
```

### ApiService3
```java
@Service
public class ApiService3 extends AbstractApiService{

    @Override
    public ApiResponseVO doApiService(RestTemplate restTemplate, ApiInfo apiInfo){

        ResponseEntity<String> response = restTemplate.postForEntity("http://localhost:8081/api/product/3", apiInfo, String.class);

        int statusCodeValue = response.getStatusCodeValue();
        ApiResponseVO apiResponseVO = new ApiResponseVO(statusCodeValue + "", response.getBody());

        return apiResponseVO;
    }
}
```

## resources/
### application.yml
```yml
spring:
  batch:

  profiles:
    active: mysql
  jpa:
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    properties:
      hibernate.format_sql: true
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
    job:
      names: ${job.name:NONE}
      enabled: false
    jdbc:
      initialize-schema: always
```

### data-mysql.sql
```sql
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (1,"user1",1000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (2,"user2",2000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (3,"user3",3000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (4,"user4",4000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (5,"user5",5000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (6,"user6",6000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (7,"user7",7000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (8,"user8",8000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (9,"user9",9000,"1");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (10,"user10",10000,"1");

INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (11,"user11",11000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (12,"user12",12000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (13,"user13",13000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (14,"user14",14000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (15,"user15",15000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (16,"user16",16000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (17,"user17",17000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (18,"user18",18000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (19,"user19",19000,"2");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (20,"user20",20000,"2");

INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (21,"user21",21000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (22,"user22",22000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (23,"user23",23000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (24,"user24",24000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (25,"user25",25000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (26,"user26",26000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (27,"user27",27000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (28,"user28",28000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (29,"user29",29000,"3");
INSERT INTO `customer` (`id`,`name`,`price`,`type`) VALUES (30,"user30",30000,"3");
```

### product_20210101.csv
```
id,name,price,type
1,item1,1000,1
2,item2,2000,1
3,item3,3000,1
4,item4,4000,1
5,item5,5000,1
6,item6,6000,1
7,item7,7000,1
8,item8,8000,1
9,item9,9000,1
10,item10,10000,1
11,item11,11000,2
12,item12,12000,2
13,item13,13000,2
14,item14,14000,2
15,item15,15000,2
16,item16,16000,2
17,item17,17000,2
18,item18,18000,2
19,item19,19000,2
20,item20,20000,2
21,item21,21000,3
22,item22,22000,3
23,item23,23000,3
24,item24,24000,3
25,item25,25000,3
26,item26,26000,3
27,item27,27000,3
28,item28,28000,3
29,item29,29000,3
30,item30,30000,3
```

### product_20210102.csv
```
id,name,price,type
31,item31,1000,1
32,item32,2000,1
33,item33,3000,1
34,item34,4000,1
35,item35,5000,1
36,item36,6000,1
37,item37,7000,1
38,item38,8000,1
39,item39,9000,1
40,item40,10000,1
41,item41,11000,2
42,item42,12000,2
43,item43,13000,2
44,item44,14000,2
45,item45,15000,2
46,item46,16000,2
47,item47,17000,2
48,item48,18000,2
49,item49,19000,2
50,item50,20000,2
51,item51,21000,3
52,item52,22000,3
53,item53,23000,3
54,item54,24000,3
55,item55,25000,3
56,item56,26000,3
57,item57,27000,3
58,item58,28000,3
59,item59,29000,3
60,item50,30000,3
```


- Run Configuration
> program arguments : --job.name=fileJob requestDate=20210101<br/>
> program arguments : --job.name=apiJob requestDate=20210101<br/>
> program arguments : 20210101
> program arguments : 20210102

- 결과1 (fileJob 실행)
> product_20210101로 product 테이블 및 데이터생성

- 결과2 (apiJob 실행 : Thread 3개 * chunk Size 10개 * 2번 = 데이터 60개)
> ApiService is started<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 960

- 결과3 (Scheduler 실행 -> ApiJobRunner(최초 테이블에 데이터 없음) -> FileJobRunner -> JobRunner(program arguments의 requestDate 사용))
> ApiService is started<br/>
> ApiService is ended<br/>
> 총 소요시간 = 538<br/>
> ApiService is started<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 910<br/>
> ApiService is started<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 941

- 결과4 (20210101 -> 20210102, 날짜마다 배치 1번만 실행)
> org.quartz.JobExecutionException : 20210101 already exists (FileJob error)<br/>
> ApiService is started (ApiJob 실행)<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 682<br/>

- 결과5 (20210101 -> 20210102, 날짜마다 배치 1번만 실행)
> org.quartz.JobExecutionException : 20210101 already exists (FileJob error)<br/>
> ApiService is started (ApiJob 실행)<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 682<br/>

- 결과6 (20210101 -> 20210102, ApiWriter product1, 2, 3.txt 파일 저장 append)
> org.quartz.JobExecutionException : 20210101 already exists (FileJob error)<br/>
> ApiService is started<br/>
> responseVO = ApiResponseVO(status=200, msg=product3 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product2 was successfully processed)<br/>
> responseVO = ApiResponseVO(status=200, msg=product1 was successfully processed)<br/>
> ApiService is ended<br/>
> 총 소요시간 = 1117

- Server
- Dependency
> lombok<br/>
> spring-web

```java
@RestController
public class ApiController { // 각각의 API 서버1, 2, 3에서 구성되었다고 가정

    @PostMapping("/api/product/1")
    public String product1(@RequestBody ApiInfo apiInfo) {

        List<ProductVO> productVOList = apiInfo.getApiRequestList().stream().map(item -> item.getProductVO()).collect(Collectors.toList());
        System.out.println("collect = " + collect);

        return "product1 was successfully processed";
    }

    @PostMapping("/api/product/2")
    public String product2(@RequestBody ApiInfo apiInfo) {

        List<ProductVO> productVOList = apiInfo.getApiRequestList().stream().map(item -> item.getProductVO()).collect(Collectors.toList());
        System.out.println("collect = " + collect);

        return "product2 was successfully processed";
    }

    @PostMapping("/api/product/3")
    public String product3(@RequestBody ApiInfo apiInfo) {

        List<ProductVO> productVOList = apiInfo.getApiRequestList().stream().map(item -> item.getProductVO()).collect(Collectors.toList());
        System.out.println("collect = " + collect);

        return "product3 was successfully processed";
    }
}
```
```java
@Data
@Builder
public class ApiInfo {

    private String url
    private List<? extends ApiRequestVO> apiRequestList;
}
```
```java
@Data
@Builder
public class ApiRequestVO {

    private long id;
    private ProductVO productVO;
}
```

- Run Configuration
> program arguments : --server.port=8081<br/>
> program arguments : --server.port=8082<br/>
> program arguments : --server.port=8083
> View > Tool Windows > Service > Run Configuration Type > Spring Boot > Running 3개