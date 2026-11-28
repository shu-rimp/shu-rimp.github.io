---
title: "JUnit 5 @Nested와 Kotlin inner class로 엔티티 단위 테스트 구조화하기"
author: shurimp
date: 2026-11-28 11:30:00 +0900
categories: ["Backend", "Test"]
tags: [junit5, nested, kotlin, inner class, 단위테스트, unit test, jpa, entity, 테스트 구조화, beforeeach, testinstance]
# 이미지 삽입 시 아래 줄 주석 제거
image: 
   path: posts/11-1.png
   alt: JUnit 5 @Nested
---

> Kotlin으로 JPA 엔티티의 검증 로직을 단위 테스트하면서 JUnit 5의 `@Nested`로 테스트를 동작별로 묶었습니다. `@Nested`가 정확히 무슨 일을 하는지, Kotlin에서는 왜 `inner class`와 함께 써야 하는지, 언제 쓰고 언제 쓰지 않는 게 좋은지 정리했습니다.

## 주요 키워드

- JUnit 5 `@Nested`, 중첩 테스트 클래스
- Kotlin `inner class`와 static 중첩 클래스의 차이
- 테스트 생명주기: `@BeforeEach` 실행 순서, 테스트마다 새 인스턴스
- `@DisplayName`, `@TestInstance(PER_CLASS)`
- 엔티티 단위 테스트: 픽스처 함수, 경계값, 예외 타입 검증

---

## 테스트할 엔티티

PT 클래스를 표현하는 `PtClass` 엔티티입니다. 생성할 때와 수정할 때 같은 `validate()`로 DB의 CHECK 제약과 같은 규칙을 검사합니다.

```kotlin
@Entity
class PtClass(
    trainer: Trainer,
    type: PtClassType,          // PERIOD(기간제) | COUNT(횟수제)
    title: String,
    startDate: LocalDate? = null,
    endDate: LocalDate? = null,
    count: Int? = null,
    amount: Int,
    capacity: Int,
) {
    // ... 필드 생략 (모두 protected set)

    var reservedCount: Int = 0
        protected set

    var state: PtClassState = PtClassState.RECRUITING
        protected set

    init {
        validate(type, startDate, endDate, count, amount, capacity)
    }

    fun updateInfo(/* ... */ capacity: Int) {
        validate(type, startDate, endDate, count, amount, capacity) // 대입 전에 검증
        // ... 대입
    }

    fun close() {
        check(state == PtClassState.RECRUITING) { "already closed" }
        this.state = PtClassState.CLOSED
    }

    private fun validate(/* ... */) {
        when (type) {
            PtClassType.PERIOD -> {
                val start = requireNotNull(startDate) { "start date is required" }
                val end = requireNotNull(endDate) { "end date is required" }
                require(!start.isAfter(end)) { "startDate must not be after endDate" }
            }
            PtClassType.COUNT -> require((count ?: 0) > 0) { "count must be > 0" }
        }
        require(amount >= 0) { "amount must be >= 0" }
        require(capacity > 0) { "capacity must be > 0" }
        require(reservedCount in 0..capacity) { "reservedCount must be between 0 and $capacity" }
    }
}
```

이 규칙들은 엔티티 안에서 끝나기 때문에 Spring이나 DB 없이 객체를 직접 만들어 테스트할 수 있습니다.

---

## @Nested를 쓴 테스트 코드

```kotlin
class PtClassTest {

    private val trainer = Trainer(
        member = Member(
            email = "trainer@test.com",
            name = "트레이너",
            role = MemberRole.TRAINER,
            password = "encoded",
        ),
    )

    // 기본값은 항상 "유효한" 값. 테스트에서는 확인할 값만 바꾼다.
    private fun periodClass(
        startDate: LocalDate? = LocalDate.of(2026, 12, 1),
        endDate: LocalDate? = LocalDate.of(2026, 12, 31),
        amount: Int = 100_000,
        capacity: Int = 10,
    ) = PtClass(
        trainer = trainer,
        type = PtClassType.PERIOD,
        title = "기간제 PT",
        startDate = startDate,
        endDate = endDate,
        amount = amount,
        capacity = capacity,
    )

    @Nested
    inner class 생성 {

        @Test
        fun `시작일과 종료일이 같은 하루짜리 기간제 클래스를 만들 수 있다`() {
            val day = LocalDate.of(2026, 12, 1)

            val ptClass = periodClass(startDate = day, endDate = day)

            assertEquals(day, ptClass.endDate)
        }

        @Test
        fun `기간제는 종료일이 시작일보다 앞서면 만들 수 없다`() {
            assertFailsWith<IllegalArgumentException> {
                periodClass(startDate = LocalDate.of(2026, 12, 2), endDate = LocalDate.of(2026, 12, 1))
            }
        }
    }

    @Nested
    inner class 마감 {

        @Test
        fun `모집 중인 클래스를 마감할 수 있다`() {
            val ptClass = periodClass()

            ptClass.close()

            assertEquals(PtClassState.CLOSED, ptClass.state)
        }

        @Test
        fun `이미 마감된 클래스는 다시 마감할 수 없다`() {
            val ptClass = periodClass().apply { close() }

            assertFailsWith<IllegalStateException> { ptClass.close() }
        }
    }
}
```

