---
title: '[Spring] Spring Batch 4. 청크 프로세스 - 3) ItemProcessor'
date: 2024-11-10 17:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## CompositeItemProcessor
1. 기본 개념
    - ItemProcessor들을 연결(Chaining)해서 위임하면 각 ItemProcessor를 실행시킨다.
    - 이전 ITemProcessor 반환 값은 다음 ItemProcessor 값으로 연결된다.

2. API
    ```java
    public ItemProcessor itemProcessor() {
        return new CompositeItemProcessorBuilder<>()
                    .delegates(ItemProcessor<?, ?>... delegates) // 체이닝할 ItemProcessor 객체 설정
                    .build();
    }
    ```

3. 구조
    - ItemReader -> CompositeItemProcessor(ItemProcessor1 -> ... -> N) -> ItemWriter
        - ItemProcessor 간 체이닝 하면서 Item을 가공 처리한다.

- `git checkout Part6.3.1`
```java
@RequiredArgsConstructor
@Configuration
public class CompositionItemConfiguration {

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
                .<String, String>chunk(10)
                .reader(new ItemReader<String>() {
                    int i = 0;
                    @Override
                    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                        i++;
                        return i > 10 ? null : "item";
                    }
                })
                .processor(customItemProcessor())
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        System.out.println(items);
                    }
                })
                .build();
    }

    @Bean
    public CompositeItemProcessor customItemProcessor() { // public ItemProcessor<? super String, String> customItemProcessor() {

        CompositeItemProcessor<String,String> compositeProcessor = new CompositeItemProcessor<>();
        List itemProcessors = new ArrayList();
        itemProcessors.add(new CustomItemProcessor1());
        itemProcessors.add(new CustomItemProcessor2());

        return new CompositeItemProcessorBuilder<>()
                .delegates(itemProcessors)
	            .build();
    }
}
```
```java
public class CustomItemProcessor1 implements ItemProcessor<String, String> {

    int cnt = 0;

    @Override
    public String process(String item) throws Exception {
        cnt++;
        return item + cnt;
    }
}
```
```java
public class CustomItemProcessor2 implements ItemProcessor<String, String> {

    int cnt = 0;

    @Override
    public String process(String item) throws Exception {
        cnt++;
        return item + cnt;
    }
}
```

- 결과
> ITEM11, ITEM22, ... , ITEM99, ITEM1010

## ClassifierCompositeItemProcessor
1. 기본 개념
    - Classifier로 라우팅 패턴을 구현해서 ItemProcessor 구현체 중에서 하나를 호출하는 역할을 한다.

2. API
    - `Classifier<C, T>`(T classify(C classifiable) : C의 분류에 따라 적절한 T를 반환)
    ```java
    public ItemProcessor itemProcessor() {
        return new ClassifierCompositeItemProcessorBuilder<>()
                    .classifier(Classifier) // 분류자 설정
                    .build();
    }
    ```

3. 구조
    - ItemReader -> ClassifierCompositeItemProcessor -> process(item) -> `Classifier<T, C>` -> classify(C classifier) : Classifier의 분류조건에 맞는 ItemProcessor 선택 -> C? -> OrderItemProcessor, PayItemProcessor -> ItemWriter

- `git checkout Part6.3.2`
```java
@RequiredArgsConstructor
@Configuration
public class ClassifierConfiguration {

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
                .<ProcessorInfo, ProcessorInfo>chunk(10)
                .reader(new ItemReader<ProcessorInfo>() {
                    int i = 0;
                    @Override
                    public ProcessorInfo read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                        i++;
                        ProcessorInfo processorInfo = ProcessorInfo.builder().id(i).build();
                        return i > 3 ? null : processorInfo;
                    }
                })
                .processor(customItemProcessor())
                .writer(new ItemWriter<ProcessorInfo>() {
                    @Override
                    public void write(List<? extends ProcessorInfo> items) throws Exception {
                        System.out.println(items);
                    }
                })
                .build();
    }

    @Bean
    public ItemProcessor customItemProcessor() { // public ItemProcessor<? super ProcessorInfo, ? extends ProcessorInfo> customItemProcessor() {

        ClassifierCompositeItemProcessor<ProcessorInfo, ProcessorInfo> processor = new ClassifierCompositeItemProcessor<>(); // classifier에게 선택 넘김

        ProcessorClassifier<ProcessorInfo, ItemProcessor<?, ? extends ProcessorInfo>> classifier = new ProcessorClassifier(); // classifier 분류 작업
        
        Map<Integer, ItemProcessor<ProcessorInfo, ProcessorInfo>> processorMap = new HashMap<>(); // classifier가 가진 map
        processorMap.put(1, new CustomItemProcessor1());
        processorMap.put(2, new CustomItemProcessor2());
        processorMap.put(3, new CustomItemProcessor3());

        classifier.setProcessorMap(processorMap);
        processor.setClassifier(classifier);

        return processor;
    }
}
```
```java
public class ProcessorClassifier<C,T> implements Classifier<C, T> {

    private Map<Integer, ItemProcessor<ProcessorInfo, ProcessorInfo>> processorMap = new HashMap<>();

    @Override
    public T classify(C classifiable) {
        return (T)processorMap.get(((ProcessorInfo)classifiable).getId()); // 1, 2, 3 선택
    }

    public void setProcessorMap(Map<Integer, ItemProcessor<ProcessorInfo, ProcessorInfo>> processorMap) {
        this.processorMap = processorMap;
    }
}
```
```java
public class CustomItemProcessor1 implements ItemProcessor<ProcessorInfo, ProcessorInfo> {
    @Override
    public ProcessorInfo process(ProcessorInfo item) throws Exception {

        System.out.println("CustomItemProcessor1");

        return item;
    }
}
```
```java
public class CustomItemProcessor2 implements ItemProcessor<ProcessorInfo, ProcessorInfo> {
    @Override
    public ProcessorInfo process(ProcessorInfo item) throws Exception {

        System.out.println("CustomItemProcessor2");

        return item;
    }
}
```
```java
public class CustomItemProcessor3 implements ItemProcessor<ProcessorInfo, ProcessorInfo> {
    @Override
    public ProcessorInfo process(ProcessorInfo item) throws Exception {

        System.out.println("CustomItemProcessor3");

        return item;
    }
}
```
```java
@Data
@Builder
public class ProcessorInfo {

    private int id;
}
```

- 결과
> CustomItemProcessor1, CustomItemProcessor2, CustomItemProcessor3