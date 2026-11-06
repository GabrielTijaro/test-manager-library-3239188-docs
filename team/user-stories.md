# User Stories — Manager Library
 
## Methodology
 
- **Format:** every story is written as **As a / I want / so that**, so each one names who needs it, what they do and why it matters.
- **Source:** stories HU-01 to HU-10 and their acceptance criteria (AC) come from `spec-python/spec.md`. The AC keep the spec numbering
  (AC-06.3 is CA-06.3 in the spec) so they can be traced.
- **Quality check (INVEST):** a story must be Independent enough to build alone, Negotiable in how, Valuable for a role, Estimable
  by its tasks in `tasks.md`, Small enough for one task group, and Testable through its acceptance criteria.
- **Acceptance criteria:** each AC is written so that a test can pass or fail it (status code, exact value, or rule).
- **Priority:** comes from `moscow.md`. Nothing in Should or Could starts before the Must stories have their evidence.
- **Done:** a story is accepted only when it meets the Definition of Done (`definition-of-done.md`): its evidence was run,
  every AC passes, and errors use the single error body.
- **Traceability:** each story points to its functional requirements (FR) and its tasks (T); the full path to code and tests is in
  `traceability-matrix.md`.
## Story map
 
| Story | As a... | I want to... | So that... | Requirements | Tasks | Priority |
|---|---|---|---|---|---|---|
| HU-01 | user | log in | I operate according to my role | FR-01, FR-02 | T-07, T-14, T-16 | Must |
| HU-02 | admin | create librarians | staff can register loans | FR-03 | T-07 | Must |
| HU-03 | admin | manage books | the catalog is up to date | FR-04 to FR-07 | T-08, T-15 | Must |
| HU-04 | user | search books | I find what a member asks for | FR-08, FR-09 | T-08 | Must |
| HU-05 | admin/librarian | manage members | I can lend to them | FR-10 | T-12, T-16 | Must |
| HU-06 | librarian | register a loan | a member takes up to 3 books | FR-11 to FR-13 | T-09, T-13 | Must |
| HU-07 | librarian | register a return | copies become available again | FR-14, FR-15 | T-10 | Must |
| HU-08 | admin/librarian | view and list loans | I follow what is lent | FR-16 | T-10, T-16 | Must |
| HU-09 | admin/librarian | see overdue books | I can chase them | FR-17 | T-10 | Must |
| HU-10 | admin | see the most-borrowed report | I know what the library lends most | FR-18 | T-11 | Must |
 
## Acceptance criteria
 
## HU-01 Log in
- AC-01.1 Valid credentials return a token valid for 8 hours.
- AC-01.2 Invalid credentials return 401 and do not reveal whether the username or the password failed.
- AC-01.3 Every endpoint except login and `/health` needs a token: no token 401, insufficient role 403.
## HU-02 Create librarians (admin)
- AC-02.1 Admin creates a user with name, password and role `librarian`.
- AC-02.2 Username is stored lowercased and trimmed; unique; duplicate returns 409.
- AC-02.3 Requesting role `admin` returns 422. The password is never returned or logged.
## HU-03 Manage books (admin)
- AC-03.1 Create a book with title, author, ISBN, category and total copies; available starts equal to total.
- AC-03.2 ISBN has 13 digits and is unique among active books; duplicate returns 409.
- AC-03.3 Total >= 1; changing total adjusts available by the same difference and never below 0 (422).
- AC-03.4 Cover (JPG/PNG, max 2 MB) is stored as an opaque key; without cover the field is `null`, never an empty string.
- AC-03.5 Delete is a soft delete; a book with open loans cannot be removed (409).
## HU-04 Search books
- AC-04.1 Partial text on title or author, category filter, active only, ordered by title, paginated (`page`, `size` max 50).
- AC-04.2 Response includes the total of results.
- AC-04.3 Unknown or soft-deleted book id returns 404.
## HU-05 Manage members
- AC-05.1 Admin and librarian create members with name and document; duplicate document returns 409.
- AC-05.2 List and search by name or document, paginated.
- AC-05.3 Members are never deleted.
## HU-06 Register a loan
- AC-06.1 One member, registered by the authenticated user, 1 to 3 lines.
- AC-06.2 The same book cannot repeat in a loan (422).
- AC-06.3 Each line decrements 1 available copy in the same operation; without copies 409 and nothing is saved.
- AC-06.4 A member cannot exceed 3 unreturned books in total (422 with the current count).
- AC-06.5 A member with an overdue book cannot borrow another (409).
- AC-06.6 Due date = `loaned_at + 14 days`; the line freezes title, ISBN and category name.
- AC-06.7 A registered loan is not edited or deleted.
## HU-07 Register a return
- AC-07.1 One line at a time; sets `returned_at` and adds 1 to available.
- AC-07.2 Returning an already returned line returns 409 and never adds twice.
- AC-07.3 Available never exceeds total (the database guarantees it).
- AC-07.4 When all lines are returned, the loan shows as closed.
## HU-08 View and list loans
- AC-08.1 A loan returns its lines and calculated status (`open`, `overdue`, `closed`).
- AC-08.2 List by date range, status and member, newest first, paginated.
- AC-08.3 Status is calculated, not stored.
## HU-09 Overdue books
- AC-09.1 Lists unreturned lines with `due_at` before now: book, member, days late.
- AC-09.2 Most days late first. Only `admin` and `librarian`.
## HU-10 Most-borrowed report
- AC-10.1 For a date range, one row per book and frozen category name, with times borrowed, descending.
- AC-10.2 End before start returns 422; a range with no loans returns an empty list, not an error.
- AC-10.3 Not broken down by librarian or member.
- AC-10.4 A closed period's report does not change if the book or category is renamed later.
- AC-10.5 Admin only.
