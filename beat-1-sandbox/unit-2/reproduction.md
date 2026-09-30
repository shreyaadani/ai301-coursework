# Unit 2 — Claim and Reproduce: issue #38

## Claim comment

Link: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902249218

> Hi! I'd like to investigate this one as a first contribution. My first step is getting the project running locally and checking which of the four edge cases the issue lists, expired tokens, malformed tokens, missing Authorization headers, and tokens signed with the wrong secret, the current suite already covers against `api/middleware/auth.py`, so I can show exactly where the gap is. I'll post what I find back here before I open anything.

**Reflection:** My first draft of this comment said I would add the integration tests the issue asks for. My own rubric rejected that draft, and it was right to. On this issue, adding those tests is the entire deliverable, so promising to add them commits me to a finished fix before I have even opened the file. I rewrote the comment to promise only investigation: getting the project running and checking what the existing suite actually covers. That version turned out to be more specific than the first one, not less, because dropping the promise left room to name the actual file and the actual first step. This matched a rule I had already written into my own voice guide before I made the mistake myself, which was a useful reminder that writing a rule down does not automatically stop you from breaking it under time pressure.

## Repro comment

Link: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902682953

> Following up on my claim above, I set the project up locally and confirmed the gap, so here is exactly where the coverage stands today.
>
> **Environment:** `main` at 2f4e82f, Python 3.11.9, pytest 9.1.1, Windows 11. Installed with `python -m venv .venv` then `pip install -e ".[dev]"`. I did not run the database migrate and seed steps or `npm install` from `make setup`. Neither of the runs below needs them.
>
> **Steps**
>
> 1. Ran the integration suite exactly as the Makefile defines it (`make test-integration`):
>
> ```
> $ python -m pytest tests/integration -v -m integration
> collecting ... collected 0 items
>
> ============================ no tests ran in 1.51s ============================
> ```
>
> 2. Ran the full unit suite with coverage pointed at the middleware:
>
> ```
> $ python -m pytest tests/unit -m unit --cov=api.middleware.auth --cov-report=term-missing -q
> 375 passed, 53 xfailed, 3 warnings in 16.19s
>
> CoverageWarning: Module api.middleware.auth was never imported. (module-not-imported)
> CoverageWarning: No data was collected. (no-data-collected)
> WARNING: Failed to generate report: No data to report.
> ```
>
> 3. Checked whether anything in the test tree reaches the middleware or makes an HTTP request at all:
>
> ```
> $ grep -rnE "middleware|get_current_user|TestClient|AsyncClient|httpx" tests/
> $ echo $?
> 1
> ```
>
> **Expected:** the four edge cases the issue lists, expired tokens, malformed tokens, missing `Authorization` headers, and tokens signed with the wrong secret, exercised against a protected endpoint somewhere in the suite.
>
> **Actual:** `tests/integration/` contains only `__init__.py`, and coverage cannot report on `api/middleware/auth.py` at all because 375 passing tests never import it.
>
> A few details I turned up that seem worth recording:
>
> - The search in step 3 returns nothing, with exit status 1, so no test currently reaches the middleware or makes an HTTP request against a protected endpoint.
> - `tests/unit/test_security.py` is thorough about `core/security.py`. It covers `decode_access_token` with invalid, malformed, empty, and tampered tokens, but it calls those functions directly. It never goes through `get_current_user`, so the 401 responses and the `WWW-Authenticate: Bearer` header the middleware is responsible for are untested.
> - `tests/conftest.py` currently provides two text fixtures and nothing else, no app, no client, no database session. That lines up with the issue naming `conftest.py` as a file to touch, since the fixtures have to exist before the integration tests can.
> - Two middleware paths look untested and worth covering alongside the four in the issue: the expiry comparison at `api/middleware/auth.py:44`, which is separate from whatever `decode_access_token` does about `exp`, and the 500 the middleware raises when the user lookup fails.
>
> **Next for me:** reading how `get_db` is provided in `core/database.py` so I can add an app and client fixture with an overridden database dependency to `conftest.py`. I'll work through the four cases from there and report back.

