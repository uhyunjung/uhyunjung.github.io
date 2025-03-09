---
title: '[Spring] Spring Security 스프링 시큐리티'
date: 2025-01-17 18:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## 초기화 과정
### Security Builder/Security Configurer
- SecurityConfig extends WebSecurityConfigureAdapter
- cors 허용, sessionManagement 정책 설정, csrf 방지, 폼 기반 로그인 인증 허용(formLogin()), http 기반 인증 허용(httpBasic()), ExceptionHandler 생성, authorizeRequest(), antMatchers()

### WebSecurity/HttpSecurity
### DelegatingFilterProxy/FilterChainProxy
- Request -> Serverlet Filter -> doFilter -> Servlet Filter -> doGet -> Servlet -> Response
- DelegatingFilterProxy -> FilterChainProxy -> Spring SecurityFilterChain
- `@Order`

### 사용자 정의 보안 설정
- 필터 동작 순서
1. DisableEncodeUrlFilter.class
2. ForceEagerSessionCreationFilter.class
3. ChannelProcessingFilter.class
4. WebAsyncManagerIntegrationFilter.class
5. **SecurityContextHolderFilter.class**
6. **SecurityContextPersistenceFilter.class**
7. HeaderWriterFilter.class
8. **CorsFilter.class**
9. **CsrfFilter.class**
10. **LogoutFilter.class**
11. X509AuthenticationFilter.class
12. AbstractPreAuthenticatedProcessingFilter.class
13. **UsernamePasswordAuthenticationFilter.class**
14. DefaultLoginPageGeneratingFilter.class
15. DefaultLogoutPageGeneratingFilter.class
16. **ConcurrentSessionFilter.class**
17. DigestAuthenticationFilter.class
18. BasicAuthenticationFilter.class
19. RequestCacheAwareFilter.class
20. SecurityContextHolderAwareRequestFilter.class
21. JaasApiIntegrationFilter.class
22. **RememberMeAuthenticationFilter.class**
23. AnonymousAuthenticationFilter.class
24. SessionManagementFilter.class
25. **ExceptionTranslationFilter.class**
26. **FilterSecurityInterceptor.class**
27. AuthorizationFilter.class
28. SwitchUserFilter.class

## 인증 프로세스
- Http Request -> AuthenticationFilter(-> UsernamePasswordAuthenticationFilter, SecurityContextHolder(SecurityContext - Authentication)) -> AuthenticationManager(ProviderManager) -> AuthenticationProvider -> UserDetailsService(UserDetails - User)

### 폼 인증 - formLogin()
### 폼 인증 필터 - UsernamePasswordAuthenticationFilter
- UsernamePasswordAuthenticationFilter : ID/비밀번호 기반 For 인증 요청을 감시하여 사용자를 인증함
- UsernamePasswordAuthenticationToken : Authentication을 implements한 AbstractAuthenticationToken의 하위 클래스

### 기본 인증 - httpBasic()
### 기본 인증 필터 - BasicAuthenticationFilter
### 기억하기 인증 - rememberMe()
### 기억하기 인증 필터 - RememberMeAuthenticationFilter
### 익명 인증 사용자 - anonymous()
### 로그아웃 - logout()
- LogoutFilter : 로그아웃 URL(/logout)로의 요청을 감시하여 해당 사용자를 로그아웃 시킴

### 요청 캐시 - RequestCache/SavedRequest

## 인증 아키텍처
- AbstractAuthenticationProcessingFilter -> AuthenticationManager -> AuthenticationProvider -> Authentication Credential Storage

### 인증 – Authentication
### 인증 컨텍스트 - SecurityContext / SecurityContextHolder
- SecurityContextPersistenceFilter : SecurityContextRepository를 통해 SecurityContext를 Load/Save 처리

### 인증 관리자 - AuthenticationManager
### 인증 제공자 - AuthenticationProvider
### 사용자 상세 서비스 - UserDetailsService
- PasswordEncoder
- GrantedAuthority

### 사용자 상세 - UserDetails

## 인증 상태 영속성
### SecurityContextRepository / SecurityContextHolderFilter
### 스프링 MVC 로그인 구현

## 세션 관리
### 동시 세션 제어 - sessionManagement().maximumSessions()
### 세션 고정 보호 - sessionManagement().sessionFixation()
### 세션 생성 정책 - sessionManagement().sessionCreationPolicy()
### SessionManagementFilter / ConcurrentSessionFilter

