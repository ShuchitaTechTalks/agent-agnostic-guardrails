# Controller Layer — AGENTS.md

> Layer-specific rules for `controller` package. See root `AGENTS.md` for full project standards.

---

## Rules

- `@RestController` + `@RequestMapping` on every controller class.
- Constructor injection only: `private final` service fields + `@RequiredArgsConstructor`.
- Inject **service interfaces** (e.g., `BookService`), never `Impl` classes, never `@Repository`.
- Keep controllers **thin** — HTTP concerns only, delegate logic to `@Service`.

### HTTP Methods & Responses
- Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` — never generic `@RequestMapping` for CRUD.
- Paths: REST-compliant, lowercase.
- **Always** return `ResponseEntity<T>` — never raw objects.
- Status codes: `200` GET/PUT, `201` POST, `204` DELETE, `400` validation, `404` not found.

### Input Validation
- Annotate request bodies with `@Valid` and use Bean Validation: `@NotBlank`, `@Positive`, `@Email`, etc.
- `GlobalExceptionHandler` catches validation failures automatically.

### Path Variables & Request Params
- `@PathVariable` for resource identifiers; `@RequestParam` for query filters.
- Optional params: `@RequestParam(required = false)`.

### Exception Handling
- Controllers do **NOT** catch exceptions — let `GlobalExceptionHandler` handle them.
- Service layer throws domain exceptions extending `CustomException`.
- Never expose stack traces or framework internals in API responses.

### Security
- Enforce authentication and authorization on all mutating endpoints (`POST`, `PUT`, `PATCH`, `DELETE`).
- Apply request size limits and basic rate limiting on public/auth-sensitive endpoints.

### Logging
- Use `@Slf4j` or manual SLF4J logger — see root `AGENTS.md` Logging Best Practices.
- Log request entry at `INFO`; never log request bodies containing PII.


### Testing
- Use `@WebMvcTest(XController.class)` — loads only the web layer, not the full context.
- Mock all service dependencies with `@MockBean`.
- Test every HTTP status code the endpoint can return (200, 201, 204, 400, 404).
- Use `MockMvc.perform(...)` + `.andExpect(status().isXxx())` and `.andExpect(jsonPath(...))`.
- Never use `@SpringBootTest` for controller unit tests.

---

## Architecture (ArchUnit-enforced)

- Controllers **may only depend on** `@Service` beans — never `@Repository` or other controllers.
- Violations fail `./mvnw verify`.

---

*Full standards: root `AGENTS.md`*
