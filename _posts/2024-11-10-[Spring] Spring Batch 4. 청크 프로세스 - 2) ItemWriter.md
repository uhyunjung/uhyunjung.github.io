---
title: '[Spring] Spring Batch 4. 청크 프로세스 - 2) ItemWriter'
date: 2024-11-10 17:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## Flat Files
### FlatFileItemWriter
1. 기본 개념
    - 2차원 데이터(표)로 표현된 유형의 파일을 처리하는 ItemWriter
    - 고정 위치로 정의된 데이터 필드나 특수 문자에 의해 구별된 데이터의 행을 기록한다.
    - Resource와 LineAggregator 두 가지가 요소가 필요하다.

2. 구조
    - FlatFileItemWriter
        - String encoding = DEFAULT_CHARSET : 문자열 인코딩, 디폴트는 Charset.defaultCharset()
        - boolean append = false : 대상 파일이 이미 있는 경우 데이터를 계속 추가할 것인지 여부
        - Resource resource : 작성해야 할 리소스
        - `LineAggregator<T>` lineAggregator : Object를 String로 변환한다.
        - FlatFileHeaderCallback headerCallback : 헤더를 파일에 쓰기 위한 콜백 인터페이스
        - FlatFileFooterCallback footerCallback : 푸터를 파일에 쓰기 위한 콜백 인터페이스
    - LineAggregator
        - Item을 받아서 String으로 변환하여 리턴한다.
        - FieldExtractor를 사용해서 처리할 수 있다.
        - 구현체 : PassThroughLineAggregator, DelimitedLineAggregator, FormatterLineAggregator
    - FieldExtractor
        - 전달 받은 Item 객체의 필드를 배열로 만들고 배열을 합쳐서 문자열을 만들도록 구현하도록 제공하는 인터페이스
        - 구현체 : BeanWrapperFieldExtractor, PassThroughFieldExtractor
    - `LineAggregator<T>`(String aggregate(T item) : 객체를 인자로 받고 문자열 반환) -> PassThroughLineAggregator(전달된 아이템을 단순히 문자열로 반환), DelimitedLineAggregator(전달된 배열을 구분자로 구분하여 문자열로 함칩), FormtatterLineAggregator(전달된 배열을 고정길이로 구분하여 문자열로 합침) -> `FieldExtractor<T>`(Object[] extract(T item) : 객체를 인자로 받고 배열 반환) -> BeanWrapperFieldExtractor(전달된 객체의 필드들을 배열로 반환), PassThroughFieldExtractor(전달된 Collection을 배열로 반환)
    - Step -> write(`List<Item>`) -> FlatFileItemWriter -> aggregate(Item) -> LineAggregator -> extract(Item) -> FieldExtractor -> Fields Value(Object[]) -> LineAggregator -> String -> FlatFileItemWriter

3. API
    ```java
    public FlatFileItemReader itemReader() {
        return new FlatFileItemWriterBuilder<T> ()
                    .name(String name)
                    .resource(Resource) // 쓰기할 리소스 설정
                    .lineAggregator(LineAggregator<T>) // 객체를 String으로 변환하는 LineAggregator 객체 설정
                    .append(boolean) // 존재하는 파일에 내용을 추가할 것인지 여부 설정
                    .fieldExtractor(FieldExtractor<T>) // 객체 필드를 추출해서 배열로 만드는 FieldExtractor 설정
                    .headerCallback(FlatFileHeaderCallback) // 헤더를 파일에 쓰기 위한 콜백 인터페이스
                    .footerCallback(FlatFileFooterCallback) // 푸터를 파일에 쓰기 위한 콜백 인터페이스
                    .shouldDeleteIfExists(boolean) // 파일이 이미 존재한다면 삭제
                    .shouldDeleteIfEmpty(boolean) // 파일의 내용이 비어 있다면 삭제
                    .delimited().delimiter(String delimiter) // 파일의 구분자를 기준으로 파일을 작성하도록 설정
                    .formatted().format(String format) // 파일의 고정길이를 기준으로 파일을 작성하도록 설정
                    .build();
    }
    ```

