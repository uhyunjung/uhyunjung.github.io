---
title: '[Spring] Spring Batch 5. 반복 및 오류 제어'
date: 2024-11-10 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Repeat
1. 기본 개념
    - Spring Batch는 얼마나 작업을 반복해야 하는지 알려줄 수 있는 기능을 제공한다.
    - 특정 조건이 충족될 때까지(또는 특정 조건이 아직 충족되지 않을 때까지) Job 또는 Step을 반복하도록 배치 어플리케이션을 구성할 수 있다.
    - 스프링 배치에서는 Step의 반복과 Chunk 반복을 RepeatOperation을 사용해서 처리하고 있다.
    - 기본 구현체로 RepeatTemplate를 제공한다.

2. 구조
    - Job -> Step -> (RepeatTemplate -> Tasklet -> (RepeatTemplate -> Chunk))
    - RepeatOperations(RepeatStatus iterate(RepeatCallback callback) : RepeatCallback을 반복 호출) -> RepeatTemplate(반복을 종료할 때까지 콜백을 반복해서 호출) -> RepeatCallback(RepeatStatus doInIteration(RepeatContext context) : 비즈니스 로직 구현) -> RepeatStatus(FINISHED, CONTINUABLE : 반복 상태), RepeatContext(상태정보 저장)
        - 일시적으로 사용할 필요가 있는 데이터 저장
        - 반복 종료와 함께 데이터 삭제
    - Step -> RepeatTemplate -> iterate() -> (RepeatCallback -> doInIteration() -> tasklet) * 반복 -> Exception? (YES : ExceptionHandler(예외 정책에 따라 반복여부를 결정할 수 있다.) -> RepeatTemplate / NO : CompletionPolicy) -> Complete? (YES(종료 정책에 따라 반복 여부를 결정할 수 있다.) -> RepeatTemplate / NO : RepeatStatus) -> FINISHED? (YES(반복 상태에 따라 반복 여부를 결정할 수 있다.) -> RepeatTemplate(반복문 종료) / NO : 반복문 유지) -> tasklet

3. **Repeat**
    1. **RepeatStatus**
        - 스프링 배치의 처리가 끝났는지 판별하기 위한 열거형(enum)
            - CONTINUABLE : 작업이 남아 있음
            - FINISHED : 더 이상의 반복 없음
    
    2. **CompletionPolicy**
        - RepeatTemplate의 iterate 메소드 안에서 반복을 중단할지 결정
        - 실행 횟수 또는 완료시기, 오류 발생 시 수행할 작업에 대한 반복 여부 결정
        - 정상 종료를 알리는데 사용된다.<br/>

        - boolean isComplete(RepeatContext context, RepeatStatus result) : 콜백의 최종 상태 결과를 참조하여 배치가 완료되었는지 확인
        - boolean isComplete(RepeatContext context) : 콜백이 완료될 때까지 기다릴 필요없이 완료되었는지 확인<br/>
        
        - CompletionPolicySupport
        - **TimeoutTerminationPolicy** : 반복시점부터 현재시점까지 소요된 시간이 설정된 시간보다 크면 반복 종료
        - DefaultResultCompletionPolicy
        - **SimpleCompletionPolicy** : 현재 반복 횟수가 Chunk 개수보다 크면 반복종료
        - CountingCompletionPolicy : 일정한 카운트를 계산 및 집계해서 카운트 제한 조건이 만족하면 반복 종료
    
    3. **ExceptionHandler**
        - RepeatCallback 안에서 예외가 발생하면 RepeatTemplate가 ExceptionHandler를 참조해서 예외를 다시 던질지 여부 결정
        - 예외를 받아서 다시 던지게 되면 반복 종료
        - 비정상 종료를 알리는데 사용된다.<br/>

        - void handleException(RepeatContext context, Throwable throwable) throws Throwable : 예외를 다시 던지기 위한 전략을 허용하는 핸들러<br/>

        - LogOrRethrowExceptionHandler : 예외를 로그로 기록할지 아니면 다시 던질지 결정
        - **RethrowOnThresholdExceptionHandler** : 지정된 유형의 예외가 임계 값에 도달하면 다시 발생
        - DefaultExceptionHandler
        - **SimpleLimitExceptionHandler** : 예외 타입 중 하나가 발견되면 카운터가 증가하고 한계가 초과되었는지 여부를 확인하고 Throwable을 다시 던짐
        - CompositeExceptionHandler

