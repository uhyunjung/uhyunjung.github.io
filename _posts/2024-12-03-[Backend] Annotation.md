---
title: '[Backend] Annotation'
date: 2024-12-03 08:10:00 +0900
categories: [백엔드, Backend]
tags: [Backend]
math: true
mermaid: true
---

## Annotation
- `@Schema`
- `@RequestBody` : Swagger 불편, JSON.Stringify
- `@PameterObject` : Swagger 편의, URLSearchParams, null 문자열로 DB에 저장
- `@RequestParam(required = false)`
- `@Schema(required = false)`X

○ @Data @Builder @Entity
	○ @Transactional @Autowired
• Mapper
	• """ 엔터 포함
	• @Param Thete is no getter dto
	• 내부 DTO @Data
• DTO 
	• @SuperBuilder
	• @DateTimeFormat
	• @Schema
• Controller
	• @Tag
	• @RequiredArgs
	• @PreAuthorize
@RestController
	• Get
		○ @Operation
		○ @ParameterObject dto
Post
requestparam body dto parameterobject
Transaction Propagation identity support