> `require`는 `IllegalArgumentException`, `check`는 `IllegalStateException`을 던집니다. 예외 타입까지 확인하면 "잘못된 입력"과 "잘못된 상태"를 구분해서 검증할 수 있습니다.

---

## @Nested의 역할

`@Nested`는 JUnit 5에게 **"이 내부 클래스도 테스트 클래스이니 안에 있는 테스트를 실행하라"**고 알려주는 표시입니다.

### 없으면 테스트가 조용히 빠진다

JUnit은 기본적으로 최상위 클래스와 static 중첩 클래스만 테스트 클래스로 찾습니다. static이 아닌 내부 클래스(Kotlin의 `inner class`)는 무시합니다.

```kotlin
class PtClassTest {
    inner class 마감 {          // @Nested 없음
        @Test
        fun `다시 마감할 수 없다`() { /* ... */ }   // 실행되지 않음
    }
}
```

컴파일 에러도 경고도 없이 테스트가 실행 목록에서 빠집니다. 빌드는 성공하니 테스트가 통과한 것처럼 보이지만, 실제로는 돌지 않은 상태입니다.

> 새 테스트를 추가했다면 테스트 결과 목록에 그 테스트가 실제로 보이는지 확인하는 습관을 들이는 게 좋습니다.

### 붙이면 JUnit이 하는 일

`마감` 그룹의 테스트 하나를 실행할 때의 순서입니다.

1. 바깥 클래스 인스턴스 생성 (`PtClassTest()`)
2. 그 인스턴스로 내부 클래스 인스턴스 생성 (`outer.마감()`)
3. 바깥 `@BeforeEach` → 안쪽 `@BeforeEach`
4. 테스트 메서드 실행
5. 안쪽 `@AfterEach` → 바깥 `@AfterEach`

이 과정을 **테스트 메서드마다 새로** 반복합니다. 그래서 한 테스트에서 `close()`를 호출해도 다른 테스트의 객체에는 영향이 없습니다.

---

## Kotlin에서 inner class가 필요한 이유

Kotlin에서 클래스 안에 그냥 `class`를 선언하면 Java의 **static 중첩 클래스**가 됩니다. Java와 기본값이 반대입니다.

| 선언 | Java로 보면 | 바깥 인스턴스 참조 |
|---|---|---|
| `class Inner` (Kotlin) | `static class Inner` | 없음 |
| `inner class Inner` (Kotlin) | `class Inner` | 있음 |

static 중첩 클래스로는 두 가지가 안 됩니다.

- **JUnit 5 규칙**: `@Nested`의 대상은 static이 아닌 내부 클래스입니다. static 중첩 클래스는 JUnit이 중첩 테스트로 다루지 않습니다.
- **바깥 멤버 접근**: 바깥 클래스의 `trainer`, `periodClass()` 같은 픽스처를 쓸 수 없어 컴파일 에러가 납니다.

생명주기 2번 단계(`outer.마감()`)처럼 내부 클래스는 바깥 인스턴스를 통해 만들어지기 때문에, 반드시 바깥 인스턴스를 참조할 수 있어야 합니다. 정리하면 이렇습니다.

> `inner`는 Kotlin 문법상 내부 클래스로 만드는 것이고, `@Nested`는 그 내부 클래스를 JUnit이 테스트로 인식하게 하는 것입니다. 둘 다 있어야 동작합니다.

---

## @Nested로 테스트를 묶으면 좋은 점

### 1. 테스트 결과가 동작별 트리로 보인다