4. 구현
    - 스프링 배치에서는 Step의 반복 구간과 Chunk의 반복 구간 때 RepeatTemplate을 사용하고 있다.
    - 스프링 배치에서 제공하는 구현체를 사용하거나 인터페이스를 직접 구현해서 사용할 수 있다.
    ```java
    RepeatTemplate repeatTemplate = new RepeatTemplate();

    @Override
    public String process(String item) throws Exception {
    
        repeatTemplate.setCompletionPolicy(new SimpleCompletionPolicy(2)); // chunk size
    //    repeatTemplate.setCompletionPolicy(new TimeoutTerminationPolicy(3000));
        repeatTemplate.setExceptionHandler(new SimpleLimitExceptionHandler());

        repeatTemplate.iterate(new RepeatCallback() {
            @Override
            public RepeatStatus doInIteration(RepeatContext context) {
                return RepeatStatus.CONTINUABLE; // 배치 작업에 대한 비즈니스 로직 수행, 얼마만큼 어떤 기준에 의해 반복을 수행하고 반복을 종료할 것인지 정책을 세움
            }
            return item;
        })
    }
    ```

- `git checkout Part7.1`
```java
@RequiredArgsConstructor
@Configuration
public class RepeatConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

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
                .<String, String>chunk(5)
                .reader(new ItemReader<String>() {
                    int i = 0;
                    @Override
                    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                        i++;
                        return i > 3 ? null : "item" + i;
                    }
                })
                .processor(new ItemProcessor<String, String>() {

                    RepeatTemplate template = new RepeatTemplate();

                    @Override
                    public String process(String item) throws Exception {

                        // 반복할 때마다 count 변수의 값을 1씩 증가
                        // count 값이 chunkSize 값보다 크거나 같을 때 반복문 종료
//                        template.setCompletionPolicy(new SimpleCompletionPolicy(2));
                        // 소요된 시간이 설정된 시간보다 클 경우 반복문 종료
//                        template.setCompletionPolicy(new TimeoutTerminationPolicy(3000));

                        // 여러 유형의 CompletionPolicy 를 복합적으로 처리함
                        // 여러 개 중에 먼저 조건이 부합하는 CompletionPolicy 에 따라 반복문이 종료됨
//                        CompositeCompletionPolicy completionPolicy = new CompositeCompletionPolicy();
//                        CompletionPolicy[] completionPolicies = new CompletionPolicy[]{new TimeoutTerminationPolicy(3000),new SimpleCompletionPolicy(2)};
//                        completionPolicy.setPolicies(completionPolicies);
//                        template.setCompletionPolicy(completionPolicy);

                        // 예외 제한 횟수만큼 반복문 실행
                        template.setExceptionHandler(simpleLimitExceptionHandler());

                        template.iterate(new RepeatCallback() {

                            public RepeatStatus doInIteration(RepeatContext context) {
                               System.out.println("repeatTest");
                               throw new RuntimeException("Exception is occurred");
//                                return RepeatStatus.CONTINUABLE;
                            }

                        });

                        return item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        System.out.println(items);
                    }
                })
                .build();
    }

    @Bean
    public SimpleLimitExceptionHandler simpleLimitExceptionHandler(){
        return new SimpleLimitExceptionHandler(2);
    }
}
```

- 참고
> RepeatTemplate, RepeatStatus
> SimpleCompletionPolicy, TimeoutTerminationPolicy, CompositeCompletionPolicy(정책 여러 개 설정 가능, 하나라도 true이면 빠져 나옴), CompletionPolicy, SimpleLimitExceptionHandler, RethrowOnThresholdExceptionHandler

- 결과
> repeatTemplate is testing 9번 실행, item1, 2, 3
> repeatTemplate is testing 3초 실행
> Exception is occurred

## FaultTolerant
1. 기본 개념
    - 스프링 배치는 Job 실행 중에 오류가 발생할 경우 장애를 처리하기 위한 기능을 제공하며 이를 통해 복원력을 향상시킬 수 있다.
    - 오류가 발생해도 Step이 즉시 종료되지 않고 Retry 혹은 Skip 기능을 활성화함으로써 내결함성 서비스가 가능하도록 한다.
    - 프로그램의 내결함성을 위해 Skip과 Retry 기능을 제공한다.
    - Skip : ItemReader, ItemProcessor, ItemWriter에 적용할 수 있다.
    - Retry : ITemProcessor, ItemWriter에 적용할 수 있다.
    - FaultTolerant 구조는 청크 기반의 프로세스 기반 위에 Skip과 Retry 기능이 추가되어 재정의되어 있다.

