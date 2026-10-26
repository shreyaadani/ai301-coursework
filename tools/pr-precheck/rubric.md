# Rubric: is this pull request ready to submit?

Proof is a line in the diff, real output from a command, or a recorded
result that names exactly what was observed. A sentence saying
something passed or works, without naming what was seen, is not
proof. Every check below reads the PR
itself against the plan, the issue, and the repo's stated standards.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diff-traces-to-plan | The diff's changed files and hunks, read against the plan's Files / Changes / Scope lines and its deviation notes (live: `plan.md` including Deviations; eval: the plan-context block). | Pass only if both directions hold. (a) Every file and every functional hunk in the diff traces to something the plan names or a deviation note records. A loose match counts: "a changelog entry" covers the changelog file, "a regression test" covers the new test file, the same fix at a sibling site of the same defect counts as the planned fix. (b) Every change the plan commits to appears in the diff, or is explicitly deferred in a plan note or deviation note. Fail if the diff adds a file, function rewrite, rename pass, config option, flag, or behavior the plan does not name and no note records; or if a planned change is missing with no note saying so. Debris (debug lines, commented-out code, formatting churn) is graded by clean-diff, not here. | required |
| description-matches-diff | Every claim the PR description makes about what changed, read against the diff. This includes fidelity claims ("implements the plan exactly", "no changes beyond the plan", "no functional changes outside X") and delivery claims ("the docs now document Y", "adds a test for Z"). | Pass only if every such claim is true of the diff: each thing the description says was changed or added is in the diff, and every fidelity claim holds when the diff is read against the plan. Fail if the description claims something the diff does not contain, or claims fidelity while the diff changes a file or behavior the plan does not name. A description that says less than the diff is graded by diff-traces-to-plan; a description that says more, or vouches for the diff falsely, fails here. | required |
| decisive-test-evidence | The test evidence, read against the plan's test plan and the repro evidence it built on. List every item the test plan says will prove the fix (each repro, each failure mode, each named before/after) before reading the evidence. A fix-proving item is one whose before result shows the bug: the failing case the issue or repro reports. An item that checks existing behavior still works (a control case, "X still works", "still renders", a full-suite run) is a non-regression item, not a fix-proving one. | Pass only if every fix-proving item on that list has, in the evidence, the actual command, named test, or exact user action AND a specific recorded result after the change, and that result differs observably from the before (shown in the evidence or in the repro evidence): an error that is gone, a value that changed, a test that now passes against the issue's input. A specific recorded result is either printed output, or a before/after transcript note that names exactly what was observed: the exact error text, message, value or listing, or a named test and its outcome. The run must exercise the changed path with the repro's own input, not a control case or a path the fix never touched. Fail if the only evidence for a fix-proving item is a non-specific claim that names no observed result ("tested locally", "works now", "colors work", "tests pass", "verified on my machine"); if the evidence runs only a control or unchanged path; or if the test plan names more fix-proving items than the evidence shows and the missing one is not disclosed (partial evidence presented as complete). A missing item passes only when the description says plainly that it was not verified and why. Secondary smoke or non-regression lines may be stated briefly; they never decide this check on their own. | required |
| clean-diff | Every hunk in the diff, read for what it does to the change. | Pass only if every hunk is part of the change the PR delivers: the fix, its tests, and docs or config the plan names. Fail if any hunk carries debris a reviewer must read past: debug prints or logging left in, commented-out code or earlier attempts, dead or unused functions, leftover TODO/FIXME markers added by this PR, or formatting, whitespace, re-indent, or import-reordering churn on lines the change does not otherwise touch. Formatting the repo's formatter applies to lines the PR already changes is not churn. | required |
| description-explains-change | The PR title and description, read as the only thing a reviewer sees before the diff. | Pass only if a reviewer who has not read the plan can tell from the description what behavior changes, why (the issue it addresses), and what was left out on purpose: every deferral or deviation the plan records is restated in the description. Terse is fine; one accurate sentence per point is enough. Fail if the description does not say what the change does, or if the plan records a deferral, shortfall, or deviation that the description never mentions. | required |
| template-and-policy-met | The repo's stated PR asks and contribution policy (live: `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`; eval: the repo-facts block's "pull requests" and "contribution policy" lines), read against the description and the diff. | First list every requirement those sources place on a PR: required template sections, checklist items, issue-link forms ("closes #N"), changelog or whatsnew entries, and any AI-use disclosure rule. Treat every package as AI-assisted work. Pass only if each listed requirement is met: each required section has real content, each checklist item is satisfied or left unticked with a reason, a required changelog file is in the diff, and, when the policy requires disclosing AI use, the description contains a disclosure that gives what that policy asks for (for example, the tool and the extent of help when the policy names those; an own-words statement when it asks only for that). A box ticked for something the evidence does not show counts as unmet. Fail if any listed requirement is missing or contradicted. If the repo states no template, no checklist, and no AI rule, this check passes; do not invent conventions the repo does not state. | required |

## Failure-mode coverage

| Reject failure mode | Checks that catch it |
|---|---|
| Silent drift: the diff does more or less than the plan, or the description claims fidelity the diff contradicts | diff-traces-to-plan, description-matches-diff |
| Not tested: missing, prose-only, wrong-path, or partial evidence | decisive-test-evidence |
| Unreviewable: debris buries the change, or the description does not let a reviewer understand it | clean-diff, description-explains-change |
| Standards wall: the repo's template, checklist, or AI-disclosure asks are unmet | template-and-policy-met |

## Verdict rule

- **accept** only if every `required` check grades `pass`.
- **reject** if any `required` check grades `fail` or `unclear`.
- `unclear` means the package does not contain enough to decide the
  check either way: for example, there is no test evidence at all, or
  the plan names no files. It counts exactly like `fail`, because a PR
  you cannot verify from the package is not ready to submit. Its
  evidence line must say what was missing.
- An honestly disclosed shortfall is not a fail. If the plan's notes or
  deviations record that something was deferred or not verified, and
  the description restates it with a reason, the checks above treat it
  as planned scope, not as drift or missing evidence. "Less than
  everything" is not "hold"; "less than claimed" is.
- `preferred` checks (none yet) are reported but never change the
  verdict.
