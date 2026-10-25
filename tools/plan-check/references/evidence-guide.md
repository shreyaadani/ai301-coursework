# Evidence guide: where evidence lives in a plan package

Eval bundles use these sections: `## Repo facts`, `## Issue`,
`## Thread highlights`, `## Repro evidence`, `## Candidate plan`,
`## Candidate plan comment`. Live mode maps them to: the repo's
CONTRIBUTING / README / AI policy files, the issue body, the issue
thread, the student's own posted repro comment on that issue, the
student's `plan.md`, and their draft comment file.

## Diagnosis and grounding

**Where it lives.** The cause is in the candidate plan's
`Cause:` / `### Diagnosis` text (calib-01: "Cause: after a push
started from the branch-commits view, that view's model is not
refreshed"), and sometimes restated in the comment ("I traced this
to..."). The behavior that cause must explain lives in `## Repro
evidence`: the numbered steps, their recorded results, any
`Control:` line, and the Expected / Actual lines. Live mode: the
student's posted repro comment, not their memory of it.

**Strong vs weak.**
- *Strong:* the cause predicts every recorded step, including the
  control. calib-01's "view model not refreshed after push" explains
  step 3 (stale color in the open view) and step 4 (re-entry rebuilds
  the view, and the color fixes itself).
- *Weak, and it fails:* a cause that a repro step rules out. calib-03
  blames pager key bindings, but repro step 3 shows the same 26 s with
  `--paging=never` (no pager at all), and step 2 shows the same pager
  seeking instantly with colors off. The evidence points at
  highlighting, so the binding diagnosis is contradicted however
  polished it reads.
- *Weak:* a cause adopted from a non-maintainer thread claim ("As
  identified in this thread...") that no repro step tests. A
  maintainer's in-thread analysis counts as grounding only when it is
  consistent with the repro steps.
- Look hardest at control and comparison steps: they are where a
  wrong cause shows up.

## Scope

**Where it lives.** The plan's `In:` / `Out:` lines or `### Scope`
section, plus any `Change:` / `### Changes` / `### Approach` list.
Every numbered change counts as scope, even when it isn't labeled as
scope. Read it against `## Issue` (what was actually reported).

**Strong vs weak.**
- *Strong:* one named site and an explicit Out line. calib-01: "In:
  the push completion callback in
  `pkg/gui/controllers/sync_controller.go`... Out: any change to how
  push status is computed, or to other views' refresh behavior."
- *Weak:* areas instead of sites ("the editor code", "the input
  stack"), undecided sites ("upstream or vendored, whichever is
  easier"), or extras the issue never asked for sitting next to the
  fix (a migration, a new option or settings panel, a rewrite or
  restructure, CI changes, "while I'm here" refactors). A small related
  test or doc line for the changed behavior is fine. A second feature
  is not.
- A plan that explicitly *defers* a larger fix ("not in scope: X,
  because Y") is bounded, not incomplete.

## Executability

**Where it lives.** The `Change` / `Approach` / `Changes` list and the
file paths it names (calib-03's numbered steps name
`minus::input::generate_default_bindings()` and `src/output.rs`).

**Strong vs weak.** A stranger could open the named file and start
work: the site, the mechanism (what call or condition changes), and
the order are stated. Weak plans hand the real decisions to build time
("profile and optimize", "investigate first", "not sure which layer").
Executability is graded inside **bounded-named-scope** (named sites
and no undecided location) and **falsifiable-test-plan**.

## Test plan

**Where it lives.** The plan's `Test:` / `### Test plan` text, read
against the repro's numbered steps and its Expected / Actual lines.

**Strong vs weak.**
- *Strong:* re-runs specific repro steps and names the observable that
  flips. calib-01: "repro steps above; at step 3 the color must flip
  without leaving the view". Today step 3 shows the unpushed color, and
  after the fix it must show the pushed color, so a failed fix is
  visible.
- *Weak, and it fails:* calib-04's "Run the full test suite (`cargo
  test --workspace`) and make sure nothing regresses". The current
  buggy code already passes that, so it proves nothing about the fix.
  Feelings ("should feel fast", "nothing else should feel broken") fail
  for the same reason.
- A new automated test passes only if the plan says what it asserts,
  and that assertion would fail on today's code.

## Honesty

**Where it lives.** A `Risks` / `Unknowns` section if present, hedges
inside the scope text (calib-04: "I would present it as opt-in
behavior for review"), the plan's `## Deviations` section (live mode,
after the build), and certainty words in the comment ("I traced this",
"brings back instant shift+G").

**Strong vs weak.** Strong plans label unknowns as unknowns and say
how they'll be resolved. Weak plans state an untested cause as fact
("I traced this to..." when the repro never isolated it). In this
rubric, false certainty about the cause is caught by
**diagnosis-fits-evidence**. A deviation recorded in `## Deviations`
is honest work. A change that appears only in the diff is not.

## Comms

**Where it lives.** The `## Candidate plan comment`, read against
`## Thread highlights` (look only at comments tagged OWNER, MEMBER,
COLLABORATOR or MAINTAINER for direction) and the `contribution
policy` line in `## Repo facts`. Live mode: the issue thread and the
repo's CONTRIBUTING.md / AI_POLICY.md.

**Strong vs weak.**
- *Strong:* engages each maintainer direction by name. calib-04's
  comment cites the owner's "deep fix is hard" note and explains why
  it stays out of the crate. calib-01's comment cites the
  CONTRIBUTING review-bandwidth note. It also meets each stated policy
  requirement: calib-04's repo requires human-written comments and
  disclosure, and the comment says "the implementation will be
  AI-assisted with me reviewing every change, and this comment is in
  my own words."
- *Weak, and it fails:* the comment proposes something else while a
  maintainer has already isolated the culprit or asked for testing, and
  never acknowledges it; or the repo's policy requires AI-use
  disclosure and the comment has none. Every package is treated as
  AI-assisted work.
- *Not a requirement:* if the thread has no maintainer direction
  (calib-01: "0 comments") and the policy states no AI or comment rule,
  boilerplate-free brevity passes. Do not invent conventions.
