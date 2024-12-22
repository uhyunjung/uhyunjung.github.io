---
title: '[Spring] Java ORM 표준 JPA 프로그래밍 3장 영속성 관리'
date: 2024-10-20 20:10:00 +0900
categories: [백엔드, ORM]
tags: [ORM]
math: true
mermaid: true
---

## 자바 ORM 표준 JPA 프로그래밍 도서
> [**자바 ORM 표준 JPA 프로그래밍**](https://product.kyobobook.co.kr/detail/S000000935744)

## 3.1 엔티티 매니저 팩토리와 엔티티 매니저
- **엔티티 매니지 팩토리**는 한 개만 만들며,여러 스레드가 동시에 접근해도 안전하며 서로 다른 스레드 간에 공유해도 된다.
- **엔티티 매니저**는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안 된다.
- **엔티티 매니저**는 보통 트랜잭션을 시작할 때 데이터베이스 커넥션을 획득한다.
- 하이버네이트를 포함한 JPA 구현체들은 EntityManagerFactory를 생성할 때 **커넥션풀**도 만드는데 이것은 J2SE 환경에서 사용하는 방법이다.

## 3.2 영속성 컨텍스트
- **영속성 컨텍스트** : 엔티티를 영구 저장하는 환경. 애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 데이터베이스 같은 역할.
- **persist()** : 엔티티 매니저를 사용해 엔티티를 영속성 컨텍스트에 저장한다.

## 3.3 엔티티의 생명주기
1. **비영속(new/transiend)** : 영속성 컨텍스트와 전혀 관계가 없는 상태
2. **영속(managed)** : 영속성 컨텍스트에 저장된 상태, 영속성 컨텍스트가 관리하는 엔티티
3. **준영속(detached)** : 영속성 컨텍스트에 저장되었다가 분리된 상태
  - **detach()** : 준영속 상태
  - **close()** : 영속성 컨텍스트 닫기
  - **clear()** : 영속성 컨텍스트 초기화
4. **삭제(removed)** : 삭제된 상태

## 3.4 영속성 컨텍스트의 특징
- 식별자 값 : 영속성 컨텍스트는 엔티티를 식별자값 `@Id`로 구분하므로 영속 상태는 식별 자 값이 반드시 있어야 한다. 없으면 에러 발생.
- 데이터베이스 저장 : JPA는 보통 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영하여 동기화하는데, 이를 **플러시**라 한다.
- 장점  
  1) 1차 캐시 : Map으로 저장, 키는 식별자, 값은 엔티티 인스턴스. `find()`로 1차 캐시에서 엔티티를 찾고, 없으면 데이터베이스에서 조회

  2) 동일성 보장 : 1차 캐시에 있는 같은 엔티티 인스턴스를 반환. 아래의 a와 b는 같은 인스턴스로 `a == b`는 `true`이다.
    - 동일성 : 실제 인스턴스가 같다. 참조 값을 비교하는 `==` 비교 값이 같다.
    - 동등성 : 실제 인스턴스가 다를 수 있찌만 인스턴스가 가지고 있는 값이 같다. `equals()` 사용

    ```java
    Member a = em.find(Member.class, "member1");
    Member b = em.find(Member.class, "member2"); 
    ```

  3) 트랜잭션을 지원하는 쓰기 지연 : 내부 쿼리 저장소에 SQL을 모아둔 후, 트랜잭션을 커밋할 때 쿼리를 데이터베이스로 보낸다. 트랜잭션이라는 작업 단위가 있기 때문에 데이터베이스와의 동기화를 최대한 늦추는 것이 가능하다.
    - 트랜잭션을 커밋하면 엔티티 매니저는 우선 영속성 컨텍스트를 플러시한다.
    - A, B 모두 트랜잭션을 커밋하면 함께 저장되고 롤백하면 함께 저장되지 않는다. 등록 쿼리를 그때 그때 데이터베이스에 전달해도 트랜잭션을 커밋하지 않으면 아무 소용이 없다. 쓰기 지연을 이용해 성능을 최적화할 수 있다.
    - 엔티티 삭제 : `remove()`로 영속성 컨텍스트에서 제거하며 가비지 컬렉션의 대상이 된다.

    ```java
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = en.getTransaction();

    transaction.begin();

    em.persist(memberA);
    em.persist(memberB);

    transaction.commit();
    ```

  4) 변경 감지 : JPA는 엔티티를 영속성에 보관할 때 최초 상태를 복사해서 스냅샷으로 저장해두는데, 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다. 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다. `update()`는 존재하지 않음.
    1. 트랜잭션을 커밋하면 엔티티 매니저 내부에서 `flush()`가 호출된다.
    2. 엔티티와 스냅샷을 비교하여 **변경된 엔티티**를 찾는다.
    3. 변경된 엔티티가 있으면 수정 쿼리를 생성해 **쓰기 지연 SQL 저장소**에 보낸다.  
      - **모든 필드 업데이트** : 수정 쿼리가 항상 같으므로 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있따. 또한, 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.  
      - 수정된 데이터만 사용해서 **동적 쿼리 생성** : `@org.hibernate.annotations.DynamicUpdate` or `@DynamicInsert`로 가능. 대략 한테이블에 컬럼이 30개 이상이 되면 정적 수정 쿼리보다 동적 수정 쿼리가 빠르다고 한다. 직접 본인 환경에서 테스트해볼 것.
    4. 쓰기 지연 저장소의 SQL을 **데이터베이스**에 보낸다.
    5. 데이터베이스 트랜잭션을 **커밋**한다.

  5) 지연 로딩 : 실제 객체 대신 프록시 객체를 로딩해두고 해당 객체를 실제 사용할 때 영속성 컨텍스트를 통해 데이터를 불러오는 방법.

