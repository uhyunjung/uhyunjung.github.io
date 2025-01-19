---
title: '[Spring] Test 가이드'
date: 2024-12-17 08:19:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## 통합 테스트
### 라이브러리
- **JUnit 5** : 자바 애플리케이션 단위 테스트 지원
- **Spring Test, Spring Boot Test** : 스프링 부트 애플리케이션에 대한 유틸리티, 통합 테스트 지원
- **AssertJ** : 단정문(assert) 지원의 라이브러리
- **Mockito** : 자바 Mock 객체 지원의 프레임워크

### 어노테이션
- `@Test`
- `@SpringBootTest` : Spring Boot, IntegrationTest

- import 주의

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.transaction.Transactional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
```

```java
@SpringBootTest
@AutoConfigureMockMvc
public class IntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @BeforeEach
    void setUp(){

    }

    @Test
    @DisplayName("통합 테스트")
    @Transactional
    void applyTest() throws Exception {

        // given
        ApplyDto ApplyDto = ApplyDto.from()
                .visitType("유형")
                .Code("코드")
                .visitorName("이름")
                .PurposeContent("목적")
                .build();

        ObjectMapper objectMapper = new ObjectMapper();
        String param = objectMapper.writeValueAsString(ApplyDto);

        // when
        ResultActions resultActions = mockMvc.perform(post("/api/apply/")
                        .contentType(APPLICATION_JSON)
                        .content(param));

        // then
        resultActions
                .andExpect(status().isFound())
                .andDo(print());
    }

}
```

## 단위 테스트
- 서버를 시작하지 않고 아래 계층들 테스트
- Spring Application Context 없이 테스트
- given-when-then 패턴 : 조건-행위(실행)-결과(검증)

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.when;
```

### Controller Test
- MVC 테스트
- 테스트 우선순위 낮음
- `@WebMvcTest` : MockApiTest
- `@AutoConfigureMockMvc` : `@Autowired`로 MockMvc 주입
- `@MockBean` : Spring Boot
- `@MockMvc`: SpringTest, mockMvc.perform()
- `mvc.perform(MockMvcRequestBuilders.get(URI).contentType().andDo(MockMvcResultHanlder.print()).andExpect(MockMvcResultMatcher.status().isOk())`

### Service Test
- Mock 테스트
- `@ExtendWith(MockitoExtension.class)`
- `@Mock` : Mockito.mock(RepositoryClass)
- `@InjectMocks` : Mockito.mock(ServiceClass)
- `Mockito.when().thenReturn()` : given 해당

- 주의사항

```
Examples of correct usage of @InjectMocks:
   @InjectMocks Service service = new Service();
   @InjectMocks Service service;
```

```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {

    @Mock
    private CommandService CommandService;

    @Mock
    private ApplyMapper applyMapper;

    @Mock
    private VisitRepository visitRepository;

    @Mock
    private VisitorRepository visitorRepository;

    @Mock
    private ApiService ApiService;

    @Mock
    private MessageAlertService messageAlertService;

    @InjectMocks
    private ServiceImpl ServiceImpl;

    @Test
    @DisplayName("신청")
    void saveVisitTest() {
        // given
        ApplyDto ApplyDto = ApplyDto.from()
                .visitType("유형")
                .AndTime(LocalDateTime.now())
                .Code("코드")
                .visitorName("이름")
                .PurposeContent("목적")
                .build();

        when(CommandService.saveVisit(any(Visit.class))).thenAnswer(invocation -> invocation.getArgument(0));
        when(applyMapper.getLastRcptNo(anyString(), anyString())).thenReturn("001");
        when(messageAlertService.sendApplyTalk(any(MessageAlertDto.class))).thenReturn(null);
        when(ApiService.saveVisit(ApplyDto)).thenAnswer(invocation -> ServiceImpl.saveVisit(ApplyDto));

        // when
        Visit result = ApiService.saveVisit(ApplyDto);

        // then
        assertThat(result).isNotNull();
        assertThat(result.getPurposeContent()).isEqualTo(ApplyDto.getPurposeContent());
    }

    @Test
    @DisplayName("수정")
    void persistTest() {
        // given
        Optional<Visit> oVisit = Optional.ofNullable(Visit.of()
                .AndTime(LocalDateTime.now().plusDays(10))
                .Code("코드")
                .visitorBelongName("이름")
                .PurposeContent("목적")
                .build());

        Optional<Visitor> oVisitor = Optional.ofNullable(Visitor.of()
                .representativeOrNo("Y")
                .VisitorName("이름")
                .visitorBirthDate("생일")
                .build());

        when(visitRepository.findByRegistrationNumber(any())).thenReturn(oVisit);
        when(visitorRepository.findById(any())).thenReturn(oVisitor);
        when(CommandService.persisVisit(any(Visit.class))).thenAnswer(invocation -> invocation.getArgument(0));

        // when
        VisitorDto VisitorDto = VisitorDto.from()
                .cud(CUD.valueOf("U"))
                .representativeOrNo("Y")
                .VisitorName("이름")
                .visitorAddress("주소")
                .visitorBirthDate("생일")
                .visitorNumber("번호")
                .build();
        List<VisitorDto> VisitorDtoList = new ArrayList<>();
        VisitorDtoList.add(VisitorDto);

        ApplyUpdateDto ApplyUpdateDto = ApplyUpdateDto.from()
                .visitType("유형")
                .AndTime(LocalDateTime.now())
                .Code("코드")
                .visitorBelongName("이름")
                .PurposeContent("목적")
                .visitors(VisitorDtoList)
                .build();
        ApplyUpdateDto.setFormNo("분류+날짜");

        Optional<Visit> result = ApiService.persist(ApplyUpdateDto);

        // then
        assertThat(result).isNotNull();
        assertThat(result).isNotEmpty();
        assertThat(result.get().getCode()).isEqualTo("코드");
    }

}
```

