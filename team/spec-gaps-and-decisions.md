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
| 8 | `data-model.md` uses schema `library`, but Laravel creates its `migrations` table before any migration runs and `infra` starts an empty database. | Use the default `public` schema. `data-model.md` was updated (header, §0 and §10 queries). | Article X: the engine wins, so the document is corrected. |
| 9 | Plan D-11 asks for JWT HS256 but names no library. | Own `Hs256TokenIssuer` (no extra package): 8 hours, claims `sub`, `role`, `iat`, `exp`. | Fewer dependencies; no personal data in the token. |
| 10 | The contract only defines `INVALID_CREDENTIALS` for login; it has no code for a missing or invalid token. | Missing, invalid or expired token -> 401 with `code = UNAUTHENTICATED`. | Keeps the single error body (NFR-4). |
| 11 | No minimum password length is specified. | Librarian passwords need at least 8 characters (422 otherwise). | Reasonable default; never logged or returned (NFR-3). |
| 12 | `days_late` is not defined for the first hours after the due date. | Whole days elapsed since `due_at` (0 during the first 24 hours). | Deterministic and testable with a fake clock. |
| 13 | PHP does not populate `$_FILES` for multipart `PUT` before PHP 8.4, but the contract uses `PUT /books/{id}/cover`. | Accept `PUT` (PHP 8.4+ uses `request_parse_body`) and also `POST` on the same path. | Clients on older PHP can still upload covers. |
| 14 | A loan request can include a book id that does not exist. | 404 `NOT_FOUND` with the field `items[i].book_id` in `details`. | Distinguishes a missing book from a validation error. |
| 15 | The contract's response shapes omit some useful ids. | Loan lines include `loan_id` and loans include `loaned_by_user_id` (extra fields). | Additive; does not break the contract. |
| 16 | Article XI asks for documentation in Spanish; the team works and documents in English. | Team documentation (`team/`) is in English; user-facing API messages stay in Spanish as the contract requires; code, columns and routes are in English. | The exam instructions ask for English work; the contract keeps Spanish messages for end users. Recorded as a conscious deviation from Article XI. |
| 17 | The spec says "ISBN has 13 digits" but does not ask for a check-digit validation. | Validate only 13 digits (`^\d{13}$`); the check digit is not verified. | Follows the spec literally; adding a rule the spec does not state would be inventing scope. |
| 18 | `PUT /books/{id}` replaces title, author, category and total; it does not list the ISBN. | The ISBN is immutable after creation. | The contract lists the replaceable fields explicitly; it also keeps frozen loan values consistent with the catalog. |
| 19 | CA-10.1 says "one row per book and frozen category name", but the grouping is by `book_id + book_title + category_name`, and ADR-004 only mentions a category change. | A book renamed inside the reported range appears in **two rows** (old title, new title), like a category change does. | Same principle as ADR-004: the report shows the frozen values and never picks a winner. |
| 20 | "Maximum 2 MB" for covers does not say if MB means 1,000,000 or 1,048,576 bytes. | 2 x 1024 x 1024 bytes (2,097,152). | Common binary reading; a 3 MB file is rejected either way (the evidence in `tasks.md`). |
| 21 | Plan D-07 lists a `DocumentNumber` value object, but T-03 only asks for `Isbn`, `Quantity` and `DateRange`. | `DocumentNumber` is not implemented as an object; the document is trimmed and required in `CreateMemberService`, unique in the engine. | `tasks.md` defines the evidence; the uniqueness rule that matters is enforced by the database. |
 
## Translation notes (Python/.NET -> Laravel/React)
 
- `FastAPI`/SQLAlchemy/Alembic -> Laravel routes/controllers, Eloquent or query builder, Laravel migrations.
- JWT HS256 with 8-hour validity is kept; implemented with a Laravel-compatible JWT or token approach, key from `JWT_SECRET`.
- Password hashing: Argon2id (Laravel supports `argon2id` in `config/hashing.php`).
- `pytest` -> PHPUnit/Pest; `import-linter` architecture tests -> a PHP architecture test (e.g. Pest arch tests) if time allows (T-19, Could).
- Error messages shown to users stay in **Spanish** as the API contract requires; code, columns and routes are in English.
 
