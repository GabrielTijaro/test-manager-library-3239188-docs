# Constitution Compliance — Manager Library (Laravel / React)

Each article of `spec-python/constitution.md` is checked against the Laravel implementation. The constitution says a principle nobody
can verify is only an intention, so every row names the check that was run and its result.

| Article | Principle | How it is verified here | Result |
|---|---|---|---|
| I | The domain depends on nothing | `grep -rn "Illuminate\|use App\\(Application\|Infrastructure\|Http\)" app/Domain` returns nothing | Met |
| II | The core declares its ports | `app/Application/Ports/*` are plain PHP interfaces with no framework types; `grep -rn "use App\\(Infrastructure\|Http\)" app/Application` returns nothing | Met |
| III | A single composition point | `app/Providers/PortBindingsServiceProvider.php` is the only place where a port is bound to an adapter; no adapter is instantiated elsewhere in `app/` | Met |
| IV | SOLID is not decorative | One service per use case (`LendBooksService`, `ReturnItemService`, ...); small ports (`Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage`) | Met |
| V | The schema belongs to the migrations | All DDL is in `database/migrations/2026_11_05_000001_create_library_schema.php`; `infra` contains no table definitions | Met |
| VI | The engine enforces what it can | `ck_book_available_range`, `ck_book_total_positive`, `ck_user_role`, unique and partial indexes and FKs are in the first migration (T-18) | Met |
| VII | Derived values are calculated | The migration has no `status` or `days_late` column; `LoanPolicy::statusOf` and `daysLate` compute them | Met |
| VIII | Frozen values do not follow the catalog | `loan_item` stores `book_title`, `isbn`, `category_name`; test `test_report_keeps_the_frozen_title_after_renaming` | Met |
| IX | No secret enters the repository or a log | `.env.example` has empty `JWT_SECRET` and `ADMIN_PASSWORD`; the real `.env` is ignored; tests assert the password never appears in a response | Met, with one note (see below) |
| X | The engine wins | `data-model.md` was corrected when the implementation differed (schema `public`, T-18 constraints marked as engine rules) | Met |
| XI | Language | Code, columns and routes are in English; user messages are in Spanish | Deviation: team documentation is in English by team decision (see `spec-gaps-and-decisions.md`, decision 16) |
| XII | Nothing is done without evidence | `traceability-matrix.md` lists the evidence of every task and marks what is still pending | Met |

## Notes

- **Article IX:** `phpunit.xml` contains test-only values (`JWT_SECRET`, `ADMIN_PASSWORD`, database password) so the suite can run.
  They are not production secrets and they are only read under `APP_ENV=testing`. Production values come from the environment.
  `.env.example` also keeps `DB_PASSWORD=library` as a documented local default. This is a conscious, recorded exception.
- **Article III:** tests may build adapters directly; the rule applies to application code.
- **Architecture tests (T-19)** are not written yet. Until then, Articles I to III are verified by the `grep` commands above.
