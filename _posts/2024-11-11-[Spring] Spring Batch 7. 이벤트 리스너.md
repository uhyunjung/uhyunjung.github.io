---
title: '[Spring] Spring Batch 7. 이벤트 리스너'
date: 2024-11-11 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 이벤트 리스너
1. 기본 개념
    - Listener는 배치 흐름 중에 Job, Step, Chunk 단계의 실행 전후에 발생하는 이벤트를 받아 용도에 맞게 활용할 수 있도록 제공하는 인터셉터 개념의 클래스
    - 각 단계별로 로그기록을 남기거나 소요된 시간을 계산하거나 실행 상태 정보들을 참조 및 조회할 수 있다.
    - 이벤트를 받기 위해서는 Listener를 등록해야 하며 등록은 API 설정에서 각 단계별로 지정할 수 있다.

2. Listeners 종류
    - Job
        - JobExecutionListener : Job 실행 전후
    - Step
        - StepExecutionListener : Step 실행 전후
        - ChunkListener : Chunk 실행 전후 (Tasklet 실행 전후), 오류 시점
        - ItemReadListener : ItemReader 실행 전후, 오류 시점, Item이 null일 경우 호출 안 됨
        - ItemProcessListener : ItemProcessor 실행 전후, 오류 시점, Item이 null일 경우 호출 안 됨
        - ItemWriteListener : ItemWriter 실행 전후, 오류 시점, Item이 null일 경우 호출 안 됨
    - SkipListener : 읽기, 쓰기, 처리 Skip 실행 시점, Item 처리가 Skip 될 경우 Item을 추적함
    - RetryListener : Retry 시작, 종료, 에러 시점

3. 구현 방법
    - 어노테이션 방식
        - 인터페이스를 구현할 필요가 없다.
        - 클래스 및 메소드명을 자유롭게 작성할 수 있다.
        - **Object 타입의 Listener로 설정하기 위해서는 어노테이션 방식으로 구현해야 한다.**
        - `@BeforeStep`, `@AfterStep`
    - 인터페이스 방식
        - `implements StepExecutionListener`, `@Override beforeStep, afterStep`

4. 구조
    - JobLauncher -> **JobExecutionListener** -> Job -> **StepExecutionListener** -> Step -> **RepeatListener** -> Tasklet -> **ChunkListener** -> Chunk -> **ItemReadListener** -> (ItemReader -> **ItemProcessListener** -> ItemProcessor -> **ItemWriteListener** -> ItemWriter) -> SkipListener, RetryListener

- `git checkout Part9.1`
```java
@RequiredArgsConstructor
@Configuration
@Slf4j
public class ListenerConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
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
                        return RepeatStatus.FINISHED;
                    }
                })
                .listener(new CustomStepListener())
                .build();
    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        return RepeatStatus.FINISHED;
                    }
                })
                .listener(new AnnotationCustomStepListener())
                .build();
    }
}
```
```java
public class CustomStepListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        System.out.println("stepExecution.getStepName() : " + stepExecution.getStepName());
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        System.out.println("stepExecution.getStatus() : " + stepExecution.getStatus());
        return null;
    }
}
```
```java
public class AnnotationCustomStepListener {

    @BeforeStep
    public void beforeStep(StepExecution stepExecution){
        System.out.println("@stepExecution.getStepName() : " + stepExecution.getStepName());
    }

    @AfterStep
    public void afterStep(StepExecution stepExecution){
        System.out.println("@stepExecution.getStatus() : " + stepExecution.getStatus());
    }
}
```

## JobExecutionListener / StepExecutionListener
1. **JobExecutionListener**
    - void beforeJob(JobExecution jobExecution) : Job의 실행 전 호출
    - void afterJob(JobExecution jobExecution) : Job의 실행 후 호출
    - Job의 성공여부와 상관없이 호출된다
    - 성공/실패 여부는 JobExecution을 통해 알 수 있다.
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .start(step())
                .next(flow4())
                .listener(JobExecutionListener)
                .listener(Object) // 어노테이션 방식
                .build();
    }
    ```
2. **StepExecutionListener**
    - void beforeStep(StepExecution stepExecution) : Step의 실행 전 호출
    - ExitStatus afterStep(StepExecution stepExecution) : Step의 실행 후 호출, ExitStatus를 변경하면 최종 종료코드로 반영된다.
    - Step의 성공여부와 상관없이 호출된다
    - 성공/실패 여부는 StepExecution을 통해 알 수 있다.
    ```java
    public Job job() {
        return stepBuilderFactory.get("step")
                .tasklet(tasklet())
                .listener(StepExecutionListener)
                .build();
    }
    ```

- `git checkout Part9.2`
```java
@RequiredArgsConstructor
@Configuration
@Slf4j
public class JobAndStepListenerConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final CustomStepListener customStepListener;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .next(step2())
                .listener(new CustomJobListener())
                // .listener(new CustomAnnotationJobListener())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