2. 구조
    - SimpleStepBuilder, SimpleChunkProcessor, SimpleChunkProvider <- FaultTolerantStep Builder, FaultTolerantChunkProvider, FaultTolerantChunkProcessor
    - StepBuilderFactory -> StepBuilder -> FaultTolerantStepBuilder -> TaskletStep
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep")
                .<I,O>chunk(10)
                .reader(ItemReader)
                .writer(ItemWriter)
                .faultTolerant() // 내결함성 기능 활성화
                .skip(Class<? extends Throwable> type) // 예외 발생 시 Skip 할 예외 타입 설정
                .skipLimit(int skipLimit) // Skip 제한 횟수 설정
                .skipPolicy(SkipPolicy skipPolicy) // Skip을 어떤 조건과 기준으로 적용할 것인지 정책 설정
                .noSkip(Class<? extends Throwable> type) // 예외 발생 시 Skip 하지 않을 예외 타입 설정
                .retry(Class<? extends Throable> type) // 예외 발생 시 Retry 할 예외 타입 설정
                .retryLimit(int retryLimit) // Retry 제한 횟수 설정
                .retryPolicy(RetryPolicy retryPolicy) // Retry를 어떤 조건과 기준으로 적용할 것인지 정책 설정
                .backOffPolicy(BackOffPolicy backOffPolicy) // 다시 Retry 하기까지의 지연시간(단위: ms)을 설정
                .noRetry(Class<? extends Throwable> type) // 예외 발생 시 Retry 하지 않을 예외 타입 설정
                .noRollback(Class<? extends Throwable> type) // 예외 발생 시 Rollback 하지 않을 예외 타입 설정
                .build();
    }
    ```

3. 구조
    - Transaction(**읽을 Item이 없을 때까지 Chunk 단위로 반복**)
        - ChunkOrientedTasklet -> provide() -> (FaultTolerant) ChunkProvider -> (read() -> ItemReader) * Chunk Size 반복 -> **Exception** -> **Skip(Skip Count만큼 예외 건너뜀)**
        - ChunkOrientedTasklet -> process(inputs) -> chunkProcessor -> execute() -> RetryTemplate -> (process() -> ItemProcesssor) * ItemReader가 읽은 아이템 개수만큼 반복 -> **Exception** -> **RetryTemplate(Retry Count만큼 재시도)**
        - ChunkOrientedTasklet -> process(inputs) -> chunkProcessor -> execute() -> RetryTemplate -> write(items) -> ItemWriter -> **Exception**

- `git checkout Part7.2`
```java
@RequiredArgsConstructor
@Configuration
public class FaultTolerantConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

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
                .<String, String>chunk(5)
                .reader(new ItemReader<String>() {
                    int i = 0;
                    @Override
                    public String read() {
                        i++;
                        if(i == 1) {
                            throw new IllegalArgumentException("skip");
                        }
                        return i > 3 ? null : "item" + i;
                    }
                })
                .processor((ItemProcessor<String, String>) item -> {
                    throw new IllegalStateException("retry");
//                    return item;
                })
                .writer(items -> System.out.println(items))
                .faultTolerant()
                .skip(IllegalArgumentException.class)
                .skipLimit(1)
                .retry(IllegalStateException.class)
                .retryLimit(2)
                .build();
    }

    @Bean
    public SimpleLimitExceptionHandler simpleLimitExceptionHandler(){
        return new SimpleLimitExceptionHandler(3);
    }
}
```

- 참고
> SimpleStepBuilder, AbstractTaskletStepBuilder, FaultTolerantStepBuilder, FaultTolerantChunkProvider, FaultTolerantChunkProcessor

## Skip
1. 기본 개념
    - Skip은 데이터를 처리하는 동안 설정된 Exception이 발생했을 경우, 해당 데이터 처리를 건너뛰는 기능이다.
    - 데이터의 사소한 오류에 대해 Step의 실패처리 대신 Skip을 함으로써, 배치수행의 빈번한 실패를 줄일 수 있게 한다.
    - ItemReader(item1, 2, 3, 4) -> `Chunk<inputs>` -> ItemProcessor(item2 error -> **Skip**) -> `Chunk<outputs>` -> ItemWriter(item1, 3, 4)
        - List로 처리하면 단건 비교 어려움
    - 오류 발생 시 스킵 설정에 의해서 Item2는 건너뛰고 Item3부터 다시 처리한다.
    - **ItemReader는 예외가 발생하면 해당 아이템만 스킵하고 계속 진행한다.**
    - **ItemProcessor와 ItemWriter는 예외가 발생하면 Chunk의 처음으로 돌아가서 스킵된 아이템을 제외한 나머지 아이템들을 가지고 처리하게 된다.**

2. 구조
    - Skip 기능은 내부적으로 **SkipPolicy**를 통해서 구현되어 있다.
    - Skip 가능 여부를 판별하는 기준
        - **1. 스킵 대상에 포함된 예외인지 여부**
        - **2. 스킵 카운터를 초과했는지 여부**
    - Step -> RepeatTemplate-> Chunk(ItemReader, ItemProcessor, ItemWriter)
    - -> Exception -> SkipPolicy(스킵 정책을 검사한다.) -> Classifier(스킵 정책을 선택하고, 스킵 여부를 결정한다.) -> Skip? skippableException && skipCount > skipLimit -> YES : 해당 아이템을 건너뛴다. -> Chunk / NO : Step 종료 -> Step
    - Step -> Exception -> Skip? **1. 스킵횟수가 2번이내일 경우 true 반환**
    - -> **(LimitCheckingItemSkipPolicy(skipLimit : 2, `Classifier<Throable, Boolen>`) -> `SubclassClassifier<Throwable, Boolean>`(`Map<class, boolean> classified`) -> {CustomSkippableException1.class, true, CustomSkippableException2.class, true, CustomNoSkippableException2.class, false})**
    - -> **2. 키로 등록되어 있는 예외클래스의 값인 true 혹은 false 값 반환**
        - {skip : true, noSkip : false}
        - 1번과 2번 모두 true일 경우 skip 기능이 작동
    <br/>

    - 스킵 정책에 따라 아이템의 skip 여부를 판단하는 클래스
    - 스프링 배치가 기본적으로 제공하는 skipPolicy 구현체들이 있으며 필요 시 직접 생성해서 사용할 수 있다. 그리고 내부적으로 Classifier 클래스들을 활용하고 있다.
    - SkipPolicy(boolen shouldSkip(Throwable t, int skipCount)) : 반환 값이 true이면 skip -> AlwaysSkipItemSkipPolicy : 항상 skip, ExceptionClassifierSkipPolicy : 예외 대상을 분류하여 skip 여부를 결정한다, CompositeSkipPolicy : 여러 SkipPolicy를 탐색하면서 skip 여부를 결정한다, **LimitCheckingItemSkipPolicy : Skip 카운터 및 예외 등록 결과에 따라 skip 여부를 결정하고, 기본값으로 설정한다**, NeverSkipItemSkipPolicy : skip을 하지 않는다.

3. API
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep")
        .<I,O>chunk(10)
        .reader(ItemReader)
        .writer(ItemWriter)
        .faultTolerant()
        .skip(Class<? extends Throwable> type) // 예외 발생 시 Skip 할 예외 타입 설정
        .skipLimit(int skipLimit) // Skip 제한 횟수 설정, ItemReader, ItemProcessor, ItemWriter 횟수 합이다.
        .skipPolicy(SkipPolicy skipPolicy) // Skip을 어떤 조건과 기준으로 적용할 것인지 정책 설정
        .noSkip(Class<? extends Throwable> type) // 예외 발생 시 Skip 하지 않을 예외 타입 설정
        .build();
    }
    ```

