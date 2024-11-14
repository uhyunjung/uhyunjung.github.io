---
title: '[Spring] Spring Batch 10. 마무리 정리'
date: 2024-11-14 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 강좌 마무리
1. **스프링 배치 기초 및 기본 이해(Fundamental 중요)**
    - 도메인 이해 : JobInstance, JobExecution, StepExecution, ExecutionContext, JobParameter, JobRepository, JobLauncher
    - Job 구성 및 API 활용 : Job, Step, Flow, Tasklet
    - Chunk 프로세스 : Chunk, ItemReader, ItemProcessor, ItemWriter
    - 반복 및 내결함성 : Repeat, Skip, Retry, Listener
    - 이벤트 리스너 : JobExecutionListener, StepListener, RetryListener, SkipListener
    - 멀티 스레드 배치 처리 : MultiThread Batch Process
    - 테스트 및 운영 : TDD & JobExplorer, JobRegistry, JobOperator

2. **스프링 배치 구조 및 원리 이해**
    - 단순한 API 사용법을 넘어 각 기능에 대한 아키텍처 및 흐름과 원리 이해
    - 기본 기능을 확장한 커스트마이징한 기능 구현

3. **스프링 배치 연동**
    - 스케줄러 필요
    - 젠킨스 : 작업 등록 및 실행, 스케줄링
    - Quartz : 실시간 스케줄링, 서버 간 클러스터 기능
    - 다양한 배치 어플리케이션 예제를 통한 실무 감각 익히기