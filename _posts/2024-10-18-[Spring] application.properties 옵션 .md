---
title: '[Spring] application.properties 옵션'
date: 2024-10-18 17:10:00 +0900
categories: [백엔드, Spring]
tags: [Spring]
math: true
mermaid: true
---

## config 옵션
```bash
spring.profiles.active=local
spring.config.activate.on-profile=local
```

## datasource 옵션
```bash
spring.datasource.url=jdbc:mysql://localhost:3306/schema
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/schema
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver

spring.datasource.url=jdbc:log4jdbc:mariadb://localhost:3306/schema
spring.datasource.driver-class-name=net.sf.log4jdbc.sql.jdbcapi.DriverSpy

spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.hikari.username=sa
spring.datasource.hikari.password=
spring.datasource.hikari.driver-class-name=org.h2.Driver
spring.batch.jdbc.initialize-schema=always

spring.datasource.initialization-mode=true
spring.datasource.continue-on-error=true
spring.datasource.data=classpath*:initdata/${database}/data.sql
```

## h2 옵션
```bash
spring.datasource.url=jdbc:h2:mem:test
spring.datasource.url=jdbc:h2:~/test
spring.datasource.url=jdbc:h2:file:~/test;AUTO_SERVER=TRUE
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.url=jdbc:h2:mem:spring_assignments;MODE=MYSQL;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.username=root
spring.datasource.password=1234
spring.datasource.hikari.minimum-idle=1
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.pool-name=H2 DB

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

## sql 옵션
```bash
spring.sql.init.mode=always
spring.sql.init.schema-locations:classpath:db/h2/schema.sql
spring.sql.init.data-locations:classpath:db/h2/data.sql
```

## jpa 옵션
```bash
spring.jpa.show-sql=true
spring.jpa.open-in-view=true
spring.jpa.generate-ddl=true
spring.jpa.defer-datasource-initialization=true
spring.jpa.hibernate.ddl-auto=update

spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.show_sql= true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.highlight_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults=true
```

## logging 옵션
```bash
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql=trace
logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
logging.level.com.zaxxer.hikari.HikariConfig=DEBUG
logging.level.root=info
logging.level.de.siegmar.logbackgelf=DEBUG
```