# NOTES

## Summary of changes
1. **SQL precedence** (`TaskRepository`, `db/queries/search_tasks.sql`, Oracle package count + cursor): `A AND B OR C AND D` parsed as `(A AND B) OR (C AND D)`, so archived tasks leaked and the status filter was ignored for title matches. Parenthesised the title/description match; added `id` as an ORDER BY tie-breaker for stable paging.
2. **Controller**: removed a `Thread.sleep` that delayed short/empty queries up to 1s. Unknown `status` now returns 400 (was 500). `page < 1` and `pageSize` outside 1..100 return 400 (negative `subList` index was a 500).
3. **`useTasks`**: stale responses could overwrite newer ones, so requests are aborted with `AbortController`. `loading` stuck on after errors and `error` never cleared; both fixed.
4. **`App`**: 300ms search debounce; reset to page 1 when query/status changes.
5. **Oracle**: `v_term VARCHAR2(257)` failed for terms over 255 chars.

## Not changed
- Pagination is still in memory. Proper fix is DB `LIMIT/OFFSET` plus a count query, a bigger diff than a patch warrants.
- LIKE wildcards (`%`, `_`) in input are not escaped.
- `System.out.println` logging left alone.

## Biggest remaining risk
In-memory pagination loads every matching row per request, and `LIKE '%x%'` cannot use an index, so this degrades as the table grows. No auth either.

## Assumptions
- 400 is right for bad input; max page size 100 is reasonable.

## Tools / AI used
Used Claude to review the code and draft fixes; I reviewed each change. Verified the SQL fix by running old vs new query on the seed data (old leaked 2 archived rows for `q=api`). Frontend builds. I could not run the Spring Boot backend in my environment, so the Java changes are untested at runtime.
