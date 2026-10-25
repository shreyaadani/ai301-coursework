# Unit 3 — Plan and Build: issue #38

**GitHub username:** shreyaadani

## Plan comment

Link: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-6392017638

> Here's my plan for this, built from [my repro above](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902682953) (`main` @ `2f4e82f`).
>
> **What the repro showed.** No test ever sends a request through `get_current_user`. `pytest tests/integration -m integration` gives `collected 0 items`, the unit suite with `--cov=api.middleware.auth` gives `375 passed` but `Module api.middleware.auth was never imported`, and grepping `tests/` for `middleware|get_current_user|TestClient|AsyncClient|httpx` exits 1. `tests/conftest.py` has no app, client or DB fixture, so nothing *could* call a protected route yet. @rose413 and @Christian101GTZ both found that the four cases already return 401, so this adds tests only and fixes nothing.
>
> **What I'll change.**
>
> - `tests/conftest.py`: a stub `get_db` session (records calls, returns or raises what the test sets), a `client` fixture that overrides `get_db` and returns `TestClient(app)` without `with`, so startup doesn't need Postgres (credit to @Christian101GTZ's follow-up for finding the `api/main.py:92` re-raise), and a `make_token` helper using `settings.secret_key` / `settings.jwt_algorithm`. `api.main` is imported inside the fixture so the unit tests never load the app.
> - `tests/integration/test_auth_middleware.py`, marked `integration`, all against `GET /profiles/{id}`:
>   - The issue's four cases: missing header, malformed, expired, wrong secret. Each asserts 401 and `WWW-Authenticate: Bearer`, and that the DB was never called. For the expired token I won't assert the detail text, because the `"Token has expired"` branch at `auth.py:44-49` is one classmates here found unreachable. I haven't tested that myself, so the test shouldn't pin either message.
>   - A valid-token control → 404 `Profile not found`, so a broken fixture can't pass as a 401.
>   - The two paths I flagged in my repro: unknown user → 401 (`auth.py:68-69`), lookup error → 500 `Database error` (`auth.py:61-66`). Same function, same fixture. If a reviewer would rather have them in a follow-up, they come out cleanly.
> - `.github/workflows/ci.yml`, `test-integration` job only. This part isn't in either plan above:
>   - Remove the exit-5 shim, which says "Delete this shim once the directory holds at least one test."
>   - Change `DATABASE_URL` to `postgresql+asyncpg://`. With CI's current `postgresql://` URL the app can't be imported, because `core/database.py` builds an async engine at import:
>
>     ```
>     $ DATABASE_URL=postgresql://pathreview:pathreview@localhost:5432/pathreview_test python -c "import api.main"
>     Traceback (most recent call last):
>       File "<string>", line 1, in <module>
>       File "api\main.py", line 8, in <module>
>         from api.routes import auth, health, profiles, reviews
>       File "api\routes\auth.py", line 10, in <module>
>         from core.database import get_db
>       File "core\database.py", line 11, in <module>
>         engine = create_async_engine(
>       ...
>     ModuleNotFoundError: No module named 'psycopg'
>     $ echo $?
>     1
>     ```
>
>     (Repo paths shortened. The `...` is SQLAlchemy's internal frames, ending in `sqlalchemy\dialects\postgresql\psycopg.py`, `import psycopg`. With the default `postgresql+asyncpg://` URL the same import exits 0.)
>
>     That job has never imported the app before, so the first PR to add an integration test would go red for this reason alone.
>
> **Not changing:** `api/middleware/auth.py`, `core/security.py`, `core/database.py`, `api/main.py`.
>
> **How I'll know it worked.** I'll re-run my three repro commands on the branch. Integration goes from `collected 0 items` to `7 passed` with Docker stopped. Coverage on `api.middleware.auth` goes from "never imported" to a real row with the 401/500 lines covered. The grep goes from exit 1 to matches in the two test files. To check the tests can fail, I'll temporarily make `auth.py:36` return instead of raise and expect the malformed and wrong-secret tests to fail, then revert. `make test-unit` should still show `375 passed, 53 xfailed`.
>
> **Not verified yet:** I haven't run the no-`with` `TestClient` approach myself, so it's the first thing I'll check. If it doesn't hold, I'll mount `api.routes.profiles.router` on a test-only app and say so. I also can't run CI locally, so the asyncpg fix gets confirmed on the PR.
>
> Branch: `fix/38-auth-integration-tests` on my fork.

## Branch

`fix/38-auth-integration-tests`, three commits (`9868159` → `e56a3d4`).

## Evidence

**Before** (`main` @ `2f4e82f`):
