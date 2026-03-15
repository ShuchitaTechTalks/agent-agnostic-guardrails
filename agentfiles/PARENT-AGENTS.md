# AGENTS.md — Spring Boot Agent-Agnostic Coding Standards

> **Source of truth** for AI coding agents. All code MUST comply before the build passes.
> Enforced by: **Checkstyle**, **SpotBugs**, **ArchUnit**.
> Layer-specific rules live in nested `AGENTS.md` files under each package.

---

## Project Overview

| Property       | Value                              |
|----------------|------------------------------------|
| Framework      | Spring Boot 3.2.x                  |
| Java Version   | Java 21                            |
| Build Tool     | Maven                              |
| Architecture   | Layered (Controller→Service→Repo)  |
| Database       | JPA / H2 (dev), PostgreSQL (prod)  |

---

## Build and Test Commands

```bash
./mvnw verify              # Full build with all quality checks
./mvnw test                # Unit + architecture tests only
./mvnw checkstyle:check    # Checkstyle only
./mvnw spotbugs:check      # SpotBugs only
./mvnw jacoco:report       # Generate coverage HTML report (target/site/jacoco)
./mvnw verify -DskipChecks # Skip checks (local dev only — never in CI)
```

## Definition of Done

Before marking any task complete:

1. Run `./mvnw verify` and confirm it exits with **`BUILD SUCCESS`**.
2. Fix every reported Checkstyle, SpotBugs, or ArchUnit violation — do not skip or suppress without documented justification.
3. Do not consider the task done until `./mvnw verify` passes clean.

## Companion Playbooks

For common, repeatable tasks (run checks, add endpoints, fix violations), see: `skills.md`.

> **CI runs `./mvnw verify`**. Any Checkstyle, SpotBugs, or ArchUnit failure **blocks the merge**.

---

## Logging Best Practices

> Logging rules apply to **all layers**. Violations are caught by Checkstyle (`Regexp` for `System.out`) and SpotBugs (`SBSC_*` for string concatenation).

### Logger Setup
- Use **SLF4J only** — never `System.out.println`, `System.err`, or `java.util.logging`.
- Prefer Lombok `@Slf4j` on the class. Otherwise: `private static final Logger log = LoggerFactory.getLogger(ClassName.class);`

### Log Levels
| Level   | When to use                                        | Example                                      |
|---------|----------------------------------------------------|----------------------------------------------|
| `ERROR` | Unrecoverable failures, caught exceptions          | `log.error("Failed to save book", ex);`      |
| `WARN`  | Recoverable issues, fallback paths taken           | `log.warn("Book not found: id={}", id);`     |
| `INFO`  | Business events, state changes                     | `log.info("Book created: id={}", id);`       |
| `DEBUG` | Execution flow, variable values during development | `log.debug("Fetching book id={}", id);`      |

### Mandatory Rules
- **Parameterized logging only** — never concatenate: `log.info("id={}", id)` not `log.info("id=" + id)`.
- **NEVER log sensitive data**: passwords, tokens, API keys, PII (emails, SSNs, names).
- **Never log and re-throw** the same exception — log once at the layer that handles it.
- **Never swallow exceptions** with empty catch blocks.
- Pass the exception object as the **last argument** to preserve the stack trace: `log.error("msg", ex)`.
- Keep log messages **concise and grep-friendly** — use `key=value` pairs for structured data.

---

## Spring Boot Coding Rules

### 1. Dependency Injection
- **ALWAYS** constructor injection — never `@Autowired` on fields.
- All dependencies: `private final` fields + Lombok `@RequiredArgsConstructor`.


### 2. Exception Handling
- All domain exceptions extend `CustomException` (base `RuntimeException`).
- Centralized handling via `@RestControllerAdvice` + `@ExceptionHandler`.
- Exception classes in the `exception` package.

### 3. JavaDoc
- All **public** classes, interfaces, methods must have JavaDoc.
- Include `@param`, `@return`, `@throws` where applicable.