//                        throw new RuntimeException("failed");
                        return RepeatStatus.FINISHED;
                    }
                })
                .listener(customStepListener) // 빈 설정O -> 데이터 공유O
                // .listener(new CustomStepListener()) // 빈 설정X -> 데이터 공유X
                .build();
    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        return RepeatStatus.FINISHED;
                    }
                })
                .listener(customStepListener)
                // .listener(new CustomStepListener()) // 빈 설정X -> 데이터 공유X
                .build();
    }
}
```
```java
public class CustomAnnotationJobListener {

    @BeforeJob
    public void AJob(JobExecution JobExecution) {
        System.out.println("JobExecution.getJobName() : " + JobExecution.getJobInstance().getJobName());
    }

    @AfterJob
    public void BJob(JobExecution JobExecution) {
        System.out.println("JobExecution.getStatus() : " + JobExecution.getStatus());
    }
}
```
```java
public class CustomJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution JobExecution) {
        System.out.println("JobExecution.getJobName() : " + JobExecution.getJobInstance().getJobName());
    }

    @Override
    public void afterJob(JobExecution JobExecution) {
        System.out.println("JobExecution.getStatus() : " + JobExecution.getStatus());
    }
}
```
```java
@Component // Bean 등록
public class CustomStepListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        System.out.println("stepExecution.getStepName() : " + stepExecution.getStepName());
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        System.out.println("stepExecution.getStatus() : " + stepExecution.getStatus());
        return ExitStatus.COMPLETED;
    }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {

    private long id;
    private String firstName;
    private String lastName;
    private Date birthdate;
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

- 결과
> Job is started<br/>
> jobName : batchJob<br/>
> Executing step: [step1]<br/>
> stepName = step1<br/>
> exitStatus = exitCode=COMPLETED; exitDescription = status=COMPLETED<br/>
> name = user1<br/>
> Step: [step1] executed in 1m52s535ms<br/>
> Executing step: [step2]<br/>
> stepName = step2<br/>
> exitStatus = exitCode=COMPLETED; exitDescription = status=COMPLETED<br/>
> name = user1

- 참고
> AbstractJob

## ChunkListener / ItemReadListener / ItemProcessListener / ItemWriteListener
1. **ChunkListener**
    - void beforeChunk(ChunkContext context) : 트랜잭션이 시작되기 전 호출, ItemReader의 read() 메소드를 호출하기 전이다.
    - void afterChunk(ChunkContext context) : Chunk가 커밋된 후 호출, ItemWriter의 write() 메소드를 호출한 후이다, 롤백 되었다면 호출하지 않는다.
    - void afterChunkError(ChunkContext context) : 오류 발생 및 롤백이 되면 호출
    - Chunk 주기 따름
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .chunk(chunkSize)
                .listener(ChunkListener)
                .build();
    }
    ```

2. **ItemReadListener**
    - void beforeRead(T item) : read() 메소드를 호출하기 전 매번 호출
    - void afterRead(T item) : read() 메소드를 호출이 성공할 때마다 호출
    - void onReadError(Exception ex) : onReadError()는 읽는 도중 오류가 발생하면 호출
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .chunk(chunkSize)
                .reader(ItemReader)
                .listener(ItemReadListener)
                .build();
    }
    ```