- `git checkout Part7.3`
```java
@RequiredArgsConstructor
@Configuration
public class SkipConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

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
                .<String, String>chunk(5)
                .reader(new ItemReader<String>() {
                    int i = 0;
                    @Override
                    public String read() throws SkippableException { // NoSkippableException
                        i++;
                        if(i == 3) {
                            throw new SkippableException("skip"); // NoSkippableException
                        }
                        System.out.println("ItemReader : " + i);
                        return i > 20 ? null : String.valueOf(i);
                    }
                })
                .processor(processor())
                .writer(writer())
                .faultTolerant() // skip or retry 사용
//                .noSkip(SkippableException.class) // 아래 설정이 위의 설정을 덮어씀
//                .skipPolicy(limitCheckingItemSkipPolicy()) // new AlwaysSkipItemSkipPolicy(), new NeverSkipItemSkipPolicy()
//                .retry(SkippableException.class)
//                .retryLimit(2)
                .skip(SkippableException.class)
                .skipLimit(2)
//                .noRollback(SkippableException.class)
                .build();
    }

    @Bean
    public LimitCheckingItemSkipPolicy limitCheckingItemSkipPolicy(){

        Map<Class<? extends Throwable>, Boolean> skippableExceptionClasses = new HashMap<>();
        skippableExceptionClasses.put(SkippableException.class, true);

        LimitCheckingItemSkipPolicy limitCheckingItemSkipPolicy = new LimitCheckingItemSkipPolicy(3, skippableExceptionClasses); // 예외 초과 횟수, 예외 대상 클래스

        return limitCheckingItemSkipPolicy;
    }

    @Bean
    public SkipItemProcessor processor() { // public ItemProcessor<? super String> itemProcessor {
        SkipItemProcessor processor = new SkipItemProcessor();
        return processor;
    }

    @Bean
    public SkipItemWriter writer() { // public ItemWriter<? super String> itemWriter {
        SkipItemWriter writer = new SkipItemWriter();
        return writer;
    }
}
```
```java
public class SkipItemProcessor implements ItemProcessor<String, String> {

	private int cnt = 0;

	@Override
	public String process(String item) throws Exception {

		if(item.equals("6") || item.equals("7")) {
			System.out.println("ItemProcessor : " + item);
			cnt++;
			throw new SkippableException("Process failed. cnt:" + cnt);
		}
		else {
			System.out.println("ItemProcessor : " + item);
			return String.valueOf(Integer.valueOf(item) * -1);
		}
	}
}
```
```java
public class SkipItemWriter implements ItemWriter<String> {

	private int cnt = 0;

	@Override
	public void write(List<? extends String> items) throws Exception {
		for (String item : items) {
			if(item.equals("-12")) {
				System.out.println("ItemWriter : " + item);
				cnt++;
				throw new SkippableException("Write failed. cnt:" + cnt);
			}
			else {
				System.out.println("ItemWriter : " + item);
			}
		}
	}
}
```
```java
public class SkippableException extends Exception {

	public SkippableException() {
		super();
	}

	public SkippableException(String msg) {
		super(msg);
	}
}
```
```java
public class NoSkipException extends Exception {

	public NoSkipException() {
		super();
	}

	public NoSkipException(String msg) {
		super(msg);
	}
}
```