4. 예제
    - DelimitedLineAggregator
        1. 기본 개념
            - 객체의 필드 사이에 구분자를 삽입해서 한 문자열로 변환한다.

        2. 구조
            - ExtractorLineAggregator
                - `FieldExtractor<T>` fieldExtractor : 객체를 인자로 받고 배열 반환
                - abstract String doAggregate(Object[] fields) : 배열 필드를 문자열로 만들어서 반환
            - DelimitedLineAggregator
                - String delimiter : 구분자, 기본값은 콤마(,)
                - String doAggregate(Object[] fields) : 배열 필드를 구분자로 구분해서 문자열을 만든 후 반환

    - `git checkout Part6.2.1.1`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlatFilesDelimitedWriteConfiguration {

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
                    .<String,String>chunk(1)
                    .reader(customItemReader())
                    .writer(customItemWriter())
                    .build();
        }

        @Bean
        public ListItemReader customItemReader() {

            List<Customer> customers = Arrays.asList(new Customer(1, "hong gil dong1", 41),
                    new Customer(2, "hong gil dong2", 42),
                    new Customer(3, "hong gil dong3", 43));

            ListItemReader<Customer> reader = new ListItemReader<>(customers); // customers 대신 Collections.emptyList()
            return reader;
        }

        @Bean
        public FlatFileItemWriter<Customer> customItemWriter() throws Exception {
            return new FlatFileItemWriterBuilder<Customer>()
                    .name("customerWriter")
                    .resource(new FileSystemResource("C:\\jsw\\inflearn\\spring-batch-lecture\\src\\main\\resources\\customer.csv")) // customer.txt
                    .append(true)
                    .delimited()
                    .delimiter(",")
                    .names(new String[] {"id", "name", "age"})
                    .build();
        }

        /*@Bean
        public FlatFileItemWriter<Customer> customItemWriter() throws Exception {

            BeanWrapperFieldExtractor<Customer> fieldExtractor = new BeanWrapperFieldExtractor<>();
            fieldExtractor.setNames(new String[] {"id","name","age"});
            fieldExtractor.afterPropertiesSet();

            DelimitedLineAggregator<Customer> lineAggregator = new DelimitedLineAggregator<>();
            lineAggregator.setDelimiter(",");
            lineAggregator.setFieldExtractor(fieldExtractor);

            return new FlatFileItemWriterBuilder<Customer>()
                    .name("CustomerWriter")
                    .resource(new ClassPathResource("customer.csv"))
                    .lineAggregator(lineAggregator)
                    .build();
        }*/
    }
    ```
    ```java
    @Data
    @AllArgsConstructor
    public class Customer {

        private long id;
        private String name;
        private int age;
    }
    ```

    - 참고
    > ExtractorLineAggregator<T>, DelimitedLineAggregator<T>, BeanWrapperFieldExtractor<T>

    - FormatterLineAggregator
        1. 기본 개념
            - 객체의 필드를 사용자가 설정한 Formatter 구문을 통해 문자열로 변환한다.
        
        2. 구조
            - FormatterLineAggregator
                - String format : 포맷 설정
                - int maximumLength = 0 : 최대 길이 설정
                - int minimumLength = 0 : 최소 길이 설정
    
    - `git checkout Part6.2.1.2`
    ```java
    @RequiredArgsConstructor
    @Configuration
    public class FlatFilesFormattedConfiguration {

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
        public ListItemReader customItemReader() {

            List<Customer> customers = Arrays.asList(new Customer(1, "hong gil dong1", 41),
                    new Customer(2, "hong gil dong2", 42),
                    new Customer(3, "hong gil dong3", 43));

            ListItemReader<Customer> reader = new ListItemReader<>(customers);
            return reader;
        }

        @Bean
        public FlatFileItemWriter<Customer> customItemWriter() throws Exception {
            return new FlatFileItemWriterBuilder<Customer>()
                    .name("customerWriter")
                    .resource(new ClassPathResource("customer.csv")) // customer.txt
        //            .append(true)
                    .formatted()
                    .format("%-2d%-15s%-2d") // 길이 및 타입
                    .names(new String[] {"id", "name", "age"})
                    .build();
        }
    }
    ```

    - 참고
    > ExtractorLineAggregator<T>, FormatterLineAggregator<T>, BeanWrapperFieldExtractor<T>

## XML
### StaxEventItemWriter
1. 기본 개념
    - XML 쓰는 과정은 읽기 과정에 대칭적이다.
    - StaxEventItemWriter는 Resource, marshaller, rootTagName가 필요하다.

2. API
    ```java
    public StaxEventItemReader itemReader() {
        return new StaxEventItemWriterBuilder<T>()
                    .name(String name)
                    .resource(Resource) // 쓰기할 리소스 설정
                    .rootTagName() // 조각 단위의 루트가 될 이름 설정
                    .overwriterOutput(boolean) // 파일이 존재하면 덮어 쓸 것인지 설정
                    .marshaller(Marhshaller) // Marshaller 객체 설정
                    .headerCallback() // 헤더를 파일에 쓰기 위한 콜백 설정
                    .footerCallback() // 푸터를 파일에 쓰기 위한 콜백 설정
                    .build();
    }
    ```

3. 구조
    - Step -> write(`List<Item>`) -> StaxEventItemWriter -> marshall(Item) -> XStreamMArshaller -> Trade -> XMLEventWriter
    - Marshaller : XMLEventWriter를 사용해서 StartDocument와 EndDocument 이벤트를 필터링해서 Resource에 쓴다.
    - Resource : 작성할 파일을 나타내는 스프링 Resource
    - Root Element Name : 루트 엘리먼트 이름
    - 이미 존재하는 파일을 덮어쓸 것이지 설정
    - XStreamMarshaller
        - StaxEventItemReader와 동일한 설정이다.
        - 맵으로 alias 지정할 수 있다.
        - 첫 번째 키는 조각의 루트 엘리먼트, 값은 바인딩할 객체 타입
        - 두 번째부터는 하위 엘리먼트와 각 클래스 타입

- `git checkout Part6.2.2`
```java
@RequiredArgsConstructor
@Configuration
public class XMLConfiguration {

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
                .<Customer, Customer>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> customItemReader() {

        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setFetchSize(10);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where firstname like :firstname");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);
        reader.setQueryProvider(queryProvider);

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "A%");

