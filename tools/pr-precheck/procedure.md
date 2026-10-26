# Procedure: how this tool grades a PR package

Adapted from the plan-check procedure (Unit 3), rewritten for pull
requests: the plan is no longer the thing being graded. It is the fixed
yardstick the PR is graded against. Almost every step here is a
side-by-side: the diff against the plan, the evidence against the test
plan, the description against the diff.

## Read order

1. **Mode and scope.** Decide the mode. Eval mode: the input is a
   bundle file, so skip `scope.md` and `voice-guide.md` entirely, fetch
   nothing and run nothing. Live mode: read `scope.md` first. If its
   `Repo:` line still holds the placeholder, stop and report that. If
   the issue or PR is not in that repo, refuse to grade. Note the house
   rules (branch from the student's fork named `fix/<issue>-<slug>`,
   one PR per plan, every template section gets real content).
2. **Load the judgment files.** Read `rubric.md` and
   `references/evidence-guide.md`. Write down the check names in table
   order, the verdict rule, what `unclear` does, and the
   honest-shortfall rule.
3. **Read the whole package in this order before grading anything.**
   The plan comes before the diff, and the description comes last, so
   the yardstick is fixed before the PR's own wording can frame it. A
   confident description read first makes a drifting diff look like
   it matches.
   1. **Issue and thread.** Note the behavior the issue says is wrong,
      and any maintainer direction (a requested approach, files to
      touch, a request to test something).
   2. **The plan**, including its repro evidence and every deviation or
      deferral note. Write down six lists:
      - *Planned files*: every file, function or code site it names.
      - *Planned changes*: every change it commits to, including tests,
        docs, changelog and config edits.
      - *Out of scope*: everything it says it will not touch.
      - *Notes*: every deferral, shortfall or deviation, quoted.
      - *Fix-proving items*: every repro, failure mode or before/after
        its test plan says will prove the fix, with the repro's own
        input and its recorded before result. An item belongs here only
        if its before result shows the bug. Items that check existing
        behavior still works (a control, "X still works", a full-suite
        run) go on a separate *Non-regression items* list.
   3. **Repo standards.** From the repo-facts block (live: the PR
      template and `docs/CONTRIBUTING.md`), write down every
      requirement placed on a PR: required template sections,
      checklist items, the issue-link form, changelog entries, and any
      AI-use disclosure rule.
   4. **Commits and diff.** List every changed file. Within each file,
      list every hunk with a few words on what it does.
   5. **Test evidence.** Note every command or named test and the
      output printed under it. Mark any line that claims a result
      without showing output.
   6. **Title and description.** Note every claim about what changed,
      every fidelity claim ("as planned", "no other changes"), every
      claim of delivery ("adds", "now documents"), every statement
      about what was left out, and every ticked checklist box.

   Live mode gets these from: the issue page or API for 3.1;
   `plan.md` in the working copy for 3.2; `.github/PULL_REQUEST_TEMPLATE.md`
   and `docs/CONTRIBUTING.md` for 3.3; `git log main..HEAD --oneline`,
   `git diff main...HEAD --stat` and `git diff main...HEAD` for 3.4;
   the student's test-evidence file for 3.5; the student's PR draft
   (title on line one) for 3.6. Grade only committed branch content
   for 3.4.

## Evidence gathering

For each check, build exactly this record before grading it. In eval
mode, copy lines from the bundle verbatim. In live mode, use the
locations in the evidence guide.

4. **diff-traces-to-plan.** Make a hunk table: one row per hunk, with
   file, what it does, and what it traces to (a quoted plan line, a
   quoted deviation note, or "none"). Mark debris rows (debug lines,
   commented-out code, pure formatting) "debris" and leave them for
   clean-diff. Then take the *Planned changes* list and mark each one
   "in diff" (name the file and hunk), "deferred" (quote the note) or
   "missing". A loose match counts as tracing: a planned "changelog
   entry" covers the changelog file, a planned "regression test" covers
   the new test file, the same fix at a sibling site of the same defect
   covers that site.
5. **description-matches-diff.** For every claim from read-order step
   3.6, record where in the diff it is true (file and hunk), or "not in
   diff". For every fidelity claim, record whether step 4's hunk table
   has any functional row marked "none"; if so, the fidelity claim is
   false.
