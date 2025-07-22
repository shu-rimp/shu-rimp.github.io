---
title: "[Spring Data JPA #1] 개요와 핵심 기능"
author: shurimp
date: 2025-04-23 14:21:00 +0900
categories: ["Backend", "DB"]
tags: [SpringDataJPA, DB]
# 이미지 삽입 시 아래 줄 주석 제거
image: 
   path: posts/6-1.png
   alt: Spring Data JPA
---

> 서론
> 
> 스프링을 사용할 경우, Spring에서 제공하는 Spring Data JPA 라이브러리를 사용할 수 있습니다. 기존 JPA에서 편의성을 추가한 기능들을 제공해 더욱 빠르고 효율적인 개발을 돕습니다. 단순 CRUD의 경우 작성해야 하는 코드의 양은 많은데 비해 거의 테이블명만 바뀌는 반복작업인 경우가 많습니다. 이를 구현체를 생성해서 자동으로 구현해주어 좀 더 핵심 비즈니스에 집중할 수 있도록 도와줍니다.

## 주요 키워드
- Spring Data JPA
- JpaRepository
- 메서드 이름 쿼리
- @Query
- 페이징 지원 쿼리
- 벌크성 수정 쿼리
- EntityGraph
- Query Hint & Lock
- 사용자 정의 리포지토리

## Spring Data JPA란?

Spring Data JPA는 기존 JPA의 기능을 **추상화하고 확장한 Spring 기반 모듈**로,  
`JpaRepository` 등의 인터페이스를 통해 **별도 구현 없이 CRUD를 자동 제공**해 개발 생산성을 크게 높여줍니다.

### 사용하는 목적 및 장점

- CRUD 메서드 자동 생성
- 메서드 이름만으로 쿼리 정의 가능
- 페이징, 정렬, 바인딩 기능 등 다양한 편의 기능
- 유지보수성과 생산성 향상         

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsername(String username);
}
```

### 참고) Setter 지양, 생성자 사용 권장

JPA를 사용할 때 뿐만 아니라, 기본적으로 값을 변경할 때 Setter는 직접 호출하지 않는 것이 좋습니다.       

- 기본 생성자(`protected`)는 JPA를 위해 필요
- 필드 값 변경이 필요한 경우 setter 대신 **명시적인 메서드로** 정의하는 것이 좋습니다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String username;

    protected Member() {} //기본 생성자

    public Member(String username) {
        this.username = username;
    }

    public void changeUsername(String username) { //변경 메서드
        this.username = username;
    }
}
```

---

## JpaRepository, CrudRepository

Spring Data JPA는 `JpaRepository`를 통해 기본 CRUD 기능을 제공합니다.  
구현체 없이도 프록시 기반의 자동 구현 클래스를 생성하며, 런타임에 `ProxyXXX` 형태로 확인할 수 있습니다.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

## 메서드 이름으로 쿼리 생성

```java
List<User> findByUsernameAndAgeGreaterThan(String username, int age);
```

`find`, `By`, `And`, `GreaterThan`과 같이 메서드 명으로 조건을 지정할 수 있습니다.

> 조건이 길어지면 메서드명이 과도하게 길어지는 단점이 있어, 복잡한 조건은 `@Query` 사용 권장

---

## @Query - 직접 쿼리 정의

```java
@Query("SELECT u FROM User u WHERE u.username = :name")
List<User> findUsers(@Param("name") String name);
```

- 쿼리 오류를 **앱 로드 시점에서 감지**
- DTO 조회 시에는 **패키지명 포함 + new 키워드 필요**

```java
@Query("SELECT new com.example.dto.UserDto(u.id, u.username) FROM User u")
List<UserDto> findUserDtos();
```

---

## 파라미터 바인딩

- **순서 기반 (`?1`, `?2`)** 보다는
- **이름 기반 (`:name`, `:age`)** 사용을 권장

```java
@Query("SELECT u FROM User u WHERE u.username IN :names")
List<User> findByNames(@Param("names") List<String> names);
```

---

## Optional 반환

Spring Data JPA의 단건 조회는 `Optional<T>` 반환을 사용합니다.

```java
Optional<User> findById(Long id);

// 사용 시 주의
userRepository.findById(id).orElse(null); // 또는 ifPresent 등으로 처리
```

- `get()` 사용은 지양 → NPE 발생 가능성 

---

## 페이징 지원

Spring Data JPA에서는 페이징 처리를 위한 다양한 방법을 제공합니다.

| 타입       | 설명                                                  |
| ---------- | ----------------------------------------------------- |
| `Page<T>`  | 전체 count 쿼리 포함                                  |
| `Slice<T>` | 다음 페이지 여부만 확인(화면이 무한 스크롤 방식일 때) |
| `List<T>`  | 단순 조회, 페이징 정보 없음                           |

```java
Page<User> findByAge(int age, Pageable pageable);
```

- count 쿼리 최적화 필요 시 `@Query(countQuery = "select count(u.id) from User u")` 옵션으로 카운트 쿼리를 직접 지정할 수도 있음

---

## 벌크성 수정 쿼리

```java
@Modifying(clearAutomatically = true)
@Query("UPDATE User u SET u.age = u.age + 1 WHERE u.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

- 반드시 `@Modifying` 필요
- DB에 직접 적용되므로 **영속성 컨텍스트 초기화 필요**
- `clearAutomatically = true` 사용 시 자동 초기화

---

## @EntityGraph

지연로딩을 즉시 로딩으로 전환하고 싶을 때 사용합니다.

```java
@EntityGraph(attributePaths = {"team"})
List<User> findAll();
```

- `JPQL`, 메서드 이름, `NamedEntityGraph`와 함께 사용 가능

---

## Query Hint & Lock

### @QueryHints

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
User findReadOnlyByUsername(String username);
```

- JPA가 변경감지를 하기 위해서는 변경 전 / 변경 후 엔티티 정보를 모두 알고 있어야 하기 때문에 그만큼 성능을 요구함. 따라서 필요하지 않을 때는 더티체킹 하지 않도록 힌트를 줌
- 읽기 전용 메서드에 QueryHint를 사용해 성능 최적화가 가능
- 모든 읽기 전용 메서드에 사용할 필요는 없고 필요에 따라 선택

### @Lock

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<User> findLockByUsername(String username);
```

- 트래픽이 많은 실시간 서비스에서는 주의해서 사용

---

## 사용자 정의 리포지토리 구현

복잡한 쿼리, QueryDSL 등을 사용할 경우 사용자 정의 리포지토리 작성이 필요할 수 있습니다.

### 방법 요약

1. 커스텀 인터페이스 작성  
2. 구현 클래스 작성 (접미사 반드시 `Impl`)  
3. JpaRepository에서 인터페이스 상속

```java
public interface UserRepositoryCustom {
    List<User> search(String keyword);
}

public class UserRepositoryImpl implements UserRepositoryCustom {
    @Override
    public List<User> search(String keyword) {
        // 구현
    }
}

public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {
}
```

> 커스텀 리포지토리가 반드시 필요한 건 아니며, 단순 DTO 조회용 리포지토리는 클래스로 분리해 직접 사용해도 무방


### 참고) toString 주의점

`@ToString` 등으로 객체 출력 시에는 **연관관계 필드를 제외**하도록 합니다.

```java
@ToString(exclude = "team")
```

- 순환참조 발생 위험
- Lazy 로딩 문제 발생 가능

---
