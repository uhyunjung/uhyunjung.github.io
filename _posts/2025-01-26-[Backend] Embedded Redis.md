---
title: '[Backend] Embedded Redis'
date: 2025-01-26 15:30:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## Redis 사용 방법
1. Embedded Redis : RedisTemplate 사용, <https://github.com/codemonstur/embedded-redis>
2. Spring Data Redis : JPA Repository처럼 사용

## pom.xml
```xml
<!-- https://mvnrepository.com/artifact/com.github.codemonstur/embedded-redis -->
<dependency>
    <groupId>com.github.codemonstur</groupId>
    <artifactId>embedded-redis</artifactId>
    <version>1.4.3</version>
    <scope>provided</scope> # test 가능
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## application.yml
```yml
spring:
  config:
    activate:
      on-profile:
        - local
  data: # 버전에 따라 data 생략
    redis:
        host: localhost
        port: 6379
```

## EmbeddedRedis
### EmbeddedRedisConfig.java
- RedisServer 시작 방법

1. `@Bean` : 항상 애플리케이션과 함께 시작
```java
@Bean
public RedisServer redisServer() throws IOException {
    RedisServer redisServer = new RedisServer(6379);
    redisServer.start();
    return redisServer;
}
```

2. `@PostConstruct` : 조건에 따라 Redis 서버를 실행, `@Bean`보다 `@PostConstruct` 사용
```java
@PostConstruct
private void start() throws IOException {
    int port = isRedisRunning()? getRandomPort() : redisPort;
    redisServer = new RedisServer(port);
    redisServer.start();
}
```

3. `InitializingBean`, `DisposableBean`
```java
public class EmbeddedRedisConfig implements InitializingBean, DisposableBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        int port = isRedisRunning()? getRandomPort() : redisPort;
        redisServer = new RedisServer(port);
        redisServer.start();
    }

    @Override
    public void destroy() throws IOException {
        if (this.redisServer != null) {
            this.redisServer.stop();
        }
    }
}
```

4. `ApplicationRunner`
```java
public class EmbeddedRedisConfig implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        try {
            int port = isRedisRunning()? getRandomPort() : this.redisPort;
            this.redisServer = new RedisServer(port);
            this.redisServer.start();
        }
        catch (Exception ex) {
            log.error("xxxxxxxxxxxxxxxxxxxxxxxxxx {}", ex);
            // TODO ignore
        }
    }
}
```

5. `CommandLineRunner`
```java
public class EmbeddedRedisConfig implements CommandLineRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        try {
            int port = isRedisRunning()? getRandomPort() : this.redisPort;
            this.redisServer = new RedisServer(port);
            this.redisServer.start();
        }
        catch (Exception ex) {
            log.error("xxxxxxxxxxxxxxxxxxxxxxxxxx {}", ex);
            // TODO ignore
        }
    }
}
```

- Process 실행 방법

1. `Runtime.exec()`
```java
    return Runtime.getRuntime().exec(shell);
```

2. `ProcessBuilder`, `ProcessHandle` : `processBuilder.redirectErrorStream(true)` : 표준 출력 스트림과 표준 오류 스트림 합침
```java
ProcessBuilder processBuilder = new ProcessBuilder(shell); // ProcessBuilder
processBuilder.redirectErrorStream(true);
Process process = processBuilder.start();
ProcessHandle processHandle = process.toHandle(); // ProcessHandle
System.out.println("Process ID: " + processHandle.pid());
process.onExit().thenRun(() -> System.out.println("Process finished"));
```

- EmbeddredRedisConfig.java

```java
import redis.embedded.RedisServer;

@Slf4j
@Profile({"local", "default", "test"})
@Configuration
public class EmbeddedRedisConfig implements ApplicationRunner {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    private RedisServer redisServer;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try {
            if (isArmArchitecture()) { // ARM <-> x86
                Objects.requireNonNull(getRedisServerExecutable());
            }

            int port = isRedisRunning()? getRandomPort() : this.redisPort;

            // this.redisServer = RedisServer.newRedisServer()
            // .port(port)
            // .setting("bind 127.0.0.1")
            // .slaveOf("localhost", 6378)
            // .setting("daemonize no")
            // .setting("appendonly no")
            // .setting("maxmemory 128M")
            // .build();

            this.redisServer = new RedisServer(port);

