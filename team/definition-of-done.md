# Definition of Done — Manager Library

Derived from Article XII of the constitution: *nothing is done without evidence*. A task is finished when its evidence exists and
can be executed, not when it is "already coded".

## Per task / user story

A task or story is **Done** only if all of these are true:

- [ ] Its evidence in `tasks.md` exists and was **run** (command or test output checked).
- [ ] Every acceptance criterion of the story passes (see `user-stories.md`).
- [ ] Business rules RN-1 to RN-6 are respected.
- [ ] Errors use the single error body (`code`, `message`, `details`) and the right status code.
- [ ] No secret is committed; the real `.env` is ignored and `.env.example` is updated.
- [ ] Domain code has no framework or ORM dependency.
- [ ] The schema change, if any, lives in a Laravel migration and `data-model.md` is updated in the same commit.
- [ ] Work is committed in small commits with Conventional Commit messages in English and **pushed to the fork's `main`**.

## Per repository

| Repo | Done when |
|---|---|
| `api` | Core T-01 to T-17 have evidence; contract test covers all routes and status codes |
| `app` | Full walkthrough works end to end: login, catalog, members, lend, return, overdue, report |
| `infra` | One command starts empty DB, API and frontend; contains no `CREATE TABLE` |
| `tool` | After running, `/loans/overdue` returns 3 lines and the report has rows |
| `page` | Opens without the API; correct on mobile and desktop |
| `docs` | `team/` contains roles, requirements, user stories, MoSCoW, DoD and spec decisions |

## For the exam delivery

- [ ] All six forks have the latest commits visible on GitHub before **11:00 p.m. Colombia time**.
- [ ] README in each fork explains how to run it.
- [ ] A last `git status` shows nothing left uncommitted in any repo.
