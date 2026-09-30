Following up on my claim above — I set the project up locally and confirmed the gap, so here is exactly where the coverage stands today.

**Environment:** `main` at 2f4e82f, Python 3.11.9, pytest 9.1.1, Windows 11. Installed with `python -m venv .venv` then `pip install -e ".[dev]"`. I did not run the database migrate/seed or `npm install` steps from `make setup`; neither of the runs below needs them.

**Steps**

1. Ran the integration suite exactly as the Makefile defines it (`make test-integration`):

```
$ python -m pytest tests/integration -v -m integration
collecting ... collected 0 items

============================ no tests ran in 1.51s ============================
```

2. Ran the full unit suite with coverage pointed at the middleware:

```
$ python -m pytest tests/unit -m unit --cov=api.middleware.auth --cov-report=term-missing -q
375 passed, 53 xfailed, 3 warnings in 16.19s

CoverageWarning: Module api.middleware.auth was never imported. (module-not-imported)
CoverageWarning: No data was collected. (no-data-collected)
WARNING: Failed to generate report: No data to report.
```

3. Checked whether anything in the test tree reaches the middleware or makes an HTTP request at all:

```
$ grep -rnE "middleware|get_current_user|TestClient|AsyncClient|httpx" tests/
$ echo $?
1
```

**Expected:** the four edge cases the issue lists — expired tokens, malformed tokens, missing `Authorization` headers, and tokens signed with the wrong secret — exercised against a protected endpoint somewhere in the suite.

**Actual:** `tests/integration/` contains only `__init__.py`, and coverage cannot report on `api/middleware/auth.py` at all because 375 passing tests never import it.

A few details I turned up that seem worth recording:

- The search in step 3 returns nothing (exit status 1), so no test currently reaches the middleware or makes an HTTP request against a protected endpoint.
- `tests/unit/test_security.py` is thorough about `core/security.py` — it covers `decode_access_token` with invalid, malformed, empty and tampered tokens — but it calls those functions directly. It never goes through `get_current_user`, so the 401 responses and the `WWW-Authenticate: Bearer` header the middleware is responsible for are untested.
- `tests/conftest.py` currently provides two text fixtures and nothing else: no app, no client, no database session. That lines up with the issue naming `conftest.py` as a file to touch — the fixtures have to exist before the integration tests can.
- Two middleware paths look untested and worth covering alongside the four in the issue: the expiry comparison at `api/middleware/auth.py:44`, which is separate from whatever `decode_access_token` does about `exp`, and the 500 the middleware raises when the user lookup fails.

**Next for me:** reading how `get_db` is provided in `core/database.py` so I can add an app + client fixture with an overridden database dependency to `conftest.py`. I'll work through the four cases from there and report back.
