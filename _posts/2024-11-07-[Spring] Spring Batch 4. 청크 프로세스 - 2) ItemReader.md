---
title: '[Spring] Spring Batch 4. 청크 프로세스 - 2) ItemReader'
date: 2024-11-07 09:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Flat Files
### FlatFileItemReader
 1. 기본 개념
    - FlatFiles
    - 2차원 데이터(표)로 표현된 유형의 파일을 처리하는 ItemReader
    - 일반적으로 고정 위치로 정의된 데이터 필드나 특수 문자에 의해 구별된 데이터의 행을 읽는다.
    - Resource와 LineMapper 두 가지 요소가 필요하다.

2. 구조
    - FlatFileItemReader
        - String encoding = DEFAULT_CHARSET : 문자열 인코딩, 디폴트는 Charset.defaultCharset()
        - int linesToSkip : 파일 상단에 있는 무시할 라인 수
        - String[] comments : 해당 코멘트 기호가 있는 라인은 무시한다.
        - Resource resource : 읽어야 할 리소스
        - `LineMapper<T> lineMapper` : String을 Object로 변환한다.
        - LineCallbackHandler skippedLinesCallback : 건너뛸 라인의 원래 내용을 전달하는 인터페이스, linesToSkip이 2이면 두 번 호출된다.
    - Resource
        - FileSystemResource : new FileSystemResource("resource/path/config.xml")
        - ClassPathResource : new ClassPathResource("classpath:path/config.xml")
    - LineMapper
        - 파일의 라인 한 줄을 Object로 변환해서 FlatFileItemReader로 리턴한다.
        - 단순히 문자열을 받기 때문에 문자열을 토큰화해서 객체로 매핑하는 과정이 필요하다.
        - LienTokenizer와 FieldSetMApper를 사용해서 처리한다.
        - `LineMapper <T>`
            - T mapLine(String line, int lineNumber) : 현재 라인과 라인넘버를 인자로 받고 객체 반환
            - DefaultLineMapper
                - `LineTokenizer`
                    - FieldSet tokenize(@Nullable String line) : 라인을 인자로 받고 FieldSet 반환
                        - DelimitedLineTokenizer : 구분자를 사용해 필드로 구분하는 파일에 사용
                        - FixedLengthTokenizer : 각 필드를 고정된 길이로 정의하는 파일에 사용
                - `FieldSetMapper<T>`
                    - T mapFieldSet(FieldSet fieldSet) : FieldSet를 사용해 객체 매핑 후 반환
                        - BeanWrapperFieldSetMapper : 객체의 필드명과 일치하는 FieldSet의 필드를 자동으로 매핑<br/>
        
        - FieldSet
            - 라인을 필드로 구분해서 만든 배열 토큰을 전달하면 토큰 필드를 참조할 수 있도록 한다.
            - JDBC의 ResultSet과 유사하다. ex) fs.readString(0), fs.readString("name")
        - LineTokenizer
            - 입력받은 라인을 FieldSet으로 변환해서 리턴한다.
            - 파일마다 형식이 다르기 때문에 문자열을 FieldSet으로 변환하는 작업을 추상화시켜야 한다.
        - FieldSetMapper
            - FieldSet 객체를 받아서 원하는 객체로 매핑해서 리턴한다.
            - JdbcTemplate의 RowMapper와 동일한 패턴을 사용한다.<br/>
    
    - Step에서의 처리 과정
        - Step -> read() -> FlatFileItemReader -> readLine() -> mapLine(line) -> LineMapper -> mapFieldSet() -> FieldSetMapper(FieldSet이 필요) -> tokenize() -> LineTokenizer(배열 토큰 전달) -> tokens[] -> FieldSet
        - FieldSet -> LineTokenizer(FieldSet) -> FieldSetMapper(Customer) -> LineMapper(Customer) -> FlatFileItemReader

3. API 소개
    ```java
    public FlatFileItemReader itemReader() {
        return new FlatFileItemReaderBuilder<T> ()
                    .name(String name) // 이름 설정, ExecutionContext 내에서 구분하기 위한 key로 저장
                    .resource(Resource) // 읽어야 할 리소스 설정
                    .delimited().delimiter("|") // 파일의 구분자를 기준으로 파일을 읽어들이는 설정
                    .fixedLength() // 파일의 고정길이를 기준으로 파일을 읽어들이는 설정
                    .addColumns(Range..) // 고정길이 범위를 정하는 설정
                    .names(String[] fieldNames) // LineTokenizer로 구분된 라인의 항목을 객체의 필드명과 매핑하도록 설정
                    .targetType(Class class) // 라인의 각 항목과 매핑할 객체 타입 설정
                    .addComment(String Comment) // 무시할 라인의 코멘트 기호 설정
                    .strict(boolean) // 라인을 읽거나 토큰화할 때 Parsing 예외가 발생하지 않도록 검증 생략하도록 설정
                    .encoding(String encoding) // 파일 인코딩 설정
                    .linesToSkip(int linesToSkip) // 파일 상단에 있는 무시할 라인 수 설정
                    .saveState(boolean) // 상태정보를 저장할 것인지 설정
                    .setLineMapper(LineMapper) // LineMapper 객체 설정
                    .setFieldSetMapper(FieldSetMapper) // FieldSetMapper 객체 설정
                    .setLineTokenizer(LineTokenizer) // LineTokenizer 객체 설정
                    .build();
    }
    ```

- `git checkout Part6.1.1.1`
```java
@RequiredArgsConstructor
@Configuration
public class FlatFilesConfiguration {

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
                .writer(new ItemWriter() {
                    @Override
                    public void write(List items) throws Exception {
                        System.out.println("items = " + items);
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

    @Bean
    public ItemReader itemReader(){

        FlatFileItemReader<Customer> itemReader = new FlatFileItemReader<>();
        itemReader.setResource(new ClassPathResource("/customer.csv"));

        DefaultLineMapper<Customer> lineMapper = new DefaultLineMapper<>();
        lineMapper.setLineTokenizer(new DelimitedLineTokenizer());
        lineMapper.setFieldSetMapper(new CustomerFieldSetMapper());

        itemReader.setLineMapper(lineMapper);
        itemReader.setLinesToSkip(1);

        return itemReader;
    }
}
```
```java
@Data
public class Customer {

    private String name;
    private int age;
    private String year;
}
```
```java
public class DefaultLineMapper<T> implements LineMapper<T> {

    private LineTokenizer tokenizer;
    private FieldSetMapper<T> fieldSetMapper;


    public T mapLine(String line, int lineNumber) throws Exception {
        return fieldSetMapper.mapFieldSet(tokenizer.tokenize(line));
    }

    public void setLineTokenizer(LineTokenizer tokenizer) {
        this.tokenizer = tokenizer;
    }

    public void setFieldSetMapper(FieldSetMapper<T> fieldSetMapper) {
        this.fieldSetMapper = fieldSetMapper;
    }

}
```
```java
public class CustomerFieldSetMapper implements FieldSetMapper<Customer> {

    @Override
    public Customer mapFieldSet(FieldSet fs) {

        if(fs == null){
            return null;
        }

        Customer customer = new Customer();
        customer.setName(fs.readString(0));
        customer.setAge(fs.readInt(1));
        customer.setYear(fs.readString(2));

        return customer;
    }
}
```