            if (!isRedisRunning()) {
                this.redisServer.start();
            }
        }
        catch (Exception ex) {
            log.error("xxxxxxxxxxxxxxxxxxxxxxxxxx {}", ex);
            // TODO ignore
        }
    }
    
    @PreDestroy
    private void stop() throws IOException {
        if (this.redisServer != null) {
            this.redisServer.stop();
        }
    }

    private boolean isRedisRunning() throws IOException {
        return isRunning(executeGrepProcessCommand(redisPort));
    }

    private Process executeGrepProcessCommand(int port) throws IOException {
        String os = System.getProperty("os.name").toLowerCase();

        String [] shell;
        if (os.contains("win")) {
            String command = String.format("netstat -an | findstr LISTENING | findstr :%d", port); // Windows
            shell = new String[] {"cmd", "/c", command};
        } else {
            String command = String.format("netstat -nat | grep LISTEN | grep %d", port); // Linux
            shell = new String[] {"/bin/sh", "-c", command};
        }

        ProcessBuilder processBuilder = new ProcessBuilder(shell);
        processBuilder.redirectErrorStream(true);
        Process process = processBuilder.start();

        return process;
    }

    private boolean isRunning(Process process) {
        String line;
        StringBuilder pidInfo = new StringBuilder();

        try (BufferedReader input = new BufferedReader(new InputStreamReader(process.getInputStream()))) {
            while ((line = input.readLine()) != null) {
                pidInfo.append(line);
            }
        } catch (Exception ignored) {
            // TODO ignore
        }
        return StringUtils.hasLength(pidInfo.toString());
    }

    private int getRandomPort() {
        ServerSocket server = null;

        try {
            server = new ServerSocket(0);
            server.setReuseAddress(true);
            return server.getLocalPort();
        } catch (IOException e) {
            throw new Error(e);
        } finally {
            if (server != null) {
                try {
                    server.close();
                } catch (IOException ignore) {
                    // TODO ignore
                }
            }
        }
    }

    private boolean isArmArchitecture() {
        return System.getProperty("os.arch").contains("aarch64");
    }

    private File getRedisServerExecutable() throws IOException {
        try {
            //return  new ClassPathResource("binary/redis/redis-server-linux-arm64-arc").getFile();
            return new File("src/main/resources/binary/redis/redis-server-linux-arm64-arc");
        } catch (Exception e) {
            throw new IOException("Redis Server Executable not found");
        }
    }

}
```

### RedisConfig.java
- 에러
```bash
Expected :RedisTest.CustomObject(value=value)
Actual   :{"value"="value"}
java.lang.IllegalArgumentException: Cannot construct instance of `CustomObject` (although at least one Creator exists): cannot deserialize from Object value (no delegate- or property-based Creator)
java.lang.ClassCastException: class CustomObject cannot be cast to class java.lang.String (CustomObject is in unnamed module of loader org.springframework.boot.devtools.restart.classloader.RestartClassLoader @39303dfc; java.lang.String is in module java.base of loader 'bootstrap')
java.lang.ClassCastException: class java.lang.Object cannot be cast to class java.lang.String (java.lang.Object and java.lang.String are in module java.base of loader 'bootstrap')
java.lang.ClassCastException: class java.util.LinkedHashMap cannot be cast to class CustomObject (java.util.LinkedHashMap is in module java.base of loader 'bootstrap'; CustomObject is in unnamed module of loader 'app')
```

- 해결 방법
    1. get할 때 `objectMapper.writeValueAsstring, objectMapper.convertValue` 사용
    2. config에 `objectMapper.activateDefaultTyping(validator, ObjectMapper.DefaultTyping.NON_FINAL);` 사용(권장)
    3. profile에 `@Profile({ "local, default, test" })`, `@Profile({ "local|default|test" })` 사용 불가능

- RedisConfig.java

```java
@Profile({ "local", "default", "test" })
@RequiredArgsConstructor
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() { // Connection 필수 (EmbeddedRedisConfig.java에 추가 가능)
        return new LettuceConnectionFactory(this.redisHost, this.redisPort);
    }

    @Bean
    @Primary
    public RedisService defaultService(RedisTemplate<String, Object> redisTemplate) {
        return new DevRedisService(redisTemplate);
    }

