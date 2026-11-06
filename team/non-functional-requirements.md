# Non-Functional Requirements — Manager Library

Source: `spec-python/spec.md` §5 and `constitution.md`. Each requirement states how it is verified.

## 1. Requirements from the spec

| ID | Requirement | How it is measured |
|---|---|---|
| NFR-1 | Book search answers in under 300 ms with 5,000 books | Load test using the seeder in `tool` |
| NFR-2 | Two librarians lending the **last copy** at the same time: one wins, the other receives 409 | Concurrent test against real PostgreSQL |
| NFR-3 | Passwords never travel or are stored in plain text and never appear in logs | Review of logs and responses; Argon2id/bcrypt hashing |
| NFR-4 | Every HTTP error uses the same body (`code`, `message`, `details`) | Contract test over all routes |
| NFR-5 | The system starts with one command and the database starts empty | `infra` README: `docker compose up` |

## 2. Principles from the constitution that act as quality requirements

| ID | Requirement | Verification |
|---|---|---|
| NFR-6 | Domain logic does not depend on framework, ORM or HTTP | Architecture test; domain classes are plain PHP |
| NFR-7 | The schema belongs to the application migrations; `infra` contains no `CREATE TABLE` | `grep` over `infra` |
| NFR-8 | Constraints the database can enforce are enforced there (`CHECK`, `UNIQUE`, FK) | Direct `INSERT` via `psql` that violates them fails |
| NFR-9 | Derived values (loan status, days late) are calculated, never stored | `data-model.md` lists no `status` or `days_late` column |
| NFR-10 | Frozen values do not follow the catalog | Test: rename a book after lending, report keeps the old title |
| NFR-11 | No secrets in the repository; real `.env` ignored, only `.env.example` versioned | `.gitignore` check + pattern `grep` |
| NFR-12 | Dates are UTC instants (`timestamptz`) | Contract test: ISO 8601 with `Z` |

## 3. Team-added requirements

| ID | Requirement | How it is measured |
|---|---|---|
| NFR-13 | Public site works without the API and renders correctly on mobile and desktop | Open `page` offline from the API; browser responsive mode |
| NFR-14 | Small, frequent commits pushed to the forks throughout the exam | Git history |