```java
@ExtendWith(MockitoExtension.class)
class CommandServiceTest {

    @Mock
    private VisitRepository visitRepository;

    @Mock
    private VisitorRepository visitorRepository;

    @InjectMocks
    private CommandService CommandService;

    @Test
    void saveVisitTest() {
        // given
        Visit Visit = Visit.of()
                .id(1L)
                .StatusCode(C"상태")
                .DateAndTime(LocalDateTime.now())
                .RegistrationNumber("분류"+"날짜")
                .TypeCode("유형")
                .AndTime(LocalDateTime.now())
                .Code("코드")
                .visitorName("이름")
                .visitorTelephoneNumber("010-0000-0000")
                .PurposeContent("목적")
                .build();

        when(visitRepository.saveAndFlush(Visit)).thenReturn(Visit);
        when(visitRepository.findById(1L)).thenReturn(Optional.of(Visit));

        // when
        CommandService.saveVisit(Visit);
        Optional<Visit> result = visitRepository.findById(Visit.getId());

        // then
        assertThat(result.isPresent()).isTrue();
        assertThat(result.get().getCode()).isEqualTo(Visit.getCode());
    }

    @Test
    void persisVisitTest() {
        // given
        Visit Visit = Visit.of()
                .id(1L)
                .build();

        when(visitorRepository.findMaxSerialNumberById(Visit.getId())).thenReturn(Optional.of(1));
        when(visitorRepository.findRepresentativeSerialNumberById(Visit.getId())).thenReturn(1);
        
        // when
        Visit result = CommandService.persisVisit(Visit);

        // then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(Visit.getId());
    }

}
```

### Repository Test
- MyBatis 테스트 : 테스트 우선순위 낮음
- JPA 테스트
- `@DataJpaTest` : `@Transactional` 포함, 기본값 임베디드 데이터베이스 사용

```java
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
```

```java
@DataJpaTest
class VisitRepositoryTest {

    @Autowired
    private VisitRepository visitRepository;

    @Test
    void findByRegistrationNumber() {
        // given
        Visit Visit = Visit.of()
                .id(1L)
                .StatusCode("상태")
                .DateAndTime(LocalDateTime.now())
                .RegistrationNumber("분류"+"날짜")
                .TypeCode("유형")
                .AndTime(LocalDateTime.now())
                .Code("코드")
                .visitorName("이름")
                .visitorTelephoneNumber("010-0000-0000")
                .PurposeContent("목적")
                .build();

        // when
        visitRepository.saveAndFlush(Visit);
        Optional<Visit> result = visitRepository.findById(Visit.getId());

        // then
        assertThat(result.isPresent()).isTrue();
        assertThat(result.get().getCode()).isEqualTo(Visit.getCode());
    }

}
```

### Entity Test
- Domain 테스트
- POJO 테스트

## 세부 모듈
- JUnit5
    - JUnit Platform
    - JUnit Jupiter
    - JUnit Vintage
- assertJ
    - `assertThat(actual, expected)`

## CRUD
- Read는 다른 CUD 테스트에 포함되어 있을 수 있음

## Jacoco 테스트 커버리지

### 참고
- <https://medium.com/simform-engineering/testing-spring-boot-applications-best-practices-and-frameworks-6294e1068516>
- <https://spring.io/guides/gs/testing-web>
- <https://github.com/spring-guides/gs-testing-web>
- <https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/html/boot-features-testing.html>
- <https://cheese10yun.github.io/spring-guide-test-1/>
- <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md>
- <https://mangkyu.tistory.com/184>
- <https://github.com/rahul-baghaniya/spring-unit-tests/tree/master/src/test/java/com/baghaniya/springunittests>
- <https://brunch.co.kr/@springboot/418>
- <https://velog.io/@beomsun/Service계층-단위테스트>