//    @Bean
//    @Primary
//    @ConditionalOnMissingBean(RedisIndexedSessionRepository.class)
//    public RedisIndexedSessionRepository redisSessionRepository(
//            RedisOperations<String, Object> redisConnectionFactory) {
//        return new RedisIndexedSessionRepository(redisConnectionFactory);
//    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<byte[], byte[]> redisTemplate = new RedisTemplate<>();

        PolymorphicTypeValidator validator = BasicPolymorphicTypeValidator.builder()
                .allowIfSubType(Object.class)
                .build();

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.activateDefaultTyping(validator, ObjectMapper.DefaultTyping.NON_FINAL); // 클래스 타입 정보 json에 활성화
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false); // 빈 객체 직렬화 예외 방지
        objectMapper.registerModule(new JavaTimeModule()); // 날짜 설정 필요

        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer(objectMapper);

        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(serializer);
        redisTemplate.setHashValueSerializer(serializer);

        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }

}
```

### RedisService.java
- 의존성 주입 확인 방법
```java
public class RedisService {
    @Autowired
    List<RedisTemplate<?, ?>> list;

    public void check {
        this.list.forEach(x -> {
            log.info("xxxxxxxxxxxxxxxxxxxxxxxx {}", x);
        });
        
        ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        MyService myService = context.getBean(MyService.class);
    }
}
```

- 제너릭 선언
```bash
Unchecked Cast Object to T
```

- RedisService.java

```java
@Service
@Getter
@RequiredArgsConstructor
public class RedisService<T> {

    @Autowired
    private final RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private final StringRedisTemplate stringRedisTemplate;

    @Autowired
    List<RedisTemplate<?, ?>> list;

    public void set(String key, Object value) {
        set(key, value, Duration.ofMinutes(5L));
    }

    public void set(String key, Object value, Duration duration) {
        getRedisTemplate().opsForValue().set(key, value, duration);
    }

    public boolean setIfAbsent(String key, Object value, Long seconds) {
        return Boolean.TRUE.equals(getRedisTemplate().opsForValue().setIfAbsent(key, value, Duration.ofSeconds(seconds)));
    }

    public T get(String key) {
        return (T) getRedisTemplate().opsForValue().get(key);
    }

    public boolean delete(String key) {
        return Boolean.TRUE.equals(getRedisTemplate().delete(key));
    }

}
```
```java
@Service
@RequiredArgsConstructor
public class RedisService<T> {

    @Autowired
    private final RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private final StringRedisTemplate stringRedisTemplate;

    @Autowired
    List<RedisTemplate<?, ?>> list;

    public void set(String key, Object value) {
        set(key, value, Duration.ofMinutes(5L));
    }

    public void set(String key, Object value, Duration duration) {
        this.redisTemplate.opsForValue().set(key, value, duration);
    }

    public boolean setIfAbsent(String key, Object value, Long seconds) {
        return Boolean.TRUE.equals(this.redisTemplate.opsForValue().setIfAbsent(key, value, Duration.ofSeconds(seconds)));
    }

    public T get(String key) {
        return (T) this.redisTemplate.opsForValue().get(key);
    }

    public boolean delete(String key) {
        return Boolean.TRUE.equals(this.redisTemplate.delete(key));
    }

    // 생략 가능
    public void setStringValue(String key, String value) {
        setStringValue(key, value, Duration.ofMinutes(5L));
    }

    public void setStringValue(String key, String value, Duration duration) {
        this.stringRedisTemplate.opsForValue().set(key, value, duration);
    }

    public String getStringValue(String key) {
        return (String) this.stringRedisTemplate.opsForValue().get(key);
    }

    public void setStringExpireTime(Duration duration) {
        setStringValue(key, value, duration);
    }

    public boolean isExists(String key) {
        return Boolean.TRUE.equals(this.redisTemplate.hasKey(key));
    }

}
```

## ConcurrentHashMap
- IntelliJ에서 RedisServer가 실행되지 않고 ClassNotFoundException 발생 -> 로컬에서 ConcurrentHashMap을 대신 Redis처럼 사용하기로 결정

### RedisService.java
```java
public interface RedisService {

    void set(String key, Object value, Duration duration);

    boolean setIfAbsent(String key, Object value, Duration duration);

    <T> T get(String key);

    boolean delete(String key);

}
```

### LocalRedisService.java
```java
//@Profile("!dev & !prod")
//@ConditionalOnProperty(name = "spring.data.redis.host", matchIfMissing = true)
@Service
@RequiredArgsConstructor
public class LocalRedisService implements RedisService {

    private final ConcurrentHashMap<String, Object> map = new ConcurrentHashMap<>();

    public void set(String key, Object value, Duration duration) {
        this.map.put(key, value);
    }

