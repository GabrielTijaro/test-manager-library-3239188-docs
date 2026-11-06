# User Stories — Manager Library

Methodology: user stories in the form **As a / I want / so that**, each with testable acceptance criteria (AC) taken from
`spec-python/spec.md`. A story is finished when it meets the Definition of Done (`definition-of-done.md`).
Priority comes from `moscow.md`.

| Story | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| HU-01 | user | log in | I operate according to my role | Must |
| HU-02 | admin | create librarians | staff can register loans | Must |
| HU-03 | admin | manage books | the catalog is up to date | Must |
| HU-04 | user | search books | I find what a member asks for | Must |
| HU-05 | admin/librarian | manage members | I can lend to them | Must |
| HU-06 | librarian | register a loan | a member takes up to 3 books | Must |
| HU-07 | librarian | register a return | copies become available again | Must |
| HU-08 | admin/librarian | view and list loans | I follow what is lent | Must |
| HU-09 | admin/librarian | see overdue books | I can chase them | Must |
| HU-10 | admin | see the most-borrowed report | I know what the library lends most | Must |

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
