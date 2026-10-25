# Procedure: how this skill grades a plan package

Adapted from the repro-check workflow (Unit 2), rewritten for plans:
the repro report is no longer the thing being graded. It is the fixed
ground truth the plan is graded against.

## Read order

1. **Mode and scope.** Decide the mode. Eval mode: the input is a
   bundle file, so skip `scope.md` and `voice-guide.md` entirely.
   Live mode: read `scope.md` first. If its `Repo:` line still holds a
   placeholder, stop and report that. If the issue URL is not in that
   repo, refuse to grade. Note the house rules (they matter for the
   comms check: a classmate's earlier plan comment does not block this
   one, but piggybacking on it fails).
2. **Load the judgment files.** Read `rubric.md` and
   `references/evidence-guide.md`. Write down the list of check names
   in table order and the verdict rule, including what `unclear` does.
3. **Read the package in this order, and read all of it before
   grading anything.** Ground truth comes first, so you form a picture
   of the bug before the plan's confident wording can shape it:
   1. Issue context (title, body, reported environment). Note what
      behavior the issue says is wrong.
   2. Repro evidence. Note every numbered step and its recorded
      result, and mark any control or comparison step (same input with
      one variable changed, a timing matrix, "works when X"). Note the
      Expected and Actual lines. These notes are the yardstick for
      diagnosis-fits-evidence and falsifiable-test-plan.
   3. Thread highlights. Note every comment by a MAINTAINER, OWNER,
      COLLABORATOR or MEMBER that gives direction, and treat the issue
      body as direction too when one of those roles opened the issue
      (for example, files it names to touch) (an isolated culprit,
      a requested approach, a request to test something, a pointer to
      an existing PR). Claims by non-maintainers are hypotheses, not
      direction and not evidence.
   4. Repo facts. Note the contribution policy's requirements on
      contributions or comments, especially any AI-use rule.
   5. The candidate plan. Note the cause, scope, files, changes and
      test plan as written.
   6. The candidate plan comment. Note what it claims and what it
      commits to.

## Evidence gathering

For each rubric check, gather exactly this before grading. In eval
mode, copy the relevant lines from the bundle verbatim. In live mode,
use the locations in the evidence guide: `gh issue view <url>
--comments` or the issue page for the thread, the student's own posted
repro comment for repro evidence, the repo's CONTRIBUTING / README /
AI policy files for conventions, and the student's `plan.md` and
comment draft for the candidate side. Grade only what the drafts
actually contain and quote, not other files in the working directory.

4. **diagnosis-fits-evidence:** Record the plan's cause in one
   sentence. Then make a table: one row per repro step, with the
   recorded result and what the plan's cause predicts for that step
   (same / different / not addressed). Record any step whose result the
   cause cannot produce. Record whether the cause comes from a thread
   claim, and if so, whether any repro step tests it.
5. **bounded-named-scope:** List every file, function or code site the
   plan names. List every distinct change the plan commits to,
   including "also", "while we're at it" and follow-up items placed
   inside the plan. For each change, note which part of the issue or
   repro needs it, or "none". The same fix at a sibling site of the
   same defect, a regression test, an audit for the same pattern, and
   "note it in the PR rather than fix it" all count as justified by
   the issue. A minimal CI or config edit counts as justified only if
   the plan quotes evidence that the fix's own tests cannot run or pass
   in the repo's required checks without it (an error transcript, or a
   repo comment asking for exactly that edit). A different subsystem, a
   migration, a new option, a rewrite or restructure, or any other CI
   work (new jobs, matrices, upgrades) gets "none". Record any wording that leaves the
   location or approach undecided ("or", "somewhere", "investigate",
   "not sure", "whichever"). A named fallback ("if X doesn't hold,
   I'll do Y instead, and report which") is not undecided wording when
   the main approach is chosen and both X and Y name concrete sites. It
   counts as an honest contingency.
6. **falsifiable-test-plan:** Copy the test plan's action(s) and the
   observable each one checks. For each observable, write what the
   buggy code produces today (from the repro's Actual line) and what
   the plan says the fixed code produces. Record whether they differ.
7. **comment-honors-thread-and-policy:** Take the maintainer directions
   from read-order step 3.3 and the policy requirements from step 3.4.
   For each one, quote the part of the plan comment that addresses it,
   or record "not addressed".

## Check execution

8. Run the checks in rubric table order. Each check uses only its own
   gathered evidence from steps 4–7. Do not re-read the package for
   one check unless that check's gathered evidence is empty, and then
   re-read only the parts that check's Evidence column names.
9. Apply the pass condition exactly as written in `rubric.md`:
   - **diagnosis-fits-evidence:** fail if the step 4 table has any step
     whose result the cause cannot produce, or if the cause is only a
     thread claim no repro step tests. Pass only if every row is "same"
     or is a step that doesn't bear on the cause.
   - **bounded-named-scope:** fail if any change in the step 5 list has
     "none" as its justification, or if undecided-location wording was
     recorded, or if no specific file, function or code site was named.
   - **falsifiable-test-plan:** fail if no observable in step 6
     differs between today's result and the fixed result, or if the
     only action is a generic suite run or a feeling.
   - **comment-honors-thread-and-policy:** fail if any item in step 7
     is "not addressed". Pass if the step 7 list is empty.
10. Grade `unclear` only when the package truly lacks what the check
    needs (for example, no cause is stated at all, or no repro step
    bears on it). It must never mean "I did not look". The grader must
    not fill the gap with outside knowledge of the codebase.
11. Write one evidence line per check: the fact or quote that decided
    it. For a fail, quote the offending plan text or the contradicting
    repro step.
12. Polish never moves a grade. A short plan that passes the
    conditions passes. A long, confident plan that fails one fails.

## Verdict assembly

13. Apply the verdict rule from `rubric.md`: `accept` only if every
    `required` check is `pass`. Any `required` check that is `fail` or
    `unclear` gives `reject`. `preferred` checks never change the
    verdict. There is no third verdict.
14. In the summary, name the deciding check(s) for a reject and quote
    their evidence lines. For an accept, list one line per check.
15. Live mode only: hold the draft comment against `voice-guide.md`,
    and list each broken rule in the summary, quoting the rule. This
    does not change the verdict.
16. Note any point where this procedure did not tell you what to do
    (a procedure gap) in the summary. Do not invent a step.
17. Output the fenced JSON block from SKILL.md last, with `checks` in
    rubric table order, each using the rubric's check name.