## 3.5 플러시
- 플러시 : 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영
  1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해 수정된 엔티티를 찾고, 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
  2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 저장한ㄷ.(등록, 수정, 삭제 쿼리)
- 영속성 컨텍스트를 플러시하는 방법
  1. `em.flush()` 직접 호출
  2. 트랜잭션 커밋 : 자동 `flush()` 호출
  3. JPQL 쿼리 실행 : 자동 `flush()` 호출. 쿼리를 실행하기 전에 영속성 컨텍스트를 플러시해서 변경 내용을 데이터베이스에 반영해야 하기 때문이다.
- 플러시 모드 옵션(javax.persistence.FlushModeType)
  1. `FlushModeType.AUTO` : 커밋이나 쿼리를 실행할 때 플러시. Default값.
  2. `FlushModeType.COMMIT` : 커밋할 때만 플러시

## 3.6 준영속
- 영속 상태의 엔티티를 준영속 상태로 만드는 방법
  1. `em.detach(entity)` : 특정 엔티티만 준영속 상태로 전환. 1차 캐시부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보 제거. 영속성 컨텍스트로부터 분리된 상태.
  2. `em.clear()` : 영속성 컨텍스트를 완전히 초기화
  3. `em.close()` : 영속성 컨텍스트 종료
- 특징
  1. 거의 비영속 상태에 가깝다.
  2. 식별자 값을 가지고 있다.
  3. 지연 로딩을 할 수 없다.
- 병합
  1. `merge()` : 준영속 상태의 엔티티를 받아서 그 정보로 새로운 영속 상태의 엔티티를 반환한다.
  2. 준영속, 비영속을 신경 쓰지 않는다.
  3. 파라미터로 넘어온 엔티티는 병합 후에도 준영속 상태로 남아 있다.

  ```java
  public class ExamMergeMain {
    static EntityManagerFactory emf = Persistence.createEntityManagerFActory("jpabook");
    
    public static void main(String args[]) {
      Member member = createMember("memberA", "회원1");
      member.setUsername("회원명변경");
      mergeMember(member);
    }

    static Member createMember(String id, String username) {
      EntityManager em1 = emf.createEntityManager();
      EntityTransaction tx1 = em1.getTransaction();
      tx1.begin();

      Member member = new Member();
      member.setId(id);
      member.setUsername(username);

      em1.persist(member);
      tx1.commit();

      em1.close();

      return member;
    }

    static void mergeMember(Member member) {
      EntityManager em2 = emf.createEntityManager();
      EntityTransaction tx2 = em2.getTransaction();

      tx2.begin();
      Member mergeMember = em2.merge(member);
      tx2.commit();

      System.out.println("member = " + member.getUsername()); // member = 회원명변경
      System.out.println("mergeMember = " + mergeMember.getUsername()); // mergeMember = 회원명변경
      System.out.println("em2 contains member = " + em2.contains(member)); // em2 contains member = false
      System.out.println("em2.contains mergeMember = " + em2.contains(mergeMember)); // em2 contains mergeMember = true

      em2.close();
    }
  }
  ```