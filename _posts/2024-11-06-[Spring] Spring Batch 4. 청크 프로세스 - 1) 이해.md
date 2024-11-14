---
title: '[Spring] Spring Batch 4. 청크 프로세스 - 1) 이해'
date: 2024-11-06 19:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Chunk
1. 기본 개념
    - Chunk란 여러 개의 아이템을 묶은 하나의 덩어리, 블록을 의미
    - 한 번에 하나씩 아이템을 입력 받아 Chunk 단위의 덩어리로 만든 후 Chunk 단위로 트랜잭션을 처리함, 즉 Chunk 단위의 Commit과 Rollback이 이루어짐
    - 일반적으로 대용량 데이터를 한 번에 처리하는 것이 아닌 청크 단위로 쪼개어서 더 이상 처리할 데이터가 없을 때까지 반복해서 입출력하는데 사용됨
    - `Chunk<I>` vs `Chunk<O>`
        - `Chunk<I>` : ItemReader로 읽은 하나의 아이템을 Chunk에서 정한 개수만큼 반복해서 저장하는 타입
        - `Chunk<O>` : ItemReader로부터 전달받은 `Chunk<I>`를 참조해서 ItemProcessor에서 적절하게 가공, 필터링한 다음 ItemWriter에 전달하는 타입
        - Source -> read item -> ItemReader -> `Chunk<I>` -> ItemProcessor -> transform -> `Chunk<O>` -> items -> ItemWriter

2. 아키텍처
    - Source -> ItemReader -> `Chunk<I>` -> `List<Item>`, Chunk size? -> NO : ItemReader or **YES : inputs -> ItemProcessor(Iterator.nxt) <-> inputs(item)** -> transform -> `Chunk<O>` -> List<Item>, outputs -> ItemWriter -> DB
    - ItemReader와 ItemProcessor는 Chunk 내 개별 아이템을 처리한다.
    - ItemWriter는 Chunk 크기만큼 아이템을 일괄 처리한다.
    - Iterable <- Chunk, List items, `List<SkipWrapper>` skips, `List<Exception>` errors, ChunkIterator iterator(return new ChunkIterator(items) : Chunk에 저장된 Item을 추출하기 위함) -> ChunkIterator(`Iterator<W>` iterator = items.iterator()) -> Iterator

