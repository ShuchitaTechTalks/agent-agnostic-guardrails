# Repository Layer — AGENTS.md

Rules for `repository` package. Extends root `AGENTS.md`.

## Rules

- All repositories must be Spring Data interfaces — never concrete classes.
- Extend `JpaRepository<Entity, ID>`. Annotate with `@Repository`.
- Naming: `EntityName + Repository` (e.g., `BookRepository`).

## Query Methods

- Prefer derived query methods (`findBy`, `existsBy`, `countBy`).
- For complex queries, use `@Query` with JPQL and `@Param` for parameter binding.
- Always parameterize queries — never concatenate user input (SQL injection risk).
- Avoid native SQL unless necessary. If used, set `nativeQuery = true` and add a comment explaining why.

## Return Types

- `Optional<T>` for single results that may not exist.
- `List<T>` for multiple results (empty list if none).
- `boolean` for existence checks (`existsBy*`).
- `long` for count queries (`countBy*`).
- `Page<T>` for paginated results (use `Pageable` parameter).
- Never return `null`.

## No Business Logic

- Repositories handle data access only — no calculations, validations, or transformations.
- Complex queries are fine; complex logic belongs in `@Service`.

## Performance

- Avoid N+1 queries: use fetch joins, `@EntityGraph`, or JPQL projections for known access paths.
- For read-only list views, return DTO projections instead of full entities.
- Use `saveAll` for bulk writes; never call `save` in a loop.

## Relationships & Transactions

- Let JPA/Hibernate handle lazy loading and proxies.
- Do not manage transactions in repositories — that's the service layer's job.

## Testing

- Use `@DataJpaTest` — loads only JPA slice (H2 in-memory), no full context.
- Test every custom `@Query` method; do not test Spring Data derived methods.
- Verify `Optional.empty()` is returned correctly for non-existent records.
- Use `@Sql` or `TestEntityManager` to seed data — never rely on pre-existing state.