    public boolean setIfAbsent(String key, Object value, Duration duration) {
        if (this.map.containsKey(key)) {
            return false;
        }
        else {
            this.map.putIfAbsent(key, value);
            return true;
        }
    }

    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        return (T) this.map.get(key);
    }

    public boolean delete(String key) {
        if (this.map.containsKey(key)) {
            this.map.remove(key);
            return true;
        }
        else {
            return false;
        }
    }

}
```

### DevRedisService.java
```java
//@Profile({"dev", "prod"})
@Service
@RequiredArgsConstructor
public class DevRedisService implements RedisService {

    private final RedisTemplate<String, Object> redisTemplate;

    public void set(String key, Object value, Duration duration) {
        this.redisTemplate.opsForValue().set(key, value, duration);
    }

    public boolean setIfAbsent(String key, Object value, Duration duration) {
        return Boolean.TRUE.equals(this.redisTemplate.opsForValue().setIfAbsent(key, value, duration));
    }

    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        return (T) this.redisTemplate.opsForValue().get(key);
    }

    public boolean delete(String key) {
        return Boolean.TRUE.equals(this.redisTemplate.delete(key));
    }

}
```

### RedisController.java
```java
@Slf4j
@RestController
@RequestMapping("home")
public class RedisController {

    @Autowired
    private RedisService redisService;

    @GetMapping("")
    public void getRedis() {
        this.redisService.set("key", "value", Duration.ofMinutes(5L));
        log.info("redis={} {}", this.redisService.getClass(), this.redisService.get("key"));
    }

}
```

## Spring Data Redis
### Token.java
```java
@Getter
@Builder
@RedisHash(value = "token", timeToLive = 60)
public class Token {

    @Id
    private String id;

    private String value;

    private LocalDateTime refreshTime;
}
```

### TokenRepository.java
```java
public interface TokenRepository extends CrudRepository<Token, String> {
}
```

## 테스트
### RedisTest.java
```java
@SpringBootTest
@Import({EmbeddedRedisConfig.class, RedisConfig.class})
@ActiveProfiles("test")
@DisplayName("Redis Test")
public class RedisCRUDTest {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @Autowired
    private RedisService redisService;

    @Autowired
    private TokenRepository tokenRepository;

    @AfterEach
    void tearDown() {
        tokenRepository.deleteAll();
    }

    @Test
    public void testEmbeddedRedis() {

        // given
        redisTemplate.opsForValue().set("key", "value");

        // when
        String value = redisTemplate.opsForValue().get("key");

        // then
        assertThat(value).isEqualTo("value");
    }

    @Test
    public void testRedisService() {

        // given
        CustomObject customObject = new CustomObject("value");

        // when
        this.redisService.set("key", customObject, Duration.ofSeconds(1L));
        CustomObject result = this.redisService.get("key");

        // then
        assertThat(result).isEqualTo(customObject);
    }
    
    @Test
    public void selectRedis() {

        //given
        String id = "id";
        String value = "value";
        LocalDateTime refreshTime = LocalDateTime.of(2025, 5, 26, 0, 0);
        Token token = Token.builder()
                .id(id)
                .value(value)
                .refreshTime(refreshTime)
                .build();

        //when
        tokenRepository.save(token);

        //then
        Token savedToken = tokenRepository.findById(id).get();
        assertThat(savedToken.getValue()).isEqualTo(value);
    }

    @Test
    public void updateRedis() {

        //given
        String id = "id";
        LocalDateTime refreshTime = LocalDateTime.of(2018, 5, 26, 0, 0);
        tokenRepository.save(Token.builder()
                .id(id)
                .build());

        //when
        Token savedToken = tokenRepository.findById(id).get();
        //savedToken.refresh(2000L, LocalDateTime.of(2018,6,1,0,0));
        tokenRepository.save(savedToken);

        //then
        Token refreshToken = tokenRepository.findById(id).get();
        //assertThat(refreshToken.getAmount()).isEqualTo(2000L);
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class CustomObject {

        private String value;

    }
}
```

## 참고
- <https://green-bin.tistory.com/77>
- <https://jojoldu.tistory.com/297>
- <https://github.com/ku-kim/springboot-redis-example/>

- <https://github.com/Kugaaa/embedded-redis-resoved-macOS14>
- <https://kyeong8139.tistory.com/70>
- <https://soobindeveloper8.tistory.com/1006>
- <https://velog.io/@kimsei1124/<Spring-Data-Redis-%EB%A1%9C%EC%BB%AC%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0>

- <https://brunch.co.kr/@springboot/212>
- <https://velog.io/@bagt/Redis-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94-%EC%82%BD%EC%A7%88%EA%B8%B0-feat.-RedisSerializer>