- `git checkout Part5.1`
```java
@RequiredArgsConstructor
@Configuration
public class ChunkConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(2)
                .reader(new ListItemReader<>(Arrays.asList("item1", "item2", "item3","item4", "item5", "item6")))
                .processor(new ItemProcessor<String, String>() {
                    @Override
                    public String process(String item) throws Exception {
                        Thread.sleep(300);
                        System.out.println(item);
                        return "my_" + item;
                    }
                })
                .writer(new ItemWriter<String>() { // outputs : items(my_item1,2,3,4,5,6)
                    @Override
                    public void write(List<? extends String> items) throws Exception { // 일괄 처리
                        Thread.sleep(1000);
                        System.out.println(items);
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

- 참고
> ListItemReader, SimpleChunkProvider, SimpleChunkProcessor

## ChunkOrientedTasklet
1. 기본 개념
    - ChunkOrientedTasklet은 스프링 배치에서 제공하는 Tasklet의 구현체로서 Chunk 지향 프로세싱을 담당하는 도메인 객체
    - ItemReader, ItemWriter, ItemProcessor를 사용해 Chunk 기반의 데이터 입출력 처리를 담당한다.
    - TaskletStep에 의해서 반복적으로 실행되며 ChunkOrientedTasklet이 실행될 때마다 매번 새로운 트랜잭션이 생성되어 처리가 이루어진다.
    - exception이 발생할 경우, 해당 Chunk는 롤백되며 이전에 커밋한 Chunk는 완료된 상태가 유지된다.
    - 내부적으로 ItemReader를 핸들링하는 ChunkProvider와 ItemProcessor, ItemWriter를 핸들링하는 ChunkProcessor 타입의 구현체를 가진다.
2. 구조
    - Tasklet(RepeatStatus execute(StepContribution, ChunkContext)) <- ChunkOrientedTasklet(ChunkProvider(Chunk 단위 아이템 제공자), ChunkProcessor(Chunk 단위 아이템 처리자))
    - TaskletStep -> execute() -> ChunkOrientedTasklet -> provide() -> (ChunkProvider -> read() -> ItemReader) * Chunk Size 반복
    - ChunkOrientedTasklet -> process(inputs) -> (ChunkProcessor -> process() -> ItemProcessor) * ItemReader가 전달한 아이템 개수만큼 반복
    - ChunkProcessor -> write(items) -> ItemWriter
    - ChunkOrientedTasklet부터 ItemWriter까지 읽을 Item이 없을 때까지 Chunk 단위 반복

    ```java
    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) {
        Chunk<I> inputs = (Chunk<I>) chunkContext.getAttribute(INPUTS_KEY); // Chunk 처리 중 예외가 발생하여 재시도할 경우 다시 데이터를 읽지 않고 버퍼에 담아 놓았던 데이터를 가지고 옴
        if (input == null) {
            inputs = chunkProvider.provide(contribution); // Item을 Chunk size만큼 반복해서 읽은 다음 Chunk<I>에 저장하고 반환
            if (buffering) {
                chunkContext.setAttribute(INPUTS_KEY, inputs); // Chunk를 캐싱하기 위해 ChunkContext 버퍼에 담음
            }
        }

        chunkProcessor.process(contribution, inputs); // ChunkProvider로부터 받은 Chunk<I>의 아이템 개수만큼 데이터를 가공하고 저장
        chunkProvider.postProcess(contribution, iinputs);

        if (inputs.isBusy()) {
            logger.debug("Inputs still busy");
            return RepeatStatus.CONTINUABLE;
        }

        chunkContext.removeAttribute(INPUTS_KEY); // Chunk 단위 입출력이 완료되면 버퍼에 저장한 Chunk 데이터 삭제
        chunkContext.setComplete();

        if (logger.isDebugEnabled()) {
            logger.debug("Inputs not busy, ended: " + inputs.isEnd());
        }

        return RepeatStatus.continueIf(!inputs.isEnd()); // 일글 Item이 더 존재하는지 체크해서 존재하면 Chunk 프로세스 반복하고 null일 경우 RepeatStatus.FINISHED 반환하고 Chunk 프로세스 종료
    }
    ```

    - StepBuilderFactory -> StepBuilder -> SimpleStepBuilder -> TaskletStep
    - StepBuilderFactory -> get(stepName) -> StepBuilder
        - -> tasklet(tasklet()) -> TaskletStepBuilder
        - -> <I,O> chunk(chunkSize), <I, O> chunk(completionPolicy) -> TaskletStepBuilder
        - partitioner(stepName, partitioner), partitioner(step) -> PartitionStepBuilder
        - job(job) -> JobStepBuilder
        - flow(flow) -> FlowStepBuilder

    ```java
    public Step chunkStep() {
        return stepBuilderFactory.get("chunkStep")
                .<I, O> chunk(10) // chunk size 설정, chunk size는 commit interval을 의미함, input, output 제너릭 타입 설정
                .<I, O> chunk(CompletionPolicy) // chunk 프로세스를 완료하기 위한 정책 설정 클래스 지정
                .reader(itemReader()) // 소스로부터 Item을 읽거나 가져오는 ItemReader 구현체 설정
                .writer(itemWriter()) // Item을 목적지에 쓰거나 보내기 위한 ItemWriter 구현체 설정
                .processor(itemProcessor()) // Item을 변형, 가공, 필터링 하기 위한 ItemProcessor 구현체 설정
                .stream(itemStream()) // 재시작 데이터를 관리하는 콜백에 대한 스트림 등록
                .readerIsTransactionalQueue() // Item이 JMS, Message Queue Server와 같은 트랜잭션 외부에서 읽혀지고 캐시할 것인지 여부, 기본값은 false
                .listener(ChunkListener) // Chunk 프로세스가 진행되는 특정 시점에 콜백 제공받도록 ChunkListener 설정
                .build();
    }
    ```

- `git checkout Part5.2`
```java
@RequiredArgsConstructor
@Configuration
public class ChunkOrientedTaskletConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    @JobScope
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(3)
                .reader(new ListItemReader<>(Arrays.asList("item1", "item2", "item3","item4", "item5", "item6")))
                .processor(new ItemProcessor<String, String>() {
                    @Override
                    public String process(String item) throws Exception {
                        return "my_" + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        items.forEach(item -> System.out.println(item));
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

- 참고
> AbstractTaskletStepBuilder, SimpleStepBuilder, SimpleChunkProvider, SimpleChunkProcessor, Chunk

## ChunkProvider/ChunkProcessor
1. ChunkProvider
    - 기본 개념
        - ItemReader를 사용해서 소스로부터 아이템을 Chunk size만큼 읽어서 Chunk 단위로 만들어 제공하는 도메인 객체
        - `Chunk<I>`를 만들고 내부적으로 반복문을 사용해서 ItemReader.read()를 계속 호출하면서 Item을 Chunk에 쌓는다.
        - 외부로부터 ChunkProvider가 호출될 때마다 항상 새로운 Chunk가 생성된다.
        - 반복문 종료 시점
            - Chunk size만큼 item을 읽으면 반복문 종료되고 ChunkProcessor로 넘어감
            - ItemReader가 읽은 item이 null일 경우 반복문 종료 및 해당 Step 반복문까지 종료
        - 기본 구현체로서 SimpleChunkProvider와 FaultTolerantChunkProvider가 있다.
    - 구조
        - `ChunkProvider<I>`(`Chunk<T> provide(StepContributioin contribution)` : Item 읽고 Chunk 단위로 저장해서 반환) <- `SimpleChunkProvider<I>`(`ItemReader<I>`, `RepeatOperations` : Item 읽기를 실행하는 반복기, `I read(StepContribution contribution, Chunk<I> chunk)` : ItemReader로부터 Item을 한 건씩 읽음)

    ```java
    public Chunk<I> provide(final StepContribution contribution) throws Exception {

        final Chunk<I> inputs = new Chunk(); // Item을 담을 Chunk 생성, provide() 호출마다 새롭게 생성
        repeatOperations.iterate(new RepeatCallback() { // Chunk size만큼 반복문 실행하면서 read() 호출

            @Override
            public RepeatStatus doInIteration(final RepeatContext context) throws Exception {
                I item = null;
                Timer.Sample sample = Timer.start(Metrics.globalRegistry);
                String status = BatchMetrics.STATUS_SUCCEED;

                try {
                    item = read(contribution, inputs); // doRead(), ItemReader가 Item 한 개씩 읽어서 리턴
                }
                catch (SkipOverflowException e) {
                    status = BatchMetrics.STATUS_FAILURE;
                    return RepeatStatus.FINISHED;
                }
                finally {
                    stopTimer(sample, contribution.getStepExecution(), status);
                }

                if (item == null) { // 더 이상 읽을 Item이 없는 null일 경우 반복문 종료 및 전체 Chunk 프로세스 종료
                    inputs.setEnd();
                    return RepeatStatus.FINISHED;
                }

                inputs.add(item); // ItemReader로부터 받은 Item을 Chunk size만큼 Chunk에 저장
                contribution.incrementReadCount();
                return RepeatStatus.CONTINUABLE;
            }
        });

        return inputs;
    }
    ```
    ```java
    protected final I doRead() throws Exception {
        try {
            listener.beforeRead();
            I item = itemReader.read();
            if (item != null) {
                listener.afterRead(item);
            }
            return item;
        }
    }
    ```

2. ChunkProcessor
    - 기본 개념
        - ItemProcessor를 사용해서 Item을 변형, 가공, 필터린라도 ItemWriter를 사용해서 Chunk 데이터를 저장, 출력한다.
        - `Chunk<O>`를 만들고 앞에서 넘어온 `Chunk<I>`의 item을 한 건씩 처리한 후 `Chunk<O>`에 저장한다.
        - 외부로부터 ChunkProcessor가 호출될 때마다 항상 새로운 Chunk가 생성된다.
        - ItemProcessor는 설정 시 선택사항으로서 만약 객체가 존재하지 않을 경우 ItemReader에서 읽은 item 그대로가 `Chunk<O>`에 저장된다.
        - ItemProcessor 처리가 완료되면 `Chunk<O>`에 있는 `List<Item>`을 ItemWriter에게 전달한다.
        - ItemWriter 처리가 완료되면 Chunk 트랜잭션이 종료하게 되고 Step 반복문에서 ChunkOrientedTasklet가 새롭게 실행된다.
        - ItemWriter는 Chunk size만큼 데이터를 Commit 처리하기 때문에 Chunk size는 곧 Commi Interval이 된다.
        - 기본 구현체로서 SimpleChunkProcessor와 FaultTolerantChunkProcessor가 있다.
    - 구조
        - `ChunkProcessor<I>`(`void process(StepContribution, Chunk<I>)` : Chunk Item을 변형하고 저장)
        - `SimpleChunkProcessor<I, O>`(`ItemProcessor<I, O>`, `ItemWriter<O>`, `Chunk<O> transform(StepContribution, Chunk<I>)` : ItemProcessor로부터 Item을 변형 후 `Chunk<O>` 반환, `void write(StepContribution, Chunk<I>, Chunk<O>)` : ItemWriter로부터 Items을 저장)
    
    ```java
    public final void process(StepContribuion contribution, Chunk<I> inputs) {
        initializeUserData(inputs);

        if (isComplete(inputs)) {
            return;
        }

        // inputs : chunkProvider.provide()에서 받은 Chunk<I>
        Chunk<O> outputs = transform(contribution, inputs); // ItemProcessor에서 가공처리 된 아이템을 담은 Chunk<O> 반환
        contribution.incrementFilterCount(getFilterCount(inputs, outputs)); // ItemProcessor 필터링 된 아이템 개수 저장
        write(contribution, inputs, getAdjustedOutpus(inputs, outpus)); // 가공 처리된 Chunk<O>의 List<Item>를 ItemWriter에게 전달
    }
    ```
    ```java
    protected Chunk<O> transform(StepContribution contribution, Chunk<I> inputs) throws Exception {
        Chunk<O> outputs = new Chunk<>(); // 아이템 가공 및 저장을 Chunk<O> 생성
        for (Chunk<I>.ChunkIterator iterator = inputs.iterator(); iterator.hasNext();) { // 읽은 아이템 개수만큼 반복하면서 doProcess() 호출
            final I item = iterator.next();
            O output;
            Timer.Sample sample = BatchMetrics.createTimerSample();
            String status = BatchMetrics.STAUTS_SUCCESS;

            try {
                output = doProcess(item);
            }
            ...
        }
    }
    ```
    ```java
    protected final O doProcess(I item) throws Exception {
        if (itemProcessor == null) { // ItemProcessor가 없다면 item을 그대로 반환
            O result = (O) item;
            return result;
        }

        try { // ItemProcessor의 process(item)로 가공 후 반환
            listener.beforeProcess(item);
            O result = itemProcessor.process(item);
            listener.afterProcess(item, result);
            return result;
        }
        ...
    }
    ```
    ```java
    protected void write(StepContribution contribution, Chunk<I> inputs, Chunk<O> outputs) {
        Timer.Sample sample = BatchMetrics.createTimerSample();
        String status = BatchMetrics.STATUS_SUCCESS;
        try {
            doWrite(outputs.getItems()); // Chunk의 Item 리스트를 전달
        }
        ...
    }
    ```
    ```java
    protected final void doWrite(List<O> items) {
        if (itemWriter == null) {
            return;
        }

        try {
            listener.beforeWrite(items);
            writeItems(items);
            doAfterWrite(items);
        }
        ...
    }
    ```
    ```java
    protected void writeItems(List<O> items) throws Exception {
        if (itemWriter != null) {
            itemWriter.write(items); // ItemWriter의 write(items)로 Item들을 일괄 처리
        }
    }
    ```

- `git checkout Part5.2`
```java
@RequiredArgsConstructor
@Configuration
public class ChunkProviderChunkProcessorConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    @JobScope
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(2) // 2개씩
                .reader(new ListItemReader<>(Arrays.asList("item1", "item2", "item3","item4", "item5", "item6")))
                .processor(new ItemProcessor<String, String>() {
                    @Override
                    public String process(String item) throws Exception {
                        return "my_" + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        items.forEach(item -> System.out.println(item));
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

- 참고
> TaskletStep, ChunkOrientedTasklet, SimpleChunkProvider, SimpleChunkProcessor, Chunk, ListItemReader, RepeatContextSupport, SimpleCompletionPolicy, DefaultResultCompletionPolicy, RepeatTemplate, StepContribution

## ItemReader/ItemWriter/ItemProccessor
1. **ItemReader**
    - 기본 개념
        - 다양한 입력으로부터 데이터를 읽어서 제공하는 인터페이스
        - ChunkOrientedTasklet 실행 시 필수적 요소로 설정해야 한다.

        - Flat 파일 : csv, txt(고정 위치로 정의된 데이터 필드나 특수문자로 구별된 데이터의 행)
        - XML, Json
        - Database
        - JMS, RabbitMQ와 같은 Message Queueing 서비스
        - Custom Reader : 구현 시 멀티 스레드 환경에서 스레드에 안전하게 구현할 필요가 있음

    - 구조
        - `ItemReader<T>`(`T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException`)
            - T read()
                - 입력 데이터를 읽고 다음 데이터로 이동한다.
                - 아이템 하나를 리턴하며 더 이상 아이템이 없는 경우 null 리턴
                - 아이템 하나는 파일의 한 줄, DB의 한 row 혹은 XML 파일에서 하나의 엘리먼트가 될 수 있다.
                - 더 이상 처리해야 할 Item이 없어도 예외가 발생하지 않고 ItemProcessor와 같은 다음 단계로 넘어간다.
            - `ItemReader<T>`, `ItemStream` <- `ItemStreamReader<T>`, `AmqpItemReader<T>`, `ListItemReader<T>`, `JmsItemReader<T>`, `ItermStreamSupport`
                - `SynchronizedItemStreamReader<T>`, `AbstractItemStreamItemReader<T>`, `MultiResourceItemReader<T>`, `KafkaItemReader<K, V>`, `AbstractItemCountingItemStreamItemReader<T>`, `JsonItemReader<T>`, `FlatFileItemReader<T>`, `JpaCursorItemReader<T>`, `AbstractPagingItemReader<T>`, `AbstractCursorItemReader<T>`, `StaxEventItemReader<T>`, `JdbcPagingItemReader<T>`, `JpaPagingItemReader<T>`
        - 다수의 구현체들이 ItemReader와 ItemStream 인터페이스를 동시에 구현하고 있음
            - 파일의 스트림을 열거나 종료, DB 커넥션을 열거나 종료, 입력 장치 초기화 등의 작엄
            - ExecutionContext에 read와 관련된 여러 가지 상태 정보를 저장해서 재시작 시 다시 참조하도록 지원
        - 일부를 제외하고 하위 클래스들은 기본적으로 스레드에 안전하지 않기 때문에 병렬 처리시 데이터 정합성을 위한 동기화 처리 필요

    - 종류
        - FILE : FlatFileItemReader, StaxEventItemReader, JsonItemReader, MultiResourceItemReader
        - DB : JdbcCursorItemReader, JpaCursorItemReader, JdbcPagingItemReader, JpaPagingItemReader, ItemReaderAdapter
        - Support/Custom : SynchronizedItemStreamReader, CustomItemReader

2. **ItemWriter**
    - 기본 개념
        - Chunk 단위로 데이터를 받아 일괄 출력 작업을 위한 인터페이스
        - 아이템 하나가 아닌 아이템 리스트를 전달받는다.
        - ChunkOrientedTasklet 실행 시 필수적 요소로 설정해야 한다.
        - Flat 파일 : csv, txt
        - XML, Json
        - Database
        - JMS, RabbitMQ와 같은 Message Queueing 서비스
    
    - 구조
        - `ItemWriter<T>`(void write(List<? extends T> items) throws Exception)
            - 출력 데이터를 아이템 리스트로 받아 처리한다.
            - 출력이 완료되고 트랜잭션이 종료되면 새로운 Chunk 단위 프로세스로 이동한다.
        - `ItemWriter<T>`, `ItemStream` <- `ItemStreamWriter<T>`, `ItemWriterAdapter<T>`, `JpaItemWriter<T>`, `ListItemWriter<T>`, `MongoItemWriter<T>`, `JdbcBatchItemWriter<T>`, `SimpleMailMessageItemWriter`, `ItermStreamSupport`
            - `AbstractItemStreamItemWriter<T>`, `SynchronizedItemStreamWriter<T>`, `StaxEventItemWriter<T>`, `MultiResourceItemWriter<T>`, `AbstractFileItemWriter<T>`, `JsonFileItemWriter<T>`, `FlatFileItemWriter<T>`, `JmsItemWriter<T>`, `AmqpItemWriter<T>`, `HibernateItemWriter<T>`, `KeyValueItemWriter<K, V>`, `KafkaItemWriter<K, T>`
        - 다수의 구현체들이 ItemWriter와 ItemStream 인터페이스를 동시에 구현하고 있음
            - 파일의 스트림을 열거나 종료, DB 커넥션을 열거나 종료, 입력 장치 초기화 등의 작엄
        - 보통 ItemREader 구현체와 1:! 대응 관계인 구현체들로 구성되어 있다.
    
    - 종류
        - FILE : FlatItemWriter, StaxEventItemWriter, JsonFileItemWriter, MultiResourceItemWriter
        - DB : JdbcBatchItemWriter, JpaItemWriter, ItemWriterAdapter
        - Support/Custom : CustomWriter

3. ItemProcessor
    - 기본 개념
        - 데이터를 출력하기 전에 데이터를 가공, 변형, 필터링하는 역할
        - ItemReader 및 ItemWriter와 분리되어 비즈니스 로직을 구현할 수 있다.
        - ItemReader로부터 받은 아이템을 특정 타입으로 변환해서 ItemWriter에 넘겨줄 수 있다.
        - ItemReader로부터 받은 아이템들 중 필터과정을 거쳐 원하는 아이템들만 ItemWriter에게 넘겨줄 수 있다.
            - ItemProcessor에서 process() 실행결과 null을 반환하면 `Chunk<O>`에 저장되지 않기 때문에 결국 ItemWriter에 전달되지 않는다.
        - ChunkOrientedTasklet 실행 시 선택적 요소이기 때문에 청크 기반 프로세싱에서 ItemProcessor 단계가 반드시 필요한 것은 아니다.
    
    - 구조
        - `ItemProcessor<I, O>`(O process(@NonNull I item) throws Exception)
        - O process
            - `<I>` 제너릭은 ItemReader에서 받을 데이터 타입 지정
            - `<O>` 제너릭은 ItemWriter에게 보낼 데이터 타입 지정
            - 아이템 하나씩 가공 처리하며 null 리턴할 경우 해당 아이템은 `Chunk<O>`에 저장되지 않음
        - `ItemProcessor<I, O>` -> `ValidatingItemProcessor<T>`, `ClassifierCompositeItemProcessor<I, O>`, `CompositeItemProcessor<I, O>`, `FunctionItemProcessor<I, O>`, `ScriptItemProcessor<I, O>`
            - `BeanValidatingItemProcessor<T>`, `ItemProcessorAdapter<I, O>`
        - ItemStream을 구현하지 않는다.
        - 거의 대부분 Customizing해서 사용하기 때문에 기본적으로 제공되는 구현체가 적다.
    - 종류
        - ClassifierCompositeItemProcessor, CompositeItemProcessor, CustomItemProcessor


```java
@RequiredArgsConstructor
@Configuration
public class ItemReader_ItemProcessor_ItemWriter_Configuration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(3)
                .reader(itemReader())
                .processor(itemProcessor())
                .writer(itemWriter())
                .build();
    }

    @Bean
    public ItemReader itemReader() {
        return new CustomItemReader(Arrays.asList(new Customer("user1"), new Customer("user2"), new Customer("user3")));
    }

    @Bean
    public ItemProcessor itemProcessor() { // public ItemProcessor<? super Customer, ? extends Customer> itemProcessor() {
        return new CustomItemProcessor(); // ItemProcessor만 custom 해서 주로 사용
    }

    @Bean
    public ItemWriter itemWriter() { // public ItemWriter<? super Customer> itemWriter() {
        return new CustomItemWriter();
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
@Data
@AllArgsConstructor
public class Customer {
    private String name;
}
```
```java
public class CustomItemReader implements ItemReader<Customer> {

    private List<Customer> list;

    public CustomItemReader(List<Customer> list) {
        this.list = new ArrayList<>(list);
    }

    @Override
    public Customer read() {
        if (!list.isEmpty()) {
            return list.remove(0);
        }
        return null;
    }
}
```
```java
public class CustomItemWriter implements ItemWriter<Customer> {

    @Override
    public void write(List<? extends Customer> items) throws Exception {
        items.forEach(item -> System.out.println(item));
    }
}
```
```java
public class CustomItemProcessor implements ItemProcessor<Customer, Customer> {

    @Override
    public Customer process(Customer customer) throws Exception {
        customer.setName(customer.getName().toUpperCase());
        return customer;
    }
}
```

## ItemStream
1. 기본 개념
    - ItemReader와 ItemWriter 처리 과정 중 상태를 저장하고 오류가 발생하면 해당 상태를 참조하여 실패한 곳에서 재시작하도록 지원
    - 리소스를 열고(open) 닫아야(close) 하며 입출력 장치 초기화 등의 작업을 해야 하는 경우
    - ExecutionContext(Map 형태)를 매개변수로 받아서 상태 정보를 업데이트(update) 한다.
    - ItemReader 및 ItemWriter는 ItemStream을 구현해야 한다.

2. 구조
    - ItemStream(void open(ExecutionContext executionContext) throws ItemSteamException : read, write 메소드 호출 전에 파일이나 커넥션이 필요한 리소스에 접근하도록 초기화 작업, void update(ExecutionContext executionContext) throws ItemStreamException : 현재까지 진행된 모든 상태를 저장, void close() throws ItemStreamException : 열려 있는 모든 리소스를 안전하게 해제하고 닫음)
    - Step -> ItemStream(open(ExecutionContext) : 리소스 열고 초기화, 최초 1회) -> ItemReader -> ItemStream(update(ExecutionContext) : 현재 상태정보 저장, Chunk size만큼 반복) -> DB 저장 -> ItemProcessor -> InputStream(open(ExecutionContext) : 리소스 열고 초기화, 최초 1회) -> ItemWriter -> ItemStream(update(ExecutionContext) : 현재 상태정보 저장, Chunk size만큼 반복) -> DB 저장 -> ItemStream(close() : 모든 리소스 닫음)

```java
@RequiredArgsConstructor
@Configuration
public class ItemStreamConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(5)
                .reader(itemReader())
                .writer(itemWriter())
                .build();
    }

    @Bean
    public CustomItemStreamReader itemReader() {
        List<String> items = new ArrayList<>(10);

        for(int i = 1; i <= 10; i++) {
            items.add(String.valueOf(i));
        }

        return new CustomItemStreamReader(items);
    }

    @Bean
    public ItemWriter itemWriter() { // public ItemWriter<? super String> itemWriter() {
        return new CustomItemWriter();
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
public class CustomItemStreamReader implements ItemStreamReader<String> {

    private final List<String> items;
    private int index = -1;
    private boolean restart = false;

    public CustomItemStreamReader(List<String> items) {
        this.items = items;
        this.index = 0;
    }

    @Override
    public String read() throws Exception {
        String item = null;

        if(this.index < this.items.size()) {
            item = this.items.get(index);
            index++;
        }

        if(this.index == 6 && !restart) {
            throw new RuntimeException("Restart is required.");
        }

        return item;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {
        if(executionContext.containsKey("index")) {
            index = executionContext.getInt("index");
            this.restart = true;
        }
        else {
            index = 0;
            executionContext.put("index", index);
        }
    }

    @Override
    public void update(ExecutionContext executionContext) throws ItemStreamException {
        executionContext.put("index", index);
    }

    @Override
    public void close() throws ItemStreamException {
        System.out.println("close");
    }
}
```
```java
public class CustomItemWriter implements ItemStreamWriter<String> { // ItemStremWriter 사용

    @Override
    public void write(List<? extends String> items) throws Exception {
        items.forEach(item -> System.out.println(item));
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {
        System.out.println("");
    }

    @Override
    public void update(ExecutionContext executionContext) throws ItemStreamException {
        System.out.println("");
    }

    @Override
    public void close() throws ItemStreamException {
        System.out.println("");
    }
}
```

- 참고
> TaskletStep, AbstractStep, CompositeItemStream, SimpleChunkProvider, SimpleChunkProcessor

## Chunk Process 아키텍처
1. Chunk 단위마다 트랜잭션 생성, 트랜잭션 내 Chunk 프로세스 반복
    - Job -> TaskletStep -> RepeatTemplate -> Transaction(ChunkOrientedTasklet -> Chunk(SimpleChunkProvider -> RepeatTemplate -> Begin Transaction -> ItemReader -> read() -> source(DB, File) -> **null? YES** : RepeatStatus.FINISHED(ItemReader가 읽은 Item이 null일 경우 모든 반복문 및 Chunk 프로세스 종료, TaskletStep 종료) or **NO : add(item)** -> `Chunk<I>` -> Limit Chunk size? NO : Chunk size만큼 반복하면서 ItemReader 실행 or YES : 읽은 아이템이 담긴 inputs 전달 -> SimpleChunkProcessor -> ItemProcessor -> Iterator -> inputs -> process(item) -> ItemProcessor -> add(Item) -> `Chunk<O>` -> write(`List<Item>`) -> ItemWriter -> Chunk Size = Commit Interval -> DB)) -> Commit Transaction(Chunk Size = Commit Interval) -> RepeatTemplate(ItemWriter 처리가 완료되면 Chunk 트랜잭션이 종료하게 되고 Chunk 단위의 프로세스가 다시 실행된다.)
    - RollbackTransaction(ItemReader, ItemProcessor, ItemWriter) -> Exception -> TaskletStep
2. Transaction(ChunkOrientedTasklet -> Chunk(SimpleChunkProvider -> `Chunk<I>` -> inputs(Chunk size만큼 반복하여 item을 `Chunk<I>`에 담는다.) -> ItemReader -> `add(item) -> inputs`, `read() -> source` -> `Chunk<I>` 전달 -> SimpleChunkProcessor -> `Chunk<O>` -> inputs -> ItemProcessor(읽은 아이템만큼 반복하여 item을 `Chunk<O>`에 담는다.) -> `add(item) -> outputs`, `process(item) -> inputs` -> write(`List<Item>`) -> ItemWriter) * 반복)
    - read(), process(item) : Item 단위로 개별 처리 : Chunk size 내에서 item 한 건씩 처리가 이루어진다.
    - write(`List<item>`) : Chunk 단위로 일괄 처리, Chunk size만큼 `List<item>`가 전달되어 일괄적으로 출력작업 처리가 이루어진다. Chunk size는 곧 Commit 간격이 된다.
    - **ItemReader vs Item Processor : ItemReader에서 item을 한 개 읽고 나서 ItemProcessor로 바로 전달하는 것이 아닌 Chunk size만큼 item을 모두 읽은 다음 ItemProcessor에게 전달하면 읽은 item 개수만큼 반복 처리한다.**


- `git checkout Part5.5`
```java
@RequiredArgsConstructor
@Configuration
public class ChunkArchitectureConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(3)
                .reader(itemReader())
                .processor(itemProcessor())
                .writer(itemWriter())
                .build();
    }

    @Bean
    public ItemReader itemReader() {
        return new CustomItemReader(Arrays.asList(new Customer("user1"), new Customer("user2"), new Customer("user3")));
    }

    @Bean
    public ItemProcessor itemProcessor() {
        return new CustomItemProcessor();
    }

    @Bean
    public ItemWriter itemWriter() {
        return new CustomItemWriter();
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
@Data
@AllArgsConstructor
public class Customer {
    private String name;
}
```
```java
public class CustomItemReader implements ItemReader<Customer> {

    private List<Customer> list;

    public CustomItemReader(List<Customer> list) {
        this.list = new ArrayList<>(list);
    }

    @Override
    public Customer read() {
        if (!list.isEmpty()) {
            return list.remove(0);
        }
        return null;
    }
}
```
```java
public class CustomItemProcessor implements ItemProcessor<Customer, Customer> {

    @Override
    public Customer process(Customer customer) throws Exception {
        customer.setName(customer.getName().toUpperCase());
        return null;
    }
}
```
```java
public class CustomItemWriter implements ItemWriter<Customer> {

    @Override
    public void write(List<? extends Customer> items) throws Exception {
        items.forEach(item -> System.out.println(item));
    }
}
```