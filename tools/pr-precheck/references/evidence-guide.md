# Evidence guide: where evidence lives in a PR package

Each family below says where its evidence lives in an eval bundle
(with `calib-01`, the delta theme-config package, as the worked
example), where the same evidence lives in a live Path Review PR (the
issue #38 auth-tests PR as the worked example), and what good evidence
looks like. The four families map one to one onto the harness's
reject categories; a package that clears all four is a clear accept.

Rule for every family: proof is a line in the diff, real printed
output, or a recorded result that names exactly what was observed. A
sentence saying something passed or works, without naming what was
seen, is a claim, not evidence.

## Plan fidelity (harness category: silent-drift)

Rubric checks: `diff-traces-to-plan`, `description-matches-diff`.

**Where it lives, eval bundle.** The plan's boundary is in the
"Plan context" block: its "Plan:", "Not in scope:" and "Files:"
sentences, plus any note recording a deferral or deviation. What
actually changed is the "Candidate PR" block's "Diff" (each
`--- a/` / `+++ b/` file header and each `@@` hunk) and "Commits"
list. Fidelity claims are in the "Description".

- calib-01: the plan says "Files: `themes.gitconfig`", "Remove the
  first, shadowed `file-decoration-style` line", and "Not in scope:
  any other theme". The diff touches only `themes.gitconfig`, with one
  hunk removing `file-decoration-style = "#434C5E" ul` under
  `[delta "gruvmax-fang"]`. The description claims "Removes the
  shadowed first `file-decoration-style` line", which that hunk shows.