3. **ItemProcessListener**
    - void beforeProcess(T item) : process() 메소드를 호출하기 전 호출
    - void afterProcess(T item, @Nullable S result) : process() 메소드 호출이 성공할 때 호출
    - void onProcessError(T item, Exception e) : 처리 도중 오류가 발생하면 호출
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .chunk(chunkSize)
                .reader(ItemReader)
                .processor(ItemProcessor)
                .listener(ItemProcessListener)
                .build();
    }
    ```

4. **ItemWriteListener**
    - void beforeWrite(`List<? extends S>` items) : write() 메소드를 호출하기 전 호출
    - void afterWrite(`List<? extends S>` items) : write() 메소드 호출이 성공할 때 호출
    - void onWriteError(Exception exception, `List<? extends S>` items) : 쓰기 도중 오류가 발생하면 호출
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .chunk(chunkSize)
                .reader(ItemReader)
                .writer(ItemWriter)
                .listener(ItemWriteListener)
                .build();
    }
    ```

- `git checkout Part9.3`
```java
@RequiredArgsConstructor
@Configuration
public class ChunkListenerConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final CustomChunkListener customChunkListener;

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
                .<Integer, String>chunk(10)
                .listener(new CustomChunkListener())
                .listener(new CustomItemReadListener())
                .listener(new CustomItemProcessListener())
                .listener(new CustomItemWriteListener())
                .reader(listItemReader())
                .processor((ItemProcessor) item -> {
                    return "item" + item;
                    // throw new RuntimeException("failed");
                })
                .writer((ItemWriter<String>) items -> {
                    throw new RuntimeException("failed");
                })
                .build();
    }

    @Bean
    public ItemReader<Integer> listItemReader() {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        return new ListItemReader<>(list);
    }
}
```
```java
@Component
public class CustomChunkListener {

	private int i; // 공유 데이터

	@BeforeChunk // 어노테이션
	public void beforeChunk(ChunkContext context) {
		System.out.println(">> Before the chunk : "+ context.getStepContext().getStepExecution().getStepName());
		Object count = context.getStepContext().getStepExecution().getExecutionContext().get("count");
		if(count == null){
			context.getStepContext().getStepExecution().getExecutionContext().putInt("count", ++i);
		}
	}

	@AfterChunk
	public void afterChunk(ChunkContext context) {
		System.out.println(">> After the chunk : "+ context.getStepContext().getStepExecution().getStepName());
		int count = (int)context.getStepContext().getStepExecution().getExecutionContext().get("count");
		System.out.println(">> count : "+ count);
	}

	@AfterChunkError
	public void afterChunkError(ChunkContext context) { // Chunk error
		System.out.println(">> After the chunk : "+ context.getStepContext().getStepExecution().getStepName());
		int count = (int)context.getStepContext().getStepExecution().getExecutionContext().get("count");
		System.out.println(">> count : "+ count);
	}
}
```
```java
@Component
public class CustomItemReadListener implements ItemReadListener {

	@Override
	public void beforeRead() {
		System.out.println(">> beforeRead"); // item 이 없을 때까지 반복하므로 최종 한번 더 호출된다
	}

	@Override
	public void afterRead(Object item) {
		System.out.println(">> afterRead : "+ item);
	}

	@Override
	public void onReadError(Exception ex) {
		System.out.println(">> onReadError : " + ex.getMessage());
	}
}
```
```java
@Component
public class CustomItemProcessListener implements ItemProcessListener {

	@Override
	public void beforeProcess(Object item) {
		System.out.println(">> beforeProcess");
	}

	@Override
	public void afterProcess(Object item, Object result) {
		System.out.println(">> afterProcess : "+ item);
		System.out.println(">> afterProcess : "+ result);
	}

	@Override
	public void onProcessError(Object item, Exception e) { // Processor error
		System.out.println(">> onProcessError : " + e.getMessage());
		System.out.println(">> onProcessError : " + item);
	}
}
```
```java
@Component
public class CustomItemWriteListener implements ItemWriteListener { // implements ItemWriteListener<String>

	@Override
	public void beforeWrite(List items) { // public void beforeWrite(List<?extends String> items)
		System.out.println(">> beforeWrite");
	}

	@Override
	public void afterWrite(List items) {
		System.out.println(">> afterWrite : "+ items);
	}

	@Override
	public void onWriteError(Exception exception, List items) {
		System.out.println(">> onWriteError : " + exception.getMessage());
		System.out.println(">> onWriteError : " + items);
	}
}
```