- resources
> /customer.csv
```
name,age,year
user1,30,2000
user2,29,2001
user3,28,2002
user4,27,2003
user5,26,2004
user6,25,2005
user7,24,2006
user8,23,2007
user9,22,2008
user10,21,2009
```

- 참고
> FlatFileItemReader, AbstractLineTokenizer, LineTokenizer, TaskletStep

4. 예제
    - DelimitedLineTokenizer
        - 기본 개념
            - 한 개 라인의 String 구분자 기준으로 나누어 토큰화하는 방식
        - 구조
            - AbstractLineTokenizer
                - String[] names : FieldSet에서 인덱스가 아닌 필드명으로 객체와 매핑할 수 있도록 함
                - boolean strict = true : 토큰화 검증을 적용할 것인지 여부, 기본값은 true
                - FieldSetFactory fieldSetFactory : FieldSet 생성 팩토리 객체
            - DelimitedLineTokenizer
                - String delimiter : 구분자, 기본값은 콤마(,)
                - FieldSet tokenize(@Nullable String line) : String line을 토큰화해서 FieldSet에게 넘겨주고 반환
    
    - `git checkout Part6.1.1.2`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlatFilesDelimitedConfiguration {

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
                    .writer(new ItemWriter() {
                        @Override
                        public void write(List items) throws Exception {
                            items.forEach(item -> System.out.println(item));
                        }
                    })
                    .build();
        }

        public FlatFileItemReader itemReader() {
            return new FlatFileItemReaderBuilder<Customer>()
                    .name("flatFile")
                    .resource(new ClassPathResource("customer.csv"))
                    .fieldSetMapper(new CustomerFieldSetMapper())
    //                .targetType(Customer.class)
                    .linesToSkip(1)
                    .delimited().delimiter(",")
                    .names("name","year","age")
                    .build();
        }

        public FlatFileItemReader itemReader2() {
            return new FlatFileItemReaderBuilder<Customer>()
                    .name("flatFile")
                    .resource(new ClassPathResource("customer.csv"))
                    .fieldSetMapper(new BeanWrapperFieldSetMapper<>())
                    .targetType(Customer.class)
                    .linesToSkip(1)
                    .delimited().delimiter(",") // 구분자
                    .names("name","year","age")
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
    @Data
    public class Customer {

        private String name;
        private String year;
        private int age;
    }
    ```
    ```java
    public class CustomerFieldSetMapper implements FieldSetMapper<Customer> {

        @Override
        public Customer mapFieldSet(FieldSet fieldSet) {
            Customer customer = new Customer();

            customer.setName(fieldSet.readString(0));
            customer.setYear(fieldSet.readString(1));
            customer.setAge(fieldSet.readInt(2));

            return customer;
        }
    }
    ```

    - FixedLengthTokenizer
        - 기본 개념
            - 한 개 라인의 String을 사용자가 설정한 고정길이 기준으로 나누어 토큰화하는 방식
            - 범위는 문자열 형식으로 설정할 수 있다.
                - "1-4" 또는 "1-3, 4-6, 7" 또는 "1-2, 4-5, 7-10"
                - 마지막 범위가 열려 있으면 나머지 행이 해당 열로 밝혀진다.
        - 구조
            - FixedLengthTokenizer
                - Range[] ranges : 열의 범위를 설정
                - int maxRange = 0 : 최대 범위 설정
                - Boolean open = false : 마지막 범위가 열려 있는지 여부
            - FlatFileItemReader
            - `DefaultLineMapper<T>`
            - DelimitedLineTokenizer

    - `git checkout Part6.1.1.3`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlatFilesFixedLengthConfiguration {

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
                    .writer(new ItemWriter() {
                        @Override
                        public void write(List items) throws Exception {
                            items.forEach(item -> System.out.println(item));
                        }
                    })
                    .build();
        }

        public FlatFileItemReader itemReader() {
            return new FlatFileItemReaderBuilder<Customer>()
                    .name("flatFile")
                    .resource(new FileSystemResource("C:\\lecture\\src\\main\\resources\\customer.txt"))
                    .fieldSetMapper(new BeanWrapperFieldSetMapper<>())
                    .targetType(Customer.class)
                    .linesToSkip(1)
                    .fixedLength()
                    .addColumns(new Range(1,5)) // 범위 없으면 끝까지 읽음
                    .addColumns(new Range(6,9))
                    .addColumns(new Range(10,11))
                    /*.addColumns(new Range(1))
                    .addColumns(new Range(6))
                    .addColumns(new Range(10))*/
                    .names("name","year","age")
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
    @Data
    public class Customer {

        private String name;
        private String year;
        private int age;
    }
    ```
    ```java
    public class CustomerFieldSetMapper implements FieldSetMapper<Customer> {

        @Override
        public Customer mapFieldSet(FieldSet fieldSet) {
            Customer customer = new Customer();

            customer.setName(fieldSet.readString(0));
            customer.setYear(fieldSet.readString(1));
            customer.setAge(fieldSet.readInt(2));

            return customer;
        }
    }
    ```

5. Exception Handling
    - 기본 개념
        - 라인을 읽거나 토큰화할 때 발생하는 Parsing 예외를 처리할 수 있도록 예외 계층 제공
        - 토큰화 검증을 엄격하게 적용하지 않도록 설정하면 Parsing 예외가 발생하지 않도록 할 수 있다.
    - 구조
        - ItemReaderException <- FaltFileParseException : ItemReader에서 파일을 읽어들이는 동안 발생하는 예외
        - FlatFileFormatException : LineTokenizer에서 토큰화하는 도중 발생하는 예외, FlatFileException보다 좀 더 구체적인 예외
            - <- IncorrectTokenCountException : DelimitedLineTokenizer로 토큰화할 때 컬럼 개수와 실제 토큰화한 컬럼 수와 다를 때 발생하는 예외
            - <- IncorrectLineLengthException : FixedLengthLineTokenizer으로 토큰화할 때 라인 전체 길이와 컬럼 길이의 총합과 일치하지 않을 때 발생
    - 토큰화 검증 기준 설정
        1. tokenizer.setColumns(new Range[] { new Range(1,5), newRange(6, 10) }); : 토큰 길이 10자
        2. tokenizer.setStrict(false); : 토큰화 검증을 적용하지 않음
        3. FieldSet tokens = tokenizer.tokenize("12345"); : 라인 길이 5자

        - LineTokenizer의 Strict 속성을 **false**로 설정하게 되면 Tokenizer가 라인 길이를 검증하지 않는다.
        - Tokenizer가 라인 길이나 컬럼명을 검증하지 않을 경우 예외가 발생하지 않는다.
        - FieldSet은 성공적으로 리턴이 되며 두 번째 범위 값은 빈토큰을 가지게 된다.

    - `git checkout Part6.1.1.4`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class ExceptionHandlingConfiguration {

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
                    .writer(new ItemWriter() {
                        @Override
                        public void write(List items) throws Exception {
                            items.forEach(item -> System.out.println(item));
                        }
                    })
                    .build();
        }

        public FlatFileItemReader itemReader() {
            return new FlatFileItemReaderBuilder<Customer>()
                    .name("flatFile")
                    .resource(new ClassPathResource("customer.txt"))
                    .fieldSetMapper(new BeanWrapperFieldSetMapper<>())
                    .targetType(Customer.class)
                    .linesToSkip(1)
                    .fixedLength()
                    .strict(false) // 라인 길이 검증X
                    .addColumns(new Range(1,5))
                    .addColumns(new Range(6,9))
                    .addColumns(new Range(10,11))
                    .names("name","year","age")
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
    @Data
    public class Customer {

        private String name;
        private String year;
        private int age;
    }
    ```
    ```java
    public class CustomerFieldSetMapper implements FieldSetMapper<Customer> {

        @Override
        public Customer mapFieldSet(FieldSet fieldSet) {
            Customer customer = new Customer();

            customer.setName(fieldSet.readString(0));
            customer.setYear(fieldSet.readString(1));
            customer.setAge(fieldSet.readInt(2));

            return customer;
        }
    }
    ```

## XML
### StaxEventItemReader
1. Java XML API
    - DOM 방식
        - 문서 전체를 메모리에 로드한 후 Tree 형태로 만들어서 데이터를 처리하는 방식, pull 방식
        - 엘리먼트 제어는 유연하나 문서 크기가 클 경우 메모리 사용이 많고 속도가 느림
    - SAX 방식
        - 문서의 항목을 읽을 때마다 이벤트가 발생하여 데이터를 처리하는 push 방식
        - 메모리 비용이 적고 속도가 빠른 장점은 있으나 엘리먼트 제어가 어려움
    - **StAX 방식 (Streaming API for XML)**
        - DOM과 SAX의 장점과 단점을 보완한 API 모델로서 push와 pull을 동시에 제공함
        - XML 문서를 읽고 쓸 수 있는 양방향 파서기 지원
        - XML 파일의 항목에서 항목으로 직접 이동하면서 Stax 파서기를 통해 구문 분석
        - 유형
            - Iterator API 방식
                - XMLEventReader의 nextEvent()를 호출해서 이벤트 객체를 가지고 옴
                - 이벤트 객체는 XML 태크 유형(요소, 텍스트, 주석 등)에 대한 정보를 제공함
            - Cursot API 방식
                - JDBC Resultset처럼 작동하는 API로서 XMLStreamReader는 XML 문서의 다음 요소로 커서를 이동한다.
                - 커서에서 직접 메소드를 호출하여 현재 이벤트에 대한 자세한 정보를 얻는다.

2. Spring-OXM
    - 스프링의 Object XML Mapping 기술로 XML 바인딩 기술을 추상화함
        - Marshaller
            - marshall : 객체를 XML로 직렬화하는 행위
        - Unmarshaller
            - unmarshall : XML를 객체로 역직렬화하는 행위
        - Marshaller와 Unmarshaller 바인딩 기능을 제공하는 오픈소스로 axB2, Castor, XmlBeans, Xstream 등이 있다.
    - 스프링 배치는 특정한 XML 바인딩 기술을 강요하지 않고 Spring OXM에 위임한다.
        - 바인딩 기술을 제공하는 구현체를 선택해서 처리하도록 한다.

3. Spring Batch XML
    - 스프링 배치에서는 StAX 방식으로 XML 문서를 처리하는 StaxEventItemReader를 제공한다.
    - XML을 읽어 자바 객체로 매핑하고 자바 객체를 XML로 쓸 수 있는 트랜잭션 구조를 지원

4. StAX 아키텍처
    - XML 전체 문서가 아닌 조각 단위(Fragment)로 구문을 처리할 수 있다.
    - 루트 엘리먼트인 `<trade>`와 `<trade>` 사이에 있는 것들은 전부 하나의 조각(Fragment)을 구성한다.
    - 조각을 읽을 때 DOM의 pull 방식을 사용하고 조각을 객체로 바인딩 처리하는 것은 SAX의 Push 방식을 사용한다.

    ```java
    public StaxEventItemReader itemReader() {
        return StaxEventItemReaderBuilder<T> ()
                .name(String name)
                .resource(Resource) // 읽어야 할 리소스 설정
                .addFragmentRootElements(String... rootElements) // Fragment 단위의 루트 엘리먼트 설정, 이 루트 조각 단위가 객체와 매핑하는 기준
                .unmarshaller(Unmarshaller) // Unmarshaller 객체 설정
                .saveState(boolean) // 상태 정보 저장 여부 설정, 기본값은 true
                .build();
    }
    ```
    ```xml
    <!-- rootElements -->
    <trade>
        <isin>XYZ0001</isin>
        <quantity>5</quantity>
        <price>11.39</price>
        <customer>Customer1</customer>
    </trade>
    ```

5. 예제
    - 기본 개념
        - Stax API 방식으로 데이터를 읽어들이는 ItemReader
        - Spring-OXM과 Xstream 의존성을 추가해야 한다.
        - `StaxEventItemReader<T>`
            - FragmentEventReader fragmentReader : XML 조각을 독립형 XML 문서로 처리하는 것을 지원하는 이벤트 판독기
            - XMLEventReader eventReader : XML 이벤트 구문 분석을 위한 최상위 인터페이스
            - Unmarshaller unmarshaller : XML 문서를 객체로 직렬화하는 인터페이스
            - Resource resource : 다양한 리소스에 접근하도록 추상화한 인터페이스
            - `List<QName>` fragmentRootElementNames : 조각 단위의 루트 엘리먼트명을 담은 리스트 변수

    - 구조
        - Step -> read() -> StaxEventItemReader -> unmarshall() -> XStreamMarshaller -> **DOM Tree** -> XMLEventReader 
        - StaxEventItemReader
            - Resource : 읽을 파일을 나타내는 스프링 Resource
            - Root Element Name : 객체에 매핑되는 조각을 감싸고 있는 루트 엘리먼트 이름
            - Unmarshaller : Spring OXM이 지원하는 XML의 조각을 객체에 매핑시키는 언마샬러 지정
        - XStreamMarshaller
            - 맵으로 alias 지정할 수 있다.
            - 첫 번째 키는 조각의 루트 엘리먼트, 값은 바인딩할 객체 타입
            - 두 번째부터는 하위 엘리먼트와 각 클래스 타입

    - Dependency
    > org.springframework.spring-oxm, com.thoughtworks.xstream.xstream

    - `git checkout Part6.1.2`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class XMLConfiguration {

        private final JobBuilderFactory jobBuilderFactory;
        private final StepBuilderFactory stepBuilderFactory;

        @Bean
        public Job job() {
            return jobBuilderFactory.get("batchJob")
                    .incrementer(new RunIdIncrementer())
                    .start(step1())
                    .build();
        }

        @Bean
        public Step step1() {
            return stepBuilderFactory.get("step1")
                    .<Customer, Customer>chunk(3)
                    .reader(customItemReader())
                    .writer(customItemWriter())
                    .build();
        }

        @Bean
        public StaxEventItemReader<Customer> customItemReader() { // public ItemReader<? extends Customer> customItemReader() {
            return new StaxEventItemReaderBuilder<Customer>()
                    .name("xmlFileItemReader")
                    .resource(new ClassPathResource("customer.xml"))
                    .addFragmentRootElements("customer")
                    .unmarshaller(itemUnMarshaller()) // UnMarshaller
                    .build();
        }

        /*@Bean
        public StaxEventItemReader<Customer> customItemReader() {

            XStreamMarshaller unmarshaller = new XStreamMarshaller();

            Map<String, Class> aliases = new HashMap<>();
            aliases.put("customer", Customer.class);
            aliases.put("id", Long.class);
            aliases.put("firstName", String.class);
            aliases.put("lastName", String.class);
            aliases.put("birthdate", Date.class);

            unmarshaller.setAliases(aliases);

            StaxEventItemReader<Customer> reader = new StaxEventItemReader<>();

            reader.setResource(new ClassPathResource("customers.xml"));
            reader.setFragmentRootElementName("customer");
            reader.setUnmarshaller(unmarshaller);

            return reader;
        }*/

        @Bean
        public ItemWriter<Customer> customItemWriter() {
            return items -> {
                for (Customer item : items) {
                    System.out.println(item.toString());
                }
            };
        }

        @Bean
        public XStreamMarshaller itemUnMarshaller() {
            Map<String, Class<?>> aliases = new HashMap<>();
            aliases.put("customer", Customer.class);
            aliases.put("id", Long.class);
            aliases.put("name", String.class);
            aliases.put("age", Integer.class);
            XStreamMarshaller xStreamMarshaller = new XStreamMarshaller();
            xStreamMarshaller.setAliases(aliases);
            return xStreamMarshaller;
        }
    }
    ```
    ```java
    @Data
    public class Customer {

        private final long id;
        private final String name;
        private final int age;
    }
    ```
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <customers>
        <customer id="1">
            <id>1</id>
            <name>hong gil dong1</name>
            <age>40</age>
        </customer>
        <customer>
        <id>2</id>
            <name>hong gil dong2</name>
            <age>42</age>
        </customer>
        <customer>
        <id>3</id>
            <name>hong gil dong3</name>
            <age>43</age>
        </customer>
    </customers>
    ```

    - resources
    > /xml.csv

## Json
### JsonItemReader
1. 기본 개념
    - Json 데이터의 Parsing과 Binding을 JsonObjectReader 인터페이스 구현체에 위임하여 처리하는 ItemReader
    - 두 가지 구현체 제공
        - JacksonJsonObjectReader
        - GsonJsonObjectReader

2. 구조
    - `JsonItemReader<T>`
        - Resource resource : 다양한 리소스에 접근하도록 추상화한 인터페이스
        - JsonObjectReader jsonObjectReader : Json 구문을 객체로 반환
    - `JacksonJsonObjectReader<T>`
        - `Class<? extends T> itemType` : Json 데이터를 매핑할 객체 타입
        - JsonParser jsonParser : Json 구문을 분석하는 파서기
        - ObjectMapper mapper : Json을 Object로 매핑하는 매퍼
        - InputStream inputStream : Json 파일로부터 읽는 입력 스트림
    - ChunkOrientedTasklet -> read() -> JsonItemReader -> read() -> JsonObjectReader <-> ObjectMapper -> readValue(JsonParser, Customer.class)

- `git checkout Part6.1.3`
```java
@RequiredArgsConstructor
@Configuration
public class JsonConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(3)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JsonItemReader<Customer> customItemReader(){
        return new JsonItemReaderBuilder<Customer>()
                .jsonObjectReader(new JacksonJsonObjectReader<>(Customer.class))
                .resource(new ClassPathResource("customer.json"))
                .name("jsonItemReader")
                .build();
    }

    @Bean
    public ItemWriter<Customer> customItemWriter() {
        return items -> {
            for (Customer item : items) {
                System.out.println(item.toString());
            }
        };
    }
}
```
```java
@Data
@NoArgsConstructor
public class Customer {

	private long id;
	private String name;
	private int age;
}
```
```json
[
    {
        "id" : 1,
        "name" : "hong gil dong1",
        "age" : 41
    },
    {
        "id" : 2,
        "name" : "hong gil dong2",
        "age" : 42
    },
    {
        "id" : 3,
        "name" : "hong gil dong3",
        "age" : 43
    }
]
```

- resources
> /customer.json

- 참고
> JsonItemReader, AbstractItemCountingItemStreamItemReader, JacksonJsonObjectReade

## DB
### Cursor Based & Paging Based
1. 기본 개념
    - 배치 어플리케이션은 실시간적 처리가 어려운 대용량 데이터를 다루며 이 때 DB I/O의 성능 문제와 메모리 자원의 효율성 문제를 해결할 수 있어야 한다.
    - 스프링 배치에서는 대용량 데이터 처리를 위한 두 가지 해결방안을 제시하고 있다.

2. Cursor Based 처리
    - JDBC ResultSet의 기본 메커니즘을 사용
    - **현재 행에 커서를 유지하며 다음 데이터를 호출하면 다음 행으로 커서를 이동**하며 데이터 반환이 이루어지는 Streaming 방식의 I/O이다.
    - ResultSet이 open 될 때마다 next() 메소드가 호출되어 Database의 데이터가 반환되고 객체와 매핑이 이루어진다.
    - **DB Connection이 연결되면 배치 처리가 완료될 때까지 데이터를 읽어오기 때문에 DB와 SocketTimeout을 충분히 큰 값으로 설정 필요**
    - 모든 결과를 메모리에 할당하기 때문에 메모리 사용량이 많아지는 단점이 있다.
    - **Connection 연결 유지 시간과 메모리 공간이 충분하다면 대량의 데이터 처리에 적합할 수 있다. (fetchSize 조절)**
    - Cursor Base -> read, fetchSize -> DB(Cursor(Connection), fetch(Streaming)) -> return ITem

3. Paging Based 처리
    - 페이징 단위로 데이터를 조회하는 방식으로 Page Size 만큼 한 번에 메모리로 가지고 온 다음 한 개씩 읽는다.
    - **한 페이지를 읽을 때마다 Connection을 맺고 끊기 때문에 대량의 데이터를 처리하더라도 SocketTimeout 예외가 거의 일어나지 않는다.**
    - 시작 행 번호를 지정하고 페이지에 반환시키고자 하는 행의 수를 지정한 후 사용 (Offset, Limit)
    - 페이징 단위의 결과만 메모리에 할당하기 때문에 메모리 사용량이 적어지는 장점이 있다.
    - **Connection 연결 유지 시간이 길지 않고 메모리 공간을 효율적으로 사용해야 하는 데이터 처리에 적합할 수 있다.**
    - Paging Based -> read, offset/size -> DB(offset, size -> Paging(Connection)) -> return `List<Item>`

### JdbcCursorItemReader
1. 기본 개념
    - Cursor 기반의 JDBC 구현체로서 ResultSet과 함께 사용되며 Datasource에서 Connection을 얻어와서 SQL을 실행한다.
    - Thread 안정성을 보장하지 않기 때문에 멀티 스레드 환경에서 사용할 경우 동시성 이슈가 발생하지 않도록 별도 동기화 처리가 필요하다.

2. API
    ```java
    public JdbcCursorItemReader itemReader() {
        return new JdbcCursorItemReaderBuilder<T> ()
                    .name("cursorItemReader")
                    .fetchSize(int chunkSize) // Cursor 방식으로 데이터를 가지고 올 때 한 번에 메모리에 할당할 크기를 설정한다.(= Chunk Size)
                    .dataSource(DataSource) // DB에 접근하기 위해 Datasource 설정
                    .rowMapper(RowMapper) // 쿼리 결과로 반환되는 데이터와 객체를 매핑하기 위한 RowMapper 설정
                    .beanRowMapper(Class <T>) // 별도의 RowMapper을 설정하지 않고 클래스 타입을 설정하면 자동으로 객체와 매핑
                    .sql(String sql) // ItemReader가 조회할 때 사용할 쿼리 문장 설정
                    .queryArguments(Object... args) // 쿼리 파라미터 설정
                    .maxItemCount(int count) // 조회할 최대 item 수
                    .currentItemCount(int count) // 조회 item의 시작 지점
                    .maxRows(int maxRows) // ResultSet 오브젝트가 포함할 수 있는 최대 행 수
                    .build();
    }
    ```

3. 구조
    - Step -> read() -> JdbcCusorItemReader -> Open Cursor -> DB Connection, PreparStatement, ResultSet
    - Step -> open() -> (ItemStream -> readCursor() -> RowMapper -> ResultSet -> next() -> DB -> ResultSet -> RowMApper -> return Object -> JdbcCursorItemReader) * Chunk Size 반복(**Streaming**)
    - Step -> close() -> ItemStream -> Close Cursor -> Close ResultSet, Close PrepareStatement, Close Connection

- `git checkout Part6.1.4.1`
```java
@RequiredArgsConstructor
@Configuration
public class JdbcCursorConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JdbcCursorItemReader<Customer> customItemReader() {
        return new JdbcCursorItemReaderBuilder()
                .name("jdbcCursorItemReader")
                .fetchSize(10) // chunkSize
                .sql("select id, firstName, lastName, birthdate from customer where firstName like ? order by lastName, firstName")
                .beanRowMapper(Customer.class)
                .queryArguments("A%")
                .maxItemCount(3)
                .currentItemCount(2)
                .maxRows(100)
                .dataSource(dataSource)
                .build();
    }

    @Bean
    public ItemWriter<Customer> customItemWriter() {
        return items -> {
            for (Customer item : items) {
                System.out.println(item.toString());
            }
        };
    }
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Customer {

	private Long Id;
	private String firstName;
	private String lastName;
	private String birthdate;
}
```
```sql
CREATE TABLE `customer` (
  `id` mediumint(8) unsigned NOT NULL auto_increment,
  `firstName` varchar(255) default NULL,
  `lastName` varchar(255) default NULL,
  `birthdate` varchar(255),
  PRIMARY KEY (`id`)
) AUTO_INCREMENT = 1;
```
```sql
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (1,"Reed","Edwards","1952-08-16 12:34:53"),(2,"Hoyt","Park","1981-02-18 08:07:58"),(3,"Leila","Petty","1972-06-11 08:43:55"),(4,"Denton","Strong","1989-03-11 18:38:31"),(5,"Zoe","Romero","1990-10-02 13:06:31"),(6,"Rana","Compton","1957-06-09 12:51:11"),(7,"Velma","King","1988-02-02 05:52:25"),(8,"Uriah","Carter","1972-08-31 07:32:05"),(9,"Michael","Graves","1958-04-13 18:47:44"),(10,"Leigh","Stone","1967-06-23 23:41:43");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (11,"Iliana","Glenn","1965-02-27 14:33:56"),(12,"Harrison","Haley","1956-06-28 03:15:41"),(13,"Leonard","Zamora","1956-03-28 15:03:09"),(14,"Hiroko","Wyatt","1960-08-22 23:53:50"),(15,"Cameron","Carlson","1969-05-12 11:10:09"),(16,"Hunter","Avery","1953-11-19 12:52:42"),(17,"Aimee","Cox","1976-10-15 12:56:50"),(18,"Yen","Delgado","1990-02-06 10:25:36"),(19,"Gemma","Peterson","1989-04-02 23:42:09"),(20,"Lani","Faulkner","1970-09-18 17:22:14");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (21,"Iola","Cannon","1954-01-12 16:56:45"),(22,"Whitney","Shaffer","1951-03-19 01:27:18"),(23,"Jerome","Moran","1968-03-16 05:26:22"),(24,"Quinn","Wheeler","1979-06-19 16:24:22"),(25,"Mira","Wilder","1961-12-27 12:11:07"),(26,"Tobias","Holloway","1968-08-13 20:36:19"),(27,"Shaine","Schneider","1958-03-08 09:47:10"),(28,"Harding","Gonzales","1952-04-11 02:06:25"),(29,"Calista","Nieves","1970-02-17 13:29:59"),(30,"Duncan","Norman","1987-09-13 00:54:49");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (31,"Fatima","Hamilton","1961-06-16 14:29:11"),(32,"Ali","Browning","1979-03-27 17:09:37"),(33,"Erin","Sosa","1990-08-23 10:43:58"),(34,"Carol","Harmon","1972-01-14 07:19:39"),(35,"Illiana","Fitzgerald","1970-08-19 02:33:46"),(36,"Stephen","Riley","1954-06-05 08:34:03"),(37,"Hermione","Waller","1969-09-08 01:19:07"),(38,"Desiree","Flowers","1952-06-25 13:34:45"),(39,"Karyn","Blackburn","1977-03-30 13:08:02"),(40,"Briar","Carroll","1985-03-26 01:03:34");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (41,"Chaney","Green","1987-04-20 18:56:53"),(42,"Robert","Higgins","1985-09-26 11:25:10"),(43,"Lillith","House","1982-12-06 02:24:23"),(44,"Astra","Winters","1952-03-13 01:13:07"),(45,"Cherokee","Stephenson","1955-10-23 16:57:33"),(46,"Yuri","Shaw","1958-07-14 15:10:07"),(47,"Boris","Sparks","1982-01-01 10:56:34"),(48,"Wilma","Blake","1963-06-07 16:32:33"),(49,"Brynne","Morse","1964-09-21 01:05:25"),(50,"Ila","Conley","1953-11-02 05:12:57");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (51,"Sharon","Watts","1964-01-09 16:32:37"),(52,"Kareem","Vaughan","1952-04-18 15:37:10"),(53,"Eden","Barnes","1954-07-04 01:26:44"),(54,"Kenyon","Fulton","1975-08-23 22:17:52"),(55,"Mona","Ball","1972-02-11 04:15:45"),(56,"Moses","Cortez","1979-04-24 15:26:46"),(57,"Macy","Banks","1956-12-31 00:41:15"),(58,"Brenna","Mendez","1972-10-02 07:58:27"),(59,"Emerald","Ewing","1985-11-28 21:15:20"),(60,"Lev","Mcfarland","1951-05-20 14:30:07");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (61,"Norman","Tanner","1959-07-29 15:41:45"),(62,"Alexa","Walters","1977-12-06 16:41:17"),(63,"Dara","Hyde","1989-08-04 14:06:43"),(64,"Hu","Sampson","1978-11-01 17:10:23"),(65,"Jasmine","Cardenas","1969-02-15 20:08:06"),(66,"Julian","Bentley","1954-07-11 03:27:51"),(67,"Samson","Brown","1967-10-15 07:03:59"),(68,"Gisela","Hogan","1985-01-19 03:16:20"),(69,"Jeanette","Cummings","1986-09-07 18:25:52"),(70,"Galena","Perkins","1984-01-13 02:15:31");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (71,"Olga","Mays","1981-11-20 22:39:27"),(72,"Ferdinand","Austin","1956-08-08 09:08:02"),(73,"Zenia","Anthony","1964-08-21 05:45:16"),(74,"Hop","Hampton","1982-07-22 14:11:00"),(75,"Shaine","Vang","1970-08-13 15:58:28"),(76,"Ariana","Cochran","1959-12-04 01:18:36"),(77,"India","Paul","1963-10-10 05:24:03"),(78,"Karina","Doyle","1979-12-01 00:05:21"),(79,"Delilah","Johnston","1989-03-04 23:50:01"),(80,"Hilel","Hood","1959-08-22 06:40:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (81,"Kennedy","Hoffman","1963-10-14 20:18:35"),(82,"Kameko","Bell","1976-06-08 15:35:54"),(83,"Lunea","Gutierrez","1964-06-07 16:21:24"),(84,"William","Burris","1980-05-01 17:58:23"),(85,"Kiara","Walls","1955-12-27 18:57:15"),(86,"Latifah","Alexander","1980-06-19 10:39:50"),(87,"Keaton","Ward","1964-10-12 16:03:18"),(88,"Jasper","Clements","1970-03-05 00:29:49"),(89,"Claire","Brown","1972-02-11 00:43:58"),(90,"Noble","Morgan","1955-09-05 05:35:01");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (91,"Evangeline","Horn","1952-12-28 14:06:27"),(92,"Jonah","Harrell","1951-06-25 17:37:35"),(93,"Mira","Espinoza","1982-03-26 06:01:16"),(94,"Brennan","Oneill","1979-04-23 08:49:02"),(95,"Dacey","Howe","1983-02-06 19:11:00"),(96,"Yoko","Pittman","1982-09-12 02:18:52"),(97,"Cody","Conway","1971-05-26 07:09:58"),(98,"Jordan","Knowles","1981-12-30 02:20:01"),(99,"Pearl","Boyer","1957-10-19 14:26:49"),(100,"Keely","Montoya","1985-03-24 01:18:09");
```

- resources
> schema-mysql.sql, data-mysql.sql

- 참고
> JdbcCursorItemReader, AbstractCursorItemReader,  AbstractItemStreamItemReader, AbstractItemCountingItemStreamItemReader

### JpaCursotItemReader
1. 기본 개념
    - Spring Batch 4.3 버전부터 지원
    - Cursor 기반의 JPA 구현체로서 EntityManagerFactory 객체가 필요하며 쿼리는 JPQL을 사용한다.

2. API
    ```java
    public JpaCursorItemReader itemReader() {
        return new JpaCursorItemReaderBuilder<T> ()
                    .name("cursorItemReader")
                    .queryString(String JPQL) // ItemReader가 조회할 때 사용할 JPQL 문장 설정
                    .EntityManagerFactory(EntityManagerFactory) // JPQL을 실행하는 EntityManager를 생성하는 팩토리
                    .parameterValue(Map<String, Object> parameters) // 쿼리 파라미터 설정
                    .maxItemCount(int count) // 조회할 최대 Item 수
                    .currentItemCount(int count) // 조회 Item의 시작 지점
                    .build();
    }
    ```

3. 구조
    - Step -> open() -> ItemStream -> Create EntityManager, Create Query, Create ResultStream
    - Step -> read() -> (JpaCursorItemReader -> doRead() -> ResultStream -> next() -> Iterator -> ResultStream -> return Object -> JpaCursorItemReader) * Chunk Size 반복(**Stream Processing**)
    - Step -> close() -> ItemStream -> Close EntityManager

- `git checkout Part6.1.4.2`
```java
@RequiredArgsConstructor
@Configuration
public class JpaCursorConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final EntityManagerFactory entityManagerFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(2)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JpaCursorItemReader<Customer> customItemReader() {

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "A%");

        return new JpaCursorItemReaderBuilder()
                .name("jpaCursorItemReader")
                .queryString("select c from Customer c where firstname like :firstname")
                .entityManagerFactory(entityManagerFactory)
                .parameterValues(parameters)
//                .maxItemCount(10)
//                .currentItemCount(2)
                .build();
    }

    @Bean
    public ItemWriter<Customer> customItemWriter() {
        return items -> {
            for (Customer item : items) {
                System.out.println(item.toString());
            }
        };
    }
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Customer {

	@Id
	@GeneratedValue
	private Long Id;
	private String firstname; // 변수명 매핑
	private String lastname;
	private String birthdate;
}
```

- 참고
> JpaCursorItemReader, JpaCursorItemReaderBuilder, AbstractITemCountingITemStreamItemReader

### JdbcPagingItemReader
1. 기본 개념
    - Paging 기반의 JDBC 구현체로서 쿼리에 시작 행 번호 (offset)와 페이지에서 반환할 행 수 (limit)을 지정해서 SQL을 실행한다.
    - 스프링 배치에서 offset과 limit을 PageSize에 맞게 자동으로 생성해주며 페이징 단위로 데이터를 조회할 때마다 새로운 쿼리가 실행한다.
    - 페이지마다 새로운 쿼리를 실행하기 때문에 피이징 시 결과 데이터의 순서가 보장될 수 있도록 order by 구문이 작성되도록 한다.
    - 멀티 스레드 환경에서 Thread 안정성을 보장하기 때문에 별도의 동기화를 할 필요가 없다.
    - PagingQueryProvider
        - 쿼리 실행에 필요한 쿼리문을 ItemReader에게 제공하는 클래스
        - 데이터베이스마다 페이징 전략이 다르기 때문에 각 데이터베이스 유형마다 다른 PagingQueryProvider를 사용한다.
        - Select 절, from 절, sortKey는 필수로 설정해야 하며, where, group by 절은 필수가 아니다.
        - SqlPagingQueryProviderFactoryBean : providers.put(MYSQL, new MySqlPagingQueryProvider());
            - Datasource 설정값을 보고 Provider 중 하나를 자동으로 선택

2. API
    ```java
    public JdbcPagingItemReader itemReader() {
        return new JdbcPagingItemReaderBuilder<T> ()
                    .name("pagingItemReader")
                    .pageSize(int pageSize) // 페이지 크기 설정 (쿼리당 요청할 레코드 수)
                    .dataSource(DataSource) // DB에 접근하기 위해 Datasource 설정
                    .queryProvider(PagingQueryProvider) // DB 페이징 전략에 따른 PagingQueryProvider 설정
                    .rowMapper(Class<T>) // 쿼리 결과로 반환되는 데이터와 객체를 매핑하기 위한 RowMapper 설정
                    .selectClause(String selectClause) // select 절 설정 (PagingQueryProvider API : selectClause ~ sortKeys)
                    .fromClause(String fromClause) // from 절 설정
                    .whereClause(String whereClause) // where 절 설정
                    .groupClause(String groupClause) // group 절 설정
                    .sortKeys(Map<String, Order> sortKeys) // 정렬을 위한 유니크한 키 설정
                    .parameterValues(Map<String, Object> parameters) // 쿼리 파라미터 설정
                    .maxItemCount(int count) // 조회할 최대 item 수
                    .currentItemCount(int count) // 조회 item의 시작 지점
                    .maxRows(int maxRows) // 조회 item의 시작 지점
                    .build(); // ResultSet 오브젝트가 포함할 수 있는 최대 행 수
    }
    ```

3. 구조
    - Step -> open() -> ItemStream -> update -> ExecutionContext
    - Step -> read() -> (JdbcPagingItemReader -> doReadPage() -> JdbcTemplate -> query() -> ResultSet -> next() -> DB -> data -> ResultSet -> rowMapper -> List -> return List) * Chunk Size 반복(**Paging Process**)
    - Step -> close() -> ItemStream -> close Paging

- `git checkout Part6.1.4.3`
```java
@RequiredArgsConstructor
@Configuration
public class JdbcPagingConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    public final DataSource dataSource;

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
                .<Customer, Customer>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> customItemReader() throws Exception {

        HashMap<String, Object> parameters = new HashMap<>(); // parameter 설정
        parameters.put("firstname", "A%");

        return new JdbcPagingItemReaderBuilder<Customer>()
                .name("jdbcPagingItemReader")
                .pageSize(10)
                .fetchSize(10)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Customer.class))
                .queryProvider(createQueryProvider())
				.parameterValues(parameters)
                .build();
    }

    @Bean
    public PagingQueryProvider createQueryProvider() throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause("id,firstName,lastName,birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where firstName like :firstname");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        return queryProvider.getObject();
    }

   /* @Bean
    public JdbcPagingItemReader<Customer> customItemReader() {
        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setFetchSize(5);
        reader.setPageSize(5);
        reader.setSaveState(true);
        reader.setRowMapper(new CustomerRowMapper());

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "A%");
        reader.setParameterValues(parameters);

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where firstname like :firstname");
        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);
        reader.setQueryProvider(queryProvider);

        return reader;
    }*/

    @Bean
    public ItemWriter<Customer> customItemWriter() {
        return items -> {
            for (Customer item : items) {
                System.out.println(item.toString());
            }
        };
    }
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		Customer customer = new Customer();

		customer.setId(rs.getInt("id"));
		customer.setFirstname(rs.getString("firstname"));
		customer.setLastname(rs.getString("lastname"));
		customer.setLastname(rs.getString("birthdate"));

		return customer;
	}
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Customer {

	private Long id;
	private String firstName;
	private String lastName;
	private String birthdate;
}
```

- 참고
> SqlPagingQueryProviderFactoryBean, JdbcPagingItemReader, AbstractPagingItemReader, AbstractSqlPagingQueryProvider, AbstractItemCountingItemStreamItemReader, SimpleChunkProvider

### JpaPagingItemReader
1. 기본 개념
    - Paging 기반의 JPA 구현체로서 EntityManagerFactory 객체가 필요하며 쿼리는 JPQL을 사용한다.

2. API
    ```java
    public JpaPAgingItemReader itemReader() {
        return new JpaPagingItemReaderBuilder<T>()
                .name("pagingItemReader")
                .pageSize(int count) // 페이지 크기 설정(쿼리당 요청할 레코드 수)
                .queryString(String JPQL) // ItemReader가 조회할 때 사용할 JPQL 문장 설정
                .EntityManagerFactory(EntityManagerFactory) // JPQL을 실행하는 EntityManager를 생성하는 팩토리
                .parameterValue(Map<String, Object> parameters) // 쿼리 파라미터 설정
                .build();
    }
    ```

3. 구조
    - Step -> open() -> ItemStream -> Create EntityManager
    - Step -> read() -> (JpaPagingItemReader -> doReadPAge() -> EntityManager -> createQuery() -> Query -> ResultList -> return ResultList -> JpaPAgingItemReader) * Chunk Size 반복(Paging Processing)
    - Step -> close() -> ItemStream -> Close EntityManager

- `git checkout Part6.1.4.4`
```java
@RequiredArgsConstructor
@Configuration
public class JpaPagingConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final EntityManagerFactory entityManagerFactory;

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
                .<Customer, Customer>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JpaPagingItemReader<Customer> customItemReader() { // public ItemReader<? extends Customer> customerItemReader() {

        return new JpaPagingItemReaderBuilder<Customer>()
                .name("jpaPagingItemReader")
                .entityManagerFactory(entityManagerFactory)
                .pageSize(10)
                .queryString("select c from Customer c join fetch c.address") // 1개의 쿼리, select c from Customer c : 10개의 쿼리
                .build();
    }

    @Bean
    public ItemWriter<Customer> customItemWriter() {
        return items -> {
            for (Customer customer : items) {
                System.out.println(customer.getAddress().getLocation());
            }
        };
    }
}
```
```java
@Getter
@Setter
@Entity
public class Customer {

