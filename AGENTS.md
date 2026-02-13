# AGENTS.md

이 문서는 본 레포에서 작업하는 모든 AI 워커의 작업 계약서이다.
이 프로젝트는 장기 유지 가능한 이메일 기반 알고리즘 복습 시스템이다.

AI는 규칙을 추론하지 않는다. 이 문서를 따른다.

---

# 0. Commands (가장 먼저 참고)

## Development

- Run server: `./gradlew bootRun`
- Run tests: `./gradlew test`
- Clean build: `./gradlew clean build`

설명보다 위 명령을 기준으로 행동한다.

---

# 1. Project Overview

이 서비스는 사용자가 이메일과 백준 아이디로 가입하고 이메일 인증을 완료한 뒤, solved.ac API 기반으로 문제 풀이 이력을 수집해 `solve_event`를 생성하고 이를
바탕으로 `review_schedule`을 만든 다음, 지정한 시점에 복습 메일을 발송하는 구조를 가진다.

이 철학을 절대 변경하지 않는다.

---

# 2. Tech Stack

- Java 21
- Spring Boot 4.x
- JPA (Hibernate 7)
- MySQL 8+
- Gradle


- 버전 혼합 금지
- 기술 스택 임의 변경 금지

---

# 3. Project Structure

```
global/
├── aop/
├── config/
├── entity/
├── exception/

domain/user/
├── controller/
├── dto/{request,response}/
├── entity/
├── repository/
└── service/
```

원칙:

- Controller → Service → Repository 구조 유지
- 비즈니스 로직은 Service에 위치
- DTO는 API 경계 전용
- Controller → Repository 직접 접근 금지

---

# 4. Git Convention

## Branch

- 기본 브랜치: `dev`
- 브랜치명 형식: `<type>/<이슈번호>-short-desc`
- short-desc: 영어 소문자 + 하이픈
    - 예: `feat/1-add-reminder`, `fix/2-null-check`
- `dev`에서 직접 작업/커밋/푸시 금지
- 모든 작업은 기능 브랜치에서 시작하고 PR로만 반영
- 커밋/푸시 전에는 반드시 사용자에게 확인을 받는다.

## Commit

형식:

```
<type>: <작업 내용> (#<이슈번호>)
```

예:

```
feat: 이메일 인증 기능 구현 (#1)
fix: 복습 스케줄 중복 생성 방지 (#2)
```

커밋은 작게 나눈다.

### Type

| type     | 설명       |
|----------|----------|
| feat     | 기능 추가    |
| fix      | 버그 수정    |
| docs     | 문서 수정    |
| chore    | 환경 설정    |
| refactor | 리팩토링     |
| style    | 코드 포맷 수정 |
| test     | 테스트 작성   |

## PR

- PR 없이 `dev` 브랜치 직접 push 금지
- PR에는 관련 이슈 연결 (`close #이슈번호`)
- Approve 후 merge
- Merge 전 반드시 `dev` 최신 내용 merge

예:

```
git checkout dev
git pull
git checkout feat/1-add-reminder
git merge dev
```

---

# 5. Code Convention

## Naming

- 클래스: PascalCase
- 변수/메서드: camelCase
- DB: snake_case, 단수형

## Repository 변수 명명 예시

- 단건 저장: `savedUser`
- 복수 조회: `users`
- 전체 조회: `allUsers`

## Controller / Service 메서드 규칙

- 생성: create
- 조회: get (외부 노출), find (내부 조회)
- 단건 조회: getUserByEmail
- 전체 조회: getAllUsers
- 수정: update
- 삭제: delete

## Formatting & Linting

- `spotless` 사용 (코드 포맷 자동화)
- `checkstyle` 사용 (스타일 규칙 검사)
- 민감정보 로그 금지

---

# 6. Final Principle

이 프로젝트는 실험용 코드가 아니다.
장기 확장형 시스템이다.

- AI는 “빠르게”보다 “일관되게”를 우선한다.
- 임의 추정 금지
- 경계 위반 금지
- 설계 철학 우선
- 모든 작업은 나에게 허락받을 것

---

# 7. Execution Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions
as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant
clarification.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to
overcomplication, and clarifying questions come before implementation rather than after mistakes.