- 참고
> SimpleStepBuilder, FaultTolerantStepBuilder, AbstractTaskletStepBuilder, ChunkOrientedTasklet, LimitCheckingItemSkipPolicy

- 결과1 (.skip(SkippableException.class).skipLimit(2))
> ItemReader 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5
> ItemWriter -1, -2, -4, -5
> ItemReader 7, 8, 9, 10, 11
> ItemProcess 7
> Skip limit of '2' exceeded(3, 6 skip 후)

- 결과2 (.skip(SkippableException.class).skipLimit(3))
> ItemReader 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5
> ItemWriter -1, -2, -4, -5
> ItemReader 7, 8, 9, 10, 11
> ItemProcess 7, 8, 9, 10, 11
> ItemWriter -8, -9, -10, -11
> ItemReader 12, 13, 14, 15, 16
> ItemProcess 12, 13, 14, 15, 16
> ItemWriter -12
> ItemProcess 12
> ItemWriter -12
> Skip limit of '3' exceeded(3, 6, 7 skip 후)

- 결과3 (.skip(SkippableException.class).skipLimit(4), .skipPolicy(limitCheckingItemSkipPolicy()), .skipPolicy(new AlwaysSkipItemSkipPolicy()))
> ItemReader 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5, 6
> ItemProcess 1, 2, 4, 5
> ItemWriter -1, -2, -4, -5
> ItemReader 7, 8, 9, 10, 11
> ItemProcess 7, 8, 9, 10, 11
> ItemWriter -8, -9, -10, -11
> ItemReader 12, 13, 14, 15, 16
> ItemProcess 12, 13, 14, 15, 16
> ItemWriter -12
> ItemProcess 12
> ItemWriter -12
> ItemProcess 13
> ItemWriter -13
> ItemProcess 14
> ItemWriter -14
> ItemProcess 15
> ItemWriter -15
> ItemProcess 16
> ItemWriter -16
> ItemReader 17, 18, 19, 20, 21
> ItemProcess 17, 18, 19, 20
> ItemWriter -17, -18, -19, -20

- 결과4(.skipPolicy(new NeverSkipItemSkipPolicy()) or .noSkip(SkippableException.class))
> ItemReader 1, 2
> Non-skippable exception during read

## Retry
1. 기본 개념
    - Retry는 ItemProcess, ItemWriter에서 설정된 Exception이 발생했을 경우, 지정한 정책에 따라 데이터 처리를 재시도하는 기능이다
    - Skip과 마찬가지로 Retry를 함으로써, 배치 수행의 빈번한 실패를 줄일 수 있게 한다.
    - Step(RepeatTemplate) -> `Chunk<inputs>` : **cached** -> ItemProcessor(Item2 error -> ?**Retry**) -> `Chunk<outputs>` -> ItemWriter
    - 오류 발생 시 재시도 설정에 의해서 Chunk 단계의 처음부터 다시 시작한다.
    - 아이템은 ItemReader에서 캐시로 저장한 값을 사용한다.

