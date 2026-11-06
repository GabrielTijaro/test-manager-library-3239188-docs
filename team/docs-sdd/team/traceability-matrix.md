# Traceability Matrix — Manager Library

SDD evidence: every requirement of the spec is traced to the decision that interprets it, the code that implements it and the
test that proves it. Source of truth: `spec-python/spec.md`, `api-contract.md`, `data-model.md` and `tasks.md`.
Decisions where the spec was silent or had to be translated to Laravel are in `spec-gaps-and-decisions.md` (numbers below).

Status legend: **Done** = implemented and its test exists; **Partial** = implemented, evidence incomplete; **Pending** = not started.
A test counts as evidence only after it was **run** (Article XII). Team members confirm the run result in the last column of §3.

## 1. User stories to code and tests (repo `api`)

| Story | Acceptance criteria | Decision | Implementation | Evidence (test) | Status |
|---|---|---|---|---|---|
| HU-01 Log in | CA-01.1 to CA-01.3 | #9, #10 | `LoginService`, `Hs256TokenIssuer`, `AuthenticateToken`, `RequireRole` | `LibraryApiTest`: login failure message, 8-hour token, token and role checks; `ContractCoverageTest`: 401 on every protected route | Done |
| HU-02 Create librarians | CA-02.1 to CA-02.3 | #11 | `CreateLibrarianService`, `Username` | `LibraryApiTest::test_admin_creates_librarians_but_not_admins`; `ContractCoverageTest`: short password, no password in responses | Done |
| HU-03 Manage books | CA-03.1 to CA-03.5 | #13 | `CreateBookService`, `UpdateBookService`, `DeleteBookService`, `UploadCoverService`, `Book` | ISBN unique and total >= 1; total adjusts available; 2 MB cover rejected; cover upload/read/remove; open loans block deletion | Done |
| HU-04 Search books | CA-04.1 to CA-04.3 | #3 | `SearchBooksService`, `DbBookRepository::search` | size capped at 50; category filter; 404 for unknown or deleted | Done |
| HU-05 Manage members | CA-05.1 to CA-05.3 | — | `CreateMemberService`, `SearchMembersService`, `GetMemberService` | `ContractCoverageTest`: create, duplicate 409, search, detail with `active_loans` | Done |
| HU-06 Register a loan | CA-06.1 to CA-06.7 | #1, #4, #5, #14 | `LendBooksService`, `LoanPolicy`, `Loan`, `DbBookRepository::decrementIfAvailable` | last copy (201 then 409); shape 422; limit of 3; overdue blocks; frozen title; 14-day due date; `LoanTest` | Done |
| HU-07 Register a return | CA-07.1 to CA-07.4 | — | `ReturnItemService`, `DbLoanRepository::markReturned`, `Loan::returnItem` | returning twice gives 409 and never adds twice; loan becomes `closed`; `LoanTest` | Done |
| HU-08 View and list loans | CA-08.1 to CA-08.3 | #12 | `GetLoanService`, `ListLoansService`, `LoanPresenter` | `ContractCoverageTest`: filters by range, status and member; calculated status | Done |
| HU-09 Overdue books | CA-09.1, CA-09.2 | #12 | `ListOverdueService` | `test_overdue_blocks_new_loans_and_is_listed_with_days_late` | Done |
| HU-10 Most-borrowed report | CA-10.1 to CA-10.5 | #6 | `MostBorrowedReportService`, `DbLoanRepository::mostBorrowed` | `test_report_keeps_the_frozen_title_after_renaming`; ordering and no per-member breakdown; invalid range 422 | Done |

## 2. Business rules and non-functional requirements

| Rule | Where it is enforced | Evidence |
|---|---|---|
| RN-1 available between 0 and total | Engine: `ck_book_available_range`; domain: `Book` | `BookTest` (never below 0, never above total) |
| RN-2 14 days, max 3 unreturned | `LoanPolicy`, `LendBooksService` | `DomainTest`, `LoanTest`, limit test |
| RN-3 overdue blocks new loans | `LendBooksService` (409 `MEMBER_HAS_OVERDUE`) | overdue test |
| RN-4 frozen values | `loan_item.book_title/isbn/category_name` | frozen-title and report tests |
| RN-5 no deletes | no delete operation exists; FK RESTRICT; soft delete | open-loans test |
| RN-6 UTC and Spanish messages | `Iso::format`; Spanish `message` text | `ContractCoverageTest::test_loan_response_has_the_contract_shape_and_utc_dates` |
| NFR-2 last copy under concurrency | conditional update + range `CHECK` (decision #1) | **Partial:** the test is sequential; a real concurrent test is pending |
| NFR-3 password never leaks | `password_hash` never selected into responses | `ContractCoverageTest::test_password_never_appears_in_any_response` |
| NFR-4 single error body | `bootstrap/app.php` exception renderer | `assertErrorBody` helper over all error tests |
| NFR-7 schema owned by migrations | `database/migrations/…create_library_schema.php`; `infra` has no DDL | `grep` over `infra` |

## 3. Tasks and evidence (`tasks.md`)

| Task | Status | Evidence | Run confirmed by |
|---|---|---|---|
| T-01 Skeleton and `/health` | Done | `test_health_is_public` | team |
| T-02 Initial migration and seed | Done | migration file; categories test | team |
| T-03 Value objects | Done | `DomainTest` | team |
| T-04 `Book` aggregate | Done (not yet wired into services) | `BookTest` | pending run |
| T-05 `Loan` aggregate | Done (not yet wired into services) | `LoanTest` | pending run |
| T-06 `Member` and `User` | Partial: `Username` only | `DomainTest` | team |
| T-07 to T-11 Use cases | Done | feature tests per story (§1) | team |
| T-12, T-13 Persistence | Done; concurrency evidence partial | repositories + feature tests | team |
| T-14 Hash and token | Done | login and password tests | team |
| T-15 Cover storage | Done | cover tests | pending run |
| T-16 HTTP and contract test | Done | `LibraryApiTest` + `ContractCoverageTest` | pending run |
| T-17 Composition and startup | Done | `library:ensure-admin` | team |
| T-18 DB-level invariants | Done (in the first migration) | `ck_book_total_positive`, `ck_user_role`; `data-model.md` §4 updated | team |
| T-19 Architecture tests | Pending | — | — |
| T-20 `infra` | Delivered, not yet run with Docker | `docker-compose.yml`, README | pending run |
| T-21 `tool` seeder | Pending | — | — |
| T-22 `app` | Pending | — | — |
| T-23 `page` | Pending | — | — |

## 4. Known gaps (honest status)

1. Real concurrency test for NFR-2 (T-13).
2. `Book` and `Loan` exist and are tested but the services still use SQL conditional updates, which is what protects the last copy under concurrency.
3. `Member` and `User` aggregates (T-06) and T-19 are not done.
4. The spec text and the constitution ask for documentation in Spanish; this team documentation is written in English by team decision.
