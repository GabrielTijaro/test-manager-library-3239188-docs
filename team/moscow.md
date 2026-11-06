# MoSCoW Prioritization — Manager Library

Basis: `spec-python/tasks.md` defines the **core** as reaching task T-17 with evidence; tasks T-18 to T-23 are what separates a
good result from an outstanding one. This prioritization follows that rule. Time box: Thursday 2026-11-05, 7:00 p.m. to 11:00 p.m. (Colombia).

## Must have (core, tasks T-01 to T-17)

| Task | Deliverable | Evidence |
|---|---|---|
| T-01 | Laravel skeleton and `GET /health` | Returns `{"status":"ok"}` |
| T-02 | Initial migration: six tables, FKs, unique indexes, five seeded categories | Queries in `data-model.md` §10 match |
| T-03 to T-06 | Domain: value objects, `Book`, `Loan`, `Member`, `User` | Unit tests |
| T-07 to T-11 | Use cases: auth, books, lend, return, overdue, most-borrowed report | Stories HU-01 to HU-10 pass |
| T-12 to T-13 | Persistence, including optimistic/conditional update for the last copy | Two concurrent loans: 201 and 409 (NFR-2) |
| T-14 | Password hashing and token | Password never in logs or responses (NFR-3) |
| T-15 | Cover storage | 2 MB ok, 3 MB 422, no cover -> `null` |
| T-16 | HTTP routes of `api-contract.md`, single error body, role authorization | Contract test |
| T-17 | Composition and startup, initial admin from environment | Login as admin on an empty database |

## Should have

| Task | Deliverable |
|---|---|
| T-22 | `app` (React): login, catalog, members, lend, return, overdue, report |
| T-20 | `infra`: `docker compose up` starts empty DB, API and frontend |
| T-18 | Push `ck_book_total_positive` and `ck_user_role` down to the database |

## Could have

| Task | Deliverable |
|---|---|
| T-21 | `tool`: demo seeder through the API (>= 40 books, 10 members, 15 loans, 3 overdue) |
| T-23 | `page`: static public site |
| T-19 | Architecture tests (articles I, II, III) |

## Won't have (out of scope, not reopened)

- Fines and payments
- Reservations
- Renewals
- Email or SMS notifications
- Bulk import
- Loans between branches
- Extra book attributes (publisher, year, language)
- Granting the `admin` role through the API

## Ordering rule

Nothing in Should or Could starts until the Must items have their evidence. One task with evidence is worth more than three half-done.