        reader.setParameterValues(parameters);

        return reader;
    }

    @Bean
    public StaxEventItemWriter customItemWriter() {
        return new StaxEventItemWriterBuilder<Customer>()
                .name("customersWriter")
                .marshaller(itemMarshaller())
                .resource(new FileSystemResource("customer.xml"))
                .rootTagName("customer")
                .overwriteOutput(true)
                .build();

    }

    @Bean
    public XStreamMarshaller itemMarshaller() {
        Map<String, Class<?>> aliases = new HashMap<>();
        aliases.put("customer", Customer.class);
        aliases.put("id", Long.class);
        aliases.put("firstName", String.class);
        aliases.put("lastName", String.class);
        aliases.put("birthdate", Date.class);
        XStreamMarshaller xStreamMarshaller = new XStreamMarshaller();
        xStreamMarshaller.setAliases(aliases);
        return xStreamMarshaller;
    }
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

- 참고
> StaxEventItemWriter, AbstractMarshaller

## Json
### JsonFileItemWriter
1. 기본 개념
    - 객체를 받아 JSON String으로 변환하는 역할을 한다.

2. API
    ```java
    public JsonFileItemWriterBuilder itemReader() {
        return new JsonFileItemWriterBuilder<T> ()
                    .name(String name)
                    .resource(Resource) // 쓰기할 리소스 설정
                    .append(boolean) // 존재하는 파일에 내용을 추가할 것인지 여부 설정
                    .jsonObjectMarshaller(JsonObjectMarshaller) // JsonObjectMarshaller 객체 설정
                    .headerCallback(FlatFileHeaderCallback) // 헤더를 파일에 쓰기 위한 콜백 인터페이스
                    .footerCallback(FlatFileFooterCallback) // 푸터를 파일에 쓰기 위한 콜백 인터페이스
                    .shouldDeleteIfExists(boolean) // 파일이 이미 존재한다면 삭제
                    .shouldDeleteIfEmpty(boolean) // 파일의 내용이 비어 있다면 삭제
                    .build();
    }
    ```