2. 구조
    - RetryTemplate
    - -> RetryOperations
        - T execute(`RetryCallback<T,E>` retryCallback) : RetryCallback을 재시도 설정만큼 반복 호출
        - T execute(`RetryCallback<T,E>` retryCallback, `RecoveryCallback<T>` recoveryCallback) : RetryCallback을 재시도 호출이 모두 소진되면 RecoveryCallback 호출
    - -> RetryCallback
        - T doWithRetry(RetryContext contxt) : 비즈니스 로직 구현
    - -> RecoveryCallback
        - T recover(RetryContext context) : 비즈니스 로직 구현
    <br/>

    - Retry 기능은 내부적으로 **RetryPolicy**를 통해서 구현되어 있다.
    - Retry 가능 여부를 판별하는 기준은 다음과 같다.
        - **1. 재시도 대상에 포함된 예외인지 여부**
        - **2. 재시도 카운터를 초과했는지 여부**
    - Step -> RepeatTemplate -> RetryTemplate(재시도 정책을 검사한다. 최초 1회는 무조건 실행.)
    - -> Retry? retryableException && retryCount > retryLimit -> YES : RetryCallback / NO : RecoveryCallback -> Chunk(ItemProcessor, ItemWriter)
    - -> Exception -> RetryPolicy -> Retry? retryableException && retryCount > retryLimit -> YES : BackOffPolicy -> Step 반복분 처음부터 다시 시작한다.
    - -> RepeatTemplate / NO : Step 종료 -> Step
    <br/>

    - Step -> Exception -> Retry? **1. 재시도 횟수가 2번 이내일 경우 true 반환**
    - -> SimpleRetryPolicy(retryLimit : 2, `Classifier<Throwable, Boolean>`) -> `SubclassClassifier<Throwable, Boolean>` `Map<class, boolean> classified` -> {CustomSkippableException1.class : true, CustomSkippableException2.class : true, CustomNoSkippableException2.class : false}
    - -> **2. 키로 등록되어 있는 예외 클래스의 값인 true 혹은 false 값 반환**
        - 1번과 2번 모두 true일 경우 재시도 기능이 작동한다.
    <br/>

    - RetryPolicy
        - 재시도 정책에 따라 아이템의 retry 여부를 판단하는 클래스
        - 기본적으로 제공하는 RetryPolicy 구현체들이 있으며 필요 시 직접 생성해서 사용할 수 있다.
        - boolean canRetry(RetryContext context)
        - RetryContext open(RetryContext parent)
        - void close(RetryContext context)
        - void registerThrowable(RetryContext context, Throwable throwable)
        - 반환 값이 true 이면 skip 한다.
        <br/>

        - -> AlwaysRetryPolicy : 항상 재시도를 허용한다.
        - -> ExceptionClassifierRetryPolicy : 예외대상을 분류하여 재시도 여부를 결정한다.
        - -> CompositeRetryPolicy : 여러 RetryPolicy를 탐색하면서 재시도 여부를 결정한다.
        - -> SimpleRetryPolicy : 재시도 횟수 및 예외 등록 결과에 따라 재시도 여부를 결정한다. 기본값으로 설정된다.
        - -> MaxAttempltsRetryPolicy : 재시도 횟수에 따라 재시도 여부를 결정한다.
        - -> TimeoutRetryPolicy : 주어진 시간 동안 재시도를 허용한다.
        - -> NeverRetryPolicy : 최초 한 번만 허용하고 그 이후로는 허용하지 않는다.
    <br/>

    - BackOffPolicy
        - 다시 재시도하기까지의 지연시간(단위 : ms)을 설정한다.
        - 처리 시간이 긴 데이터가 있을 경우 backOffPolicy로 재시도의 시간 간격을 조절할 수 있음
    <br/>
    
    - RetryPolicy와 BackOffPolicy를 사용해서 재시도 정책을 설정한다.
    - RetryCallback과 RecoveryCallback를 호출해서 재시도 로직을 수행한다.

3. API
    ```java
    public Step batchStep() {
        return stepBuilderFactory.get("batchStep")
                .<I,O>chunk(10)
                .reader(ItemReader)
                .writer(ItemWriter)
                .faultTolerant()
                .retry(Class<? extends Throwable> type) // 예외 발생 시 Retry 할 예외 타입 설정
                .retryLimit(int skipLimit) // Retry 제한 횟수 설정
                .retryPolicy(SkipPolicy skipPolicy) // Retry를 어떤 조건과 기준으로 적용할 것인지 정책 설정
                .noRetry(Class<? extends Throwable> type) // 다시 Retry 하기까지의 지연시간(단위 : ms)을 설정
                .backOffPolicy(BackOffPolicy backOffPolicy) // 예외 발생 시 Retry 하지 않을 예외 타입 설정
                .noRollback(Class<? extends Throwable> type) // 예외 발생 시 Rollback 하지 않을 예외 타입 설정
                .build();
    }
    ```

