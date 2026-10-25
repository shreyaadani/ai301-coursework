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

```
$ python -m pytest tests/integration -v -m integration
collecting ... collected 0 items
============================ no tests ran in 0.36s ============================
exit status: 5

$ python -m pytest tests/unit -m unit --cov=api.middleware.auth --cov-report=term-missing -q
375 passed, 53 xfailed, 2 warnings in 22.97s
exit status: 0

$ python -m pytest tests/integration -m integration --cov=api.middleware.auth --cov-report=term-missing -q
CoverageWarning: Module api.middleware.auth was never imported. (module-not-imported)
CoverageWarning: No data was collected. (no-data-collected)
WARNING: Failed to generate report: No data to report.
no tests ran in 0.42s
exit status: 5

$ grep -rnE "middleware|get_current_user|TestClient|AsyncClient|httpx" tests/
exit status: 1
```

**After** (`fix/38-auth-integration-tests` @ `e56a3d4`):

```
$ python -m pytest tests/integration -v -m integration
collecting ... collected 7 items
tests/integration/test_auth_middleware.py::test_missing_authorization_header PASSED [ 14%]
tests/integration/test_auth_middleware.py::test_malformed_token PASSED   [ 28%]
tests/integration/test_auth_middleware.py::test_expired_token PASSED     [ 42%]
tests/integration/test_auth_middleware.py::test_wrong_signing_secret PASSED [ 57%]
tests/integration/test_auth_middleware.py::test_valid_token_reaches_route PASSED [ 71%]
tests/integration/test_auth_middleware.py::test_unknown_user PASSED      [ 85%]
tests/integration/test_auth_middleware.py::test_user_lookup_error PASSED [100%]
======================== 7 passed, 3 warnings in 2.63s ========================
exit status: 0

$ python -m pytest tests/unit -m unit --cov=api.middleware.auth --cov-report=term-missing -q
375 passed, 53 xfailed, 2 warnings in 22.33s
exit status: 0

$ python -m pytest tests/integration -m integration --cov=api.middleware.auth --cov-report=term-missing -q
Name                     Stmts   Miss  Cover   Missing
------------------------------------------------------
api\middleware\auth.py      38      5    87%   41, 45, 53-55
------------------------------------------------------
TOTAL                       38      5    87%
7 passed, 3 warnings in 3.86s
exit status: 0

$ grep -rnE "middleware|get_current_user|TestClient|AsyncClient|httpx" tests/
tests/conftest.py:88:    """Return a TestClient for the real app with get_db overridden by stub_db.
tests/conftest.py:91:    load it. TestClient is not entered as a context manager, so the startup
tests/conftest.py:94:    from fastapi.testclient import TestClient
tests/conftest.py:103:    yield TestClient(app)
tests/integration/test_auth_middleware.py:1:"""Integration tests for authentication edge cases in api/middleware/auth.py (#38)."""
tests/integration/test_auth_middleware.py:40:    # branch at api/middleware/auth.py:44-49, discussed on #38.
tests/integration/test_auth_middleware.py:58:    """Test a valid token passes auth, so the 401s above come from the middleware."""
tests/integration/test_auth_middleware.py:83:    """Test a database failure during the user lookup gets 500 from the middleware."""
Binary file tests/integration/__pycache__/test_auth_middleware.cpython-311-pytest-9.1.1.pyc matches
Binary file tests/__pycache__/conftest.cpython-311-pytest-9.1.1.pyc matches
exit status: 0
```

Full unfiltered output: [`before-raw.txt`](before-raw.txt), [`after-raw.txt`](after-raw.txt).

## Run history