- 결과1
> before Chunk
> before Read
> after Read
... (Chunk Size 만큼 반복)
> before Read
> after Read
> before Process
> after Process
... (Chunk Size 만큼 반복)
> before Process
> after Process
> before Write
> items = [items1, ... , 10]
> after Write
> after Chunk
> before Chunk (처음으로 돌아감, 데이터가 없으므로 종료)
> before Read
> after Chunk

- 결과2 (throw new RuntimeException("failed"))
> after Chunk Error

- 참고
> SimpleChunkProvider, TaskletStep, AbstractJob

## SkipListener & RetryListener
1. SkipListener
    - void onSkipInRead(Throwable t) : read 수행 중 Skip이 발생할 경우 호출
    - void onSkipInWrite(S item, Throwable t) : write 수행 중 Skip이 발생할 경우 호출
    - void onSkipInProcess(T item, Throwable t) : process 수행 중 Skip이 발생할 경우 호출
    - ItemReader -> **onSkipInRead()** -> SkipListener
    - -> 3번 read 중 예외 발생 -> 1,2,4,5,6,7,8,9,10
    - -> ItemProcessor -> **onSkipInProcess()** -> SkipListener
    - -> 6번 process 중 예외 발생 -> 1,2,4,5,7,8,9,10
    - -> ItemWriter -> **onSkipInWrite()** -> SkipListener
    - -> 9번 write 중 예외 발생 -> 1,2,4,5,7,8,10
    - -> DB

- `git checkout Part9.4`
```java
@RequiredArgsConstructor
@Configuration
public class SkipListenerConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final CustomSkipListener customSkipListener;

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
                .<Integer, String>chunk(10)
                .reader(listItemReader())
                .processor(new ItemProcessor<Integer, String>() {
                    @Override
                    public String process(Integer item) throws Exception {
                        if (item == 4) {
                            throw new CustomSkipException("process skipped");
                        }
                        System.out.println("process : " + item);
                        return "item" + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        for (String item : items) {
                            if (item.equals("item5")) {
                                throw new CustomSkipException("write skipped");
                            }
                            System.out.println("write : " + item);
                        }
                    }
                })
                .faultTolerant()
                .skip(CustomSkipException.class)
                .skipLimit(3)
                .listener(customSkipListener)
                .build();
    }

    @Bean
    public ItemReader<Integer> listItemReader() {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        return new LinkedListItemReader<>(list);
    }
}
```
```java
@Component
public class CustomSkipListener implements SkipListener {

	@Override
	public void onSkipInRead(Throwable t) {
		System.out.println(">> onSkipInRead : "+ t.getMessage());
	}

	@Override
	public void onSkipInWrite(Object item, Throwable t) {
		System.out.println(">> onSkipInWrite : "+ item);
//		System.out.println(">> onSkipInWrite : "+ t.getMessage());
	}

	@Override
	public void onSkipInProcess(Object item, Throwable t) {
		System.out.println(">> onSkipInProcess : "+ item);
		System.out.println(">> onSkipInProcess : "+ t.getMessage());
	}
}
```
```java
public class CustomSkipException extends Exception {

    public CustomSkipException() {
        super();
    }

    public CustomSkipException(String message) {
        super(message);
    }
}
```
```java
public class LinkedListItemReader<T> implements ItemReader<T> {

    private List<T> list;

    public LinkedListItemReader(List<T> list) {
        if (AopUtils.isAopProxy(list)) {
            this.list = list;
        }
        else {
            this.list = new LinkedList<>(list);
        }
    }

    @Nullable
    @Override
    public T read() throws CustomSkipException {

        if (!list.isEmpty()) {
            T remove = (T)list.remove(0);
            if ((Integer)remove == 3) {
                throw new CustomSkipException("read skipped : " + remove);
            }
            System.out.println("read : " + remove);
            return remove;
        }
        return null;
    }
}
```