### 4. Null Safety
- Return `Optional<T>` instead of `null`.
- Use `.orElseThrow()` / `.orElse()` — never bare `.get()`.
- Annotate with `@NonNull` / `@Nullable` where the contract matters.

### 5. Style Rules (Checkstyle)
- Refer to `checkstyle.xml` and `checkstyle-suppressions.xml` for details.

---

## Architecture Rules (ArchUnit)

```
Controller → Service → Repository
```

- **Controllers** depend only on **Services** — never Repositories or other Controllers.
- **Services** depend on Repositories and other Services — never Controllers.
- **Repositories** depend only on Entity/Model classes — never Services or Controllers.
- **No circular dependencies**.
- All `@Service` exceptions must extend `CustomException`.
- All controller methods must return `ResponseEntity`.

---

## Security

- No credentials in source — use env vars or Vault.
- Deny by default: require explicit allow-rules for CORS origins, methods, and headers.
- Restrict Actuator exposure to required endpoints only; never expose sensitive endpoints publicly.

---

## Performance & Scalability

- All list/search APIs must support pagination (`Pageable`) with safe defaults and a max page size cap.
- Set explicit timeouts for outbound HTTP/DB interactions; no infinite waits.

> Layer-specific performance rules (N+1, projections, caching, batching) live in `service/AGENTS.md` and `repository/AGENTS.md`.

---

## Java 21 Best Practices

- Prefer `record` for immutable DTO/value carriers.
- Use pattern matching and switch expressions when they improve readability and reduce branching noise.
- Use virtual threads for I/O-bound concurrency; do not use them as a blanket optimization for CPU-bound work.
- Prefer immutable collections for outward-facing responses/config snapshots.

---

## Testing Standards

### Structure & Naming
- Test class name mirrors the class under test: `BookServiceImplTest`, `BookControllerTest`.
- Method name format: `methodName_stateUnderTest_expectedBehavior` — e.g., `findById_whenNotFound_throwsException`.
- Use **Arrange / Act / Assert** (AAA) sections, separated by a blank line. No comments needed if structure is obvious.

### Test Types by Layer
| Layer      | Tool                  | Scope                                      |
|------------|-----------------------|--------------------------------------------|
| Controller | `@WebMvcTest`         | HTTP layer only — mock service with Mockito |
| Service    | Plain JUnit 5 + Mockito | Business logic — mock repository           |
| Repository | `@DataJpaTest`        | JPA queries against in-memory H2           |
| Full stack | `@SpringBootTest`     | Integration tests only — use sparingly     |

### Mandatory Rules
- Every new public method in `service` and `controller` packages must have at least one test.
- Use `@ExtendWith(MockitoExtension.class)` for unit tests — never load the full Spring context for pure logic tests.
- Prefer `@WebMvcTest` over `@SpringBootTest` for controller tests.
- Assert on **behaviour and output**, not on internal implementation details.
- Never use `Thread.sleep` in tests — use `Awaitility` for async assertions.
- Do not share mutable state between tests — each test must be fully independent.

### Coverage Thresholds (enforced by JaCoCo at `verify`)
| Metric         | Minimum |
|----------------|---------|
| Line coverage  | 80 %    |
| Branch coverage| 80 %    |

> Excluded from coverage: `model`, `dto`, `exception` packages and the main application entry class (boilerplate).

---

## Enforcement Plugins

| Tool           | Config                  | Phase      | Purpose                              |
|----------------|-------------------------|------------|--------------------------------------|
| **Checkstyle** | `checkstyle.xml`        | `validate` | Style, formatting, naming, JavaDoc   |
| **SpotBugs**   | `spotbugs-exclude.xml`  | `verify`   | Bugs, null safety, resource leaks    |
| **ArchUnit**   | `ArchitectureTest.java` | `test`     | Layer separation, dependency rules   |
| **JaCoCo**     | `pom.xml`               | `verify`   | Line & branch coverage gates         |

---
