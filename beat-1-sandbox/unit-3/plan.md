# Plan: issue #38, integration tests for authentication edge cases

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38
My repro: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902682953
Repo state: `main` at `2f4e82f`. Branch: `fix/38-auth-integration-tests` on my fork.

## Diagnosis

This is a coverage gap, not a runtime bug. Nothing in the test suite
ever sends a request through `get_current_user` in
`api/middleware/auth.py`, so a change that let a bad token through
would not fail any test. My repro shows this in three steps:

Step 1, the integration suite as `make test-integration` runs it:

```
$ python -m pytest tests/integration -v -m integration
collecting ... collected 0 items

============================ no tests ran in 1.51s ============================
```

Step 2, the whole unit suite with coverage on the middleware:

```
$ python -m pytest tests/unit -m unit --cov=api.middleware.auth --cov-report=term-missing -q
375 passed, 53 xfailed, 3 warnings in 16.19s

CoverageWarning: Module api.middleware.auth was never imported. (module-not-imported)
CoverageWarning: No data was collected. (no-data-collected)
WARNING: Failed to generate report: No data to report.
```

Step 3, anything in `tests/` that reaches the middleware or makes an HTTP request:

```
$ grep -rnE "middleware|get_current_user|TestClient|AsyncClient|httpx" tests/
$ echo $?
1
```

Why the gap exists, from the same report: "`tests/conftest.py`
currently provides two text fixtures and nothing else: no app, no
client, no database session". With no app or client fixture, no test
can call a protected endpoint. And `tests/unit/test_security.py`
"calls those functions directly. It never goes through
`get_current_user`, so the 401 responses and the `WWW-Authenticate:
Bearer` header the middleware is responsible for are untested."

Each repro step fits this cause. Step 1: no integration tests exist.
Step 2: the unit tests never import the module. Step 3: no test
references it or makes a request.

## Scope

**In scope:** tests that send real HTTP requests through
`get_current_user` on a real protected route, plus the fixtures they
need, plus the two CI lines that would otherwise stop those tests from
running or count against them.