## 예외 처리
### 예외 처리 - exceptionHandling()
### 예외 필터 - ExceptionTranslationFilter
- ExceptionTranslationFilter : 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달

## 악용 보호
### CORS (Cross Origin Resource Sharing)
### CSRF (Cross Site Request Forgery)
### CSRF 토큰 유지 및 검증
### CSRF 통합
### SameSite

## 인가 프로세스
- FilterSecurityInterceptor : 접근 권한 확인을 위해 요청을 AccessDecisionManager로 위임. 이 필터가 실행되는 시점에는 사용자가 인증됐다고 판단.
- DynamicSecurityFilter : 동적 권한
- SecurityMetadataSource : 사용자가 요청한 자원에 필요한 권한 정보를 조회해서 전달
    - DefaultFilterInvocationSecurityMetadataSource
    - FilterInvocationSecurityMetadataSource : UrlFilterInvocationSecurityMetadataSource 구현체
    - MethodSecurityMetadataSource
    - UrlResourcesMapFactoryBean
- AccessDecisionManager

### 요청 기반 권한 부여 - HttpSecurity.authorizeHttpRequests()
### 표현식 및 커스텀 권한 구현
### 요청 기반 권한 부여 - HttpSecurity.securityMatcher()
### 메서드 기반 권한 부여 - @PreAuthorize, @PostAuthorize
- Custom Expression : `@Service`, AccessControlService(ACL)

### 메서드 기반 권한 부여 - @PreFilter, @PostFilter
### 메서드 기반 권한 부여 - @Secured, JSR-250 및 부가 기능
### 정적 자원 관리
### 계층적 권한 - RoleHirerachy

## 인가 아키텍처
### 인가 – Authorization
### 인가 관리자 이해 - AuthorizationManager
### 요청 기반 인가 관리자 - AuthorityAuthorizationManager 외 클래스 구조 이해
### 요청 기반 Custom AuthorizationManager 구현
### RequestMatcherDelegatingAuthorizationManager 로 인가 설정 응용하기
### 메서드 기반 인가 관리자 - PreAuthorizeAuthorizationManager 외 클래스 구조 이해
### 메서드 기반 Custom AuthorizationManager 구현
### 포인트 컷 메서드 보안 구현하기 - AspectJExpressionPointcut / ComposablePointcut
### AOP 메서드 보안 구현하기 - MethodInterceptor, Pointcut, Advisor

## 이벤트 처리
### 인증 이벤트 - Authentication Events
### 인증 이벤트 - AuthenticationEventPublisher 활용
### 인가 이벤트 - Authorization Events

## 통합하기
### Servlet API 통합 - SecurityContextHolderAwareRequestFilter
### Spring MVC 통합 - @AuthenticationPrincipal
### Spring MVC 비동기 통합 - WebAsyncManagerIntegrationFilter

## 고급설정
### 다중 보안 설정
### Custom DSLs - HttpSecurity.with(AbstractHttpConfigurer)
### Redis 를 활용한 이중화 설정 - @EnableRedisHttpSession

## 회원 인증 시스템
### 회원가입 / PasswordEncoder
### 커스텀 UserDetailService 구현하기
### 커스텀 AuthenticationProvider 구현하기
### 커스텀 로그아웃
### 커스텀 인증상세 구현하기 - WebAuthenticationDetails / AuthenticationDetailsSource
### 커스텀 인증성공 핸들러 - AuthenticationSuccessHandler
### 커스텀 인증실패 핸들러 - AuthenticationFailureHandler
### 커스텀 접근 제한하기 - AccessDeniedHandler

## 비동기 인증
### Rest 인증 보안 및 화면 구성
### Rest 인증 필터 구현 - RestAuthenticationFilter
### RestAuthenticationProvider 구현하기
### Rest 인증 성공 / 실패 핸들러
### Rest 인증 상태 영속하기 - SecurityContextRepository 설정하기
### Rest 예외 처리 - RestAuthenticationEntryPoint / RestAccessDeniedHandler
### Rest 로그아웃 구현
### Rest CSRF 구현
### Rest DSLs 구현

## 회원 관리 시스템
### 회원 관리 시스템 기본 구성
### 프로그래밍 방식의 인가 구현 - 메모리 기반
### 프로그래밍 방식의 인가 구현 – DB 연동
### 인가 설정 실시간 반영하기
### 계층적 권한 적용하기 - RoleHierarchy
### 프로그래밍의 인가구현 - 필터에 의한 DB 연동

## OAuth2.0
## JWT