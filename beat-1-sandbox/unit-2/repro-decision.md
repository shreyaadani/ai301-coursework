# Reproduction decisions — issue #38

Companion to `decision-log.md` (which covers the skill and the eval).
This file records how the reproduction itself was produced: what I
ran, what I chose not to run, and why. My own rubric grades the
package on evidence, so the reasoning behind each choice is written
down here rather than crammed into the comment.

Issue: codepath/pathreview-ai301-fa26-s3#38 — integration tests for
authentication edge cases (expired tokens, malformed tokens, missing
`Authorization` header, tokens signed with the wrong secret).
Repo state: `main` at 2f4e82f.

## What "reproduce" means on an enhancement issue

This issue is labelled `enhancement`, not `bug`. There is no crash to
trigger, so the thing to demonstrate is the *absence* the issue
asserts: that the authentication middleware has no test coverage. A
me-too comment ("yes, there are no tests") would assert exactly what
the issue already says and add nothing. The reproduction therefore had
to produce an artifact showing the gap rather than describing it.

The strongest available artifact turned out to be a coverage run:
coverage reporting `Module api.middleware.auth was never imported` is
the machine saying the gap exists, and it cannot be argued with the way
a prose claim can.

## Setup: what I ran and what I skipped

`make setup` does six things: venv, `pip install -e ".[dev]"`,
pre-commit install, `alembic upgrade head`, seed the database, and
`npm install` for the frontend.

I ran the first two only. The last three need Postgres running via
docker-compose and Node installed, and neither of my two runs touches
a database or the frontend — the integration suite collects zero tests,
and the unit suite is marked "fast, no external dependencies" in
`pyproject.toml`.

**This deviation is stated in the comment.** My rubric's
`deviation-acknowledged` check fails a package for a silent deviation
from the documented setup; an acknowledged one passes. Saying "I did
not run the migrate/seed or npm steps; neither run below needs them"
costs one sentence and makes the environment record honest.

## The two artifacts, and why both

1. **`pytest tests/integration -v -m integration` → `collected 0
   items`.** Run exactly as `make test-integration` defines it, so the
   command is the project's own, not one I invented. Proves the
   integration suite is empty.

2. **`pytest tests/unit -m unit --cov=api.middleware.auth` → `Module
   api.middleware.auth was never imported`, alongside `375 passed, 53
   xfailed`.** This is the load-bearing one. The first artifact alone
   would be weak — someone could reasonably assume the middleware is
   covered by unit tests instead. The second closes that door: the
   entire passing unit suite never loads the module.

The pairing matters. One shows the empty directory; the other shows
nothing elsewhere compensates for it.

3. **`grep -rnE "middleware|get_current_user|TestClient|AsyncClient|
   httpx" tests/` → no output, exit status 1.** Added after the first
   live-mode grade, which passed the package but flagged that this
   finding was stated in prose without its output. Voice-guide Rule 2
   ("I show the output, not my confidence in it") is satisfied by
   showing the command and its empty result.

   The `echo $?` matters here: grep prints nothing when it finds
   nothing, so an empty block alone is ambiguous — it could mean the
   command was never run. Exit status 1 is grep's own way of saying
   "searched, matched nothing", which turns an absence into an
   artifact.

   **The command in the comment is one I actually ran.** The original
   search was done with a different tool; rather than paste a
   plausible-looking `grep` line that had never been executed, I ran
   the real command and pasted its real result. Writing a command into
   a report because it would *probably* produce that output is
   fabricated evidence, and it is the precise failure my `no-evidence`
   eval packages are built from.

## Findings that go beyond the issue text

Four things I found while looking, all included in the comment because
they are useful to whoever works on this (me or a classmate):

- **Nothing in `tests/` references the middleware.** Grepping the tree
  for `middleware`, `get_current_user`, `TestClient`, `AsyncClient`,
  `httpx` returns no matches. No test makes an HTTP request against a
  protected endpoint at all.
- **`test_security.py` is a near-miss, not coverage.** It tests
  `decode_access_token` with invalid, malformed, empty and tampered
  tokens — which *sounds* like it covers three of the issue's four
  cases — but calls the functions directly. The middleware's 401
  responses and `WWW-Authenticate: Bearer` header are never exercised.
  Worth stating explicitly, because a maintainer skimming file names
  could reasonably think that file already does the job.
- **`conftest.py` has no app, client or DB fixtures**, only two text
  fixtures. This explains why the issue names `conftest.py` as a file
  to touch: the fixtures must exist before any of these tests can be
  written. It is also the real first task.
- **Two extra untested paths**: the expiry comparison at
  `api/middleware/auth.py:44` (separate from whatever
  `decode_access_token` does with `exp`) and the 500 raised when the
  user lookup fails. Neither is in the issue's list of four.

## Things I deliberately did not do

- **Did not write any tests yet.** The claim comment promised
  investigation and a report back; writing the tests before reporting
  would make that comment retroactively dishonest, and the fixtures
  question should be settled in the thread first.
- **Did not diagnose beyond what I ran.** I noticed the middleware's
  expiry check looks redundant with the decoder's, but I did not claim
  it is a bug — I have not tested it. My rubric's
  `conclusion-within-evidence` check exists to catch exactly that kind
  of unearned diagnosis, and voice-guide Rule 2 says the same thing.
- **Did not disclose AI assistance in the comment.** The repo's
  `docs/CONTRIBUTING.md` has no AI policy and no disclosure
  requirement, so `disclosure-when-required` passes without one, and
  voice-guide Rule 5 scopes disclosure to repos that ask. A deliberate
  call rather than an oversight; if the course wants disclosure
  regardless of repo policy, one sentence in the comment covers it.

## Environment record, for reference

```
repo      codepath/pathreview-ai301-fa26-s3, main @ 2f4e82f
python    3.11.9
pytest    9.1.1
os        Windows 11 Home Single Language
install   python -m venv .venv; pip install -e ".[dev]"
skipped   alembic upgrade head, scripts/seed_db.py, npm install
```
