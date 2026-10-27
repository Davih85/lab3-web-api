# Lab 3 Web API -- Project Report

## What I specified

Before coding I read, for each method, if it is safe and idempotent and what the test must show.

- `POST /employees` : not safe, not idempotent. It calls `save`. Two POSTs must give two ids (1 and 2), `save` called twice with `null`, no reads.
- `GET /employees/{id}` : safe, idempotent. `findById(1)` twice (200) and `findById(2)` twice (404). No `save`, `deleteById` or `findAll`.
- `GET /employees` : safe, idempotent. `findAll` twice with the same list. No `save`, `deleteById` or `findById`.
- `PUT /employees/{id}` : not safe, idempotent. First PUT 201, second 200, same state. `save` twice with the same argument.
- `DELETE /employees/{id}` : not safe, idempotent. `deleteById(1)` twice, both 204. No `save`, `findById` or `findAll`.

## What I changed

I changed `src/test/kotlin/es/unizar/webeng/lab3/ControllerTests.kt` and the `REPORT.md`.

In `ControllerTests.kt` I replaced the five `TODO(...)` placeholders with the
MockK blocks from `docs/GUIDE.md` the  mock setup before the HTTP calls and the
verification blocks after them.

## Technical decisions

- **POST**: used `answers` + `andThenAnswer` (ids 1 and 2). A single `answers` would fail because the second POST would get id 1. Verified `save` twice with `null`, because the controller sets `id = null` before saving.
- **GET item**: two stubs, `findById(1)` = `Optional.of`, `findById(2)` = `Optional.empty()`. Rejected one `any()` stub because it cannot give 200 and 404 in the same test.
- **PUT**: `findById(1)` with `andThenAnswer`, because the first PUT creates and the second finds. Verified `save(Tom, Manager, 1)` twice with the same argument cause the state does not change.
- **DELETE**: `justRun` because `deleteById` returns `Unit`. Verified `deleteById(1)` twice with the same argument for the same final state.
- **GET collection**: `findAll` returns `listOf(Employee("Mary", "Manager", 1))`. Verified `findAll()` twice, and `save`, `deleteById`, `findById` at zero.

## How I verified

- `./gradlew check` on the starter fails on purpose cause the five tests stop on
  `TODO(...)`.
- `./gradlew test` after filling the blocks four tests passed and one failed. 
- The cause was in my own verification block I had
  copied the `exactly = 0` list from the item GET, so it contained `findAll()`
  while the same test already asserted `findAll()` twice. I replaced
  `findAll()` with `deleteById(any())` in that block.
- After the fix: `./gradlew test` and  `./gradlew check` passed
- Also checked with `./gradlew ktlintFormat`, `./gradlew ktlintCheck` and `./gradlew check` and all was green.

## AI disclosure



- **Tools / skills:** Deepseek used as an assistive tool.
- **Purpose:** Debugging and understanding concepts
- **Representative prompts:** "Explicame este precondicion para terminar de entenderlo 
    * Replaces the employee at this URI, or creates it there.
    * PUT is idempotent and not safe (RFC 9110 §9.2.1, §9.2.2, §9.3.4).
    * The request URI is the target resource. An id in the body is not.
    * 201 Created (§15.3.2) when this URI has no current representation.
    * Location may be omitted because that URI is the target (§15.3.2).
    * 200 OK (§15.3.1) when a later identical PUT stores the same state.
    * The status may change from 201 to 200. The resource state does not,
    * which is what idempotency requires (§9.2.2).
    * Content-Location (§8.7) identifies the representation in the body."

- **Affected files/sections:** None. The assistant was used only to understand the concepts and what each block from `docs/GUIDE.md` does.
- **Validation steps:** I ran `./gradlew test` before and after the fix, then `./gradlew ktlintFormat`, `./gradlew ktlintCheck` and `./gradlew check` and all was green.
- **Citations:** the MockK blocks are from `docs/GUIDE.md`
- **Human-reviewed:** I understood the concepts and applied the blocks from `docs/GUIDE.md`.