**Not in scope:**
- No change to `api/middleware/auth.py`, `core/security.py`,
  `core/database.py` or `api/main.py`. Classmates on this thread report
  that all four cases already return 401 (rose413's and
  Christian101GTZ's repros), so these tests lock in existing behavior.
  They don't fix anything.
- The `"Token has expired"` branch at `api/middleware/auth.py:44-49`.
  I flagged that line in my repro as "separate from whatever
  `decode_access_token` does about `exp`". Both classmates' plans say
  it can't be reached, because `decode_access_token` returns `None` for
  an expired token first. The expired-token test asserts 401 +
  `WWW-Authenticate: Bearer` only, not the detail text, so it doesn't
  pin either message. Whether to delete or fix that branch is a
  separate issue.
- No real database. The tests override `get_db` with a stub session.
- No other routes, no other middleware, no lint or typing cleanup
  outside the new files.

## Files to touch

1. `tests/conftest.py` (named in the issue). Add the fixtures, and
   import `api.main` *inside* the fixture body, so the unit tests that
   share this conftest never import the app:
   - `stub_db`: an object with an async `execute()` that returns a
     result whose `scalar_one_or_none()` gives a value the test sets,
     or raises if the test sets an error. It also records how many
     times it was called.
   - `client`: sets `app.dependency_overrides[get_db]` to yield
     `stub_db`, returns `TestClient(app)` **without** a `with` block,
     and clears the overrides on teardown. Leaving out `with` follows
     Christian101GTZ's finding in this thread: the startup hook re-raises
     when Postgres is down (`api/main.py:92`), and `TestClient` only
     runs startup inside `with`.
   - `make_token(sub, exp_delta, secret)`: `jose.jwt.encode` with
     `settings.jwt_algorithm` and, by default, `settings.secret_key`.
2. `tests/integration/test_auth_middleware.py` (new, the filename the
   issue gives). It's marked `pytest.mark.integration`, and every test
   calls `GET /profiles/00000000-0000-0000-0000-000000000000`:

   | Test | Request | Expected |
   |---|---|---|
   | `test_missing_authorization_header` | no header | 401, `WWW-Authenticate: Bearer`, `stub_db` not called |
   | `test_malformed_token` | `Bearer not.a.jwt` | 401, `Bearer`, `stub_db` not called |
   | `test_expired_token` | real secret, `exp` 5 min in the past | 401, `Bearer`, `stub_db` not called |
   | `test_wrong_signing_secret` | valid `exp`, signed with `"not-the-app-secret"` | 401, `Bearer`, `stub_db` not called |
   | `test_valid_token_reaches_route` (control) | real secret, valid `exp`, stub returns a user, then no profile | 404 `Profile not found`, which proves the 401s above come from auth and not from a broken fixture |
   | `test_unknown_user` | valid token, stub returns `None` | 401 (`auth.py:68-69`) |
   | `test_user_lookup_error` | valid token, stub raises | 500 `Database error` (`auth.py:61-66`) |

   The first four are the issue's four cases. The control makes the
   test plan falsifiable. The last two are the "two middleware paths
   [that] look untested" from my repro. They cover the same function
   with the same fixture and no new code under test.
3. `.github/workflows/ci.yml`, two edits to the `test-integration`
   job only:
   - Delete the exit-code-5 shim (lines 80-94). Its own comment says:
     "Delete this shim once the directory holds at least one test."
     After this PR the directory holds tests.
   - Change `DATABASE_URL` (line 96) from `postgresql://...` to
     `postgresql+asyncpg://...`. `core/database.py` builds an async
     engine at import time, and with the plain `postgresql://` URL that
     fails. I checked this locally:

     ```
     $ DATABASE_URL=postgresql://pathreview:pathreview@localhost:5432/pathreview_test python -c "import api.main"
     ...
     ModuleNotFoundError: No module named 'psycopg'
     ```

     The integration job has never imported the app before, because
     the directory was empty. So the first PR that adds a test there
     would go red for a reason unrelated to the tests, and
     CONTRIBUTING requires all five CI jobs to be green.

## Approach (order of work)

1. Add the conftest fixtures, then write `test_missing_authorization_header`
   only, and run it with Docker stopped. This tests the
   no-`with`-block assumption on my own machine before anything else
   depends on it.
2. Add the other six tests.
3. Make the two `ci.yml` edits.
4. Run `make lint`, `make typecheck` and `make test-unit`, then the
   test plan below.

## Test plan

Re-run my three repro commands on the branch. Each one has a concrete
output today and a different expected output after:

| Repro step | Today (`main` @ `2f4e82f`) | Expected on branch |
|---|---|---|
| 1. `python -m pytest tests/integration -v -m integration` | `collected 0 items`, `no tests ran` | `collected 7 items`, `7 passed`, with Docker stopped |
| 2. same as step 1, plus `--cov=api.middleware.auth --cov-report=term-missing` (step 2 pointed at the suite that now has the tests) | `Module api.middleware.auth was never imported` | a coverage row for `api/middleware/auth.py` whose Missing column does not list 36 (bad-token raise), 59-60 (user lookup), 61-66 (500 path) or 69 (unknown user) |
| 3. `grep -rnE "middleware\|get_current_user\|TestClient\|AsyncClient\|httpx" tests/` | no output, exit 1 | matches in `tests/conftest.py` and `tests/integration/test_auth_middleware.py`, exit 0 |

Two more checks:
- **The tests can fail:** temporarily change `auth.py:36` from
  `raise credentials_exception` to `return None`. The malformed and
  wrong-secret tests must fail (expected 401, got something else).
  Then revert, and confirm with `git diff` that `auth.py` is
  unchanged.
- **No regressions:** `make test-unit` still shows `375 passed, 53
  xfailed`, the same as step 2 of my repro.

## Risks and unknowns

- **The no-`with` `TestClient` trick is someone else's result.**
  Christian101GTZ reported it, and I haven't run it myself yet, which
  is why it's step 1 of the approach. If it doesn't hold, I'll build a
  test-only `FastAPI()` that mounts `api.routes.profiles.router` and
  record that under Deviations.
- **CI isn't something I can run locally.** The asyncpg URL fix is
  based on the local import failure above. I'll confirm it on the PR's
  `test-integration` job, which needs a maintainer to release CI for
  first-time fork PRs.
- **`datetime.utcfromtimestamp` / `utcnow`** at `auth.py:44` are
  deprecated in Python 3.12 and may warn. That's not mine to change
  here; I'll mention it in the PR if it shows up.
- **The two extra tests (unknown user, lookup error) go beyond the
  issue's list of four.** If a reviewer wants them in a separate PR,
  they come out cleanly, since they're independent test functions.

## Deviations
Nothing about the approach changed from the plan. Same files, same fixtures, and the no-with TestClient trick worked on the first try, so I never needed the fallback approach I’d mentioned as a backup.

A few small things came out slightly different than I expected. Breaking auth.py:36 on purpose failed three tests instead of the two I predicted, because I hadn’t realized decode_access_token returns None for expired tokens too, not just malformed ones. My repro’s original coverage command, pointed at the unit test suite, still shows the middleware as never imported after the fix, since the new tests live in tests/integration/ instead. Pointing that same coverage flag at the integration tests instead is what actually shows the fix working: the middleware goes from unimported to 87% covered. I also ended up running ruff, black, mypy, and pytest directly instead of through make, since make isn’t installed on my Windows machine. And I split the CI config change into its own separate commit, since CONTRIBUTING’s commit-type rules treat it differently from a test change.

None of this required a re-grade of the plan or a new comment on the issue, since the actual work matched what I said I’d do.