```
PtClassTest
├─ 생성
│  ├─ 시작일과 종료일이 같은 하루짜리 기간제 클래스를 만들 수 있다
│  └─ 기간제는 종료일이 시작일보다 앞서면 만들 수 없다
└─ 마감
   ├─ 모집 중인 클래스를 마감할 수 있다
   └─ 이미 마감된 클래스는 다시 마감할 수 없다
```

"마감할 때의 규칙"을 찾고 싶으면 `마감` 그룹만 보면 됩니다. 테스트 이름이 곧 도메인 규칙 문서가 되고, 테스트가 수십 개로 늘어날수록 차이가 커집니다.

### 2. 테스트 이름이 짧아진다

그룹 이름이 앞부분을 맡아서 `마감 > 이미 마감된 클래스는 다시 마감할 수 없다`처럼 읽힙니다. 그룹이 없으면 메서드 이름마다 "마감 시 ~"를 반복해서 붙여야 합니다.

### 3. 그룹별 준비 코드를 따로 둘 수 있다

그룹의 테스트가 모두 같은 상태에서 시작한다면, 그 그룹 안에만 `@BeforeEach`를 둘 수 있습니다.

```kotlin
@Nested
inner class 마감 {
    private lateinit var ptClass: PtClass

    @BeforeEach
    fun setUp() {
        ptClass = periodClass()
    }

    @Test
    fun `모집 중인 클래스를 마감할 수 있다`() {
        ptClass.close()
        assertEquals(PtClassState.CLOSED, ptClass.state)
    }
}
```

바깥 클래스의 `@BeforeEach`가 먼저 실행되고, 그다음 안쪽 것이 실행됩니다.

---

## 함께 알아두면 좋은 것

- **여러 단계 중첩**: `@Nested` 안에 `@Nested`를 또 둘 수 있습니다(예: `마감 > 예약자가 있을 때 > ...`). 다만 두 단계를 넘어가면 오히려 읽기 어려워집니다.
- **표시 이름 바꾸기**: 기본으로 클래스 이름이 그룹 이름이 됩니다. 영문 클래스명을 쓰고 화면에는 한글로 보이게 하고 싶다면 `@DisplayName("마감할 때")`를 붙이면 됩니다.
- **`@BeforeAll` 제약**: Kotlin의 `inner class`에는 `companion object`를 둘 수 없습니다. 중첩 클래스 안에서 `@BeforeAll`을 쓰려면 `@TestInstance(TestInstance.Lifecycle.PER_CLASS)`를 붙여야 합니다. 엔티티 단위 테스트에서는 거의 필요 없습니다.

---

## 언제 쓰고, 언제 쓰지 않을까

`@Nested`는 필수가 아닙니다. 들여쓰기가 한 단계 깊어지고, 그룹이 하나뿐이면 오히려 군더더기가 됩니다.

| 상황 | 추천 |
|---|---|
| 테스트할 메서드나 상황이 2~3개 이상 | `@Nested`로 묶기 |
| 테스트가 몇 개 안 되거나 그룹이 하나뿐 | 평평하게 작성 |
| 그룹마다 준비 상태가 다름 | `@Nested` + 그룹별 `@BeforeEach` |

어느 쪽을 고르든 **프로젝트 안에서 한 가지 방식으로 통일**하는 것이 더 중요합니다.

---

## 마무리: 엔티티 단위 테스트 작성 원칙

`@Nested` 외에 엔티티 테스트를 쓰면서 지킨 원칙도 함께 정리합니다.

- **테스트 하나에 규칙 하나**: 테스트 이름만 읽어도 어떤 규칙인지 알 수 있게 씁니다.
- **기본값 있는 픽스처 함수**: 유효한 객체를 만드는 함수를 두고, 각 테스트에서는 확인할 값만 바꿉니다. 실패 원인이 어떤 값인지 바로 드러납니다.
- **경계값 위주**: 시작일과 종료일이 같은 날, `count = 0`, `capacity = 0`, `amount = 0` 같은 값을 확인합니다. 버그는 대부분 경계에서 나옵니다.
- **단위 테스트와 통합 테스트 구분**: 엔티티 안의 규칙은 단위 테스트로 확인합니다. CHECK 제약, 조건부 UPDATE, unique 인덱스처럼 DB에 맡긴 규칙은 `@DataJpaTest`와 Testcontainers로 실제 DB에서 확인합니다.