3. 구조
    - Step -> write(`List<Item>`) -> JsonFileItemWriter -> marshal(Item) -> JacksonJsonObjectMarshaller -> ObjectMapper -> writeValueAsString(Item)

- `git checkout Part6.2.3`
```java
@RequiredArgsConstructor
@Configuration
public class JsonConfiguration {

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
                .<Customer, Customer>chunk(10)
                .reader(customItemReader())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> customItemReader() {

        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setFetchSize(10);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where firstname like :firstname");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);
        reader.setQueryProvider(queryProvider);

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "A%");

        reader.setParameterValues(parameters);

        return reader;
    }

    @Bean
    public JsonFileItemWriter<Customer> customItemWriter() { // public ItemWriter<? super Customer> customItemWriter() {
        return new JsonFileItemWriterBuilder<Customer>()
                .jsonObjectMarshaller(new JacksonJsonObjectMarshaller<>())
                .resource(new ClassPathResource("customer.json"))
                .name("customerJsonFileItemWriter")
                .build();
    }
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

## DB
### JdbcBatchItemWriter
1. 기본 개념
    - JdbcCursorItemReader 설정과 마찬가지로 datasource를 지정하고, sql 속성에 실행할 쿼리를 설정
    - JDBC의 Batch 기능을 사용하여 bulk insert/update/delete 방식으로 처리
    - 단건 처리가 아닌 일괄처리이기 때문에 성능에 이점을 가진다.

2. API
    ```java
    public JdbcBatchItemWriter itemWriter() {
        return new JdbcBatchItemWriterBuilder<T> ()
                    .name(String name)
                    .datasource(Datasource) // DB에 접근하기 위해 Datasource 설정
                    .sql(String sql) // ItemWriter가 사용할 쿼리 문장 설정
                    .assertUpdates(boolean) // 트랜잭션 이후 적어도 하나의 항목이 행을 업데이트 혹은 삭제하지 않을 경우 예외 발생 여부를 설정함, 기본값은 true
                    .beanMapped() // Pojo 기반으로 Insert SQL의 Values를 매핑
                    .columnMapped() // Key, Value 기반으로 Insert SQL의 Values를 매핑
                    .build();
    }
    ```

3. 구조
    - Step -> write(`List<Item>`) -> JdbcBatchItemWriter -> map? Map 타입 -> columnMapped() -> ColumnMapItemPreparedStatementSetter -> JdbcTemplate -> batchUpdate(sql) -> DB / map? 일반 클래스 타입 -> beanMapped() -> BeanPropertyItemSqlParameterSourceProvider -> JdbcTemplate -> Insert 문 -> DB
    - ColumnMapped로 설정한 경우 Map 타입의 아이템 배치처리
    - BeanMapped로 설정한 경우 Pojo 타입의 아이템 배치처리

- `git checkout Part6.2.4.1`
```java
@RequiredArgsConstructor
@Configuration
public class JdbcBatchConfiguration {

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
        reader.setFetchSize(100);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
//        queryProvider.setWhereClause("where firstname like :firstname");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);
        reader.setQueryProvider(queryProvider);

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "A%");

//        reader.setParameterValues(parameters);

        return reader;
    }

    @Bean
    public JdbcBatchItemWriter<Customer> customItemWriter() { // public ItemWriter<? super Customer> customItemWriter() {
        return new JdbcBatchItemWriterBuilder<Customer>()
                .dataSource(dataSource)
                .sql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)")
                .beanMapped() // 설정
//                .columnMapped()
                .build();
    }

    /*@Bean
    public JdbcBatchItemWriter<Customer> customItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }*/
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

