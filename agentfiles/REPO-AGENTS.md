# Repository Layer — AGENTS.md

> Layer-specific rules for `repository` package. See root `AGENTS.md` for full project standards.

---

## Rules

### Structure
- All repositories must be **Spring Data interfaces** — never concrete classes.
- Extend `JpaRepository<Entity, ID>`. Annotate with `@Repository`.
- Naming: `EntityName + Repository` (e.g., `BookRepository`).

### Query Methods
- Prefer **derived query methods** (`findBy`, `existsBy`, `countBy`) — Spring generates SQL automatically.
- For complex queries, use `@Query` with **JPQL** and `@Param` for parameter binding.
- **Always parameterize** queries — never concatenate user input (SQL injection risk).
- Avoid native SQL unless absolutely necessary (database-specific features). If used, set `nativeQuery = true` and add a comment explaining why.

### Return Types
- `Optional<T>` — single result that may not exist.
- `List<T>` — multiple results (empty list if none).
- `boolean` — existence checks (`existsBy*`).
- `long` — count queries (`countBy*`).
- `Page<T>` — paginated results (use `Pageable` parameter).
- **Never** return `null`.

### No Business Logic
- Repositories handle **data access only** — no calculations, validations, or transformations.
- Complex queries are fine; complex logic belongs in `@Service`.

### Performance
- Avoid N+1 queries: use fetch joins, `@EntityGraph`, or JPQL projections for known access paths.
- For read-only list views, return DTO projections instead of full entities.
- Use `saveAll` for bulk writes; never call `save` in a loop.

### Relationships & Transactions
- Let JPA/Hibernate handle lazy loading and proxies.
- Do **not** manage transactions in repositories — that's the service layer's job.



### Testing
- Use `@DataJpaTest` — loads only JPA slice (H2 in-memory), no full context.
- Test every custom `@Query` method; do not test Spring Data derived methods.
- Verify `Optional.empty()` is returned correctly for non-existent records.
- Use `@Sql` or `TestEntityManager` to seed data — never rely on pre-existing state.

---

## Architecture (ArchUnit-enforced)

- Repositories **may only depend on** Entity/Model classes and Spring Data interfaces.
- Repositories **must NOT depend on** `@Service`, `@RestController`, or servlet classes.
- SpotBugs flags unsanitized SQL queries.
- Violations fail `./mvnw verify`.

---

*Full standards: root `AGENTS.md`*