| # | Command | Changed before this run | Result |
|---|---|---|---|
| 1 | `--limit 3` (smoke) | First full drafts of rubric, procedure, evidence guide | 2/3 partial. `pkg-02` (gold accept) failed on `bounded-named-scope` |
| 2 | Full, `--save-run` | Loosened `bounded-named-scope`: fixing sibling sites of the same defect, regression tests, pattern audits, and "note it, don't fix it" no longer count as creep | 19/20, PASS. All categories matched. Only miss: `pkg-14` |
| — | (grading my own plan) | — | My own plan drafts graded reject, because the CI edits counted as scope creep |
| 3 | `--only pkg-02,pkg-04,pkg-06,pkg-15,pkg-20` (canaries) | Added a narrow CI carve-out: a CI/config edit only counts as part of the fix with quoted evidence the fix's own tests need it | 5/5 partial, no misses |
| 4 (saved) | Full, `--save-run` | Procedure.md: a named fallback with concrete specifics now counts as an honest contingency, not undecided wording | 19/20, PASS. All categories matched. Only miss: `pkg-14`, left as-is |

## Package analysis

I'm using **pkg-14** for this, since it's the one package my rubric still disagrees with the gold label on. The gold label is accept. My rubric rejects it.

The plan in pkg-14 says: "Files: the client attach/reattach path in `zellij-server`... exact functions to be pinned in the PR after tracing." My `bounded-named-scope` check requires a change to be pinned to named files or functions a stranger could open, not a general area, and this plan explicitly defers naming the exact location until later. That's why it fails.

The gold note itself calls this package "arguable," so I don't think my rubric is simply wrong here. I considered loosening the check to let an unnamed-but-promised-later location pass, but decided against it. The same looseness that would let pkg-14 through would also open the door for packages like `pkg-17` ("gocui? tcell? not sure") and `pkg-18` ("whichever is easier"), which are genuinely bad plans that my check is specifically built to catch. I'd rather keep one arguable miss than weaken the check against two clear ones.

## Check rationale

The check is `bounded-named-scope`. As it reads in my uploaded `rubric.md`:

"The plan's change and scope statements (In / Out / Not in scope, Files, Changes list) read against the issue body and the repro evidence. Pass only if (a) the change is pinned to named files, functions or code sites that a stranger could open, not areas like 'the parser' or 'somewhere in the input stack', and (b) every listed change is needed to fix the behavior the issue and repro describe. These count as part of the fix, not scope creep: applying the same fix to sibling sites of the same defect in the same function or file, a regression test or doc line for the changed behavior, auditing for the same bug pattern, noting (not fixing) related problems for the PR, and a minimal CI or config edit that the plan shows is strictly required for the fix's own tests to run or pass in the repo's required checks (with quoted evidence such as an error transcript or a repo comment asking for it). New CI jobs, matrices, upgrades, or CI work the fix's own tests don't need are still creep. Fail if the plan leaves the location undecided ('X or Y, whichever is easier', 'investigate first'), or if it bundles work the issue never asked for (refactors, migrations, new options or settings, redesigns, CI changes) next to the fix."

I chose this wording because this check went through more revision than any other in my rubric, and each revision was forced by a real disagreement rather than something I anticipated upfront. It started out failing `pkg-02`, a clear accept, because fixing the same bug at a sibling location read as scope creep under the original wording. After loosening it for that, it then failed my own actual plan for issue #38, because my CI fix for `ci.yml` also looked like unrelated extra work under the same rule. I added a narrow carve-out specifically for a CI edit backed by quoted evidence that the fix's own tests need it, which is exactly my situation with the `psycopg`/`asyncpg` driver mismatch. Both changes were tested with canary packages afterward to make sure the check hadn't become too lenient toward actual scope creep in the process.

## Trade-offs

Being this strict about scope buys real protection: a plan that leaves its fix location vague, or bundles in unrelated refactors and redesigns, gets correctly held, not accidentally waved through. That's clearly the right call for packages like `pkg-17` and `pkg-18`, which are genuinely unbuildable as written.

What it costs is `pkg-14`. A plan that's honest about not knowing the exact function yet, but is clearly planning to investigate before actually touching code, gets treated the same as a plan that's just being lazy about naming a location. The check can't currently tell the difference between "I haven't traced it yet, but I will before writing any code" and "I'm not going to bother figuring out where this goes." I think that's an acceptable trade-off given how much it protects against the worse failure mode, but it's a real limitation, not a free loosening I just haven't gotten around to.
    