**Where it lives, live (issue #38).** The plan is `plan.md` in the
fork's top folder: "Scope" (In scope / Not in scope), "Files to touch"
(numbered, with the CI edits named), "Approach", and "Deviations".
The diff is `git diff main...HEAD` on `fix/38-auth-integration-tests`,
with `git diff main...HEAD --stat` for the file list and
`git log main..HEAD --oneline` for the commits. Fidelity claims are in
`pr_draft.md`'s Summary, Changes and Notes for Reviewers sections.

- For #38: "Files to touch" names `tests/conftest.py`,
  `tests/integration/test_auth_middleware.py` and two edits to
  `.github/workflows/ci.yml`; "Not in scope" names
  `api/middleware/auth.py`, `core/security.py`, `core/database.py` and
  `api/main.py`. The `--stat` file list should contain exactly the
  three named files and none of the four out-of-scope ones.

**What good looks like.** Every changed file and every functional hunk
traces to a plan line or a deviation note (a loose match is fine: a
planned "regression test" covers the new test file). Every change the
plan commits to is in the diff, or a note defers it. Every claim the
description makes about the change is visible in the diff. Silent
drift in either direction fails: a hunk no plan line covers (a new
option, a rename pass, a rewrite), or a planned change missing with no
note, or a description claiming "exactly as planned" or "now
documents X" over a diff that says otherwise.

## Test evidence (harness category: not-tested)

Rubric check: `decisive-test-evidence`.

**Where it lives, eval bundle.** The yardstick is the plan context's
"Repro evidence" (the before result, with the repro's own input) and
its "Test plan:" sentence (the items that will prove the fix). The
proof is the "Candidate PR" block's "Test evidence" section: commands
in code blocks and the output printed under them.

- calib-01: the test plan names two items: "re-run the configparser
  repro, expect a clean parse" and a `delta --show-syntax-themes` smoke
  check. The evidence shows the repro command
  `python3 -c "import configparser; ..."` with its before
  (`DuplicateOptionError ... already exists`) and its after
  (`parsed OK`). That decides the fix-proving item. The smoke check is
  stated in a sentence ("renders gruvmax-fang unchanged"): a secondary
  line, which never decides the check on its own.

**Where it lives, live (issue #38).** The yardstick is `plan.md`'s
"Test plan" table (three repro steps, each with "Today" and "Expected
on branch") and its two extra checks (the deliberate `auth.py:36`
break, and the unit suite count). The before results are also quoted
in `plan.md`'s "Diagnosis". The proof is `test_evidence.md` in the
fork's top folder (raw output of the repro steps on `main` and on the
branch, and of the repo's four required checks), and whatever of it
`pr_draft.md`'s Testing section pastes. A reviewer sees only the PR,
so evidence that stays in `test_evidence.md` and never reaches the
description does not count as shown.

- For #38: the decisive after-outputs are `collected 7 items` /
  `7 passed` for the integration run (before: `collected 0 items`,
  `no tests ran`), a coverage row
  `api\middleware\auth.py  38  5  87%  41, 45, 53-55` (before:
  `Module api.middleware.auth was never imported`), and grep matches
  with exit 0 (before: exit 1).

**What good looks like.** For every item the test plan says will prove
the fix: the command, named test or exact user action, a specific
recorded result after the change, and an observable difference from
the before, run on the repro's own input through the changed path. A
specific recorded result is printed output, or a before/after
transcript note naming exactly what was seen (the exact error text
that is gone, the exact message or value that appears, a named test
and its outcome); an interactive fix with no command to print is
evidenced this way. Not good: "tests pass", "works on my machine",
"colors work now" or "verified locally", naming no observed result; a run of a
control case or an unchanged path; or a test plan naming two
fix-proving items with evidence for only one and no sentence saying
the second was not verified.

## Diff quality (harness category: unreviewable)

Rubric checks: `clean-diff`, `description-explains-change`.

**Where it lives, eval bundle.** The change itself is the "Diff"
section (every `+` and `-` line, hunk by hunk) and the "Commits" list.
What a reviewer is told about it is the "Title" and "Description".

- calib-01: the diff is one hunk with one removed line and unchanged
  context; nothing else rides along. The one commit is
  `themes: remove shadowed file-decoration-style from gruvmax-fang`.
  The description says what changes (the shadowed line is removed),
  why (#2211, strict parsers error on the duplicate), and that
  rendering is unchanged.

**Where it lives, live (issue #38).** The full `git diff main...HEAD`
output, read hunk by hunk, and the three commits
(`9868159`, `3d304e0`, `e56a3d4`). What a reviewer is told is
`pr_draft.md`: its first line (the title) and its Summary, Changes and
Notes for Reviewers sections.

- For #38: every hunk should be a fixture, a test, or one of the two
  CI edits. Black normalized line endings when it ran, so check that
  the diff has no whitespace-only or line-ending-only hunks
  (`git diff main...HEAD --stat` showing insertions only on the test
  files is the quick read). The plan's Deviations (the third test that
  failed on the deliberate break, the unchanged unit coverage command,
  running checks directly because `make` is not installed, the CI
  edit split into its own commit) must be restated in the description.

**What good looks like.** Every hunk is part of the change: the fix,
its tests, and docs or config the plan names. No debug prints, no
commented-out code or earlier attempts, no dead functions, no TODOs
added by this PR, no formatting, whitespace or import churn on lines
the change does not otherwise touch. The description lets a reviewer
who has not read the plan say what behavior changes, why, and what
was left out on purpose; terse is fine.

## Standards and comms (harness category: standards-wall)

Rubric check: `template-and-policy-met`.

**Where it lives, eval bundle.** The repo's asks are in the "Repo
facts" block: its "pull requests:" line (template sections, checklist
items, issue-link form, changelog asks) and its "contribution policy"
line (including any AI-use rule). Whether the PR meets them is in the
"Description" (sections, checklist, disclosure) and the "Diff" (for a
required changelog file). Explicit maintainer direction is in "Thread
highlights".

- calib-01: "no PR template; the contribution docs ask that PRs stay
  small and reference the issue" and "no stated AI policy". The PR is
  one line and the description references `#2211`, so every stated ask
  is met and nothing else is required.

**Where it lives, live (issue #38).** The asks are in the fork's
`.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`, plus the
house rules in this skill's `scope.md`. Whether the PR meets them is in
`pr_draft.md`, the branch name, and the commit messages.

- Path Review's template sections: Summary, Issue (`Closes #`),
  Changes, Testing (a checklist: CI green on all five jobs, the four
  `make` checks, new tests cover the changes, xfail marker removed for
  a seeded bug), Screenshots / Demo, Notes for Reviewers.
- `docs/CONTRIBUTING.md` asks: fill the template completely; all five
  CI jobs green; Conventional Commits (`<type>(<scope>): ...`); the
  `<type>/<issue>-<slug>` branch format; remove the `xfail` marker when
  fixing a seeded bug; no bulk lint fixes. It states no AI-use policy.
- `scope.md` house rules: PR from the student's fork branch
  `fix/<issue-number>-<slug>`, one PR per plan, every template section
  gets real content.

**What good looks like.** Every required section has real content, not
the template's placeholder comment. Every checklist box is either
ticked with evidence behind it or left unticked with a reason (for #38,
"CI is green" stays unticked until CI has actually run on the PR).
The issue link uses the stated form (`Closes #38`). When the stated
policy requires AI disclosure, the description's disclosure gives what
that policy asks for (the tool and extent when it names them). If the repo states no template, no checklist and no AI
rule, there is nothing to meet; conventions the repo never states are
not requirements.
