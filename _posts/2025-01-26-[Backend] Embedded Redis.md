---
title: '[Backend] Embedded Redis'
date: 2025-01-26 15:30:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## Dependency
- Embedded Redis : <https://github.com/codemonstur/embedded-redis>
- Spring Data Redis

## pom.xml
```xml
<!-- https://mvnrepository.com/artifact/com.github.codemonstur/embedded-redis -->
<dependency>
    <groupId>com.github.codemonstur</groupId>
    <artifactId>embedded-redis</artifactId>
    <version>1.4.3</version>
    <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>3.4.0</version>
</dependency>
```

## application.yml
```yml
spring:
  config:
    activate:
      on-profile:
        - local
  redis:
    host: localhost
    port: 6379
```

## test 패키지
### EmbeddedRedisConfig.java
```java
import redis.embedded.RedisServer;

@Profile("local")
@Configuration
public class EmbeddedRedisConfig {

    @Value("${spring.redis.port}")
    private int redisPort;

    private RedisServer redisServer;

    @PostConstruct
    private void start() throws IOException {
        int port = isRedisRunning() ? getRandomPort() : redisPort;
        redisServer = new RedisServer(port);// x86, ARM
        redisServer.start();
    }

    @PreDestroy
    private void stop() throws IOException {
        if (redisServer != null) {
            redisServer.stop();
        }
    }

    private boolean isRedisRunning() throws IOException {
        return isRunning(executeGrepProcessCommand(redisPort));
    }

    private Process executeGrepProcessCommand(int port) throws IOException {
        String command = String.format("netstat -an | findstr LISTENING | findstr :%d", port); // Windows
        String[] shell = {"cmd", "/c", command};

//        String command = String.format("netstat -nat | grep LISTEN | grep %d", port); // Linux
//        String[] shell = {"/bin/sh", "-c", command};

        return Runtime.getRuntime().exec(shell);
    }

    private boolean isRunning(Process process) {
        String line;
        StringBuilder pidInfo = new StringBuilder();

        try (BufferedReader input = new BufferedReader(new InputStreamReader(process.getInputStream()))) {
            while ((line = input.readLine()) != null) {
                pidInfo.append(line);
            }
        } catch (Exception ignored) {
            // ignore
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
                    // ignore
                }
            }
        }
    }
}
```

- EmbeddedRedisConfig.java에서 RedisServer 시작 방법
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
        int port = isRedisRunning() ? getRandomPort() : redisPort;
        redisServer = new RedisServer(port);
        redisServer.start();
    }
```

### RedisCRUDTest.java
```java
@SpringBootTest
@Import({EmbeddedRedisConfig.class})
@ActiveProfiles("local")
@DisplayName("Redis CRUD Test")
public class RedisCRUDTest {

    @Autowired
    private StringRedisTemplate redisTemplate;

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
}
```

## main 패키지
### RedisRepositoryConfig.java
```java
@Configuration
@EnableRedisRepositories
public class RedisRepositoryConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setEnableTransactionSupport(true);
        return redisTemplate;
    }
}
```

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

## 참고
- <https://green-bin.tistory.com/77>
- <https://jojoldu.tistory.com/297>
- <https://github.com/ku-kim/springboot-redis-example/>

- <https://github.com/Kugaaa/embedded-redis-resoved-macOS14>
- <https://kyeong8139.tistory.com/70>
- <https://soobindeveloper8.tistory.com/1006>
- <https://velog.io/@kimsei1124/<Spring-Data-Redis-%EB%A1%9C%EC%BB%AC%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0>