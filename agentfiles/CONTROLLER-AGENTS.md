# Controller Layer — AGENTS.md

Rules for `controller` package. Extends root `AGENTS.md`.

## Rules

- `@RestController` + `@RequestMapping` on every controller class.
- Constructor injection only: `private final` service fields + `@RequiredArgsConstructor`.
- Inject service interfaces (e.g., `BookService`), never `Impl` classes, never `@Repository`.
- Keep controllers thin — HTTP concerns only, delegate logic to `@Service`.
- Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` — never generic `@RequestMapping` for CRUD.
- Paths: REST-compliant, lowercase.
- Always return `ResponseEntity<T>` — never raw objects.
- Status codes: `200` GET/PUT, `201` POST, `204` DELETE, `400` validation, `404` not found.

## Validation

- Annotate request bodies with `@Valid` and use Bean Validation: `@NotBlank`, `@Positive`, `@Email`, etc.
- `@PathVariable` for resource identifiers; `@RequestParam` for query filters.
- Optional params: `@RequestParam(required = false)`.

## Exception Handling

- Controllers do NOT catch exceptions — let `GlobalExceptionHandler` handle them.
- Never expose stack traces or framework internals in API responses.

## Security

- Enforce auth on all mutating endpoints (`POST`, `PUT`, `PATCH`, `DELETE`).
- Apply request size limits and basic rate limiting on public/auth-sensitive endpoints.

## Testing

- Use `@WebMvcTest(XController.class)` — loads only web layer, not full context.
- Mock all service dependencies with `@MockBean`.
- Test every HTTP status code the endpoint can return (200, 201, 204, 400, 404).
- Use `MockMvc.perform(...)` + `.andExpect(status().isXxx())` and `.andExpect(jsonPath(...))`.
- Never use `@SpringBootTest` for controller unit tests.
