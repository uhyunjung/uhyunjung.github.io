---
title: '[Backend] Spring Annotation 스프링 어노테이션'
date: 2025-02-05 08:10:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## Annotation
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
- `@RestController`
- `@Controller`
- `@Tag`
- `@PreAuthorize`

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
- `@Primary`