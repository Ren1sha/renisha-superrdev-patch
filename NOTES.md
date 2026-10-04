# Patch notes

## Summary of changes

Fixed search correctness and request handling, not the UI.

SQL `AND`/`OR` precedence in `TaskRepository`, `search_tasks.sql`, and the Oracle package let archived rows leak via description matches and skipped the status filter on title matches. Grouped the predicate. Escaped `LIKE` wildcards so `%`/`_` are literals.

Removed `Thread.sleep` in `TaskController` (up to 1s on short queries). Invalid status now returns 400 instead of 500. Clamped page/pageSize.

Frontend: abort in-flight fetches, always clear loading (errors were stuck on “Loading…”), debounce search 300ms, reset page when query/status changes.

## What I did not change

Did not add DB-level pagination (still loads the filtered set then `subList`). Fine for this seed size; a real Oracle path already pages. Did not add indexes, auth, or structured logging. Did not rewrite ROWNUM to `OFFSET/FETCH`. Left `System.out` logging.

## Biggest remaining risk

Search still does `LIKE '%term%'` on title and description with no index — that will not scale, and pagination is in memory. A production Oracle path should keep predicates parenthesized, push `OFFSET/FETCH` (or keyset) into SQL, and use a text index.

## Tools / AI

Cursor Grok 4.6: read the repo, traced AND/OR, confirmed against seed archived “api” rows, applied the patch, then curl + browser smoke tests. I wrote the debounce in `App.jsx` myself (300ms) rather than a separate hook.
