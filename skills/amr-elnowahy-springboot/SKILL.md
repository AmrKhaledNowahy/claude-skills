---
name: amr-elnowahy-springboot
description: Code structure, readability and comment style for Spring Boot backend projects - services, helpers, guards, transactions, mapping, Javadoc and inline comments. Use when writing or editing Java service, controller, mapper or repository code.
---

# Backend Patterns (Spring Boot)

Existing project conventions win; follow them where they differ.

## Service structure
- New services: bare interface + `impl/XxxServiceImpl` (interface has no Javadoc, one blank line between methods). If the project's existing services are plain classes, match the project.
- Class order: annotations (`@Service @RequiredArgsConstructor`, logger only if used), constants, `final` dependencies, public `@Override` methods, private helpers.
- Over ~150 lines of helpers, group them with `//region <name>` ... `//endregion <name>`.
- Target: public method <= 25 lines, class <= 250 lines. Split steps into named private helpers (validate, resolve, build, map).
- Helpers are small and single-purpose. Build boolean conditions from named predicates (`isPendingAndVisibleToday`), not long inline expressions.

## Readability rules
- Guards first, throw early, happy path last. Max 2 nesting levels.
- No lambdas longer than 3 lines; extract a named method.
- Constants for repeated or meaningful literals.
- Entities via `.builder()`; mapping in a mapper or a `toResponse` helper, never inline in the flow.
- Errors: `BaseException(ErrorCode.X)`; `orElseThrow(() -> new BaseException(...))`.
- `@Transactional` on every public service method; `readOnly = true` for reads; `REQUIRES_NEW` only with a stated reason.
- One style per file for `var` vs explicit types: follow the project.
- Clean formatting: single blank lines, no stray spaces before commas, no unused imports or return types.

## Libraries and types
- Lombok everywhere it removes boilerplate: `@RequiredArgsConstructor`, `@Builder`, `@Getter/@Setter` or `@Data` on DTOs, a logger annotation. No hand-written getters, setters or constructors.
- MapStruct for all entity <-> DTO mapping (`@Mapper(componentModel = "spring")`); no manual mapping helpers when a mapper fits.
- DTOs: use `@FieldDefaults(level = AccessLevel.PRIVATE)` so `private` is not repeated on every field.
- Enums over string/int constants and flags. Put behavior on the enum (static helpers like `requiresRating`, abstract or per-constant methods) instead of `if/else` chains on the value.
- Use a design pattern when it removes branching or duplication and makes the code easier to read: Strategy (enum or bean map instead of `switch`/`if` chains), Factory, Builder, Template Method. Do not add a pattern for a single case; state the pattern in one line when applying it.

## Controllers
- Thin: map the request, call one service method, return. No business logic, no try/catch.
- Return the response DTO directly with `@ResponseStatus` where not 200; `ResponseEntity` only when headers or a variable status are needed.
- `@Valid @RequestBody` on inputs; constraints live on the Request DTO.

## Repositories
- Derived methods for 1-2 conditions; `@Query` (JPQL) beyond that.
- Projections or DTO interfaces for read-only lists; `@EntityGraph` or `join fetch` against N+1; `Pageable` for lists.

## Exceptions
- `ErrorCode` enum holds code, HTTP status and message key; throw `BaseException(ErrorCode.X)`.
- One `@RestControllerAdvice` maps exceptions to the standard error body.

## Logging
- Parameterized messages (`log.info("x {}", id)`), never concatenation; none inside tight loops.
- Never log secrets or PII (tokens, passwords, emails, national IDs); log ids.
- Levels: ERROR needs action, WARN unexpected but recoverable, INFO business events, DEBUG diagnostics. Log an exception once, at the handler.
- Minimal: only where an operator needs it; no entry/exit logs.

## Naming
- API boundary: `XxxRequest` / `XxxResponse`; internal transfer objects `XxxDto`.
- Package by feature : each feature package holds its own controller, service, repository, dto, mapper and entity, so a feature change stays in one place. Shared code goes in a common/shared package. Existing project layout wins when it differs.

