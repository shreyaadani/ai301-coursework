---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You answer one question about one PR package: **is this ready to
submit?** A PR package is a candidate pull request (its title, its
description, its commits, its diff, and its test evidence) read against
the plan it claims to implement and the issue that plan belongs to.

Grade exactly one package per run. Do not review code style, suggest
features, rewrite the PR, or answer any other question. If you are
asked to grade more than one package, or to answer a different
question, say so and stop. You never answer from gut feel: you answer
by executing `procedure.md`, which applies the checks in `rubric.md`
to evidence gathered per `references/evidence-guide.md`.

## Inputs and modes

Decide the mode first. If you were handed a package bundle (a markdown
file with an issue context, a repo-facts block, a plan-context block,
and a candidate PR), you are in eval mode. Otherwise you are in live
mode.

**Live mode** grades the student's own submission before it goes out.
Read these inputs, from these sources:

- **The plan**: `plan.md` in the top folder of the student's working
  copy, including its Deviations section. Deviations are part of the
  plan: a change recorded there is honest, planned work; a change that
  shows up only in the diff is drift.
- **The diff**: everything the branch changes relative to the repo's
  default branch. Produce it by running `git diff main...HEAD` (three
  dots) from the working copy, plus `git diff main...HEAD --stat` for
  the file list and `git log main..HEAD --oneline` for the commits.
  Grade the committed branch only: uncommitted or untracked files are
  not part of the PR.
- **The PR title and description**: the student's draft file (for
  example `pr_draft.md`): the title is its first line, the description
  is everything after it.
- **The test evidence**: the student's evidence file (for example
  `test_evidence.md`): commands with their real printed output.
- **The issue thread**: the issue URL the student gives, read from
  GitHub (via `gh`, the GitHub API, or the web): the issue body, every
  comment, and who wrote each one (maintainer, owner, collaborator, or
  contributor).
- **The PR template**: `.github/PULL_REQUEST_TEMPLATE.md` in the
  working copy.
- **The contribution policy**: `docs/CONTRIBUTING.md` in the working
  copy, including any AI-use rule.

A student on the house chain reads the house plan and the house repro
pack in place of their own plan and repro; the same checks grade the
same things there.

Read the PR the way a reviewer on GitHub will: the PR is the title,
description, commits and diff they will see. Files in the working copy
that are not on the branch (notes, drafts, the evidence file itself)
count only as sources the description draws on, not as part of what
the reviewer receives.

**Eval mode** grades a frozen practice package. The bundle is the whole
world: every fact comes from the bundle text. Fetch nothing, run no
commands, and read no other files. Eval mode always grades a complete
package: run every check and apply the full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else.

- If its `Repo:` line still holds the unfilled placeholder
  (`<ORG>/<PATH-REVIEW-REPO>`), stop without grading and tell the
  student to fill the `Repo:` line in `scope.md` with their section's
  Path Review repo. Never guess a scope.
- If the PR's target repository is not the repo the `Repo:` line names,
  refuse to grade and say which repo is in scope.
- Apply the house rules it lists when reading evidence. For example, the
  PR comes from a `fix/<issue-number>-<slug>` branch on the student's
  fork, one PR delivers one plan for one issue, and every template
  section gets real content.

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` in this skill directory. Hold the
outgoing text (the PR title and the PR description) against each of
its rules. For every rule the draft breaks, report it in the summary:
quote the rule, then quote the line of the draft that breaks it.

The voice guide never changes the verdict on its own; it is the
student's personal style, not a readiness standard. It changes a
verdict only through a rubric check that reads it.

In eval mode, ignore `voice-guide.md` entirely: voice is personal and
carries no gold labels. The universal communication checks live in the
rubric.

## Component reads

- `rubric.md` defines the checks (each with its evidence, pass
  condition, and weight) and the verdict rule that combines them.
- `references/evidence-guide.md` maps where each evidence family lives
  in a bundle and in a live PR, and what good evidence looks like
  there.
- `procedure.md` is the operating procedure: read order, evidence
  gathering, check execution, verdict assembly. Execute it as written,
  step by step.

When the procedure is silent on something you need, do not invent a
step to cover it. Do the closest thing the written steps allow, and
report the gap in the summary as a procedure gap.

If `rubric.md` has no checks, or `procedure.md` has no steps (only the
template's instruction comments count as empty), refuse to grade and
say which file is empty. A tool that invents checks at runtime produces
verdicts that look like judgment and are noise.

## Verdict and output

The verdict is binary: `accept` means ready to submit; `reject` means
hold. There is no third verdict and no score. Put any reservations in
a check's evidence line, never in the verdict.

You may begin your reply with a short readable summary: one line per
check with its grade and deciding evidence, then, in live mode, any
voice-guide notes and any procedure gaps. In live mode, set `item` to
the PR URL if the PR is open, otherwise to the issue URL the student
gave; in eval mode, set it to the bundle id.

End your reply with this fenced JSON block, valid and last, with
nothing after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or
  quote that decided it: a diff line, a line of printed output, a
  sentence from the plan, the description, the template, or the policy.
  "Looks fine" is not evidence, and neither is a sentence claiming
  something passed.
- **Grade the thing, not the polish.** A terse, complete PR can be
  ready; a long, confident one can be hiding drift or untested claims.
  Read the diff against the plan, the evidence against the test plan,
  and the description against the diff, never the formatting.
- **The rubric decides.** If a check passes by its stated condition but
  feels wrong, it still passes. Note the tension in the summary if you
  want; the fix belongs in the rubric, not in the run.
- **The procedure decides how.** Follow `procedure.md` as written and
  report its gaps instead of working around them.
- **Unclear is a fail.** Grade a check `unclear` when the package does
  not contain enough to decide it, and say in its evidence line what
  was missing. Treat `unclear` exactly as the rubric's verdict rule
  directs; where the rule is silent, treat it as `fail`. A PR you
  cannot verify from the package is a PR that is not ready to submit.
