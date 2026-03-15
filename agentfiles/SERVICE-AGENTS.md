# Service Layer — AGENTS.md

> Layer-specific rules for `service` package. See root `AGENTS.md` for full project standards.

---

## Rules

### Structure
- `@Service` on every implementation class.
- Define a **Java interface** per service; implementation uses `Impl` suffix.
- Always inject the **interface**, not the implementation.

### Transaction Management
- `@Transactional` on write methods (create, update, delete).
- `@Transactional(readOnly = true)` on read methods — set at class level if most methods are reads.

### Exception Handling
- Throw **domain exceptions** extending `CustomException` — never raw `RuntimeException`.
- One exception class per error scenario (e.g., `BookNotFoundException`), in the `exception` package.
- Include meaningful messages with relevant data (IDs, names).

### Dependencies
- May depend on: `@Repository` beans and other `@Service` interfaces.
- **Must NOT** depend on: `@RestController`, `@Controller`, or any servlet/HTTP classes (`HttpServletRequest`, `HttpHeaders`, etc.).
- Avoid circular dependencies between services.

### Business Logic & Validation
- All business logic lives here — controllers validate syntax, services validate business rules.
- Use `Optional<T>` for nullable results; prefer `.orElseThrow()` with domain exceptions.

### Performance
- Cap incoming `Pageable` page size at a safe maximum — never pass unbounded page sizes to the repository.
- Use `@Cacheable` only for stable, read-heavy data; always define TTL and a documented cache invalidation path.


### JavaDoc
- Describe business-level behavior, not implementation details.

### Testing
- Use plain JUnit 5 + `@ExtendWith(MockitoExtension.class)` — no Spring context.
- Mock all `@Repository` and `@Service` dependencies with `@Mock`; inject via `@InjectMocks`.
- Test every branch: happy path, `Optional.empty()`, and every thrown domain exception.
- Name tests: `methodName_stateUnderTest_expectedBehavior`.

---

## Architecture (ArchUnit-enforced)

- Services **may depend on** `@Service` and `@Repository` beans.
- Services **must NOT depend on** controllers or servlet classes.
- Violations fail `./mvnw verify`.

---

*Full standards: root `AGENTS.md`*
