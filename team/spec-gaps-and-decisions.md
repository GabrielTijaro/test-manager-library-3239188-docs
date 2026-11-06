# Spec Gaps and Decisions — Manager Library

While reading the spec for the Laravel/React translation we found points the spec does not settle. Each one is recorded with the
decision taken and why, so the choices are traceable (SDD: when the spec is silent, decide explicitly and write it down).

| # | Gap | Decision | Reason |
|---|---|---|---|
| 1 | Plan D-04 / ADR-002 call for a "version token per book", but `data-model.md` §3 has no version column. Article X says the data model wins. | Use a **conditional update inside a transaction** (`UPDATE books SET available_copies = available_copies - 1 WHERE id = ? AND available_copies > 0`) plus the range `CHECK`. If 0 rows change, return 409 `BOOK_NOT_AVAILABLE`. | Satisfies NFR-2 (last copy: one 201, one 409) without changing the data model. Same optimistic spirit as ADR-002. |
| 2 | `GET /books/{id}/cover` appears only in the text of `api-contract.md`, not in the endpoint table, and has no role. | Any authenticated role may call it; returns the binary or 404. | Consistent with `GET /books` (authenticated). |
| 3 | If `size` > 50 it is unclear whether to cap or reject. | Cap to 50; do not error. | More tolerant for clients; the response always echoes the real `size`. |
| 4 | CA-06.4 does not say whether "more than 3 unreturned" includes the books requested now. | Reject with 422 if (current unreturned + requested lines) > 3, returning the current count in `details`. | Prevents a member from ending above the limit. |
| 5 | No order is given when several loan rules fail at once. | Order: (1) 422 request shape, including repeated book; (2) 409 member has overdue books; (3) 422 limit of 3; (4) 409 no copies. | Validation before state; member rules before stock; deterministic for contract tests. |
| 6 | Report `from`/`to` are dates, compared against `loaned_at` (`timestamptz`). | Inclusive on both ends: `loaned_at >= from 00:00 UTC` and `loaned_at < (to + 1 day) 00:00 UTC`. | Makes the end date inclusive without losing loans made late on the last day. |
| 7 | `ck_book_total_positive` and `ck_user_role` are marked pending until T-18. | Put them in the **first migration** and update `data-model.md` §4 in the same commit. | Article VI: enforce in the database whatever it can enforce; Laravel migrations make it free. |

## Translation notes (Python/.NET -> Laravel/React)

- `FastAPI`/SQLAlchemy/Alembic -> Laravel routes/controllers, Eloquent or query builder, Laravel migrations.
- JWT HS256 with 8-hour validity is kept; implemented with a Laravel-compatible JWT or token approach, key from `JWT_SECRET`.
- Password hashing: Argon2id (Laravel supports `argon2id` in `config/hashing.php`).
- `pytest` -> PHPUnit/Pest; `import-linter` architecture tests -> a PHP architecture test (e.g. Pest arch tests) if time allows (T-19, Could).
- Error messages shown to users stay in **Spanish** as the API contract requires; code, columns and routes are in English.
