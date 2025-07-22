---
title: "[Spring Data JPA #2] 구현체 분석 및 기타 기능"
author: shurimp
date: 2025-04-27 15:05:00 +0900
categories: ["Backend", "DB"]
tags: [SpringDataJPA, DB]
# 이미지 삽입 시 아래 줄 주석 제거
image: 
   path: posts/6-1.png
   alt: Spring Data JPA
---

## JPA 구현체 분석: `SimpleJpaRepository`

Spring Data JPA는 내부적으로 `SimpleJpaRepository` 클래스를 통해 대부분의 기능을 제공합니다.

### 주요 특징

- `findById`, `save` 등 모든 CRUD 메서드의 실제 구현체
- 트랜잭션 처리:
  - 클래스 레벨: 기본적으로 `readOnly`
  - `save` 등 데이터 변경 메서드: 메서드 단위로 트랜잭션 처리

### `save()` 동작 방식

```java
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

- **새로운 엔티티**: `persist()`
- **기존 엔티티**: `merge()`

> **주의:** 단순한 데이터 변경에는 `merge()`를 사용하는 것이 아닌, **변경 감지(dirty checking)**를 이용해야 합니다.

---

## New 엔티티 판별 기준

- 식별자가 **참조 타입**이면 `null`일 경우 신규
- 식별자가 **기본 타입(primitive)**이면 `0`일 경우 신규

### 오작동 예시

참조 타입의 신규여부를 판단할 때, 기본 동작 방식이 식별자(id)가 null일 때가 기준이기 때문에 식별자를 직접 지정하는 상황에서는 persist가 아닌 merge를 호출하게 됩니다.

엔티티 생성에 merge()를 호출할 시, 기존 엔티티를 조회 -> 엔티티가 없으면 새로 생성 하는 과정을 거치기 때문에 persist()에 비해 비효율적입니다.

```java
@Entity
public class MyEntity {
    @Id
    private Long id; // 직접 할당

    // id가 null이 아니므로 merge() 호출됨 → 비효율 발생
}
```

이걸 해결하기 위해 **Persistable<T> 인터페이스의 `isNew()` 메서드**를 재정의할 수 있습니다.

```java
public class MyEntity implements Persistable<Long> {
    
    @Id
    private Long id;
    @CreatedDate
    private LocalDateTime createdDate;

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```

식별자 기준이 아닌 생성일(createdDate) 기준으로 재정의. 이렇게 하면 id를 직접 지정할 때에도 정상적으로 신규 엔티티로 판별합니다.

---

## 기타 기능들

---

## Criteria, Example

Spring Data JPA는 JPA 표준 기능인 **Criteria API**와 **Example Matcher**를 지원하지만, 실제 현업에서는 다음 이유로 사용 빈도가 낮습니다:

| 기능     | 한계                                                           |
| -------- | -------------------------------------------------------------- |
| Criteria | 코드 복잡도 ↑, 가독성 ↓                                        |
| Example  | inner join만 가능, 중첩 제약조건 불가, 단순한 매칭 조건만 지원 |

> **대안: `QueryDSL` 사용**

---

## Example 사용 예시

```java
Member member = new Member("user1", "teamA");
Example<Member> example = Example.of(member);
memberRepository.findAll(example);
```

- Example을 사용하면 엔티티 자체를 검색조건으로 지정할 수 있습니다.
- `ExampleMatcher`로 조건에서 제외할 필드 지정 가능
- inner join만 지원, 중첩 조건 불가, 단순한 매칭 조건(ex. '=')만을 지원하므로 한계가 명확함

---

## Projections

> 엔티티 전체가 아닌 **특정 필드만 조회할 때 유용**

### 인터페이스 기반 DTO 조회

```java
public interface UsernameOnly {
    String getUsername();
}

List<UsernameOnly> findBy();
```

- 리포지토리 메서드 반환 타입에 인터페이스 지정
- Spring Data JPA가 **프록시 객체 생성**
- **중첩 인터페이스**로 조인도 일부 가능

> 루트 엔티티를 벗어나면 성능 최적화가 되지 않고 복잡도 증가  
> 복잡한 조회는 **QueryDSL** 사용 권장

---

## 네이티브 쿼리 (Native Query)

가급적 사용을 지양하는게 좋지만, 필요시 다음 주의사항 고려:

| 단점           | 설명                                                   |
| -------------- | ------------------------------------------------------ |
| 문법 확인 불가 | JPQL과 달리 컴파일 타임 검증 불가                      |
| Sort 문제      | Spring Data JPA의 `Sort` 파라미터가 정상 작동하지 않음 |
| 동적 쿼리 불가 | 정적 쿼리만 사용 가능                                  |

### 대안

- **정적 쿼리**가 필요한 경우: Native Query + 인터페이스 기반 Projection 사용(페이징 API 사용 가능)
- **복잡한 조회**: `SpringJdbcTemplate` 또는 `MyBatis` 사용 고려

---
