---
title: '[Backend] variable not initialized in the default constructor'
date: 2024-10-13 09:10:00 +0900
categories: [백엔드, 오늘의 에러]
tags: [Backend]
math: true
mermaid: true
---

## 문제
> java: variable not initialized in the default constructor
> java: cannot find symbol

## 해결 과정
### pom.xml에서 lombok 설정 수정
아래 항목 주석 처리
```xml
<!--			<plugin>-->
<!--				<groupId>org.apache.maven.plugins</groupId>-->
<!--				<artifactId>maven-compiler-plugin</artifactId>-->
<!--				<configuration>-->
<!--					<annotationProcessorPaths>-->
<!--						<path>-->
<!--							<groupId>org.projectlombok</groupId>-->
<!--							<artifactId>lombok</artifactId>-->
<!--						</path>-->
<!--					</annotationProcessorPaths>-->
<!--				</configuration>-->
<!--			</plugin>-->
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.4.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<url/>
	<licenses>
		<license/>
	</licenses>
	<developers>
		<developer/>
	</developers>
	<scm>
		<connection/>
		<developerConnection/>
		<tag/>
		<url/>
	</scm>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
		</dependency>

	<build>
		<plugins>
<!--			<plugin>-->
<!--				<groupId>org.apache.maven.plugins</groupId>-->
<!--				<artifactId>maven-compiler-plugin</artifactId>-->
<!--				<configuration>-->
<!--					<annotationProcessorPaths>-->
<!--						<path>-->
<!--							<groupId>org.projectlombok</groupId>-->
<!--							<artifactId>lombok</artifactId>-->
<!--						</path>-->
<!--					</annotationProcessorPaths>-->
<!--				</configuration>-->
<!--			</plugin>-->
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

## 결과
Lombok 확인 필요