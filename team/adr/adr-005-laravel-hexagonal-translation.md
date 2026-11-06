# ADR-005 · Translating the hexagonal architecture to Laravel

- **Status:** accepted
- **Context:** the spec describes a hexagonal architecture for Python and .NET (`domain`, `application`, `adapters`, `bootstrap`).
  The challenge is solved with PHP and Laravel, which brings its own conventions (Eloquent, service container, `app/Http`).
  A direct copy would let Eloquent models and framework classes leak into the domain, which Article I forbids.
- **Decision:**
  1. Keep the layers as namespaces under `app/`: `Domain`, `Application` (use cases + ports), `Infrastructure` (adapters) and `Http`.
  2. Ports are PHP interfaces defined by the application layer. Adapters in `Infrastructure` implement them with the Laravel
     query builder (`DB::table`), not Eloquent models, so no ORM type crosses a port.
  3. `PortBindingsServiceProvider` is the single composition point (Article III); controllers receive use-case services by injection.
  4. Errors: domain and application throw their own exceptions; only `bootstrap/app.php` turns them into the single error body.
  5. Tests: PHPUnit; domain tests need no framework; feature tests run against real PostgreSQL.
- **Alternatives rejected:** Eloquent models as domain entities (couples the domain to the ORM and to Laravel); business logic in
  controllers or in model events (breaks SRP and makes the rules untestable without HTTP).
- **Consequences:** a little more code than "Laravel by the book", in exchange for domain rules that run without a database and
  adapters that can be replaced by changing one line in the provider (OCP). Eloquent features (relations, scopes) are intentionally not used.
- **How it is verified:** `grep` over `app/Domain` and `app/Application` finds no framework imports; `PortBindingsServiceProvider` is
  the only file that references both a port and its adapter (see `constitution-compliance.md`).