**Reflection:** This issue is labeled enhancement, not bug, so there was no crash for me to trigger. The thing I actually needed to prove was an absence, that the middleware has no test coverage, and a simple statement like "yes, there are no tests" would have just repeated the issue without adding evidence. I used two test runs together to make the point solidly: the empty integration suite shows there are no integration tests, and the coverage run on the unit suite shows that no unit test imports the middleware either. Either one alone leaves room for doubt. Together they close it off. I also had to be careful about one specific line. My first draft of the report described a search of the test files in prose, without showing the actual command or its output. My own rubric let it pass, but it was a real gap against my own voice guide, which says I should show the output rather than just my confidence in it. When I fixed this, I made sure to actually run the grep command fresh and paste its real output, rather than write a plausible-looking command I had not actually executed. Writing down a command because it would probably produce a certain result is not the same as showing evidence, and it would have been exactly the kind of unverified claim my rubric's no-evidence checks are built to catch.

## Run history

| Run | Scope | Result | What changed after |
|---|---|---|---|
| Smoke | `--limit 3` | 3 out of 3 agree | Nothing. This was just a check that the setup worked. |
| Full v0 | 20 packages | 19 out of 20 agree, bar passed, all five categories matched, including disclosure at 1 out of 1 | Loosened the `steps-rerunnable` check |
| Canary | `--only pkg-05,pkg-18,pkg-06,pkg-20` | 4 out of 4 agree | Nothing. This confirmed the loosening was safe. |
| Full v1 (saved) | 20 packages | 20 out of 20 agree, bar passed, all five categories matched | Nothing. This is the run I am submitting. |

## Package analysis

I'm using pkg-05 for this section. The gold label is accept. My rubric's first full run rejected it instead, which was the one disagreement in that run.

Eight of the nine checks passed. The one that failed was `steps-rerunnable`, because the package described its environment file in prose, as a dependencies list plus a category section, rather than pasting it inline or linking to it. I had written that check with a different package in mind, one where the reproduction lived in a private repository nobody else could access, which is a genuinely unrunnable case. My pass condition for the check conflated two different problems: material that is not pasted word for word, and material that cannot actually be obtained or reconstructed at all. pkg-05's environment was described precisely enough that anyone reading it could rebuild an equivalent file in under a minute, so it should have passed. I rewrote the check's pass condition so that it accepts input that is shown inline, is publicly available, or is described specifically enough to reconstruct, and only fails when something is genuinely unobtainable or when a step skips the actual trigger for the bug. After that change, I reran the four packages most likely to be affected by a looser check, including the private-repository package the original wording was meant to catch, and all four still graded the way they should. The confirming full run afterward agreed on all twenty.

## Check rationale

The check is `steps-rerunnable`. As it is written in `rubric.md`, its pass condition reads:

"A reader outside the author's environment could assemble the inputs and run the steps: the starting state is given, the trigger is an exact command or action, and every input is shown inline, publicly obtainable, OR characterized specifically enough to rebuild an equivalent (an env.yml with a valid dependencies list plus an unrecognized category: section is rebuildable; verbatim quoting is not required). Fails when the reproduction rests on material nobody else can supply or reconstruct, an internal repo, a config whose relevant contents are never characterized, or when the steps skip the trigger itself."

I chose this wording because the check exists to catch reproductions that nobody else could actually rerun, most often because the setup lives somewhere private or undocumented. The earlier version of this check was stricter than that purpose required. It rejected a package for describing its setup in its own words instead of quoting a file verbatim, even though the description was specific enough for anyone to follow. The current wording keeps the check aimed at real unrunnability instead of at a particular writing style.

## Trade-offs

The biggest trade-off in this rubric is between strictness and flexibility in how evidence gets described. A stricter check like the original `steps-rerunnable` is easier to grade consistently, since either something is quoted verbatim or it is not, but it can punish a report that is honest and specific just because it phrases things in prose instead of pasting a file. A looser check is fairer to reports like that, but it depends more on judgment about what counts as "specific enough," which is harder to apply the same way every time.

The other trade-off I noticed while writing the repro report itself was between speed and honesty in how I gathered evidence. It would have been faster to write down a command I assumed would produce a certain result and move on. Instead, I ran the actual command and pasted its real output, even for a small supporting detail. That cost a little extra time, but it meant every claim in the report was something I had actually verified rather than something I expected to be true. Given that the whole point of this unit is proving that a stranger could rerun what I did and get the same result, I think that extra step was worth it every time.
