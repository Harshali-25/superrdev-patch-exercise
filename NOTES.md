# NOTES

## Summary of changes

1.SQL search: Fixed the `AND`/`OR` precedence problem in the task search queries. Added parentheses so archived tasks are always excluded and the status filter works correctly. Added `id` as a tie-breaker in `ORDER BY` for more stable pagination.

2.Controller: Removed the artificial `Thread.sleep()` delay. Invalid `status` now returns `400` instead of `500`. Also added validation for `page` and `pageSize`; `page < 1` and `pageSize` outside `1..100` return `400`.

3. Frontend requests: Updated `useTasks` to use `AbortController` so an older request is cancelled when a newer search, status, or page request starts. Cancelled requests do not update the state. Fixed loading and error state handling.

4. Search and pagination: Added a 300ms debounce so typing does not send a request for every key press. The page resets to page 1 when the debounced query or status changes.

5.Oracle SQL:Fixed the `VARCHAR2` term length so longer search terms can be handled.

## Not changed

- Pagination is still performed in memory. Moving it to the database with `LIMIT/OFFSET` and a count query would require a larger change.
- `LIKE` wildcards such as `%` and `_` are not escaped.
- Existing `System.out.println` logging was left unchanged.

## Biggest remaining risk

In-memory pagination loads all matching rows for every request. Also, `LIKE '%x%'` can become slow as the number of tasks grows.

## Assumptions

- `400 Bad Request` is appropriate for invalid input.
- Maximum `pageSize` of 100 is reasonable.

## Tools / AI used

I used Claude to review the code, explain the bugs, and help with the fixes. I reviewed the changes myself and ran the application to test the API and frontend behavior.