- 결과
> read : 1<br/>
> read : 2<br/>
> ...<br/>
> read : 10<br/>
> write : item1<br/>
> write : item2<br/>
> write : item1<br/>
> onSkipProcess : 4<br/>
> onSkipProcess : process skipped<br/>
> onSkipRead : read skipped : 3<br/>
> write : item2<br/>
> onSkipRead : read skipped : 3<br/>
> write : item6<br/>
> onSkipWrite : item5<br/>
> onSkipWrite : write skipped<br/>
> onSkipRead : read skipped : 3<br/>
> write : item7<br/>
> onSkipRead : read skipped : 3<br/>
> write : item8<br/>
> onSkipRead : read skipped : 3<br/>
> write : item9<br/>
> onSkipRead : read skipped : 3<br/>
> write : item10<br/>
> onSkipRead : read skipped : 3

2. RetryListener
    - boolean open(RetryContext context, `RetryCallback<T,E>` callback) : 재시도 전 매번 호출, false를 반환할 경우 retry를 시도하지 않음
    - void close(RetryContext context, `RetryCallback<T,E>` callback, Throwable throwable) : 재시도 후 매번 호출
    - void onError(RetryContext context, `RetryCallback<T,E>` callback, Throwable throwable) : 재시도 실패 시마다 호출
    - RetryTemplate -> execute() -> RetryListener -> open() -> RetryCallback
    - -> Error? NO : Chunk Process / YES : Retry Process -> RetryListener -> onError()
    - -> Retry Limit? YES : RecoveryCallback / NO : RetryListener -> close()

- `git checkout Part9.5`
```java
@RequiredArgsConstructor
@Configuration
public class RetryListenerConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final CustomRetryListener customRetryListener;

    @Bean
    public Job job(){
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1(){
        return stepBuilderFactory.get("step1")
                .<Integer, String>chunk(10)
                .reader(listItemReader())
                .processor(new CustomItemProcessor())
                .writer(new CustomItemWriter())
                .faultTolerant()
                .retry(CustomRetryException.class)
                .retryLimit(2)
                .listener(customRetryListener)

                .build();
    }

    @Bean
    public ItemReader<Integer> listItemReader() {
        List<Integer> list = Arrays.asList(1,2,3,4);
//        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        return new LinkedListItemReader<>(list);
    }
}
```
```java
@Component
public class CustomRetryListener implements RetryListener{

	@Override
	public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
		System.out.println("open");
		return true;
	}

	@Override
	public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
		System.out.println("close");
	}

	@Override
	public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
		System.out.println("Retry Count: " + context.getRetryCount());
	}
}
```
```java
public class CustomRetryException extends Exception {

    public CustomRetryException() {
        super();
    }

    public CustomRetryException(String message) {
        super(message);
    }
}
```
```java
public class LinkedListItemReader<T> implements ItemReader<T> {

    private List<T> list;

    public LinkedListItemReader(List<T> list) {
        if (AopUtils.isAopProxy(list)) {
            this.list = list;
        }
        else {
            this.list = new LinkedList<>(list);
        }
    }

    @Override
    public T read() throws CustomRetryException {

        if (!list.isEmpty()) {
            T remove = (T)list.remove(0);
            return remove;
        }
        return null;
    }
}
```
```java
public class CustomItemProcessor implements ItemProcessor<Integer,String> {

    int count = 0;

    @Override
    public String process(Integer item) throws Exception {

        if(count < 2) {
            if (count % 2 == 0) {
                count = count + 1;

            } else if (count % 2 == 1) {
                count = count + 1;
                throw new CustomRetryException();
            }
        }
        return String.valueOf(item);
    }
}
```
```java
public class CustomItemWriter implements ItemWriter<String> {
    int count = 0;
    @Override
    public void write(List<? extends String> items) throws CustomRetryException {

        for (String item : items) {
            if(count < 2) {
                if (count % 2 == 0) {
                    count = count + 1;

                } else if (count % 2 == 1) {
                    count = count + 1;
                    throw new CustomRetryException();
                }
            }
            System.out.println("write : " + item);
        }
    }
}
```

- 결과
> open<br/>
> close<br/>
> open<br/>
> Retry count : 1<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> write : 1<br/>
> Retry count : 1<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> write : 1<br/>
> write : 2<br/>
> write : 3<br/>
> write : 4<br/>
> close

- 결과
> Retry count : 1<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> write : 1<br/>
> Retry count : 1<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> close<br/>
> open<br/>
> write : 1<br/>
> write : 2<br/>
> write : 3<br/>
> write : 4<br/>
> close

- 참고
> RetryTemplate