- 참고
> JdbcBatchItemWriterBuilder, JdbcBatchItemWriter

### JpaItemWriter
1. 기본 개념
    - JPA Entity 기반으로 데이터를 처리하며 EntityManagerFactory를 주입받아 사용한다.
    - Entity를 하나씩 Chunk 크기만큼 Insert 혹은 Merge한 다음 Flush 한다.
    - ItemReader나 ItemProcessor로부터 아이템을 전달받을 때는 Entity 클래스 타입으로 받아야 한다.

2. API
    ```java
    public JpaItemWriter itemWriter() {
        return new JpaItemWriterBuilder<T> ()
                    .usePersist(boolean) // Entity를 persist()할 것인지 여부 설정, false이면 merge() 처리
                    .entityManagerFactory(EntityManagerFactory) // EntityManagerFactory 설정
                    .build();
    }
    ```

3. 구조
    - Step -> write(`List<Item>`) -> JpaItemWriter -> (EntityManager -> usePersist -> YES -> persist / NO -> merge -> flush()) * Chunk Size 반복 -> DB
        - 전달되는 Item 타입은 Entity 클래스가 되어야 한다.
        - EntityManager가 Item을 persist나 merge로 바로 인서트하기 때문이다.

- `git checkout Part6.2.4.2`
```java
@RequiredArgsConstructor
@Configuration
public class JpaConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;
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
                .<Customer, Customer2>chunk(10)
                .reader(customItemReader())
                .processor(customItemProcessor())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JpaItemWriter<Customer2> customItemWriter() {
        return new JpaItemWriterBuilder<Customer2>()
                .entityManagerFactory(entityManagerFactory)
                .usePersist(true)
                .build();
    }

    @Bean
    public ItemProcessor<? super Customer, ? extends Customer2> customItemProcessor() {
        return new CustomItemProcessor();
    }

    @Bean
    public JdbcPagingItemReader<Customer> customItemReader() {

        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setFetchSize(10);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where firstname like :firstname");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);
        reader.setQueryProvider(queryProvider);

        HashMap<String, Object> parameters = new HashMap<>();
        parameters.put("firstname", "C%");

        reader.setParameterValues(parameters);

        return reader;
    }
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getString("birthdate"));
	}
}
```
```java
public class CustomItemProcessor implements ItemProcessor<Customer,Customer2> {

     private ModelMapper modelMapper = new ModelMapper();

    @Override
    public Customer2 process(Customer item) throws Exception {
        Customer2 customer2 = modelMapper.map(item, Customer2.class);

        return customer2;
    }
}
```
```java
@Data
@AllArgsConstructor
public class Customer {

    private long id;
    private String firstName;
    private String lastName;
    private String birthdate;
}
```
```java
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Customer2 {

    @Id
    private long id;
    private String firstName;
    private String lastName;
    private String birthdate;
}
```

- Dependency
> org.modelmapper.modelmapper

- 참고
> JpaItemWriter

## ItemReaderAdapter
1. 기본 개념
    - 배치 Job 안에서 이미 있는 DAO나 다른 서비스를 ItemWriter 안에서 사용하고자 할 때 위임 역할을 한다.

- `git checkout Part6.2.4.3`
```java
@RequiredArgsConstructor
@Configuration
public class ItemWriterAdapterConfiguration {

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
                .reader(new ItemReader<String>() { // 10개 데이터 read

                    int i = 0;

                    @Override
                    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                        i++;
                        return i > 10 ? null : "item" + i;
                    }
                })
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public ItemWriterAdapter customItemWriter() {

        ItemWriterAdapter<String>  writer = new ItemWriterAdapter<>();
         writer.setTargetObject(customService());
         writer.setTargetMethod("joinCustomer");
        return  writer;
    }

    @Bean
    public CustomService customService() {
        return new CustomService();
    }
}
```
```java
public class CustomService<T> {

    public void customWrite(T item){

        System.out.println(item);
    }
}
```

- 참고
> ItemWriterAdapter