	@Id
	@GeneratedValue
	private Long Id;
	private String username;
	private int age;

	@OneToOne(mappedBy = "customer")
	private Address address;
}
```
```java
@Getter
@Setter
@Entity
public class Address {

	@Id
	@GeneratedValue
	private Long Id;
	private String location;

	@OneToOne
	@JoinColumn(name = "customer_id")
	private Customer customer;
}
```
```yml
spring:
  profiles:
    active: local
  batch:
    job:
#      enabled: false
      names: ${job.name:NONE}
  jpa:
    hibernate:
      ddl-auto: update # update 설정
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
```sql
insert into customer (id, username, age)values (1,'user1',30);
insert into customer (id, username, age)values (2,'user2',31);
insert into customer (id, username, age)values (3,'user3',32);
insert into customer (id, username, age)values (4,'user4',33);
insert into customer (id, username, age)values (5,'user5',34);
insert into customer (id, username, age)values (6,'user6',35);
insert into customer (id, username, age)values (7,'user7',36);
insert into customer (id, username, age)values (8,'user8',37);
insert into customer (id, username, age)values (9,'user9',38);
insert into customer (id, username, age)values (10,'user10',39);

insert into address (id, location, customer_id) values (1,'location1',1);
insert into address (id, location, customer_id) values (2,'location2',2);
insert into address (id, location, customer_id) values (3,'location3',3);
insert into address (id, location, customer_id) values (4,'location4',4);
insert into address (id, location, customer_id) values (5,'location5',5);
insert into address (id, location, customer_id) values (6,'location6',6);
insert into address (id, location, customer_id) values (7,'location7',7);
insert into address (id, location, customer_id) values (8,'location8',8);
insert into address (id, location, customer_id) values (9,'location9',9);
insert into address (id, location, customer_id) values (10,'location10',10);
```

- 참고
> JpaPagingItemReader, AbstractItemCountingItemStreamItemReader, SimpleChunkProvider

### ItemReaderAdapter
1. 기본 개념
    - 배치 Job 안에서 이미 있는 DAO나 다른 서비스를 ItemReader 안에서 사용하고자 할 때 위임 역할을 한다.

2. 

- `git checkout Part6.1.4.5`
```java
@RequiredArgsConstructor
@Configuration
public class ItemReaderAdapterConfiguration {

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
                .<String,String>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public ItemReaderAdapter customItemReader() {

        ItemReaderAdapter reader = new ItemReaderAdapter();
        reader.setTargetObject(customService());
        reader.setTargetMethod("joinCustomer");

        return reader;
    }

    private CustomService<String> customService() {
        return new CustomService<>();
    }

    @Bean
    public ItemWriter<String> customItemWriter() { // 무한반복
        return items -> {
                System.out.println(items);
        };
    }
}
```
```java
public class CustomService<T> {

    private int cnt = 0;

    public T joinCustomer(){
        return (T)("item" + cnt++);
    }
}
```

- 참고
> ItemReaderAdapter, SimpleChunkProvider, AbstractMethodInvokingDelegator