6. **decisive-test-evidence.** Take the *Fix-proving items* list. For
   each item record four things: the command, test name, or exact user
   action in the evidence (quote it), the after result (quote it: either
   printed output, or a before/after transcript note naming exactly what
   was observed, such as the exact error text, message, value, listing,
   or a named test's outcome), the before result (quote it from the
   evidence or the repro), and whether the run uses the repro's own
   input on the changed path (yes / no, and what it ran instead). If an
   item's only evidence is a claim that names no observed result
   ("works now", "tests pass" with no test named, "verified locally"),
   record "non-specific" with the quoted claim. If there is nothing,
   record "absent". If it is absent, look for a
   sentence in the description saying it was not verified and why, and
   quote it. Record what the evidence shows for each *Non-regression
   item* separately; those lines do not decide this check.
7. **clean-diff.** From step 4's hunk table, list every "debris" row
   and quote the offending line(s), with its type: debug output,
   commented-out code, dead or unused function, added TODO/FIXME, or
   formatting/whitespace/import churn on lines the change does not
   otherwise touch. Formatting the repo's formatter applies to lines
   the PR already changes is not debris.
8. **description-explains-change.** Quote the description's sentence
   that says what behavior changes, and the one that says why (the
   issue). Then take the plan's *Notes* list and, for each note, quote
   the description sentence that restates it, or record "not restated".
9. **template-and-policy-met.** Take the requirements list from
   read-order step 3.3. For each requirement, quote the description
   text or name the diff file that meets it, or record "missing". For
   each ticked checklist box, name the evidence that backs it, or
   record "unsupported". If the policy requires AI disclosure, quote the
   disclosure (it must give what that policy asks for, such as the tool and extent when the policy names them) or record
   "missing". Treat every package as AI-assisted. If the list is empty,
   record "no stated requirements".

## Check execution

10. Run the checks in rubric table order. Each check uses only its own
    record from steps 4-9. Re-read the package for a check only if its
    record is empty, and then only the parts that check's Evidence
    column names.
11. Apply the pass conditions in `rubric.md` exactly as written:
    - **diff-traces-to-plan:** fail if any functional hunk traces to
      "none", or any planned change is "missing".
    - **description-matches-diff:** fail if any claim is "not in diff",
      or any fidelity claim is false.
    - **decisive-test-evidence:** fail if any fix-proving item is
      "non-specific", "absent" without a quoted not-verified sentence, or
      ran something other than the repro's input on the changed path.
    - **clean-diff:** fail if the debris list has any row.
    - **description-explains-change:** fail if there is no sentence
      saying what changes, or any note is "not restated".
    - **template-and-policy-met:** fail if any requirement is "missing"
      or any ticked box is "unsupported". Pass if the record says "no
      stated requirements".
12. Apply the honest-shortfall rule before failing anything: a
    deferral or unverified item recorded in the plan's notes AND
    restated in the description is planned scope. It is not "missing",
    not drift, and not absent evidence.
13. Grade `unclear` only when the package truly lacks what the check
    needs (for example, no test evidence at all, or a plan that names
    no files). It never means "I did not look". Do not fill the gap
    with outside knowledge of the codebase or the project.
14. Write one evidence line per check: the fact or quote that decided
    it. For a fail, quote the offending diff line, claim, or missing
    item.
15. Polish never moves a grade. A terse PR that meets the conditions
    passes. A long, confident one that fails a condition fails.

## Verdict assembly

16. Apply the verdict rule from `rubric.md`: `accept` only if every
    `required` check is `pass`. Any `required` check that is `fail` or
    `unclear` gives `reject`, and `unclear` counts exactly like `fail`.
    `preferred` checks never change the verdict. There is no third
    verdict.
17. The deciding check for a reject is the first failing or unclear
    required check in rubric table order. Quote its evidence line first
    in the summary, then list any other failing checks. For an accept,
    list one line per check.
18. Live mode only: hold the PR title and description against
    `voice-guide.md`, and list each broken rule in the summary, quoting
    the rule and the line that breaks it. This does not change the
    verdict.
19. Note any point where this procedure did not tell you what to do (a
    procedure gap) in the summary. Do not invent a step.
20. Output the fenced JSON block from SKILL.md last, with `checks` in
    rubric table order, each using the rubric's check name. In live
    mode `item` is the PR URL if the PR is open, otherwise the issue
    URL; in eval mode it is the bundle id.
