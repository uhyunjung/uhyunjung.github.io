---
title: '[Backend] Spring Annotation 스프링 어노테이션'
date: 2025-02-05 08:10:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## Annotation
## Main
- `@SpringBootApplication(exclude = Configuration.class)`
- `@EnableWebSecurity(debug = false)`
- `@EnableMethodSecurity(securedEnabled = true, prePostEnabled = true, jsr250Enabled = true)`
- `@ComponentScan(basePackages = { "net.demo", "io.demo" })`

## Mapper
- """ 엔터 포함
- `@Param`
- `@Data` : 내부 DTO

## Entity
- `@Entity`
- `@NoArgsConstructor`
- `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- `@RequiredArgs`

- toDto 함수

## DTO 
- `@Builder`
- `@SuperBuilder(builderMethodName = "of")`
- `@DateTimeFormat`
- `@Schema`
- `@Schema(required = false)`X

- private 변수
- final 변수
- toEntity 함수
- null 체크

## Controller
- `@RestController` : JSON/XML 데이터를 반환하는 REST API, 자동으로 @ResponseBody 적용됨
- `@Controller` : HTML 뷰를 반환하는 전통적인 MVC 처리, @ResponseBody를 사용해야 응답 본문 반환, 서버에서 렌더링된 페이지를 반환해야 할 때
- `@Tag`
- `@PreAuthorize(hasRole="adminx" & hasPermistion="/abc")`
- `@Request("/abc.")`

- Get
	- `@Operation`
	- `@ParameterObject`
- Post
	- DTO 전달
	- `@RequestBody` : Swagger 불편, JSON.Stringify
	- `@PameterObject` : Swagger 편의, URLSearchParams, null 문자열로 DB에 저장
	- `@RequestParam(required = false)` : URLSearchParams
	- `@PathVariable("id")` : 필수

## Service
- Controller <-> Service : DTO 전달
- Service <-> Repository : Entity 전달
- `@Autowired`
- `@Transactional(rollbackFor = Throwable.class, propagation = Propagation.REQUIRED)` identity support

## 기타
- `@MappedSuperclass`
- `@EventListener`
- `@EntityListeners(AuditingEntityListener.class)`
- `@JsonIgnore`
- `@JsonIgnoreProperties(value = { "modifiedAt" }, allowGetters = true)`
- `@PostLoad`
- `@PrePersist`
- `@PreUpdate`
- `@Transient`
- `@Builder.Default`
- `@EnableJdbcHttpSession(cleanupCron = "0 */10 * * * *")` : 매 10분마다 clean
- `@CacheConfig`
- `@Value("${server.servlet.session.gtimeout:120m}")`
- `@QueryHint`
- `@Aspect`
- `@Around("within() && execution(public * *(..))")`
- `@Qualifier("jasyptSupportEncryptor")`
- `@Bean`
- `@Order`
- `@Profile`
- `@Import`
- `@Primary` : 의존성 주입 순서
- `@ConditionalOnProperty(name = "spring.data.redis.host", havingValue = "localhost", matchIfMissing=true)` : matchIfMissing은 property가 설정되지 않아도 true이면 Bean 생성, false면 Bean 생성하지 않음
- `@ConditionalOnMissingBean(RedisService.class)` : Bean이 존재하지 않을 때만 해당 Bean을 생성

## 의존성 주입
```
List<RedisService> list;

public void set() {
	list.foreach(x -> x.isYour().set()'');
	list.getFirst();
}
```