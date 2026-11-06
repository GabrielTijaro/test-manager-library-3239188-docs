# Functional Requirements — Manager Library

Source of truth: `spec-python/spec.md` (user stories HU-01 to HU-10, business rules RN-1 to RN-6) and `api-contract.md`.
The business behaviour is kept as specified; only the technology is translated to **Laravel (PHP)** and **React**.

## 1. Actors

| Actor | Responsibility | Authentication |
|---|---|---|
| Administrator (`admin`) | Maintains books and members, creates librarians, sees all reports | Username + password -> token |
| Librarian (`librarian`) | Registers loans and returns; queries catalog, members and overdue items | Username + password -> token |
| Member | Person who borrows books. **Does not log in**: it is data, not a user | — |

The `admin` role is provisioned by the deployment from environment variables. Nobody can grant it at runtime.

## 2. Scope

**In scope:** book catalog with optional cover, five fixed categories, members, loans with several lines, return per line,
overdue books, most-borrowed report, login with roles.

**Out of scope (not reopened):** fines and payments, reservations, renewals, email/SMS, bulk import, loans between branches,
extra book attributes (no publisher, year or language).

## 3. Functional requirements

| ID | Requirement | Roles | Story |
|---|---|---|---|
| FR-01 | Log in and receive a token valid for 8 hours; invalid credentials return 401 without saying which field failed | all | HU-01 |
| FR-02 | Every endpoint except login and `/health` requires a token (401 without it, 403 with insufficient role) | all | HU-01 |
| FR-03 | Admin creates librarians (username lowercased and trimmed, unique -> 409; role `admin` requested -> 422) | admin | HU-02 |
| FR-04 | Admin creates and edits books: title, author, 13-digit ISBN (unique among active books), category, total copies >= 1 | admin | HU-03 |
| FR-05 | Changing total copies adjusts available copies by the same difference and never leaves them below 0 (422) | admin | HU-03 |
| FR-06 | Upload or remove a cover (JPG/PNG, max 2 MB); stored as an opaque key, `null` when absent | admin | HU-03 |
| FR-07 | Delete a book = soft delete; blocked (409) if it has open loans | admin | HU-03 |
| FR-08 | Search books by partial title/author, filter by category, active only, ordered by title, paginated (max 50), with total | authenticated | HU-04 |
| FR-09 | Get one book by id; missing or soft-deleted -> 404 | authenticated | HU-04 |
| FR-10 | Create members (name + unique document -> 409), list and search by name/document, paginated; members are never deleted | admin, librarian | HU-05 |
| FR-11 | Register a loan for one member with 1 to 3 lines; no repeated book (422); each line decrements one available copy atomically; no copies -> 409 and nothing is saved | admin, librarian | HU-06 |
| FR-12 | A member cannot hold more than 3 unreturned books (422 with current count) and cannot borrow while having an overdue book (409) | admin, librarian | HU-06 |
| FR-13 | Due date = `loaned_at + 14 days`; each line freezes title, ISBN and category name at loan time; loans are immutable | admin, librarian | HU-06 |
| FR-14 | Return one line at a time: sets `returned_at`, adds 1 to available copies; returning twice -> 409 and never adds twice; available never exceeds total | admin, librarian | HU-07 |
| FR-15 | When all lines are returned the loan is `closed` | system | HU-07 |
| FR-16 | Get a loan with its lines and calculated status (`open`, `overdue`, `closed`); list loans by date range, status and member, newest first, paginated | admin, librarian | HU-08 |
| FR-17 | List overdue lines (book, member, days late), most late first | admin, librarian | HU-09 |
| FR-18 | Most-borrowed report for a date range: one row per book and frozen category name, count, descending; end before start -> 422; empty range -> empty list; not broken down by librarian or member | admin | HU-10 |

## 4. Business rules

| ID | Rule |
|---|---|
| RN-1 | Available copies are always between 0 and total |
| RN-2 | Loan period is 14 days; maximum 3 unreturned books per member |
| RN-3 | A member with overdue books cannot borrow until they return them |
| RN-4 | Frozen line values (title, ISBN, category) never follow the live catalog |
| RN-5 | Loans and returns are never deleted; books only get soft delete |
| RN-6 | Dates are UTC instants; user messages in Spanish, code and columns in English |

## 5. Error model

All errors share one body: `code`, `message`, `details[]`. Status codes: 200/201/204, 401, 403, 404, 409 (state conflict),
422 (validation). Errors never expose traces, SQL, server paths or passwords.
