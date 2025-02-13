---
title: '[Spring] Spring Batch 6. 멀티 스레드 프로세싱'
date: 2024-11-11 14:30:00 +0900
categories: [백엔드, Spring Batch]
tags: [Spring]
math: true
mermaid: true
---

## Spring Batch 강의
> [**Spring Batch 강의**](https://www.inflearn.com/course/스프링-배치/dashboard)<br/>
> [**Spring Batch 소스 코드**](https://github.com/onjsdnjs/spring-batch-lecture)

## 멀티 스레드 프로세싱
1. 단일 스레드 vs 멀티 스레드
    - 프로세스 내 특정 작업을 처리하는 스레드가 하나일 경우 단일 스레드 여러 개일 경우 멀티 스레드로 정의할 수 있다.
    - 작업 처리에 있어서 단일스레드와 멀티스레드의 선택 기준은 어떤 방식이 자원을 효율적으로 사용하고 성능 처리에 유리한가 하는 점이다.
    - 일반적으로 복잡한 처리나 대용량 데이터를 다루는 작업일 경우 전체 소요 시간 및 성능상의 이점을 가져오기 위해 멀티 스레드 방식을 선택한다.
    - 멀티 스레드 처리 방식은 데이터 동기화 이슈(동시성)가 존재하기 때문에 최대한 고려해서 결정해야 한다.
    <br/>
    
    - **단일 스레드(순차 실행)** : main 스레드 -> 작업 A -> 작업 B -> 작업 C
    - **멀티 스레드(병렬 실행)** : main 스레드 -> worker1 -> 작업 A, worker1 -> 작업 A, worker1 -> 작업 A

2. Main Thread
    - Step -> TaskExecutorRepeatTemplate -> TaskExecutor : 스레드 생성 및 관리 -> execute() -> ThreadPool(Thread1, 2,3) -> run()
    - -> **ExecutingRunnable** run()(RepeatCallback callback, `ResultQueue<ResultHolder>` queue, RepeatStatus result) -> execute() -> ChunkOrientedTasklet -> ItemReader, ItemProcessor, ItemWriter
    - -> ExecutingRunnable run() -> put(**Runnable**) -> BlockingQueue : 스레드 안정성 보장
    - TaskExecutorRepeatTemplate -> (**Runnable** take() -> RepeatStatus) -> BlockingQueue

3. 스프링 배치 스레드 모델
    - 스프링 배치는 기본적으로 단일 스레드 방식으로 작업을 처리한다.
    - 성능 향상과 대규모 데이터 작업을 위한 비동기 처리 및 Scale out 기능을 제공한다.
    - Local과 Remote 처리를 지원한다.
    <br/>

    1. **AsyncItemProcessor / AsyncItemWriter**
        - ItemProcessor 에게 별도의 스레드가 할당되어 작업을 처리하는 방식
    2. **Multi-threaded Step**
        - Step 내 Chunk 구조인 ItemReader, ItemProcessor, ItemWriter 마다 여러 스레드가 할당되어 실행하는 방법
    3. **Remote Chunking**
        - 분산환경처럼 Step 처리가 여러 프로세스로 분할되어 외부의 다른 서버로 전송되어 처리하는 방식
    4. **Parallel Steps**
        - Step마다 스레드가 할당되어 여러 개의 Step을 병렬로 실행하는 방법
    5. **Partitioning**
        - Master/Slave 방식으로서 Master가 데이터를 파티셔닝한 다음 각 파티션에게 스레드를 할당하여 Slave가 독립적으로 작동하는 방식

- Customer 테이블 및 데이터 생성
```sql
CREATE TABLE `customer` (
  `id` mediumint(8) unsigned NOT NULL auto_increment,
  `firstName` varchar(255) default NULL,
  `lastName` varchar(255) default NULL,
  `birthdate` varchar(255),
  PRIMARY KEY (`id`)
) AUTO_INCREMENT=1;
```
```sql
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (1,"Reed","Edwards","1952-08-16 12:34:53"),(2,"Hoyt","Park","1981-02-18 08:07:58"),(3,"Leila","Petty","1972-06-11 08:43:55"),(4,"Denton","Strong","1989-03-11 18:38:31"),(5,"Zoe","Romero","1990-10-02 13:06:31"),(6,"Rana","Compton","1957-06-09 12:51:11"),(7,"Velma","King","1988-02-02 05:52:25"),(8,"Uriah","Carter","1972-08-31 07:32:05"),(9,"Michael","Graves","1958-04-13 18:47:44"),(10,"Leigh","Stone","1967-06-23 23:41:43");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (11,"Iliana","Glenn","1965-02-27 14:33:56"),(12,"Harrison","Haley","1956-06-28 03:15:41"),(13,"Leonard","Zamora","1956-03-28 15:03:09"),(14,"Hiroko","Wyatt","1960-08-22 23:53:50"),(15,"Cameron","Carlson","1969-05-12 11:10:09"),(16,"Hunter","Avery","1953-11-19 12:52:42"),(17,"Aimee","Cox","1976-10-15 12:56:50"),(18,"Yen","Delgado","1990-02-06 10:25:36"),(19,"Gemma","Peterson","1989-04-02 23:42:09"),(20,"Lani","Faulkner","1970-09-18 17:22:14");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (21,"Iola","Cannon","1954-01-12 16:56:45"),(22,"Whitney","Shaffer","1951-03-19 01:27:18"),(23,"Jerome","Moran","1968-03-16 05:26:22"),(24,"Quinn","Wheeler","1979-06-19 16:24:22"),(25,"Mira","Wilder","1961-12-27 12:11:07"),(26,"Tobias","Holloway","1968-08-13 20:36:19"),(27,"Shaine","Schneider","1958-03-08 09:47:10"),(28,"Harding","Gonzales","1952-04-11 02:06:25"),(29,"Calista","Nieves","1970-02-17 13:29:59"),(30,"Duncan","Norman","1987-09-13 00:54:49");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (31,"Fatima","Hamilton","1961-06-16 14:29:11"),(32,"Ali","Browning","1979-03-27 17:09:37"),(33,"Erin","Sosa","1990-08-23 10:43:58"),(34,"Carol","Harmon","1972-01-14 07:19:39"),(35,"Illiana","Fitzgerald","1970-08-19 02:33:46"),(36,"Stephen","Riley","1954-06-05 08:34:03"),(37,"Hermione","Waller","1969-09-08 01:19:07"),(38,"Desiree","Flowers","1952-06-25 13:34:45"),(39,"Karyn","Blackburn","1977-03-30 13:08:02"),(40,"Briar","Carroll","1985-03-26 01:03:34");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (41,"Chaney","Green","1987-04-20 18:56:53"),(42,"Robert","Higgins","1985-09-26 11:25:10"),(43,"Lillith","House","1982-12-06 02:24:23"),(44,"Astra","Winters","1952-03-13 01:13:07"),(45,"Cherokee","Stephenson","1955-10-23 16:57:33"),(46,"Yuri","Shaw","1958-07-14 15:10:07"),(47,"Boris","Sparks","1982-01-01 10:56:34"),(48,"Wilma","Blake","1963-06-07 16:32:33"),(49,"Brynne","Morse","1964-09-21 01:05:25"),(50,"Ila","Conley","1953-11-02 05:12:57");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (51,"Sharon","Watts","1964-01-09 16:32:37"),(52,"Kareem","Vaughan","1952-04-18 15:37:10"),(53,"Eden","Barnes","1954-07-04 01:26:44"),(54,"Kenyon","Fulton","1975-08-23 22:17:52"),(55,"Mona","Ball","1972-02-11 04:15:45"),(56,"Moses","Cortez","1979-04-24 15:26:46"),(57,"Macy","Banks","1956-12-31 00:41:15"),(58,"Brenna","Mendez","1972-10-02 07:58:27"),(59,"Emerald","Ewing","1985-11-28 21:15:20"),(60,"Lev","Mcfarland","1951-05-20 14:30:07");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (61,"Norman","Tanner","1959-07-29 15:41:45"),(62,"Alexa","Walters","1977-12-06 16:41:17"),(63,"Dara","Hyde","1989-08-04 14:06:43"),(64,"Hu","Sampson","1978-11-01 17:10:23"),(65,"Jasmine","Cardenas","1969-02-15 20:08:06"),(66,"Julian","Bentley","1954-07-11 03:27:51"),(67,"Samson","Brown","1967-10-15 07:03:59"),(68,"Gisela","Hogan","1985-01-19 03:16:20"),(69,"Jeanette","Cummings","1986-09-07 18:25:52"),(70,"Galena","Perkins","1984-01-13 02:15:31");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (71,"Olga","Mays","1981-11-20 22:39:27"),(72,"Ferdinand","Austin","1956-08-08 09:08:02"),(73,"Zenia","Anthony","1964-08-21 05:45:16"),(74,"Hop","Hampton","1982-07-22 14:11:00"),(75,"Shaine","Vang","1970-08-13 15:58:28"),(76,"Ariana","Cochran","1959-12-04 01:18:36"),(77,"India","Paul","1963-10-10 05:24:03"),(78,"Karina","Doyle","1979-12-01 00:05:21"),(79,"Delilah","Johnston","1989-03-04 23:50:01"),(80,"Hilel","Hood","1959-08-22 06:40:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (81,"Kennedy","Hoffman","1963-10-14 20:18:35"),(82,"Kameko","Bell","1976-06-08 15:35:54"),(83,"Lunea","Gutierrez","1964-06-07 16:21:24"),(84,"William","Burris","1980-05-01 17:58:23"),(85,"Kiara","Walls","1955-12-27 18:57:15"),(86,"Latifah","Alexander","1980-06-19 10:39:50"),(87,"Keaton","Ward","1964-10-12 16:03:18"),(88,"Jasper","Clements","1970-03-05 00:29:49"),(89,"Claire","Brown","1972-02-11 00:43:58"),(90,"Noble","Morgan","1955-09-05 05:35:01");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (91,"Evangeline","Horn","1952-12-28 14:06:27"),(92,"Jonah","Harrell","1951-06-25 17:37:35"),(93,"Mira","Espinoza","1982-03-26 06:01:16"),(94,"Brennan","Oneill","1979-04-23 08:49:02"),(95,"Dacey","Howe","1983-02-06 19:11:00"),(96,"Yoko","Pittman","1982-09-12 02:18:52"),(97,"Cody","Conway","1971-05-26 07:09:58"),(98,"Jordan","Knowles","1981-12-30 02:20:01"),(99,"Pearl","Boyer","1957-10-19 14:26:49"),(100,"Keely","Montoya","1985-03-24 01:18:09");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (101,"Michelle","House","1964-07-21 23:28:52"),(102,"Xavier","Wagner","1975-11-10 21:24:38"),(103,"Xandra","Britt","1963-02-07 06:39:31"),(104,"Cole","Nolan","1988-06-25 18:58:47"),(105,"Charles","Hartman","1987-04-18 01:54:46"),(106,"Fiona","Newton","1971-12-26 18:29:47"),(107,"Kuame","Walton","1961-12-29 19:02:26"),(108,"Erica","Finch","1958-03-23 12:55:49"),(109,"Kamal","Steele","1966-06-22 14:36:06"),(110,"Veda","Carpenter","1977-11-01 21:22:08");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (111,"Wendy","Glenn","1977-02-21 20:54:54"),(112,"Vernon","Lara","1986-10-04 18:31:48"),(113,"Xerxes","Townsend","1962-08-18 00:00:05"),(114,"Ocean","Tran","1983-12-01 09:09:11"),(115,"Rose","Vaughan","1988-06-15 09:17:29"),(116,"Kay","Christian","1978-05-23 23:54:35"),(117,"Petra","Fisher","1954-03-16 07:04:47"),(118,"Giselle","Lambert","1972-05-01 19:17:15"),(119,"Malachi","Meadows","1974-10-15 06:51:54"),(120,"Zephania","Hensley","1970-03-20 14:13:44");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (121,"Malachi","Morrow","1950-12-10 22:21:27"),(122,"Quinn","Walter","1954-02-09 09:47:34"),(123,"Amity","Reese","1950-12-09 18:38:20"),(124,"Adria","Gilliam","1982-07-20 12:54:38"),(125,"Ann","Jennings","1956-05-09 05:31:46"),(126,"Charles","Harper","1965-03-18 03:42:55"),(127,"Giselle","Case","1951-02-15 04:14:58"),(128,"Brynn","Reeves","1978-02-10 01:42:07"),(129,"Scarlett","Mcintyre","1986-12-10 11:01:47"),(130,"Brielle","Aguirre","1976-03-04 07:34:58");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (131,"Blaze","Price","1958-09-05 03:34:33"),(132,"Dorothy","Mcmillan","1965-02-19 17:23:39"),(133,"Beverly","Bradshaw","1957-08-26 00:50:55"),(134,"Autumn","Hendricks","1960-02-14 22:18:38"),(135,"Allistair","Frederick","1970-05-14 06:01:23"),(136,"Kathleen","Petersen","1965-06-29 23:33:06"),(137,"Neil","Haney","1980-03-14 11:00:15"),(138,"Jin","Cooley","1972-11-02 03:22:45"),(139,"Hilary","Lowery","1956-09-02 21:30:02"),(140,"Bruce","Stokes","1954-01-13 04:38:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (141,"Fletcher","Mercado","1958-01-27 10:07:15"),(142,"Pandora","Osborn","1974-05-30 08:49:35"),(143,"Kaden","William","1961-01-12 10:31:38"),(144,"Myles","Humphrey","1962-10-05 08:29:53"),(145,"Lesley","Nieves","1966-05-24 16:53:03"),(146,"Christopher","Mcdaniel","1956-12-01 09:57:26"),(147,"Ruby","Hewitt","1979-12-30 16:30:36"),(148,"Oleg","Carlson","1983-03-05 21:16:05"),(149,"Jolie","Bass","1975-04-11 10:48:11"),(150,"Tanek","Maynard","1978-04-03 07:53:10");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (151,"Axel","Hansen","1956-01-03 20:01:24"),(152,"Aline","Pickett","1973-02-16 15:01:42"),(153,"Callie","Castillo","1966-05-31 07:47:17"),(154,"Galena","Fisher","1966-12-18 09:39:17"),(155,"August","Valentine","1961-04-12 09:33:43"),(156,"Davis","Maldonado","1962-02-27 15:46:53"),(157,"Christian","Mcconnell","1966-11-28 22:33:33"),(158,"Honorato","Hansen","1975-02-26 21:28:39"),(159,"Patrick","Snyder","1966-07-27 16:40:22"),(160,"Lynn","Wiggins","1976-12-10 06:02:50");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (161,"Kai","Potts","1969-10-04 10:21:12"),(162,"Vaughan","Lang","1951-08-23 04:14:24"),(163,"Mark","King","1986-07-27 19:09:22"),(164,"Xenos","Hart","1967-11-26 06:09:40"),(165,"Nell","Kirk","1964-10-22 09:09:34"),(166,"Nevada","Dodson","1988-07-16 03:47:17"),(167,"Orla","Rasmussen","1973-12-10 03:40:11"),(168,"Meghan","Oliver","1951-05-09 09:08:50"),(169,"Minerva","Morris","1970-07-31 13:56:00"),(170,"Sarah","Barton","1979-07-10 13:00:18");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (171,"Yardley","Good","1986-05-08 17:33:48"),(172,"Tara","Barrera","1984-09-19 16:45:07"),(173,"Keelie","Carney","1952-07-17 10:39:09"),(174,"Lamar","Weaver","1970-02-02 23:00:19"),(175,"Iris","Holt","1962-10-16 09:24:26"),(176,"Lisandra","Christian","1968-12-19 06:13:34"),(177,"Zeph","Schultz","1977-12-07 03:17:07"),(178,"Fallon","Morse","1984-12-15 20:55:05"),(179,"Cynthia","Blackwell","1974-07-02 11:57:10"),(180,"Angela","Gay","1986-04-20 10:38:41");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (181,"Amber","Harding","1952-03-15 02:09:44"),(182,"Katelyn","Harmon","1971-07-24 12:04:14"),(183,"Byron","Faulkner","1974-12-30 11:19:07"),(184,"Xyla","Fowler","1990-07-15 03:17:05"),(185,"Zeus","Kemp","1969-09-29 10:16:11"),(186,"Kylan","Charles","1980-07-04 09:25:28"),(187,"Dylan","Short","1967-11-28 08:03:58"),(188,"Dora","Holland","1986-09-06 02:38:54"),(189,"Dana","Jones","1979-06-13 01:47:11"),(190,"Kaye","Zimmerman","1974-05-22 19:46:19");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (191,"Lareina","Carney","1956-06-17 04:26:50"),(192,"Reece","Figueroa","1984-10-05 12:04:35"),(193,"Justina","Gates","1986-07-30 08:53:20"),(194,"Raya","Sheppard","1966-06-07 05:16:28"),(195,"Harper","Bender","1951-03-24 14:22:38"),(196,"Candace","Collins","1953-07-26 17:53:44"),(197,"Idola","Levine","1986-03-25 17:16:34"),(198,"Belle","Hays","1970-05-05 03:27:20"),(199,"Dominique","Nunez","1990-10-01 19:48:17"),(200,"Adria","Harrell","1955-04-08 12:14:55");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (201,"Kirsten","Chapman","1967-06-15 12:26:58"),(202,"Vance","Mccall","1972-03-21 16:46:50"),(203,"Duncan","Page","1980-07-06 22:17:56"),(204,"Addison","Wilkinson","1982-12-28 06:48:04"),(205,"Owen","Petty","1964-03-15 20:02:25"),(206,"Clementine","Kim","1981-02-17 22:50:30"),(207,"Yoshi","Campos","1985-08-02 10:04:25"),(208,"Dacey","Baxter","1980-12-06 21:40:18"),(209,"Bevis","Arnold","1973-09-04 06:50:34"),(210,"Sasha","Davenport","1985-12-06 20:49:29");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (211,"Josephine","Delaney","1955-12-10 03:35:58"),(212,"Avram","Farmer","1984-04-05 16:18:00"),(213,"Kai","Ferguson","1977-12-13 13:52:32"),(214,"David","Mercado","1961-08-08 06:37:49"),(215,"Suki","Foreman","1973-03-07 08:52:05"),(216,"Uma","Clayton","1964-01-28 13:50:56"),(217,"Mannix","Gutierrez","1990-08-22 21:10:28"),(218,"Adrian","Macdonald","1982-02-12 13:53:51"),(219,"Kellie","Lynch","1974-01-18 18:16:39"),(220,"Xaviera","Key","1962-03-26 23:15:01");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (221,"Sybill","Hood","1960-11-13 12:25:15"),(222,"Adena","Rhodes","1961-10-12 15:28:42"),(223,"Cheyenne","Henderson","1974-03-19 06:55:07"),(224,"Carly","Nielsen","1978-01-30 09:59:18"),(225,"Burton","Stuart","1982-04-17 00:02:52"),(226,"Alden","Barnes","1969-11-16 09:19:20"),(227,"Zeus","Walsh","1961-03-15 20:28:22"),(228,"Bruce","Soto","1967-05-20 15:59:08"),(229,"Fatima","Roth","1987-02-14 02:37:32"),(230,"Branden","Beasley","1955-11-22 20:54:38");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (231,"Petra","Davidson","1989-04-11 03:56:35"),(232,"Ursula","Park","1966-10-19 21:08:29"),(233,"Hu","Kent","1967-01-03 16:43:33"),(234,"Anne","Shields","1990-10-26 20:38:57"),(235,"Amos","Golden","1951-01-07 15:33:14"),(236,"Gavin","Conner","1986-10-09 16:46:56"),(237,"Geraldine","Warner","1969-03-14 18:42:07"),(238,"Callum","Todd","1958-05-21 23:30:56"),(239,"Alexander","Morin","1961-02-22 11:39:48"),(240,"Maisie","Morales","1986-10-15 01:34:28");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (241,"Keiko","Rodriguez","1989-02-03 18:47:44"),(242,"Amaya","Bullock","1980-11-26 18:01:44"),(243,"Aimee","Arnold","1976-03-03 20:38:51"),(244,"Jillian","Cabrera","1979-11-08 06:29:22"),(245,"Ronan","Camacho","1977-10-02 05:08:10"),(246,"Dora","Schultz","1958-02-02 05:02:29"),(247,"Chiquita","Perry","1973-06-13 00:17:33"),(248,"Rudyard","Wynn","1980-01-24 10:49:15"),(249,"Avram","Hardy","1958-11-12 13:34:53"),(250,"Odysseus","Delaney","1960-06-21 08:13:28");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (251,"Logan","Carr","1983-06-10 05:43:03"),(252,"Walker","Michael","1984-03-31 00:21:20"),(253,"Ishmael","Paul","1954-04-21 19:35:03"),(254,"Angelica","Valencia","1984-02-10 04:09:12"),(255,"Belle","Baker","1963-06-21 22:29:08"),(256,"Reuben","Harvey","1989-02-20 19:59:37"),(257,"Kai","Barry","1958-04-17 12:18:52"),(258,"Ingrid","Trevino","1986-02-02 03:58:07"),(259,"Amelia","Wong","1951-12-31 15:56:27"),(260,"Bradley","Henry","1985-08-04 23:50:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (261,"Graiden","Hinton","1972-04-13 23:54:11"),(262,"Hall","Collier","1954-12-03 09:01:47"),(263,"Lacey","Ford","1952-06-25 07:11:38"),(264,"Xenos","Terrell","1989-05-25 06:03:21"),(265,"Beck","Key","1954-09-04 19:35:15"),(266,"Veda","Wilcox","1982-05-29 18:58:12"),(267,"Ralph","Morin","1983-06-24 10:00:23"),(268,"Burke","Hicks","1976-08-24 23:32:50"),(269,"Yasir","Aguirre","1968-05-15 06:21:56"),(270,"Pearl","Cunningham","1967-07-09 02:40:59");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (271,"Alan","Paul","1955-08-17 06:47:43"),(272,"Herrod","Joyner","1968-06-29 03:01:37"),(273,"Tanek","Delgado","1960-08-12 18:41:21"),(274,"Ursa","Garcia","1960-06-23 17:32:11"),(275,"Forrest","Mcneil","1969-10-07 05:39:12"),(276,"Mariko","Hurley","1970-03-29 10:52:38"),(277,"Dale","Workman","1962-01-12 09:15:10"),(278,"Noelani","Hanson","1988-11-06 16:53:05"),(279,"Mari","Tyson","1979-03-10 05:31:00"),(280,"Ursa","England","1954-06-28 20:19:01");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (281,"Roth","Bentley","1984-09-29 01:26:17"),(282,"Carter","Payne","1953-05-10 20:21:30"),(283,"Lee","Gay","1966-11-16 06:57:50"),(284,"Isabella","Fernandez","1986-05-31 15:19:09"),(285,"Madonna","Graham","1978-03-01 00:55:12"),(286,"Armand","Frank","1982-10-04 07:39:41"),(287,"Zoe","Tran","1969-08-30 11:09:00"),(288,"Burke","Blair","1956-02-13 16:02:21"),(289,"Howard","Day","1983-07-10 22:48:56"),(290,"Xantha","Buckley","1982-10-13 10:00:28");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (291,"Wyatt","Hinton","1979-03-03 18:17:56"),(292,"Martin","Mathews","1965-06-25 11:42:51"),(293,"Zephania","George","1953-02-22 22:06:05"),(294,"Phoebe","Clay","1964-04-21 19:59:09"),(295,"Vernon","Brewer","1957-12-11 15:32:51"),(296,"Blythe","Calderon","1963-11-05 12:18:42"),(297,"Macaulay","Lara","1957-07-10 03:53:47"),(298,"Imelda","Buck","1965-11-16 14:10:59"),(299,"Neville","Riggs","1984-06-11 11:58:00"),(300,"Tasha","Duncan","1983-09-26 00:25:42");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (301,"Walker","Jarvis","1969-08-31 06:54:50"),(302,"Forrest","Kim","1956-06-28 02:16:10"),(303,"Phoebe","Tucker","1955-03-09 07:26:02"),(304,"Solomon","Hendricks","1990-08-05 05:45:47"),(305,"Tatiana","Merrill","1966-07-21 10:38:26"),(306,"Hilel","Raymond","1966-03-26 06:39:40"),(307,"Dominique","Leblanc","1987-11-26 12:58:12"),(308,"Naomi","Edwards","1964-08-10 20:02:21"),(309,"Lawrence","Ramos","1990-10-16 04:41:56"),(310,"Raymond","Mccoy","1973-09-11 01:09:56");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (311,"Xanthus","Frost","1954-05-08 18:36:03"),(312,"Summer","Ferguson","1977-06-17 21:01:16"),(313,"Giacomo","Shannon","1984-12-20 07:25:15"),(314,"Macaulay","Rowland","1990-07-25 21:06:19"),(315,"Hiram","Pennington","1988-07-29 05:55:41"),(316,"Chava","Taylor","1975-10-27 18:08:30"),(317,"Xenos","Hopper","1967-03-03 02:58:05"),(318,"Cathleen","Franklin","1969-07-06 17:30:24"),(319,"Althea","Cohen","1963-07-17 17:31:15"),(320,"Ora","Hernandez","1975-10-17 01:53:31");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (321,"Nadine","Alvarez","1981-08-27 06:18:18"),(322,"Sloane","Leach","1979-11-18 23:19:01"),(323,"Cheyenne","Graham","1954-05-06 08:08:22"),(324,"Gemma","Alvarado","1987-11-17 10:58:49"),(325,"Catherine","Small","1954-09-14 09:32:31"),(326,"Yolanda","Hahn","1974-01-25 04:13:22"),(327,"Camden","Madden","1984-06-23 04:00:09"),(328,"Hanae","Gamble","1962-06-18 15:31:20"),(329,"Lucius","Carter","1952-04-06 11:22:00"),(330,"Geoffrey","Ruiz","1961-01-09 06:57:26");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (331,"Baxter","Ashley","1971-04-16 23:43:48"),(332,"Ursa","Simpson","1960-01-21 01:26:50"),(333,"Ali","Combs","1973-08-14 03:47:04"),(334,"Kiayada","Rowland","1984-10-22 00:41:52"),(335,"Megan","Gray","1961-01-19 13:02:27"),(336,"Chelsea","Gonzalez","1968-01-01 22:27:53"),(337,"Casey","Foster","1988-08-07 20:48:55"),(338,"Genevieve","Jacobs","1969-12-13 21:26:50"),(339,"Joshua","Rodgers","1984-03-14 03:33:56"),(340,"Gary","Howell","1957-07-27 18:57:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (341,"Colby","Branch","1984-07-21 12:54:03"),(342,"Kylee","Calderon","1976-09-01 18:35:50"),(343,"Chastity","Douglas","1960-07-26 04:46:16"),(344,"Elizabeth","Vargas","1971-03-14 22:34:10"),(345,"Anne","Mcdaniel","1985-10-30 22:07:55"),(346,"Kimberley","Key","1969-03-18 00:13:28"),(347,"Ivory","Lynn","1989-03-06 02:51:54"),(348,"Jared","Leonard","1979-01-31 14:32:24"),(349,"Vivian","Robinson","1986-11-17 04:22:13"),(350,"Jenna","Barrera","1964-10-31 07:14:19");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (351,"Ava","Nieves","1980-07-17 23:40:49"),(352,"Prescott","Gallegos","1979-06-02 00:04:55"),(353,"Ila","Chase","1982-06-07 17:32:02"),(354,"Quinn","Shaffer","1954-03-03 20:01:15"),(355,"Thaddeus","Ochoa","1952-09-07 06:30:54"),(356,"Sawyer","Montoya","1975-05-25 07:15:00"),(357,"Mona","Hendrix","1955-09-13 19:40:24"),(358,"Quinn","Franco","1964-11-14 02:56:13"),(359,"Myra","Rios","1951-06-08 07:02:01"),(360,"Laurel","Malone","1984-04-12 15:21:58");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (361,"Brian","Myers","1968-12-19 11:35:12"),(362,"Joan","Jacobs","1972-10-19 09:59:00"),(363,"Aurelia","Donovan","1958-10-03 15:16:28"),(364,"Melyssa","Mcknight","1980-03-05 13:16:08"),(365,"Rina","Hughes","1978-04-14 20:02:28"),(366,"Samantha","Mayer","1973-02-19 14:09:30"),(367,"Melvin","Barry","1953-06-18 15:12:16"),(368,"Delilah","Pena","1961-01-19 14:34:33"),(369,"Violet","Floyd","1957-09-26 08:07:52"),(370,"Guinevere","Baker","1977-05-25 18:59:22");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (371,"Whitney","Graves","1979-01-06 15:23:15"),(372,"Axel","Black","1961-07-26 11:14:38"),(373,"Carl","Vasquez","1960-09-17 00:24:23"),(374,"Lydia","Ashley","1986-02-10 00:39:06"),(375,"Xerxes","Carson","1968-03-14 12:09:21"),(376,"Sara","Allen","1980-11-21 11:22:20"),(377,"Malcolm","Snow","1957-02-21 17:35:38"),(378,"Maile","Workman","1985-10-23 12:18:19"),(379,"Aubrey","Alston","1963-01-17 03:40:33"),(380,"Mark","Scott","1962-03-22 01:39:31");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (381,"Thor","Blackwell","1966-04-09 05:45:16"),(382,"Gisela","Duke","1975-12-18 13:04:42"),(383,"Martina","Ballard","1987-04-17 14:41:01"),(384,"Xavier","Mcmillan","1965-08-05 18:07:06"),(385,"Francesca","Osborne","1958-04-09 07:18:05"),(386,"Barry","Mccoy","1971-09-23 19:46:46"),(387,"Gray","Walker","1962-02-28 21:08:01"),(388,"Cleo","Potts","1987-01-03 10:35:30"),(389,"Willow","Humphrey","1957-11-24 11:47:22"),(390,"Simone","Silva","1955-03-20 04:15:23");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (391,"Devin","Frazier","1965-05-02 12:38:53"),(392,"Aquila","Henry","1971-09-28 12:40:06"),(393,"Mollie","Wyatt","1953-07-18 18:21:26"),(394,"Vaughan","Holland","1962-04-12 00:03:35"),(395,"Charlotte","Lott","1988-01-02 16:27:27"),(396,"Bruno","Johns","1989-12-01 23:39:50"),(397,"Kendall","Landry","1980-09-03 09:09:48"),(398,"Karly","George","1977-07-04 16:30:36"),(399,"Travis","Blackburn","1984-04-29 06:28:12"),(400,"Amery","Humphrey","1983-03-01 08:01:47");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (401,"Kadeem","Franklin","1960-07-08 12:00:36"),(402,"Cade","Patterson","1954-05-03 03:31:40"),(403,"Arsenio","Holman","1957-05-26 02:28:12"),(404,"Georgia","Buchanan","1971-11-11 13:11:10"),(405,"Brittany","Decker","1969-03-31 23:27:51"),(406,"Kane","Hurst","1969-12-22 20:23:43"),(407,"Halee","Terrell","1966-04-28 02:11:28"),(408,"Scott","Osborne","1962-11-23 11:49:00"),(409,"Rooney","Marquez","1967-07-28 13:09:50"),(410,"Gillian","Dejesus","1983-12-08 15:00:25");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (411,"Avye","Smith","1969-08-17 16:37:37"),(412,"Rose","Simon","1977-04-05 23:52:30"),(413,"Lee","Pate","1972-06-20 01:12:52"),(414,"Zena","Meyers","1965-04-15 05:52:56"),(415,"Steel","Lucas","1959-08-05 06:32:43"),(416,"Berk","Johnston","1971-12-23 15:17:58"),(417,"Chandler","Stokes","1969-08-18 19:13:16"),(418,"Drake","Baxter","1987-12-15 22:39:45"),(419,"Merritt","Wallace","1958-10-04 00:30:32"),(420,"Rigel","Davenport","1951-10-08 03:31:53");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (421,"Bruce","Walsh","1982-01-22 22:40:30"),(422,"Brennan","Peters","1961-06-26 20:42:37"),(423,"Zeph","Hodges","1981-12-16 07:27:29"),(424,"Sade","Horton","1978-11-26 00:16:22"),(425,"Moses","Graham","1981-07-15 09:22:05"),(426,"Emmanuel","Clemons","1974-08-19 10:57:48"),(427,"Hadley","Lawrence","1950-12-21 13:45:38"),(428,"Roth","Hoffman","1961-08-23 07:40:22"),(429,"Wynne","Buchanan","1988-03-30 09:42:02"),(430,"Geoffrey","Cook","1989-12-05 07:54:23");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (431,"Adrian","Waller","1970-02-03 02:16:36"),(432,"Laith","Lang","1972-06-14 05:20:32"),(433,"Charde","Watson","1985-04-30 17:02:39"),(434,"Christine","Bennett","1967-05-31 11:02:35"),(435,"Irene","Giles","1968-10-29 02:51:04"),(436,"Holmes","Lindsay","1956-06-02 03:41:51"),(437,"Brynn","Williamson","1959-02-18 18:30:28"),(438,"Dalton","Cunningham","1962-08-08 16:39:44"),(439,"Ila","Boone","1963-09-18 04:43:56"),(440,"Erica","Galloway","1957-03-04 02:32:26");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (441,"Ciaran","Santana","1968-02-08 07:54:00"),(442,"Cameron","Strickland","1976-11-01 15:11:13"),(443,"Connor","Velasquez","1974-04-24 16:28:01"),(444,"Blossom","Berg","1967-02-14 18:59:27"),(445,"Jakeem","Reilly","1973-11-20 06:32:41"),(446,"Fay","Rutledge","1951-06-14 18:56:14"),(447,"August","Berg","1951-06-24 21:24:59"),(448,"Naomi","Morris","1987-07-05 00:39:41"),(449,"Fiona","Vang","1983-03-21 18:58:59"),(450,"Danielle","Jensen","1963-05-31 09:42:04");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (451,"Laith","Bartlett","1955-04-06 04:55:40"),(452,"Leigh","Holcomb","1964-11-23 11:37:45"),(453,"Chantale","Chen","1968-06-22 12:42:04"),(454,"Kevyn","Wise","1974-07-24 13:35:56"),(455,"Demetrius","Snider","1956-02-11 19:06:56"),(456,"Tasha","Guy","1975-11-20 08:47:01"),(457,"Ignacia","George","1965-06-09 13:26:24"),(458,"Charissa","Landry","1979-07-01 06:15:54"),(459,"Jolene","Malone","1967-08-12 19:23:17"),(460,"Haley","Glover","1987-07-10 21:40:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (461,"Martina","Rowe","1986-03-27 06:47:21"),(462,"Kyra","Duncan","1975-06-18 16:13:19"),(463,"Dominic","Wong","1977-04-12 11:50:15"),(464,"Halee","Caldwell","1958-06-27 11:56:38"),(465,"Dane","Kramer","1960-07-17 20:14:06"),(466,"Colton","Garrison","1979-06-19 14:26:42"),(467,"Neville","Bolton","1967-02-06 23:23:11"),(468,"Dylan","Frye","1984-06-07 17:59:01"),(469,"Yuri","Palmer","1964-07-28 12:47:11"),(470,"Lamar","Alvarado","1980-01-26 15:15:44");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (471,"Bevis","William","1964-02-09 15:21:25"),(472,"Ella","Martinez","1956-05-01 11:51:54"),(473,"Walter","Golden","1986-07-11 00:01:40"),(474,"Carter","Harvey","1987-10-07 00:32:02"),(475,"Steven","Ortega","1986-03-29 14:05:03"),(476,"Christopher","Battle","1986-01-28 09:49:58"),(477,"Philip","Howe","1983-08-08 12:20:10"),(478,"Knox","Calderon","1971-07-02 08:23:02"),(479,"Brenden","Dillard","1981-03-25 21:41:47"),(480,"Sigourney","Walker","1987-09-28 10:35:56");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (481,"Lila","Prince","1975-07-04 09:13:43"),(482,"Kaitlin","Wolf","1973-07-09 12:38:19"),(483,"Athena","Whitfield","1988-05-02 08:39:23"),(484,"Quinlan","Travis","1981-03-27 03:59:45"),(485,"Angela","Booth","1972-02-16 00:40:53"),(486,"Dennis","Haynes","1976-01-12 11:39:56"),(487,"Rhiannon","Rosario","1966-06-16 23:06:54"),(488,"Paul","Baxter","1985-05-07 01:27:19"),(489,"Wallace","Powers","1952-06-09 16:22:40"),(490,"Malachi","Pruitt","1963-12-24 12:40:48");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (491,"Garrison","Randolph","1965-01-14 16:22:02"),(492,"Omar","Puckett","1967-04-06 10:37:51"),(493,"Tamekah","Kaufman","1976-03-07 18:52:34"),(494,"Erica","Peterson","1987-09-25 13:12:25"),(495,"Lacey","Bell","1970-08-24 11:20:23"),(496,"Xavier","Garza","1954-10-24 15:48:03"),(497,"Bryar","Hunt","1973-03-30 15:13:21"),(498,"Lacota","Trujillo","1975-02-27 13:00:34"),(499,"Jared","Wiley","1982-12-05 06:23:56"),(500,"Patricia","Randall","1971-04-02 23:19:55");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (501,"Nell","Bean","1966-10-03 16:32:12"),(502,"Stephen","Gentry","1983-02-24 09:05:19"),(503,"Erasmus","Reynolds","1976-07-06 08:35:11"),(504,"Nadine","Greene","1989-10-10 10:16:47"),(505,"Ruby","Weaver","1952-10-18 00:41:24"),(506,"Gareth","Fletcher","1961-11-11 16:56:00"),(507,"Dieter","Osborn","1968-12-02 08:03:03"),(508,"Amela","Hammond","1973-03-22 22:24:29"),(509,"Erasmus","Knowles","1974-05-02 12:32:51"),(510,"Yoshio","Petersen","1969-12-23 02:08:58");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (511,"Devin","Rich","1972-10-05 13:44:31"),(512,"Richard","Knowles","1958-08-03 03:20:19"),(513,"Zelda","Houston","1973-04-19 11:47:36"),(514,"Quinn","Alston","1951-11-15 05:01:07"),(515,"Risa","Farmer","1956-06-21 20:53:06"),(516,"Regan","Molina","1976-06-10 00:17:46"),(517,"Kai","Best","1987-06-04 01:00:45"),(518,"Petra","Alvarez","1990-03-13 01:57:18"),(519,"Kerry","Deleon","1954-08-19 20:32:46"),(520,"Colin","Mcneil","1956-03-15 08:35:23");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (521,"Chelsea","Mack","1957-08-30 18:55:15"),(522,"Lance","Chen","1987-11-08 12:48:48"),(523,"Lester","Allison","1966-04-17 07:54:24"),(524,"Jamal","Roach","1989-03-21 18:40:52"),(525,"Patricia","Pace","1975-01-16 13:32:57"),(526,"Arsenio","Browning","1963-02-07 23:40:02"),(527,"Elvis","Gentry","1960-09-24 00:32:57"),(528,"Elijah","Peterson","1974-08-19 06:38:42"),(529,"Erin","Herrera","1970-03-06 14:35:36"),(530,"MacKenzie","Odonnell","1983-10-17 13:08:20");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (531,"Arsenio","Mercer","1983-08-15 10:25:29"),(532,"Nichole","Gutierrez","1953-11-19 05:06:13"),(533,"Hedda","Fox","1965-09-05 21:14:37"),(534,"Dean","Barber","1953-09-04 19:33:53"),(535,"Clio","Glass","1958-09-26 23:28:51"),(536,"Idola","Frye","1985-04-25 22:12:14"),(537,"Stewart","Pierce","1987-03-28 16:12:25"),(538,"Patrick","Paul","1981-12-28 20:23:06"),(539,"Hadassah","Shields","1953-01-28 19:29:55"),(540,"Renee","Wise","1961-02-24 23:08:22");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (541,"Desiree","Roach","1964-08-03 08:27:55"),(542,"Aladdin","Mcdonald","1961-12-05 13:04:24"),(543,"Kyra","Merritt","1960-09-03 01:22:04"),(544,"Ocean","Bishop","1985-02-10 08:33:50"),(545,"Gary","Norton","1987-10-12 04:58:43"),(546,"Mannix","Cooper","1976-12-07 07:05:25"),(547,"Melodie","Matthews","1988-02-22 11:12:41"),(548,"Garrett","Walton","1979-08-08 20:10:31"),(549,"Melodie","Booth","1967-06-28 06:22:02"),(550,"Mollie","Greene","1953-09-30 04:26:31");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (551,"Victoria","Oliver","1952-06-24 03:22:53"),(552,"Herman","Heath","1961-03-19 00:59:21"),(553,"Wendy","Stout","1970-08-21 20:08:57"),(554,"Dolan","Pitts","1951-03-27 17:35:19"),(555,"Hiram","Vaughan","1952-01-12 05:00:45"),(556,"Daryl","Rollins","1956-05-11 08:52:47"),(557,"Chaim","Wells","1962-03-05 05:56:41"),(558,"Nash","Weaver","1954-12-17 06:27:15"),(559,"Kevyn","Wooten","1959-04-12 14:20:01"),(560,"Lavinia","Vance","1975-09-10 23:24:51");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (561,"Ursula","Avila","1958-01-29 07:00:27"),(562,"Cassandra","Whitfield","1975-05-08 01:06:57"),(563,"Celeste","Frank","1982-08-20 09:13:03"),(564,"Emmanuel","Henson","1979-06-28 08:37:55"),(565,"Michelle","Harrison","1959-08-26 17:39:38"),(566,"Xenos","Blair","1975-08-25 09:46:59"),(567,"Shannon","Carter","1984-07-19 14:39:39"),(568,"Holmes","Weber","1989-10-06 13:29:17"),(569,"Noelle","Shaw","1960-02-08 14:15:54"),(570,"Tamekah","Moran","1987-03-07 23:41:39");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (571,"Cairo","Witt","1988-09-28 04:42:27"),(572,"Alfreda","Hatfield","1990-11-09 19:40:51"),(573,"Beverly","Harris","1975-06-13 23:12:22"),(574,"Troy","Lucas","1972-06-15 17:36:42"),(575,"Hanna","Hopper","1963-04-25 07:10:47"),(576,"Aimee","David","1957-05-12 10:05:34"),(577,"Burton","Robles","1982-05-08 01:09:39"),(578,"Miriam","Melendez","1989-07-14 06:03:57"),(579,"Keane","Pierce","1969-04-09 13:39:58"),(580,"Rudyard","Carroll","1985-07-07 12:52:45");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (581,"Rigel","Fox","1967-09-22 12:37:11"),(582,"Iona","Gutierrez","1985-11-29 17:08:38"),(583,"Hilda","Knight","1983-06-18 16:34:04"),(584,"Paloma","Garrett","1957-04-18 02:02:12"),(585,"Callum","Hodges","1973-10-01 16:02:21"),(586,"Kiayada","Wiley","1988-03-01 11:10:38"),(587,"Plato","Brady","1989-12-20 18:02:35"),(588,"Brennan","Justice","1961-05-17 23:32:32"),(589,"Elizabeth","Molina","1985-08-29 20:05:06"),(590,"Jonas","Meyer","1952-10-19 14:17:27");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (591,"Brady","Sanchez","1978-02-21 15:02:21"),(592,"Noah","Cunningham","1951-10-16 22:21:10"),(593,"Hyatt","Goff","1957-01-27 13:18:07"),(594,"Scarlett","Baldwin","1968-08-20 14:43:52"),(595,"Zorita","Lynn","1955-04-09 21:36:01"),(596,"Mariko","Stevens","1952-10-06 04:37:00"),(597,"Nell","Watts","1984-04-15 23:36:17"),(598,"Bree","Cole","1982-02-22 20:40:47"),(599,"Chandler","Jenkins","1962-05-02 20:13:43"),(600,"Urielle","Webster","1958-10-23 17:48:27");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (601,"Jenette","Mays","1971-06-12 04:03:51"),(602,"Candace","Gonzalez","1961-04-17 12:55:36"),(603,"Leigh","Francis","1972-05-24 08:33:35"),(604,"Jamal","Carrillo","1990-08-11 12:38:26"),(605,"Kristen","Herring","1971-02-19 03:29:11"),(606,"Brandon","Weaver","1989-09-05 12:16:21"),(607,"Randall","Savage","1986-11-20 00:41:26"),(608,"Aspen","Norton","1959-09-15 15:52:49"),(609,"Tarik","Hill","1989-05-26 06:35:39"),(610,"Roary","Sullivan","1970-12-28 10:42:44");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (611,"Nolan","Miller","1966-01-29 20:22:06"),(612,"Julie","Colon","1958-12-23 18:07:12"),(613,"Robin","Weber","1960-12-31 22:59:37"),(614,"Charlotte","Vang","1987-08-19 00:51:45"),(615,"Barclay","Moss","1973-08-18 16:49:27"),(616,"Austin","Watkins","1981-03-06 03:19:58"),(617,"Holly","Velasquez","1963-12-23 17:45:29"),(618,"Constance","Jefferson","1970-10-25 01:22:02"),(619,"Vance","Clements","1973-08-07 08:57:23"),(620,"Jameson","Miller","1965-01-06 10:59:50");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (621,"Teegan","Bass","1959-05-23 03:20:41"),(622,"Georgia","Wolf","1984-09-20 19:41:47"),(623,"Donovan","Hull","1985-12-07 06:22:38"),(624,"Keane","Hernandez","1977-10-17 18:02:16"),(625,"Nigel","Valdez","1983-08-16 07:38:14"),(626,"Honorato","Justice","1978-11-07 03:50:36"),(627,"Ivor","Mejia","1988-01-19 22:17:16"),(628,"Sonya","Nichols","1958-03-02 21:18:09"),(629,"Adara","Douglas","1977-08-30 14:10:42"),(630,"Zenaida","Finch","1958-08-09 16:40:19");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (631,"Bruno","Decker","1982-06-20 19:32:46"),(632,"Dylan","Good","1969-11-25 03:46:40"),(633,"Brody","Matthews","1986-07-01 22:35:12"),(634,"Lael","Holland","1953-05-27 14:25:52"),(635,"Hyatt","Pope","1959-01-07 09:30:21"),(636,"Harriet","Joyce","1956-11-25 03:05:50"),(637,"Chaim","Wiley","1972-09-08 09:03:56"),(638,"Dylan","Mckay","1987-06-11 04:54:03"),(639,"Sybil","Velasquez","1961-12-09 11:53:56"),(640,"Violet","Santos","1979-01-07 00:57:10");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (641,"Anthony","Ayers","1964-09-27 12:30:28"),(642,"Amena","Owens","1977-07-18 05:00:16"),(643,"Oscar","Leon","1988-01-10 10:27:12"),(644,"Scarlet","Sherman","1988-05-19 06:15:41"),(645,"Yardley","Rocha","1959-10-15 18:13:04"),(646,"Rowan","Webster","1951-06-07 02:16:25"),(647,"Ayanna","Guerra","1955-09-27 22:35:45"),(648,"Wynne","Snyder","1976-04-02 19:06:36"),(649,"Rose","Sexton","1979-05-06 16:01:02"),(650,"Jael","Gilmore","1955-10-19 06:47:24");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (651,"Edward","Holmes","1980-04-16 02:56:00"),(652,"Dora","Roach","1961-08-20 02:49:33"),(653,"Chaim","Monroe","1957-06-19 13:08:53"),(654,"Owen","Foreman","1989-04-16 06:00:45"),(655,"Adam","Paul","1967-11-25 17:17:51"),(656,"Brian","Perez","1959-06-10 06:55:55"),(657,"Todd","Wilcox","1983-09-19 08:25:05"),(658,"Ishmael","Dixon","1961-06-18 12:20:20"),(659,"Marny","Medina","1965-08-22 22:41:52"),(660,"Brock","Lyons","1954-04-10 18:08:20");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (661,"Malachi","Hill","1975-08-16 20:54:27"),(662,"Inga","Weeks","1969-03-10 08:35:46"),(663,"Harper","Weeks","1954-01-09 08:06:08"),(664,"Cairo","Ramos","1966-05-07 02:27:42"),(665,"Benjamin","Walls","1957-06-01 17:13:03"),(666,"Grady","Briggs","1979-12-22 01:03:08"),(667,"Elizabeth","Christian","1960-03-27 10:14:04"),(668,"Kim","Oneil","1958-11-15 19:19:38"),(669,"Raya","Reyes","1980-09-10 15:20:53"),(670,"Haviva","Pitts","1953-06-25 15:14:19");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (671,"Illana","Oneill","1958-02-16 16:31:42"),(672,"Skyler","Vazquez","1954-01-12 06:48:33"),(673,"Quynn","Hudson","1982-11-07 03:10:23"),(674,"Autumn","Jefferson","1965-10-10 21:35:25"),(675,"Timothy","Ford","1953-05-19 13:53:21"),(676,"Amber","Wilkins","1981-09-27 04:12:09"),(677,"Knox","Sweet","1951-07-20 18:48:37"),(678,"Kasper","Horn","1974-10-29 08:29:23"),(679,"Ignatius","Sheppard","1986-10-31 02:00:37"),(680,"Breanna","Delacruz","1963-12-15 09:20:47");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (681,"Keith","Morse","1956-10-31 00:04:26"),(682,"Tanner","Mckee","1956-12-06 06:08:30"),(683,"Larissa","Holder","1987-03-01 03:41:16"),(684,"Joan","Hardy","1986-10-07 01:54:40"),(685,"Lucy","Silva","1972-11-22 13:19:53"),(686,"Samantha","Bean","1975-03-25 23:13:19"),(687,"Keiko","Cooley","1957-04-29 01:12:54"),(688,"Phelan","Davidson","1956-07-05 19:19:12"),(689,"Hayes","Hopper","1979-08-12 05:58:45"),(690,"Jenette","Hubbard","1982-08-23 21:19:16");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (691,"Gemma","Santos","1966-12-14 16:36:51"),(692,"Elvis","Patel","1979-03-16 18:25:12"),(693,"Camilla","Patrick","1986-07-23 18:18:57"),(694,"Deborah","Vazquez","1976-06-14 02:32:34"),(695,"Jerry","Hodge","1976-08-15 10:03:35"),(696,"Sean","Kirk","1962-10-30 22:55:55"),(697,"Victoria","Garrison","1957-07-09 03:12:04"),(698,"Jael","Norris","1951-04-06 11:03:14"),(699,"Joan","Byrd","1968-07-26 20:12:34"),(700,"Myles","Ballard","1952-04-29 15:09:12");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (701,"TaShya","Byers","1971-09-14 15:20:31"),(702,"Haley","Mathews","1956-06-17 18:05:40"),(703,"Astra","Martin","1957-04-13 08:27:02"),(704,"Geoffrey","Griffith","1976-08-24 02:08:41"),(705,"Magee","Hansen","1951-10-21 11:10:51"),(706,"Josiah","Sosa","1961-03-08 14:55:13"),(707,"Lesley","Hahn","1989-09-02 21:50:26"),(708,"Liberty","Pace","1966-07-19 19:56:28"),(709,"Teagan","Cervantes","1973-07-18 12:21:03"),(710,"Noelani","Cannon","1974-03-31 10:53:04");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (711,"Xenos","Thompson","1956-06-01 11:36:56"),(712,"Noel","Dickson","1987-04-03 14:25:22"),(713,"Ivor","Acevedo","1989-02-15 03:45:03"),(714,"Penelope","Hodges","1964-03-17 13:32:47"),(715,"Bernard","Bryan","1969-09-16 18:37:20"),(716,"Armando","Alford","1979-07-14 00:34:38"),(717,"Hannah","Mcclure","1983-11-18 00:52:47"),(718,"Vernon","Banks","1956-01-21 15:59:55"),(719,"Cyrus","Knight","1971-08-28 13:26:35"),(720,"Blair","Tate","1988-03-09 08:20:06");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (721,"Priscilla","Hewitt","1983-04-13 20:21:38"),(722,"Mollie","Pate","1966-03-14 14:09:43"),(723,"Evelyn","Fowler","1958-07-20 10:29:00"),(724,"Amethyst","Christian","1983-04-19 15:25:17"),(725,"Adrian","Newton","1973-08-22 05:57:56"),(726,"Ryan","Cannon","1959-02-27 19:28:31"),(727,"Rana","Bruce","1984-07-13 06:49:22"),(728,"Mohammad","Chang","1971-06-13 13:17:26"),(729,"Kylee","Vincent","1974-02-11 03:49:47"),(730,"Brian","Palmer","1968-06-21 22:20:52");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (731,"Emerald","Baker","1957-10-16 06:20:00"),(732,"Halee","Rush","1954-12-20 00:07:31"),(733,"Lisandra","Guy","1955-12-11 04:53:37"),(734,"Eliana","Cross","1963-03-11 15:29:37"),(735,"Freya","Mann","1986-07-31 17:17:45"),(736,"Tucker","Mayo","1984-09-27 12:09:44"),(737,"Dacey","Ellis","1970-11-01 14:27:51"),(738,"Lucas","Boone","1971-08-08 21:07:50"),(739,"Christine","Newton","1981-12-14 13:59:31"),(740,"Byron","Norton","1962-07-16 21:19:06");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (741,"Alisa","Salas","1971-05-12 08:02:52"),(742,"Britanney","Davidson","1990-01-18 12:44:58"),(743,"Bo","Sloan","1960-06-29 09:58:42"),(744,"Xerxes","Mcclure","1983-03-17 23:35:14"),(745,"Ciara","Gregory","1962-05-21 14:35:23"),(746,"Hannah","Valencia","1965-06-03 06:26:21"),(747,"Paki","Park","1985-04-02 04:48:50"),(748,"Thor","Kim","1984-08-29 18:49:01"),(749,"Camilla","Atkins","1965-03-19 23:55:53"),(750,"Anastasia","Santos","1978-05-03 21:32:51");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (751,"Reece","Chang","1957-11-24 23:03:28"),(752,"May","Graves","1966-08-02 12:46:29"),(753,"Mariam","Arnold","1964-04-07 01:50:05"),(754,"Karina","Blackburn","1976-07-27 16:43:33"),(755,"Cherokee","Hopper","1951-01-30 22:24:15"),(756,"Troy","Mendez","1981-02-23 00:47:29"),(757,"Rachel","Phelps","1989-10-01 03:52:38"),(758,"Lionel","White","1966-04-22 09:21:04"),(759,"Xerxes","Calhoun","1976-11-19 15:50:10"),(760,"Kelsie","Day","1983-03-26 16:29:24");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (761,"Simon","Warner","1965-09-23 04:14:11"),(762,"Jana","Frederick","1958-01-27 17:18:26"),(763,"Patrick","Goodwin","1965-03-11 22:03:49"),(764,"Ishmael","Brock","1960-05-07 04:43:22"),(765,"Amal","Pitts","1980-09-24 05:33:50"),(766,"Cullen","Mack","1978-07-22 06:21:39"),(767,"Amos","Villarreal","1980-05-24 10:54:59"),(768,"Grant","Oneal","1973-09-29 01:18:43"),(769,"Dara","Carson","1960-05-06 04:49:35"),(770,"Abra","Valdez","1988-02-23 05:03:34");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (771,"Dennis","Peters","1963-11-23 02:12:46"),(772,"Xyla","Mcclain","1963-10-13 12:31:32"),(773,"Eleanor","Duffy","1981-03-22 12:33:20"),(774,"Lionel","Stephenson","1971-03-17 09:47:21"),(775,"Emmanuel","Wheeler","1978-11-29 02:13:52"),(776,"Miranda","Mcgee","1980-11-28 18:41:48"),(777,"Byron","Mcintyre","1984-07-04 01:51:48"),(778,"Ulric","Preston","1973-10-24 09:19:32"),(779,"Athena","Payne","1956-06-04 20:30:39"),(780,"Madonna","Casey","1972-07-02 14:32:21");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (781,"Rinah","Gallegos","1989-02-19 05:55:50"),(782,"Finn","White","1963-07-21 18:23:17"),(783,"Castor","Wilkinson","1954-06-23 02:43:35"),(784,"Glenna","Richardson","1961-05-27 03:43:06"),(785,"Walter","Cook","1975-08-22 23:41:46"),(786,"Eden","Simpson","1974-09-17 00:36:59"),(787,"Tana","Cummings","1982-03-22 04:12:15"),(788,"Devin","Gamble","1965-06-04 10:27:34"),(789,"Alfreda","Mclean","1965-09-23 13:13:46"),(790,"Jolie","Daniel","1988-02-25 23:23:41");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (791,"Sawyer","Osborne","1965-08-09 21:03:16"),(792,"Lillian","Moss","1968-04-24 15:47:12"),(793,"Hyacinth","Pitts","1979-11-25 17:37:25"),(794,"Raven","Conner","1964-01-23 09:43:56"),(795,"Rigel","Rose","1968-02-15 02:20:11"),(796,"Ina","Patrick","1973-09-14 08:44:35"),(797,"Lillian","Rojas","1977-04-30 15:18:28"),(798,"Channing","Hahn","1973-08-02 16:36:13"),(799,"Aidan","Carver","1990-11-29 01:46:59"),(800,"Haley","Carney","1968-12-02 11:24:38");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (801,"Declan","Roach","1978-09-20 06:12:02"),(802,"Olga","Dalton","1958-08-11 23:38:34"),(803,"Keely","Knowles","1952-11-28 20:43:31"),(804,"Kitra","Ortega","1971-02-18 09:18:56"),(805,"Simon","Hobbs","1977-06-12 01:16:48"),(806,"Francis","Jordan","1988-04-22 19:28:55"),(807,"Elaine","Rodriguez","1971-07-28 00:36:30"),(808,"Julie","Lester","1984-01-10 05:33:04"),(809,"Joseph","Terrell","1972-07-08 23:16:41"),(810,"Ivor","Clark","1953-04-21 12:58:33");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (811,"Caryn","Cain","1968-07-19 08:23:28"),(812,"Amena","Leon","1976-07-09 06:33:45"),(813,"Trevor","Holden","1978-03-14 07:57:03"),(814,"Uriah","Brady","1976-08-23 14:24:43"),(815,"Whoopi","Waters","1986-08-25 16:47:04"),(816,"Alden","Bartlett","1956-03-29 20:31:36"),(817,"Alan","Holder","1968-07-25 18:42:29"),(818,"Kelsie","Oneal","1980-12-17 07:33:15"),(819,"Reece","Gentry","1958-07-15 12:05:10"),(820,"Lila","Clements","1986-08-27 08:44:30");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (821,"Cullen","Lancaster","1985-11-02 22:56:05"),(822,"Rachel","Mckenzie","1952-10-17 17:24:51"),(823,"Mia","Bradley","1964-03-25 07:59:41"),(824,"Macy","Erickson","1974-09-04 10:44:11"),(825,"Neve","Bonner","1950-12-01 07:14:20"),(826,"Oren","Irwin","1980-07-20 18:42:47"),(827,"Caesar","Petersen","1987-02-28 10:08:34"),(828,"Velma","Sears","1957-03-25 12:57:18"),(829,"Anjolie","Shaffer","1960-05-27 07:08:23"),(830,"Lester","Harrington","1988-05-01 15:59:47");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (831,"Mara","Jacobs","1986-08-02 03:18:57"),(832,"Barclay","Boyer","1982-03-21 21:48:45"),(833,"Stacy","Rios","1970-02-05 03:18:48"),(834,"Gage","Dyer","1990-01-03 09:23:40"),(835,"Iris","Alexander","1980-03-20 21:00:25"),(836,"Ivan","Woodward","1974-12-04 11:55:23"),(837,"Riley","Horne","1972-01-04 19:25:55"),(838,"Lacota","Sears","1979-02-08 21:25:07"),(839,"Charde","Becker","1953-10-31 10:02:34"),(840,"Kim","Arnold","1962-08-30 22:28:34");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (841,"Erasmus","Cherry","1966-11-29 23:46:27"),(842,"Kato","Underwood","1980-08-31 22:10:23"),(843,"Kasper","Hammond","1963-12-19 08:10:20"),(844,"Mark","Ford","1964-05-16 14:33:36"),(845,"Sheila","Mueller","1958-05-14 18:46:35"),(846,"Acton","Hodge","1982-03-23 19:22:23"),(847,"Carson","Woodward","1980-08-26 13:30:34"),(848,"Christen","Dillon","1956-09-22 08:10:56"),(849,"Rosalyn","Macias","1988-02-26 21:53:18"),(850,"Vance","Gilliam","1977-07-07 00:34:16");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (851,"Belle","Doyle","1980-04-01 01:53:55"),(852,"Kylan","Vance","1961-12-30 21:01:03"),(853,"Samantha","Nichols","1962-06-28 01:11:49"),(854,"Matthew","Boone","1980-10-08 07:42:48"),(855,"Inez","Bullock","1989-02-23 10:43:09"),(856,"Wing","Blevins","1985-03-15 14:35:22"),(857,"Colette","Ross","1967-10-13 04:02:35"),(858,"Yvonne","Sanders","1965-04-09 19:58:51"),(859,"Genevieve","Wyatt","1957-09-15 21:35:15"),(860,"Sylvester","Cote","1966-08-07 21:14:45");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (861,"Finn","Acevedo","1960-12-19 17:57:54"),(862,"Hadassah","Whitaker","1976-10-21 23:57:26"),(863,"Driscoll","Anderson","1956-03-04 20:30:53"),(864,"Peter","Peterson","1972-05-22 16:00:13"),(865,"Raymond","Wheeler","1965-12-25 15:32:12"),(866,"Judith","Gamble","1986-05-20 23:00:15"),(867,"Darius","Talley","1962-09-15 09:30:17"),(868,"Xenos","Mack","1980-11-13 11:27:51"),(869,"Kiara","Terry","1984-03-02 23:45:15"),(870,"Tallulah","Bowman","1959-11-24 20:54:28");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (871,"Lucy","Pace","1958-09-18 13:44:00"),(872,"Tatum","Summers","1968-08-27 15:45:21"),(873,"Edan","Giles","1964-01-03 04:20:18"),(874,"Kyra","Melton","1979-11-08 21:52:05"),(875,"Rana","Conrad","1975-08-28 14:25:16"),(876,"Azalia","Rush","1985-05-19 07:56:41"),(877,"Carla","Conley","1951-08-17 11:37:04"),(878,"Guinevere","Garrett","1989-01-30 05:07:34"),(879,"Kirk","Albert","1978-08-24 05:06:03"),(880,"Aphrodite","Patel","1953-08-03 02:41:06");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (881,"Karyn","Vargas","1988-04-19 15:54:52"),(882,"Rowan","Stokes","1970-01-03 22:17:45"),(883,"Medge","Walsh","1955-09-22 07:07:39"),(884,"Kelsey","Boyle","1963-02-06 20:42:14"),(885,"Ferris","Woodward","1982-09-05 00:41:10"),(886,"Karen","Boyer","1984-02-21 16:48:19"),(887,"Alfonso","Murray","1962-05-08 04:31:04"),(888,"Macey","Zimmerman","1990-06-16 02:24:41"),(889,"Duncan","Poole","1953-01-27 10:55:21"),(890,"Shelby","Snider","1954-02-28 09:29:42");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (891,"Hilel","Cohen","1955-08-24 17:24:46"),(892,"Lavinia","Farrell","1955-11-24 11:33:27"),(893,"Katell","Burnett","1971-09-17 05:56:52"),(894,"Hanae","Mckay","1980-06-11 02:11:32"),(895,"Jenette","Roach","1964-12-20 19:02:08"),(896,"Idona","Dickerson","1959-03-04 16:43:07"),(897,"Clinton","Clarke","1973-11-22 00:25:02"),(898,"Mark","Gonzalez","1968-06-18 18:00:21"),(899,"Galvin","Valencia","1988-06-21 05:00:11"),(900,"Angelica","Adkins","1984-01-20 19:43:21");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (901,"Thaddeus","Collins","1975-11-08 04:11:38"),(902,"Graiden","Keller","1956-09-15 22:41:55"),(903,"Allistair","Brock","1976-06-24 15:56:01"),(904,"Libby","Morgan","1957-01-03 10:11:03"),(905,"Nevada","Meyers","1964-05-07 11:45:29"),(906,"Ali","Daugherty","1982-09-02 19:26:28"),(907,"Jordan","Petersen","1972-01-05 11:29:51"),(908,"Conan","Frederick","1969-05-07 05:12:08"),(909,"Astra","Wallace","1966-10-30 04:23:29"),(910,"Thomas","Holder","1955-09-30 03:35:51");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (911,"Sophia","Irwin","1963-04-21 14:40:11"),(912,"Hilda","Sloan","1951-05-14 22:57:58"),(913,"Cullen","Mccullough","1960-12-20 09:14:12"),(914,"Yoshio","West","1989-12-24 16:09:35"),(915,"Marsden","Pearson","1985-04-18 09:25:00"),(916,"Dominic","Gay","1976-08-11 01:45:33"),(917,"Carl","Lester","1984-04-08 09:36:01"),(918,"Stella","Herrera","1965-09-05 02:51:08"),(919,"Omar","Aguirre","1956-05-20 08:42:48"),(920,"Hayley","Hoover","1977-11-10 06:46:43");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (921,"Lacota","Sloan","1981-02-20 04:33:03"),(922,"Deborah","Wagner","1961-06-20 12:38:30"),(923,"Kitra","Foster","1975-08-30 14:05:01"),(924,"Candace","Talley","1989-06-01 00:36:14"),(925,"Eric","Wiggins","1961-11-23 19:26:34"),(926,"Kay","Mathews","1972-11-19 05:11:33"),(927,"Otto","Dean","1960-08-14 22:30:58"),(928,"Kitra","Barnett","1987-10-27 14:10:42"),(929,"Paki","Andrews","1985-06-23 03:46:36"),(930,"Clark","Kerr","1962-02-19 22:11:42");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (931,"Lila","Greer","1969-03-30 11:46:12"),(932,"George","Mosley","1971-06-10 17:16:27"),(933,"Hedda","Stanley","1978-03-05 16:04:21"),(934,"Angelica","Pierce","1980-04-18 20:04:07"),(935,"Dylan","Giles","1972-10-26 08:20:27"),(936,"Cherokee","Byrd","1973-02-23 00:35:12"),(937,"Carol","Lester","1954-04-16 09:18:20"),(938,"Adria","Alford","1965-08-04 04:09:05"),(939,"Daquan","Howe","1970-08-31 12:56:58"),(940,"Lani","Robbins","1971-02-24 01:54:00");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (941,"Carissa","Nieves","1956-02-11 18:39:43"),(942,"Wynne","Randall","1958-11-11 22:09:38"),(943,"Raya","Carroll","1967-02-17 08:02:28"),(944,"Giacomo","Walsh","1975-04-16 11:14:11"),(945,"Ella","Mathis","1980-11-11 05:21:23"),(946,"Hakeem","Sims","1971-05-13 17:45:47"),(947,"Yoshi","Pacheco","1965-09-08 06:44:21"),(948,"Nola","Blackwell","1959-03-05 00:56:20"),(949,"Clare","Bates","1987-06-26 18:57:48"),(950,"Owen","Fuentes","1966-10-22 09:11:49");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (951,"Chelsea","Kinney","1977-05-25 07:22:06"),(952,"Wang","Baker","1990-06-04 02:28:57"),(953,"Rachel","Reilly","1978-12-30 02:15:35"),(954,"Tucker","Woods","1968-11-19 15:47:38"),(955,"Dacey","Cervantes","1963-01-12 01:58:42"),(956,"Aileen","Booker","1955-05-07 08:23:36"),(957,"Rylee","Doyle","1961-08-12 00:55:18"),(958,"Sarah","Mendez","1956-10-28 03:34:18"),(959,"May","Lester","1971-01-19 02:17:18"),(960,"Yael","Patrick","1989-03-04 16:55:27");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (961,"Brandon","Clarke","1953-04-26 23:31:06"),(962,"Shelby","Parrish","1952-05-26 19:30:52"),(963,"Norman","Bartlett","1966-04-12 01:21:08"),(964,"Abbot","Salinas","1966-05-24 06:17:01"),(965,"Lucas","Meadows","1973-10-11 13:08:45"),(966,"Ainsley","Yang","1976-02-21 04:56:09"),(967,"Aileen","Cannon","1978-02-03 02:23:44"),(968,"Christen","Lawson","1953-05-04 15:48:54"),(969,"Shafira","Butler","1989-01-01 14:44:51"),(970,"Skyler","Hays","1989-06-25 23:00:34");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (971,"Nathaniel","Mccall","1966-07-10 10:22:58"),(972,"Fredericka","Curtis","1990-02-26 07:14:48"),(973,"Dennis","Parker","1973-12-08 06:57:17"),(974,"Alice","Mathews","1954-03-18 10:05:07"),(975,"Emmanuel","Townsend","1989-11-08 04:02:26"),(976,"Tallulah","Leach","1987-03-17 12:36:22"),(977,"Lael","Winters","1954-11-19 05:41:54"),(978,"Amos","Floyd","1971-07-07 14:14:30"),(979,"Addison","Nichols","1989-04-08 14:24:53"),(980,"Amos","Knox","1966-06-10 11:31:45");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (981,"Regan","Campos","1968-02-13 02:30:05"),(982,"Merrill","Wolfe","1965-11-06 10:44:37"),(983,"Fredericka","Miles","1954-07-28 22:08:26"),(984,"Geraldine","Jensen","1978-08-01 02:33:04"),(985,"Sharon","Morales","1989-06-09 08:05:23"),(986,"Nyssa","Vaughn","1969-12-04 17:57:19"),(987,"Orlando","French","1990-07-04 01:29:11"),(988,"Peter","Daugherty","1983-12-10 19:40:55"),(989,"Stephanie","Hayden","1975-02-08 07:10:15"),(990,"Harriet","Burton","1973-08-22 01:29:26");
INSERT INTO `customer` (`id`,`firstName`,`lastName`,`birthdate`) VALUES (991,"Kenneth","Wheeler","1952-06-22 17:13:29"),(992,"Meghan","Clark","1972-06-02 04:42:03"),(993,"Debra","Maddox","1974-08-30 11:34:13"),(994,"Ursula","Bryant","1963-06-13 10:48:23"),(995,"Dillon","Glenn","1964-06-15 11:28:45"),(996,"Juliet","Colon","1977-03-07 07:53:18"),(997,"William","Cole","1981-08-13 15:28:44"),(998,"Doris","Craft","1985-12-14 04:12:41"),(999,"Camilla","Combs","1960-01-29 12:04:57"),(1000,"Cheyenne","Mckinney","1975-03-22 03:53:55");
```

- `git checkout Part 8.1`
```java
@RequiredArgsConstructor
@Configuration
public class ThreadConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(5)
                .reader(reader())
                .processor(new ItemProcessor<String, String>() {
                    @Override
                    public String process(String item) throws Exception {
                        System.out.println("Processor => Thread = " + Thread.currentThread().getName() + "item = " + item);
                        return item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(List<? extends String> items) throws Exception {
                        System.out.println("Writer => Thread = " + Thread.currentThread().getName() + "item = " + items);
                    }
                })
                .taskExecutor(new SimpleAsyncTaskExecutor())
                .throttleLimit(4)
                .build();
    }

    @Bean
    public SynchronizedItemStreamReader<String> reader() {

        List<String> items = new ArrayList<>();

        for(int i = 0; i < 20; i++) {
            items.add(String.valueOf(i));
        }
        CustomListItemReader<String> customListItemReader = new CustomListItemReader(items);
        SynchronizedItemStreamReader synchronizedItemStreamReader =  new SynchronizedItemStreamReader();
        synchronizedItemStreamReader.setDelegate(customListItemReader);

        return synchronizedItemStreamReader;
    }

    @Bean
    public ListItemReader<String> reader2() {

        List<String> items = new ArrayList<>();

        for(int i = 0; i < 20; i++) {
            items.add(String.valueOf(i));
        }
        CustomListItemReader<String> customListItemReader = new CustomListItemReader(items);

        return customListItemReader;
    }
}
```
```java
public class CustomListItemReader<T> extends ListItemReader<T> implements ItemStreamReader<T> {

    public CustomListItemReader(List<T> list) {
        super(list);
    }

    @Override
    public T read() {
        T read = super.read();
        System.out.println("Reader :" + read + " => Thread = " + Thread.currentThread().getName());
        return read;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {

    }

    @Override
    public void update(ExecutionContext executionContext) throws ItemStreamException {

    }

    @Override
    public void close() throws ItemStreamException {

    }
}
```

## AsyncItemProcessor / AsyncItemWriter
1. 기본 개념
    - Step 안에서 ItemProcessor가 비동기적으로 동작하는 구조
    - AsyncItemProcessor와 AsyncItemWriter가 함께 구성이 되어야 함
    - AsyncItemProcessor로부터 AsyncItemWriter가 받는 최종 결과값은 `List<Future<T>>` 타입이며 비동기 실행이 완료될 때까지 대기한다.
    - spring-batch-integration 의존성이 필요하다.

2. Main Thread
    - Job(Step) -> ItemReader -> **AsyncItemProcessor** -> delegate -> ItemProcessor(Thread)
    - Job(Step) -> ItemReader -> **AsyncItemProcessor** -> `List<Future>` -> **AsyncItemWriter** -> delegate -> ItemWriter
    <br/>

    - AsyncItemProcessor
        - `ItemProcessor<I,O> delegate` : 실제 process를 수행하는 ItemProcessor
        - TaskExecutor taskExecutor = new SyncTaskExecutor() : Thread를 생성하고 Task 할당
        - `FutureTask<O> task = new FutureTask<>(Callable<V>)` : Thread가 수행하는 Task로서 Callable를 실행시키고 결과를 `Future<V>`에 담아 반환
        - -> process()
    <br/>

    - Job -> TaskletStep -> ChunkOrientedTasklet -> ItermReader -> `Chunk<inputs>`
    - -> AsyncItemProcessor( -> delegate -> TaskExecutor(WorkerThread(FutureTask(ItemProcessor)))) -> `List<Future<T>>`
    - -> AsyncItemWriter( -> delegate -> ItemWriter(List(Future(Item)))) -> write(list) : 비동기 실행 결과 값들을 모두 받아오기까지 대기 -> DB
        - write(list) : for (`Future<T>` future : items) list.add(future.get());
            - future.get() -> **wait**

3. API
    - processor() : 스레드 풀 개수만큼 스레드가 생성되어 비동기로 실행된다. 내부적으로 실제 ItemProcessor에게 실행을 위임하고 결과를 Future에 저장한다.
    - writer() : 비동기 실행 결과 값들을 모두 받아오기까지 대기함. 내부적으로 실제 ItemWriter에게 최종 결과값을 넘겨주고 실행을 위임한다.
    ```java
    public Step step() throws Exception {
        return stepBuilderFactory.get("step")
                .chunk(100) // 청크 개수 설정
                .reader(pagingItemReader()) // ItemReader 설정 : 비동기 실행 아님
                .processor(asyncItemProcessor()) // 비동기 실행을 위한 AsyncItemProcessor 설정
                .writer(asyncItemWriter()) // AsyncItemWriter 설정
                .build(); // TaskletStep 생성
    }
    ```

- Dependency
> spring-batch-integration

- `git checkout Part8.2`
```java
@RequiredArgsConstructor
@Configuration
public class AsyncConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
//                .start(step1()) // 동기식
                .start(asyncStep1()) // 비동기식
                .listener(new StopWatchJobListener()) // 성능 비교
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .chunk(100)
                .reader(pagingItemReader())
                .processor(customItemProcessor())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public Step asyncStep1() throws Exception {
        return stepBuilderFactory.get("asyncStep1")
                .chunk(100)
                .reader(pagingItemReader())
                .processor(asyncItemProcessor())
                .writer(asyncItemWriter())
                .taskExecutor(taskExecutor())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> pagingItemReader() {
        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setPageSize(100);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        reader.setQueryProvider(queryProvider);

        return reader;
    }

    @Bean
    public ItemProcessor customItemProcessor() {
        return new ItemProcessor<Customer, Customer>() {
            @Override
            public Customer process(Customer item) throws Exception {

                Thread.sleep(10); // 비동기식은 기다리지 않음

                return new Customer(item.getId(),
                        item.getFirstName().toUpperCase(),
                        item.getLastName().toUpperCase(),
                        item.getBirthdate());
            }
        };
    }

    @Bean
    public JdbcBatchItemWriter customItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }

    @Bean
    public AsyncItemProcessor asyncItemProcessor() throws Exception {
        AsyncItemProcessor<Customer, Customer> asyncItemProcessor = new AsyncItemProcessor();

        asyncItemProcessor.setDelegate(customItemProcessor());
        asyncItemProcessor.setTaskExecutor(new SimpleAsyncTaskExecutor());
//        asyncItemProcessor.setTaskExecutor(taskExecutor());
        asyncItemProcessor.afterPropertiesSet();

        return asyncItemProcessor;
    }

    @Bean
    public AsyncItemWriter asyncItemWriter() throws Exception {
        AsyncItemWriter<Customer> asyncItemWriter = new AsyncItemWriter<>();

        asyncItemWriter.setDelegate(customItemWriter());
        asyncItemWriter.afterPropertiesSet();

        return asyncItemWriter;
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setThreadNamePrefix("async-thread-");
        return executor;
    }
}
```
```java
@Data
@AllArgsConstructor
public class Customer {

    private final long id;
    private final String firstName;
    private final String lastName;
    private final Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```
```java
public class StopWatchJobListener implements JobExecutionListener { // 성능 비교

    @Override
    public void beforeJob(JobExecution jobExecution) {
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long time = jobExecution.getEndTime().getTime() - jobExecution.getStartTime().getTime();
        System.out.println("==========================================");
        System.out.println("총 소요된 시간 : " + time);
        System.out.println("==========================================");
    }
}
```

- Run Configuration
> -job.name=batchJob message=20240101
> Customer 테이블 삭제

- 결과1 (동기식 Thread.sleep(10))
> Executing step: [step1]
> Step: [step1] executed in 176ms
> 총 소요시간: 1927

- 결과2 (동기식 Thread.sleep(30))
> Executing step: [step1]
> Step: [step1] executed in 171ms
> 총 소요시간 : 4367

- 결과3 (비동기식 Thread.sleep(10))
> Executing step: [asyncStep1]
> Step: [asyncStep1] executed in 256ms
> 총 소요시간 : 344

- 참고
> FutureTask, Future, Callable, RunnableFuture, AsyncItemProcessor, AsyncItemWriter, SimpleChunkProcessor

## Multi-threaded Step
1. 기본 개념
    - Step 내에서 멀티 스레드로 Chunk 기반 처리가 이루어지는 구조
    - TaskExecutorRepeatTemplate이 반복자로 사용되며 설정한 개수 **(throttleLimit)** 만큼의 스레드를 생성하여 수행한다.
2. 구조
    - Job -> Step -> TaskletStep -> TaskExecutorRepeatTemplate
    - -> Worker1, 2, 3, ... -> new
    - -> **Runnable**(RepeatCallback(ChunkOrientedTasklet(**Thread-safe**(**ItemReader -> ItemProcesor -> ItemWriter**) <- Worker1, 2, 3, ... (**`Chunk<inputs>` -> `Chunk<Outputs>` -> `List<Item>`**))))
        - **ItemReader는 Thread-safe인지 반드시 확인해야 한다.**
            - 데이터를 소스로부터 읽어오는 역할이기 때문에 스레드마다 중복해서 데이터를 읽어오지 않도록 **동기화가 보장**되어야 한다.
        - **스레드마다 새로운 Chunk가 할당되어 데이터 동기화가 보장된다.**
            - 스레드끼리 Chunk를 서로 공유하지 않는다.
    - TaskExecutorRepeatTemplate
        - int throttleLimit = DEFAULT_THROTTLE_LIMIT : 조절 제한 개수
        - TaskExecutor taskExecutor = new SyncTaskExecutor() : Thread를 조절 제한 수만큼 생성하고 Task를 할당
        - -> iterate() : ExecutingRunnable 객체 생성
        - -> run() : result = callback.doInIteration(context) 상태값 저장
    - ExecutingRunnable
        - RepeatCallback callback : 반복기에 의해 호출되는 콜백
        - `ResultQueue<ResultHolder>` queue : 최종 RepeatStatus를 담고 있는 Queue
        - public void run() : RepeatCallback 로직 수행

3. API
    ```java
    public Step step() throws Exception {
        return stopBuilderFactory.get("step")
                .<Customer, Customer>chunk(100)
                .reader(pagingItemReader()) // Thread-safe한 ItemReader 설정
                .processor(customerItemProcessor())
                .writer(customItemWriter())
                .taskExecutor(taskExecutor()) // 스레드 생성 및 실행을 위한 taskExecutor 설정
                .build();
    }
    ```

- `git checkout Part8.3`
```java
@RequiredArgsConstructor
@Configuration
public class MultiThreadStepConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .listener(new StopWatchJobListener())
                .build();
    }

    @Bean
    public Step step1() throws Exception {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(100)
                .reader(pagingItemReader()) // customItemReader()
                .listener(new CustomReadListener())
                .processor((ItemProcessor<Customer, Customer>) item -> item)
                .listener(new CustomProcessListener())
                .writer(customItemWriter())
                .listener(new CustomWriteListener())
                .taskExecutor(taskExecutor()) // 없으면 동기 단일스레드, 있으면 비동기 멀티스레드
//                .throttleLimit(2)
                .build();
    }

    @Bean
    public JdbcCursorItemReader<Customer> customItemReader() { // 동기화 처리되지 않은 ItemReader 클래스
        return new JdbcCursorItemReaderBuilder()
                .name("jdbcCursorItemReader")
                .fetchSize(100)
                .sql("select id, firstName, lastName, birthdate from customer order by id")
                .beanRowMapper(Customer.class)
                .dataSource(dataSource)
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Customer> pagingItemReader() { // 동기화 처리된 ItemReader 클래스
        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setPageSize(100);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        reader.setQueryProvider(queryProvider);

        return reader;
    }

    @Bean
    public JdbcBatchItemWriter customItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setThreadNamePrefix("async-thread-");
        return executor;
    }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {

    private long id;
    private String firstName;
    private String lastName;
    private Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```
```java
public class CustomReadListener implements ItemReadListener<Customer> {

    @Override
    public void beforeRead() {

    }

    @Override
    public void afterRead(Customer item) {
        System.out.println("Thread : " + Thread.currentThread().getName() + ", read item : " + item.getId());
    }

    @Override
    public void onReadError(Exception ex) {

    }
}
```
```java
public class CustomProcessListener implements ItemProcessListener<Customer, Customer> {
    @Override
    public void beforeProcess(Customer item) {

    }

    @Override
    public void afterProcess(Customer item, Customer result) {
        System.out.println("Thread : " + Thread.currentThread().getName() + ", process item : " + item.getId());
    }

    @Override
    public void onProcessError(Customer item, Exception e) {

    }
}
```
```java
public class CustomWriteListener implements ItemWriteListener<Customer> {

    @Override
    public void beforeWrite(List<? extends Customer> items) {

    }

    @Override
    public void afterWrite(List<? extends Customer> items) {
        System.out.println("Thread : " + Thread.currentThread().getName() + ", write items : " + items.size());

    }

    @Override
    public void onWriteError(Exception exception, List<? extends Customer> items) {

    }
}
```
```java
public class StopWatchJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long time = jobExecution.getEndTime().getTime() - jobExecution.getStartTime().getTime();
        System.out.println("==========================================");
        System.out.println("총 소요된 시간 : " + time);
        System.out.println("==========================================");
    }
}
```

- 결과1 (단일스레드)
> Thread : main, read item : 1, ... , 100, process item 1, ... , 100, write items : 100
> Thread : main, read item : 101, ... , 200, process item 101, ... , 200, write items : 100
> ...
> 총 소요시간 : 997

- 결과2 (멀티스레드 .taskExecutor(taskExecutor()), executor.setCorePoolSize(4))
> Thread : async-thread4, 2, 1, 3, read item : 2, 4, ... 103, 100, process item 14, ... , 203, write items : 100
> 총 소요시간 : 559

- 결과3 (멀티스레드 .taskExecutor(taskExecutor()), executor.setCorePoolSize(2))
> Thread : async-thread1, 2, read item : 1, 2, ..., 100, process item 1, ... , 100, write items : 100 (대기 상태가 발생하지 않기 때문에 async-thread3, 4 등이 발생하지 않음)
> 총 소요시간 : 739

- 결과4 (.customItemReader())
> Thread : async-thread2, read item : 2
> Thread : async-thread1, read item : 2
> Unexpected cursor position change (JdbcCursorItemReader, JpaCursorItemReader 동기화 처리되지 않음)

- 참고
> ThreadPoolExecutor, ThreadPoolTaskExecutor, ExecutorConfigurationSupport, SimpleChunkProvider, ChunkOrientedTasklet
> JdbcCusorItemReader

## Parallel Steps
1. 기본 개념
    - SplitState를 사용해서 여러 개의 Flow들을 병렬적으로 실행하는 구조
    - 실행이 다 완료된 후 FlowExecutionStatus 결과들을 취합해서 다음 단계 결정을 한다.

2. 구조
    - Job -> SimpleFlow -> SplitState(flows) -> TaskExecutor
    - -> Worker(FutureTask(SimpleFlow)) -> results -> FlowExecution ->
    - ->`Collection<FlowExecution>` -> aggregator.aggregate(results) -> FlowExecutionAggregator -> SimpleFlow
        - return FlowExecutionStatus : COMPLETED, STOPPED, FAILED, UNKNOWN
        - 최종 실행결과의 상태값을 반환하여 다음 Step 결정하도록 한다.
    - SplitState
        - `Collection<Flow>` flows : 병렬로 수행할 Flow들을 담은 컬렉션
        - TaskExecutor taskExecutor : Thread를 생성하고 Task를 할당
        - FlowExecutionAggregator aggregator : 병렬로 수행 후 하나의 종료 상태로 집계하는 클래스
        - FlowExecutionStatus handle(final FlowExecutor executor)
            - `FutureTask<FlowExecution>`

3. API
    ```java
    public Job job() {
        return jobBuilderFactory.get("job")
                .start(flow1()) // Flow1을 생성한다.
                .split(TaskExecutor).add(flow2(), flow3()) // Flow2와 3을 생성하고 총 3개의 Flow를 합친다. taskExecutor에서 Flow 개수만큼 스레드를 생성해서 각 Flow를 실행시킨다.
                .next(flow4()) // Flow4는 split 처리가 완료된 후 실행이 된다.
                .end()
                .build();
    }
    ```

- `git checkout Part8.4`
```java
@RequiredArgsConstructor
@Configuration
public class ParallelStepConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(flow1())
                .split(taskExecutor()).add(flow2()) // .next(flow2())
                .end()
                .listener(new StopWatchJobListener())
                .build();
    }

    @Bean
    public Flow flow1() {

        TaskletStep step = stepBuilderFactory.get("step1")
                            .tasklet(tasklet()).build();

        return new FlowBuilder<Flow>("flow1")
                .start(step)
                .build();
    }

    @Bean
    public Flow flow2() {

        TaskletStep step1 = stepBuilderFactory.get("step2")
                .tasklet(tasklet()).build();

        TaskletStep step2 = stepBuilderFactory.get("step3")
                .tasklet(tasklet()).build();

        return new FlowBuilder<Flow>("flow2")
                .start(step1)
                .next(step2)
                .build();
    }

    @Bean
    public Tasklet tasklet() {
        return new CustomTasklet();
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(4);
        executor.setThreadNamePrefix("async-thread-");
        return executor;
    }
}
```
```java
@RequiredArgsConstructor
@Configuration
public class ParallelStepConfiguration2 {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {

        FlowBuilder<Flow> flowBuilder = new FlowBuilder<>("split");

        Flow flow = flowBuilder.split(taskExecutor())
                .add(flow1(), flow2())
                .end();

        return jobBuilderFactory.get("batchJob2")
                .start(customStep1())
                .next(customStep2())
                .on("COMPLETED").to(flow)
                .end()
                .build();
    }

    @Bean
    public Step customStep1() {
        return stepBuilderFactory.get("customStep1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("customStep1 was executed");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step customStep2() {
        return stepBuilderFactory.get("customStep2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("customStep2 was executed");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Flow flow1() {
        return new FlowBuilder<Flow>("flow1")
                .start(stepBuilderFactory.get("step1")
                        .tasklet(tasklet()).build())
                .build();
    }

    @Bean
    public Flow flow2() {
        return new FlowBuilder<Flow>("flow2")
                .start(stepBuilderFactory.get("step2")
                        .tasklet(tasklet()).build())
                .next(stepBuilderFactory.get("step3")
                        .tasklet(tasklet()).build())
                .build();
    }

    @Bean
    public Tasklet tasklet() { // Tasklet Bean으로 생성 -> 동기화 고려 필요
        return new CustomTasklet();
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setThreadNamePrefix("parallel-thread-");
        return executor;
    }
}
```
```java
public class CustomTasklet implements Tasklet {

    private long sum = 0; // 멤버변수
    private Object lock = new Object();

    @Override
    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {

        // Thread.sleep(1000);
        // long sum = 0;
        synchronized (lock) { // 동시성 문제 해결
            for (int i = 0; i < 1000000000; i++) { // (약 10억)
                sum++;
            }
            System.out.println(String.format("%s has been executed on thread %s", chunkContext.getStepContext().getStepName(), Thread.currentThread().getName()));
            System.out.println(String.format("sum : %d", sum));
        }
        return RepeatStatus.FINISHED;
    }
}
```
```java
public class StopWatchJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long time = jobExecution.getEndTime().getTime() - jobExecution.getStartTime().getTime();
        System.out.println("==========================================");
        System.out.println("총 소요된 시간 : " + time);
        System.out.println("==========================================");
    }
}
```

- 결과1 (100000000) -> **동시성 문제 : sum, CustomTasklet 공유, 여러 Thread가 동시 쓰기 작업 때문**
> Executing step: [step1]<br/>
> Executing step: [step2]<br/>
> step1 has been executed on thread async-thread-2<br/>
> sum : 1004632415<br/>
> step2 has been executed on thread async-thread-1<br/>
> sum : 1006402238<br/>
> Step : [step2] executed in 12s793ms<br/>
> Step : [step1] executed in 12s798ms<br/>
> Executing step: [step3]<br/>
> step3 has been executed on thread async-thread-1<br/>
> sum : 2006402238<br/>
> Step: [step3] executed in 515ms<br/>
> 총 소요시간 : 13443

- 결과2 (10000)
> Executing step: [step2]<br/>
> Executing step: [step1]<br/>
> step2 has been executed on thread async-thread-1<br/>
> sum : 10000<br/>
> step2 has been executed on thread async-thread-2<br/>
> sum : 20000<br/>
> Step : [step2] executed in 67ms<br/>
> Step : [step1] executed in 55ms<br/>
> Executing step: [step3]<br/>
> step3 has been executed on thread async-thread-1<br/>
> sum : 30000<br/>
> Step: [step3] executed in 43ms<br/>
> 총 소요시간 : 249

- 결과3 (100000000, .next(flow2()))
> Executing step: [step1]<br/>
> step2 has been executed on thread main<br/>
> sum : 100000000<br/>
> Step : [step1] executed in 123ms<br/>
> Executing step: [step2]<br/>
> step2 has been executed on thread main<br/>
> sum : 200000000<br/>
> Step : [step2] executed in 92ms<br/>
> Executing step: [step3]<br/>
> step3 has been executed on thread main<br/>
> sum : 300000000<br/>
> Step: [step3] executed in 88ms<br/>
> 총 소요시간 : 209

- 결과4 (100000000, synchronized (lock))
> Executing step: [step2]<br/>
> Executing step: [step1]<br/>
> step2 has been executed on thread async-thread-1<br/>
> sum : 100000000<br/>
> Step : [step2] executed in 110ms<br/>
> step1 has been executed on thread async-thread-2<br/>
> sum : 200000000<br/>
> Step : [step1] executed in 144ms<br/>
> Executing step: [step3]<br/>
> step3 has been executed on thread async-thread-1<br/>
> sum : 300000000<br/>
> Step: [step3] executed in 104ms<br/>
> 총 소요시간 : 417

- 결과4 (sum 지역변수 할당, synchronized (lock) X)
> Executing step: [step2]<br/>
> Executing step: [step1]<br/>
> step2 has been executed on thread async-thread-1<br/>
> sum : 100000000<br/>
> Step : [step2] executed in 93ms<br/>
> step1 has been executed on thread async-thread-2<br/>
> sum : 100000000<br/>
> Step : [step1] executed in 86ms<br/>
> Executing step: [step3]<br/>
> step3 has been executed on thread async-thread-1<br/>
> sum : 100000000<br/>
> Step: [step3] executed in 73ms<br/>
> 총 소요시간 : 297

- 참고
> FlowBuilder, JobBuilderHelper, SplitState

## Partitioning
1. 기본 개념
    - MasterStep이 SlaveStep을 실행시키는 구조
    - **SlaveStep은 각 스레드에 의해 독립적으로 실행이 됨(하나만 생성, 동기화 문제 없음)**
    - **SlaveStep은 독립적인 StepExecution 파라미터 환경을 구성함(1대1 생성)**
    - SlaveStep은 ItemReader / ItemProcessor / ItemWriter 등을 가지고 동작하며 작업을 독립적으로 병렬 처리한다.
    - MasterStep은 PartitionStep이며 SlaveStep은 TaskletStep, FlowStep 등이 올 수 있다.

2. 구조
    - MainThread(Job(Step -> MasterStep -> Step)) -> SlaveStep(Worker), SlaveStep(Worker), SlaveStep(Worker) ...
    - Job -> PartitionStep(Master)
    - -> StepExecutionAggregator
    - -> PartitionHandler -> StepExecutions -> TaskExecutor(Worker(FutureTask(TaskletStep(Slave)))) -> StepExecution : 스레드별로 독립적으로 StepExecution을 실행시킨다, StepExecutionAggregator : 실행결과를 취합한다.
    - -> PartitionHandler -> handle() -> StepExecutionSplitter -> split() -> Partitioner -> **StepExecutions**(ExecutionContext(Data(1 ~ 50)), ExecutionContext(Data(51 ~ 100)), ExecutionContext(Data(101 ~ 150)))
        - gridSize만큼 StepExecution과 ExecutionContext 생성
        - gridSize만큼 태스크 및 스레드가 생성
        - ExecutionContext에 각 스레드가 공유할 데이터 설정
    - Partitioning -> StepExecutions -> TaskExecutor(TaskExecutorPartitionHandler(step.execute(stepExecution)) : step은 같아도 각각의 다른 stepExecution 정보를 DB에 저장) -> Worker1, 2, 3 -> FutureTask(TaskletStep) -> ChunkOrientedTasklet
    - -> SimpleChunkProvider -> `Chunk<Inputs>` -> SimpleChunkProcessor -> `Chunk<Outputs>` -> SimpleChunkProcessor
    - **-> ItemReaderProxy -> ItemProcessorProxy -> ItemWriterProxy** (-> Data -> Worker)
        - 각 스레드는 자신에게 할당된 StepExecution을 가지고 있다.
        - 각 스레드는 자신에게 할당된 청크 클래스를 참조한다.
        - **Thread-safe를 만족한다.**
    <br/>

    - **PartitionStep**
        - 파티셔닝 기능을 수행하는 Step 구현체
        - 파티셔닝을 수행 후 StepExecutionAggregator를 사용해서 StepExecution의 정보를 최종 집계한다.
        <br/>

        - PartitionHandler partitionHandler
        - StepExecutionSplitter stepExecutionSplitter
        - StepExecutionAggregator stepExecutionAggregator
    - **PartitionHandler**
        - PartitionStep에 의해 호출되며 스레드를 생성해서 WorkStep을 병렬로 실행한다.
        - WorkStep에서 사용할 StepExecution 생성은 StepExecutionSplitter과 Partitioner에게 위임한다.
        - WorkStep을 병렬로 실행 후 최종 결과를 담은 StepExecution을 PartitionStep에 반환한다.
        - gridSize만큼 StepExecution 생성 후 반환
        - 각 스레드가 StepExecution 담긴 WorkStep을 병렬로 실행
        - 실행 후 정보를 담고 있는 각 StepExecution을 Set에 저장 후 반환
        <br/>

        - `Collection<StepExecution>` handle(StepExecutionSplitter stepSplitter, StepExecution masterStepExecution)
    - **StepExecutionSplitter**
        - WorkStep에서 사용할 StepExecution을 gridSize만큼 생성한다.
        - Partitioner를 통해 ExecutionContext를 얻어서 StepExecution에 매핑한다.
        - Partitioner 호출
        <br/>

        - `Set<StepExecution>` split(StepExecution masterStepExecution, int gridSize)
    - **Partitioner**
        - StepExecution에 매핑할 ExecutionContext를 gridSize만큼 생성한다.
        - 각 ExecutionContext에 저장된 정보는 WorkStep을 실행하는 스레드마다 독립저그올 참조 및 활용 가능하다.
        - gridSize만큼 ExecutionContext 생성
        <br/>

        - `Map<String, ExecutionContext>` partition(int gridSize)

3. API
    ```java
    public Step step() throws Exception {
        return stepBuilderFactory.get("masterStep") // Step 기본 설정
                .partitioner("slaveStep", new ColumnRangePartitioner()) // PartitionStep 생성을 위한 PartitionStepBuilder가 생성되고 Partitioner를 설정
                .step(slaveStep()) // 슬레이브 역할을 하는 Step을 설정 : TaskletStep, FlowStep 등이 올 수 있음
                .gridSize(4) // 파티션 구분을 위한 값 설정 : 몇 개의 파티션으로 나눌 것인지 사용됨
                .taskExecutor(ThreadPoolTaskExecutor()) // 스레드 풀 실행자 설정 : 스레드 생성, 스레드 풀 관리
                .build(); // PartitionStep 생성 : MasterStep의 역할 담당
    }
    ```

- `git checkout Part8.5`
```java
@RequiredArgsConstructor
@Configuration
public class PartitioningConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(masterStep())
                .build();
    }

    @Bean
    public Step masterStep() throws Exception {
        return stepBuilderFactory.get("masterStep")
                .partitioner(slaveStep().getName(), partitioner()) // slaveStep 정보
                .step(slaveStep())
                .gridSize(4)
                .taskExecutor(new SimpleAsyncTaskExecutor())
                .build();
    }

    @Bean
    public Step slaveStep() {
        return stepBuilderFactory.get("slaveStep")
                .<Customer, Customer>chunk(1000)
                .reader(pagingItemReader(null, null)) // 빈 생성X, 프록시 객체 생성
                .writer(customerItemWriter())
                .build();
    }

    @Bean
    public ColumnRangePartitioner partitioner() {
        ColumnRangePartitioner columnRangePartitioner = new ColumnRangePartitioner();

        columnRangePartitioner.setColumn("id");
        columnRangePartitioner.setDataSource(this.dataSource);
        columnRangePartitioner.setTable("customer");

        return columnRangePartitioner;
    }

    @Bean
    @StepScope // @Value 사용하기 위해 필요
    public JdbcPagingItemReader<Customer> pagingItemReader(
            @Value("#{stepExecutionContext['minValue']}")Long minValue,
            @Value("#{stepExecutionContext['maxValue']}")Long maxValue) { // 동적 쿼리 생성(런타임 시간)
        System.out.println("reading " + minValue + " to " + maxValue);
        JdbcPagingItemReader<Customer> reader = new JdbcPagingItemReader<>();

        reader.setDataSource(this.dataSource);
        reader.setFetchSize(1000);
        reader.setRowMapper(new CustomerRowMapper());

        MySqlPagingQueryProvider queryProvider = new MySqlPagingQueryProvider();
        queryProvider.setSelectClause("id, firstName, lastName, birthdate");
        queryProvider.setFromClause("from customer");
        queryProvider.setWhereClause("where id >= " + minValue + " and id < " + maxValue);

        Map<String, Order> sortKeys = new HashMap<>(1);

        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        reader.setQueryProvider(queryProvider);

        return reader;
    }

    @Bean
    @StepScope
    public JdbcBatchItemWriter<Customer> customerItemWriter() { // customer 테이블 저장
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }
}
```
```java
public class ColumnRangePartitioner implements Partitioner {

	private JdbcOperations jdbcTemplate;

	private String table;

	private String column;

	public void setTable(String table) {
		this.table = table;
	}

	public void setColumn(String column) {
		this.column = column;
	}

	public void setDataSource(DataSource dataSource) {
		jdbcTemplate = new JdbcTemplate(dataSource);
	}

	@Override
	public Map<String, ExecutionContext> partition(int gridSize) {
		int min = jdbcTemplate.queryForObject("SELECT MIN(" + column + ") from " + table, Integer.class);
		int max = jdbcTemplate.queryForObject("SELECT MAX(" + column + ") from " + table, Integer.class);
		int targetSize = (max - min) / gridSize + 1; // (1000 -1) / 4 + 1 = 249 + 1 = 250 (4등분)

		Map<String, ExecutionContext> result = new HashMap<String, ExecutionContext>();
		int number = 0;
		int start = min;
		int end = start + targetSize - 1;

		while (start <= max) {
			ExecutionContext value = new ExecutionContext();
			result.put("partition" + number, value);

			if (end >= max) {
				end = max;
			}
			value.putInt("minValue", start);
			value.putInt("maxValue", end);
			start += targetSize;
			end += targetSize;
			number++;
		}

		return result;
	}
}
```
```java
@Data
@AllArgsConstructor
public class Customer {

    private final long id;
    private final String firstName;
    private final String lastName;
    private final Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```
```java
public class StopWatchJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long time = jobExecution.getEndTime().getTime() - jobExecution.getStartTime().getTime();
        System.out.println("==========================================");
        System.out.println("총 소요된 시간 : " + time);
        System.out.println("==========================================");
    }
}
```

- 결과
> Executing step: [masterStep]<br/>
> reading : 501 to 750<br/>
> reading : 1 to 250<br/>
> reading : 251 to 500<br/>
> reading : 751 tp 1000<br/>
> Step: [slaveStep:partition1] executed in 336ms<br/>
> Step: [slaveStep:partition2] executed in 340ms<br/>
> Step: [slaveStep:partition3] executed in 340ms<br/>
> Step: [slaveStep:partition0] executed in 340ms<br/>
> Step: [masterStep] executed in 472ms<br/>
> BATCH_JOB_EXECUTION : masterStep, slaveStep:partition0, 1, 2, 3<br/>
> BATCH_STEP_EXECUTION : READ_COUNT masterStep = 1000, slaveStep = 250, 250, 250, 250 (전부 COMPLETED)<br/>
> BATCH_STEP_EXECUTION_CONTEXT : Execution마다 Thread가 1, 501, 251, 751 ExecutionContext 저장

- 참고
> Partitioner, stepBuilder, AbstractStep, PartitionStep(step = [name=masterStep]), AbstractPartitionHandler, SimpleStepExecutionSplitter, ColumnRangePartitioner, TaskExecutorPartitionHandler(step = [name=slaveStep]), TaskletStep, StepScope(실제 Bean 생성), DefaultStepExecutionAggregator(masterStep의 stepExecution에 반영)

## SynchronizedItemStreamReader
1. 기본 개념
    - Thread-safe 하지 않은 ItemReader를 Thread-safe하게 처리하도록 하는 역할을 한다.
    - Spring Batch 4.0부터 지원한다.

2. 구조
    - **Non Thread-safe**
        - ItemReader (<- Item1, 2, 3, <- read() <- Worker) -> query() -> DB
        - 각 스레드가 동시에 동일한 Item을 중복해서 읽어올 수 있다.
    - **Thread-safe**
        - ItemReader(Synchronized) (<- read(), wait() <- Worker) -> query() -> DB
        - 각 스레드가 대기하고 있다가 순차적으로 Item을 읽어온다.
    - **SynchronizedItemStreamReader는 스레드 안전을 위한 동기화 처리를 해주고 데이터 처리는 JdbcCursorItemReader에게 위임한다.**
    - **Thread-safe 하지 않은 ItemReader를 Thread-safe 하게 처리**

- `git checkout Part8.6`
```java
@RequiredArgsConstructor
@Configuration
@Slf4j
public class NotSynchronizedConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(100)
                .reader(customItemReader())
                .listener(new ItemReadListener<Customer>() {
                    @Override
                    public void beforeRead() {

                    }

                    @Override
                    public void afterRead(Customer item) {
                        System.out.println("item.getId() : " + item.getId());
                    }

                    @Override
                    public void onReadError(Exception ex) {

                    }
                })
                .writer(customerItemWriter())
                .taskExecutor(taskExecutor())
                .build();
    }

    @Bean
    @StepScope
    public JdbcCursorItemReader<Customer> customItemReader() {
        return new JdbcCursorItemReaderBuilder<Customer>()
                .fetchSize(100)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Customer.class))
                .sql("select id, firstName, lastName, birthdate from customer")
                .name("NotSafetyReader")
                .build();
    }

    @Bean
    @StepScope
    public JdbcBatchItemWriter<Customer> customerItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setThreadNamePrefix("not-safety-thread-");
        return executor;
    }
}
```
```java
@RequiredArgsConstructor
@Configuration
@Slf4j
public class SynchronizedConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    @Bean
    public Job job() throws Exception {
        return jobBuilderFactory.get("batchJob")
                .incrementer(new RunIdIncrementer())
                .start(step1())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Customer, Customer>chunk(60)
                .reader(customItemReader())
                .listener(new ItemReadListener<Customer>() {
                    @Override
                    public void beforeRead() {

                    }

                    @Override
                    public void afterRead(Customer item) {
                        System.out.println("item.getId() : " + item.getId());
                    }

                    @Override
                    public void onReadError(Exception ex) {

                    }
                })
                .writer(customerItemWriter())
                .taskExecutor(taskExecutor())
                .build();
    }

    @Bean
    @StepScope
    public SynchronizedItemStreamReader<Customer> customItemReader() {
        JdbcCursorItemReader<Customer> notSafetyReader = new JdbcCursorItemReaderBuilder<Customer>()
                .fetchSize(60)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Customer.class))
                .sql("select id, firstName, lastName, birthdate from customer")
                .name("SafetyReader")
                .build();

        return new SynchronizedItemStreamReaderBuilder<Customer>()
                .delegate(notSafetyReader) // delegate
                .build();
    }

    @Bean
    @StepScope
    public JdbcBatchItemWriter<Customer> customerItemWriter() {
        JdbcBatchItemWriter<Customer> itemWriter = new JdbcBatchItemWriter<>();

        itemWriter.setDataSource(this.dataSource);
        itemWriter.setSql("insert into customer2 values (:id, :firstName, :lastName, :birthdate)");
        itemWriter.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider());
        itemWriter.afterPropertiesSet();

        return itemWriter;
    }

    @Bean
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setThreadNamePrefix("safety-thread-");
        return executor;
    }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {

    private long id;
    private String firstName;
    private String lastName;
    private Date birthdate;
}
```
```java
public class CustomerRowMapper implements RowMapper<Customer> {
	@Override
	public Customer mapRow(ResultSet rs, int i) throws SQLException {
		return new Customer(rs.getLong("id"),
				rs.getString("firstName"),
				rs.getString("lastName"),
				rs.getDate("birthdate"));
	}
}
```

- 결과1 (NotSynchronizedConfiguration)
> Thread : not-safety-thread-1, item.getId() : 4<br/>
> Thread : not-safety-thread-4, item.getId() : 4<br/>
> Thread : not-safety-thread-2, item.getId() : 4<br/>
> Thread : not-safety-thread-3, item.getId() : 4<br/>
> Thread : not-safety-thread-2, item.getId() : 8<br/>
> Thread : not-safety-thread-2, item.getId() : 9<br/>
> Thread : not-safety-thread-2, item.getId() : 10<br/>
> Thread : not-safety-thread-2, item.getId() : 11<br/>
> Thread : not-safety-thread-2, item.getId() : 12<br/>
> Unexpected cursor position change

- 결과2 (SynchronizedConfiguration)
> Thread : not-safety-thread-1, item.getId() : 1<br/>
> Thread : not-safety-thread-4, item.getId() : 2<br/>
> Thread : not-safety-thread-2, item.getId() : 3<br/>
> Thread : not-safety-thread-3, item.getId() : 4<br/>
> ...<br/>
> Thread : not-safety-thread-2, item.getId() : 1000


- 참고
> SynchronizedItemStreamReader