- `git checkout Part7.4`
- api/
```java
@RequiredArgsConstructor
@Configuration
public class RetryConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob1")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .<String, Customer>chunk(5)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .faultTolerant()
//                .skip(RetryableException.class)
//                .skipLimit(2)
                .retry(RetryableException.class)
                .noRetry(NoRetryException.class)
                .retryLimit(2)
//                .retryPolicy(retryPolicy())
                .build();
    }

    @Bean
    public ListItemReader<String> reader() {

        List<String> items = new ArrayList<>();

        for(int i = 0; i < 30; i++) {
            items.add(String.valueOf(i));
        }

        return new ListItemReader<>(items);
    }

    @Bean
    public ItemProcessor processor() {
        RetryItemProcessor processor = new RetryItemProcessor();
        return processor;
    }

    @Bean
    public ItemWriter writer() {
        RetryItemWriter writer = new RetryItemWriter();
        return writer;
    }

    @Bean
    public SimpleRetryPolicy limitCheckingItemSkipPolicy() {

        Map<Class<? extends Throwable>, Boolean> exceptionClass = new HashMap<>();
        exceptionClass.put(RetryableException.class, true);

        SimpleRetryPolicy simpleRetryPolicy = new SimpleRetryPolicy(2, exceptionClass);

        return simpleRetryPolicy;
    }

    @Bean
    public RetryPolicy retryPolicy() {

        Map<Class<? extends Throwable>, Boolean> exceptionClass = new HashMap<>();
        exceptionClass.put(RetryableException.class, true);

        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy(2,exceptionClass);

        return retryPolicy;
    }
}
```
```java
public class RetryItemProcessor implements ItemProcessor<String, String> {

	private int cnt = 0;

	@Override
	public String process(String item) throws Exception {
		if(item.equals("2") || item.equals("3")) {
			cnt++;
			throw new NoRetryException("Process failed. cnt:" + cnt);
		}
		return item;
	}
}
```
```java
public class RetryItemWriter implements ItemWriter<String> {

	@Override
	public void write(List<? extends String> items) throws Exception {
		items.forEach(item -> System.out.println(item));
	}
}
```

- 결과1 (.retry(RetryableException.class).retryLimit(2))
> Non-skippable exception in recoverer while processing : retry 정상동작하나 처리 안 되는 경우

- 결과2 (.skip(RetryableException.class).skipLimit(2).skipPolicy(retryPolicy()) .retry(RetryableException.class).retryLimit(2))
> 0, 1, 4, 5, 6, ... 29 (chunk = 5 : [0, 1, 4], [5, 6, 7, 8, 9], ... 29)

- 참고
> ChunkOrientedTasklet, SimpleChunkProcessor, FaultTolerantChunkProvider, FaultTolerantChunkProcessor, BatchRetryTemplate, RetryTemplate, RetryState, RetryContextSupport, SimpleRetryPolicy

- template/
```java
@RequiredArgsConstructor
@Configuration
public class RetryConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob3")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .<String, Customer>chunk(5) // Customer
                .reader(reader())
                .processor(processor())
                .writer(writer()) // items -> items.forEach(item -> System.out.println(item))
                .faultTolerant()
                // .skip(RetryableException.class)
                // .skipLimit(2)
                .retry(RetryableException.class)
                // .noRetry(NoRetryException.class)
                .retryLimit(2) // default = 1
                // .retryPolicy(retryPolicy())
                .build();
    }

    
    @Bean
    public ListItemReader<String> reader() {
        List<String> items = new ArrayList<>();

        for(int i = 0; i < 30; i++) {
            items.add(String.valueOf(i));
        }

        return new ListItemReader<>(items);
    }

    @Bean
    public ItemProcessor processor() { // public ItemProcessor<? super String, String> processor() {
        RetryItemProcessor2 processor = new RetryItemProcessor2();
        return processor;
    }

    @Bean
    public ItemWriter writer() {
        RetryItemWriter2 writer = new RetryItemWriter2();
        return writer;
    }

    @Bean
    public RetryPolicy retryPolicy() {

        Map<Class<? extends Throwable>, Boolean> exceptionClass = new HashMap<>();
        exceptionClass.put(RetryableException.class, true);

        SimpleRetryPolicy simpleRetryPolicy = new SimpleRetryPolicy(2,exceptionClass);

        return simpleRetryPolicy;
    }

    @Bean
    public RetryTemplate retryTemplate() {

        Map<Class<? extends Throwable>, Boolean> exceptionClass = new HashMap<>();
        exceptionClass.put(RetryableException.class, true);

        // FixedBackOffPolicy backOffPolicy = new FixedBackOffPolicy();
        // backOffPolicy.setBackOffPeriod(2000); // 지정한 시간만큼 대기 후 재시도 한다.

        SimpleRetryPolicy simpleRetryPolicy = new SimpleRetryPolicy(2,exceptionClass);
        RetryTemplate retryTemplate = new RetryTemplate();
        // retryTemplate.setBackOffPolicy(backOffPolicy);
        retryTemplate.setRetryPolicy(simpleRetryPolicy);

        return retryTemplate;
    }
}
```
```java
public class RetryItemProcessor2 implements ItemProcessor<String, Customer> {

	@Autowired
	private RetryTemplate retryTemplate;

	@Override
	public Customer process(String item) throws Exception {

		Classifier<Throwable, Boolean> rollbackClassifier = new BinaryExceptionClassifier(true);

		Customer result = retryTemplate.execute(new RetryCallback<Customer, RuntimeException>() { // state = null -> retry 계속 시도 -> retry count = retryLimit = 2
			@Override
			public Customer doWithRetry(RetryContext context) throws RuntimeException {
				// 설정된 조건 및 횟수만큼 재시도 수행
				if(item.equals("1") || item.equals("2")){
					throw new RetryableException("failed");
				}
				return new Customer(item);
			}
		}, new RecoveryCallback<Customer>() { // retry count = 2 -> item = 2
			@Override
			public Customer recover(RetryContext context) throws Exception {
				// 재시도가 모두 소진되었을 때 수행
				return new Customer(item);
			}
		}, new DefaultRetryState(item, rollbackClassifier)); // recovery 예외 발생 -> 처음으로 rollback -> failed
		//template - state 추가, skip 추가, backoff 추가,
		return result;
	}
}
```
```java
public class RetryItemWriter2 implements ItemWriter<Customer> {

	@Override
	public void write(List<? extends Customer> items) throws Exception {
		items.forEach(item -> System.out.println(item));
	}
}
```
```java
// @author Michael Minella
@Data
@AllArgsConstructor
public class Customer {

	private String item;
}
```

