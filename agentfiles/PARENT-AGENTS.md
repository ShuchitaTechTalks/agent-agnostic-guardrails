# AGENTS.md — Spring Boot Coding Standards

Source of truth for AI coding agents. Enforced by Checkstyle, SpotBugs, ArchUnit.
Layer-specific rules live in nested `AGENTS.md` files under each package.

## Project Overview

- Spring Boot 3.2.x, Java 21, Maven
- Layered architecture: Controller → Service → Repository
- JPA / H2 (dev), PostgreSQL (prod)

## Build Commands

```bash
./mvnw verify              # Full build with all quality checks
./mvnw test                # Unit + architecture tests only
./mvnw checkstyle:check    # Checkstyle only
./mvnw spotbugs:check      # SpotBugs only
```

## Definition of Done

Run `./mvnw verify` — it must exit with `BUILD SUCCESS`. Fix every Checkstyle, SpotBugs, or ArchUnit violation; do not skip or suppress without documented justification.

## Logging

- Use SLF4J only — never `System.out`, `System.err`, or `java.util.logging`.
- Prefer Lombok `@Slf4j`. Fallback: `private static final Logger log = LoggerFactory.getLogger(ClassName.class);`
- Levels: `ERROR` (unrecoverable), `WARN` (recoverable/fallback), `INFO` (business events), `DEBUG` (dev flow).
- Parameterized only: `log.info("id={}", id)` — never concatenate.
- Never log sensitive data (passwords, tokens, PII).
- Never log-and-rethrow the same exception; log once at the handling layer.
- Never swallow exceptions with empty catch blocks.
- Pass exception as last arg: `log.error("msg", ex)`.
- Use `key=value` pairs for structured, grep-friendly messages.

## Coding Rules

- Constructor injection only — never `@Autowired` on fields. Use `private final` + `@RequiredArgsConstructor`.
- All domain exceptions extend `CustomException` (`RuntimeException`). Centralized handling via `@RestControllerAdvice` + `@ExceptionHandler`. Exception classes in `exception` package.
- JavaDoc on all public classes, interfaces, methods. Include `@param`, `@return`, `@throws`.
- Return `Optional<T>` instead of `null`. Use `.orElseThrow()`/`.orElse()` — never bare `.get()`. Annotate `@NonNull`/`@Nullable` where needed.
- Refer to `checkstyle.xml` and `checkstyle-suppressions.xml` for style details.

## Architecture (ArchUnit)

- Controllers depend only on Services — never Repositories or other Controllers.
- Services depend on Repositories and other Services — never Controllers.
- Repositories depend only on Entity/Model classes — never Services or Controllers.
- No circular dependencies.
- All `@Service` exceptions must extend `CustomException`.
- All controller methods must return `ResponseEntity`.

## Security

- No credentials in source — use env vars or Vault.
- Deny by default for CORS origins, methods, headers.
- Restrict Actuator to required endpoints; never expose sensitive endpoints publicly.

## Performance

- All list/search APIs must support pagination (`Pageable`) with safe defaults and max page size cap.
- Set explicit timeouts for outbound HTTP/DB calls; no infinite waits.

## Java 21

- Prefer `record` for immutable DTOs/value carriers.
- Use pattern matching and switch expressions where they improve readability.
- Use virtual threads for I/O-bound concurrency only (not CPU-bound).
- Prefer immutable collections for outward-facing responses/config.

## Testing

- Test class mirrors source: `BookServiceImplTest`, `BookControllerTest`.
- Method name: `methodName_stateUnderTest_expectedBehavior`.
- Use Arrange/Act/Assert sections.
- Every new public method in `service`/`controller` must have at least one test.
- Assert on behaviour/output, not internal implementation.
- Never use `Thread.sleep` — use Awaitility for async.
- No shared mutable state between tests.
- Coverage minimums (JaCoCo): 80% line, 80% branch. Excluded: `model`, `dto`, `exception` packages and main app class.
- Layer-specific testing rules live in each layer's `AGENTS.md`.