## Testing
- Services: Mockito unit tests (`MockitoExtension`, AssertJ). Controllers: `@WebMvcTest`. Repositories: `@DataJpaTest` or Testcontainers.
- Method names: `method_condition_expected` (e.g. `approve_whenAlreadyApproved_throwsBaseException`).
- Put the Jira ticket and any test-case ticket in the test (`@Tag`/`@DisplayName` or a Javadoc line) when provided.
- Traceability: keep one MD per feature at `docs/tests/<feature>.md` with a table, updated whenever tests change:

  | Behavior | Test class#method | Layer | Jira | Test-case ticket |
  |---|---|---|---|---|

## Java practices
- Immutability: `final` fields; records for immutable value objects/DTOs where Lombok adds nothing; return unmodifiable collections (`List.of`, `toList()`).
- Streams for transforms only, no side effects; a plain loop when it reads simpler. Use a `Map` lookup instead of nested loops.
- Money: `BigDecimal` (never `double`) with explicit scale and `RoundingMode`.
- Time: `java.time`; store `Instant`/UTC; inject a `Clock` so time is testable.

## Persistence and API
- Never expose entities in the API; map to Response DTOs.
- Relations `LAZY` by default; `@Version` optimistic locking for concurrently updated entities.
- Schema changes only via Flyway/Liquibase; no `ddl-auto` beyond `validate` outside local dev.
- Every list endpoint is paginated; PUT/DELETE idempotent; version in the path; no breaking changes without a new version.

## Security
- Validate all input at the boundary; parameterized queries only.
- Method-level authorization (`@PreAuthorize`) with least-privilege roles.
- No sensitive data in responses or logs.

## Transactions, observability, performance
- No external calls (HTTP, messaging) inside a transaction; keep transactions short. Self-invocation bypasses `@Transactional`; call through another bean.
- Actuator health and metrics enabled; correlation id in logs via MDC.
- Batch writes (`saveAll`, JDBC batch); fetch only needed columns; cache only with a defined eviction rule; measure before optimizing.

## Comments
- Javadoc on a public/complex method states the rule or contract: inputs, guarantees, side effects. Not a restatement of the name.
- Inline `//` goes directly above a non-obvious step and explains WHY (ordering reason, defense in depth, side effect like "verifies AND consumes").
- Ticket or flag tags where relevant (for example `PROJ-123` on a feature flag).
- No comments on trivial code. Plain wording, no decoration.

## Naming and design
- Intent-revealing names, no abbreviations. Classes are nouns, methods are verbs, booleans start with `is/has/can`.
- No `Manager`/`Helper`/`Util` dumping grounds; name the class after its single job.
- One responsibility per class; depend on interfaces; composition over inheritance; constructor injection only; no static mutable state.

## Dead code and duplication
- Remove unused code, imports and commented-out code. No magic numbers (use constants or enums).
- Extract on the second duplicate, not the first.

## Exception hygiene
- Never swallow an exception; catch the most specific type; wrap with the cause (`new BaseException(code, e)`).
- No exceptions for control flow.

## Review heuristics
Flag and fix before finishing: long method, god class, feature envy (a method using another class's data more than its own), nesting over 2 levels, boolean flag parameters, more than 3 parameters (use a parameter object).

## Refactoring
- Only with tests green, in small steps, and never mixed with behavior changes in the same commit.

## Before finishing
- Re-read the diff against the rules above; split anything over the size targets.
- Checklist: names clear, no unused imports/code, no secrets or PII in logs, errors via `ErrorCode`, `@Transactional` set, tests added or updated (with ticket tags and the `docs/tests/<feature>.md` row), formatting clean.
- Say how to validate with the project's real commands; never claim tests passed unless run.
- Short before/after patterns: see [examples.md](examples.md).