- /
```java
public class RetryableException extends RuntimeException {

	public RetryableException() {
		super();
	}

	public RetryableException(String msg) {
		super(msg);
	}
}
```
```java
public class NoRetryException extends RuntimeException {

	public NoRetryException() {
		super();
	}

	public NoRetryException(String msg) {
		super(msg);
	}
}
```

- 참고
> FaultTolerantChunkProcessor, RetryTemplate

- 결과1 (.retry(RetryableException.class).retryLimit(2))
> Customer(item=0, 1, 2, ... 29) : 실패나 에러 발생X

- 결과2 (.retry(RetryableException.class).retryLimit(1)) + new DefaultRetryState(item, rollbackClassifier) 추가
> failed

- 결과3 (.skip(RetryableException.class).skipLimit(2).skipPolicy(retryPolicy()) .retry(RetryableException.class).retryLimit(2))
> Customer(item=0, 3, 4, ... 29)

## Skip & Retry 아키텍처
- **ItemReader** : Chunk 기반으로 Item 읽기
    - Step -> RepeatTemplate -> (ChunkOrientedTasklet -> (FaultTolerantChunkProvider -> SimpleChunkProvider)
    - -> (RepeatTemplate -> **ItemReader(read())** -> Exception? NO : RepeatTemplate / YES : **Skip**? NO : **throw NonSkippableReadException**
    - -> Step 실패 종료 -> RepeatTemplate / YES : **ItemReader(read())** -> LimitCheckingItemSkipPolicy))

- **ItemProcessor**
    - TaskletStep -> (RepeatTemplate -> ChunkOrientedTasklet -> FaultTolerantChunkProcessor -> ChunkIterator
    - -> (RetryTemplate -> (RetryCallback -> **Retry**? (NO : SimpleRetryPolicy -> **RecoveryCallback** -> **Skip**? (NO : LimitCheckingItemSkipPolicy -> **throw SkipLimitExceededException**
    - -> **Step 실패 종료** -> RepeatTemplate / YES : **Skip 아이템 제거**) -> ChunkIterator / YES : **ItemProcessor** -> Exception? (NO : ChunkIterator / YES : **Step 재시도**(예외가 발생하고 Retry Limit이 초과한 경우) -> RepeatTemplate)))))

- **ItemWriter**
    - TaskletStep -> (RepeatTemplate -> ChunkOrientedTasklet -> FaultTolerantChunkProcessor -> ChunkIterator
    - -> (RetryTemplate -> (RetryCallback -> **Retry**? (NO : SimpleRetryPolicy(예외가 발생하고 Retry Limit이 초과한 경우) -> **RecoveryCallback** -> **Skip**? (NO : LimitCheckingItemSkipPolicy
    - -> **throw ExhaustedRetryException** -> **Step 실패 종료** -> RepeatTemplate / YES : **doScan으로 복구작업**) -> RetryTemplate / YES : **ItemWriter** -> Exception? (NO : **Step 성공 종료** -> RepeatTemplate / YES : **Step 재시도** -> RepeatTemplate)))))