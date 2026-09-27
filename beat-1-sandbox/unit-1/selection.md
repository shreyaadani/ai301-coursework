# Unit 1 — Issue Selection

## Chosen issue

**https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38**
*Add integration tests for authentication edge cases*

**My skill's verdict: `accept`** (all 8 required checks pass)

| Check | Grade | Evidence |
|---|---|---|
| issue-open | pass | State: Open |
| repo-alive | pass | Not archived; newest default-branch commit 2026-09-16, 11 days before grading |
| unclaimed | pass | Assignees: none; no linked PRs; no comments in thread |
| scope-settled | pass | Four named failure scenarios and three named files; nothing marked TBD |
| bounded-change | pass | One new test file plus conftest support — all the same testing edit |
| wanted-by-project | pass | Filed by the repo owner with maintainer-applied labels |
| policy-allows-ai-assisted-pr | pass | No CONTRIBUTING.md at either standard path (404); silence passes |
| no-exotic-infra | pass | FastAPI integration tests; nothing beyond a standard laptop stated |
| maintainer-merges-prs | fail (preferred) | Repo has 0 merged PRs; last 8 commits are direct pushes |
| newcomer-signposted | fail (preferred) | Labels are api, enhancement, tests, tier-2 — no good-first-issue |
| verification-path | pass | Four explicit scenarios the tests must cover |

**Backup, if #38 stalls: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/36**
(*POST /reviews has no test for a profile with no ingested documents* — also `accept`;
`tier-1` and `good first issue`, 2–3 hours, one file. Lower risk, smaller scope.)

Why #38 fits me: it is FastAPI backend work whose deliverable is a test suite, which is
the backend-and-testing work my scope.md fit profile asks for, and its 3–5 hour estimate
matches the time I have in a week. It names all three files and all four failure cases,
so there is nothing left to guess at before I start.

---

## Run history

| # | Scope | Result |
|---|---|---|
| 1 | `--limit 3`, first filled rubric | 2/3 — `issue-01` false reject |
| 2 | `--only` × 5, revised rubric | 5/5 (items I had tuned against — weak evidence) |
| 3 | `--limit 3`, revised rubric | 3/3 |
| 4 | **full 20** | **16/20, below bar** — 4 false rejects, all `clear-accept` |
| 5 | `--only` × 8 (4 misses + 4 scope rejects) | 8/8 |
| 6 | **full 20, confirming** | **20/20, PASS** |
| 7 | `--only` × 3 (dead-repo items) | 3/3 — after demoting `maintainer-merges-prs` |
| 8 | **full 20, final — the committed run** | **19/20, PASS**, category floor held |

Run 4 is the one that taught me something. Runs 1–3 were partial and all agreed, which
made the rubric look finished; the full run showed four false rejects that no partial run
had touched. Runs 7–8 came after live mode exposed a check that could not work on the
Path Review repo (see Trade-offs).

## Issue analysis

**`issue-15`** (zulip/zulip#19589). **Gold: `reject`. My rubric: `accept`.** This is the
one disagreement in my committed run.

Gold rejects it for "years of design debate and two abandoned PRs behind a friendly
label." My rubric accepted it because two of my checks read that same evidence
differently, both by deliberate design:

- `unclaimed` passed on "linked PRs #20840 and #23123 both closed." I wrote that check so
  a **closed** linked PR does not block: if the previous attempt is closed, the lane is
  free. Gold reads the same two closed PRs as evidence the work is harder than it looks.
- `scope-settled` passed because the grader judged that a contributor's sample Slack
  payload had answered the maintainer's clarification request, leaving only an item the
  issue itself marks "Perhaps ... also" — which my wording explicitly excludes from
  counting as unsettled.

What makes this worth writing down is what the earlier run hid. In run 6, `issue-15` was
**rejected**, agreeing with gold — but the only check it failed was
`maintainer-merges-prs`, on the grounds that "no last-5 commit message uses 'Merge pull
request #N'". Zulip simply does not use merge-commit formatting. My rubric got the right
verdict for a reason that had nothing to do with why the issue is actually bad. Demoting
that check to `preferred` removed the accidental catch and revealed that `scope-settled`
never saw this issue at all. 19/20 is a more honest score than the 20/20 before it.

## Check rationale

`unclaimed`, quoted exactly as it stands in the rubric I uploaded:

> No assignee, no OPEN linked PR, and no comment within 30 days of capture claiming intent
> to work on it. A CLOSED linked PR does not count — that lane is free. A claim a
> maintainer answered by inviting anyone to try does not count

Three thresholds, each there for a reason I can point at in the eval set:

- **OPEN linked PR, not any linked PR.** `issue-09` (conda/conda#7617) is gold `accept`
  and has a closed linked PR (`conda/conda#11627`). A check that failed on any linked PR
  would reject it. A closed PR means the previous attempt ended — the work is available.
- **30 days, not forever.** `issue-09` also carries a 2022 comment claiming the issue. A
  claim with no PR four years later is not a claim. 30 days is long enough that someone
  three weeks in is still protected, short enough that a stale claim does not freeze an
  issue permanently.
- **A maintainer inviting others cancels a claim.** On `issue-09` the maintainer answered
  the claim with "Think you can just give it a try if you are interested" — the claim was
  explicitly opened back up.

## Trade-offs

**Closed linked PRs: free lane, or warning sign?** I could not have both `issue-09` and
`issue-15`. `issue-09` is gold `accept` with one closed linked PR; `issue-15` is gold
`reject` with two. Treating closed PRs as abandoned attempts would catch `issue-15` and
lose `issue-09`. I chose "free lane" because it is the reading that is true more often —
most closed PRs mean the attempt ended and the work is open — and because the failure
direction is safer: I would rather look at an issue that turns out to be hard than skip
one that was available. The cost is visible and I am not hiding it: `issue-15` is the
single disagreement in my committed run.

**`maintainer-merges-prs` had to stop gating.** I first wrote it as `required`: a repo
that merges no PRs will leave yours unreviewed. It passed the eval that way. Then live
mode on the Path Review repo rejected all three candidates at once — the repo has zero
merged PRs, because staff push straight to main. The check was correct for real
open-source repos and wrong for a classroom one, and `scope.md`'s own house rule says
credit attaches to the PR you open, not to whether it merges. I demoted it to `preferred`
so it ranks candidates instead of gating them; `repo-alive` still gates liveness, and the
three dead-repo bundles are all still rejected without it. A rubric that scores well on
the eval set and rejects every issue in the repo it is pointed at is not finished.

**Silence passes on policy.** `policy-allows-ai-assisted-pr` treats a repo with no stated
AI policy as a pass rather than an unknown. With `unclear` counting as a fail, the strict
reading would have rejected most of the eval set, since most repos state nothing. The risk
I accept is a policy that exists somewhere I did not look.

---

## Reflection prompts 

1.Realizing the eval and the real world aren't the same test:

The biggest surprise was that a rubric can score a perfect 20/20 on the eval set and still fail completely the moment it meets a real repo. My eval bundles were all snapshots of established open-source projects with real merge histories, so a check like "requires a merged external PR in the last 90 days" looked airtight, since it correctly caught every dead-repo case in the set. But the Path Review repo is staff-seeded for the class, with zero merged PRs by design, since credit attaches to opening a PR rather than getting it merged. The same check that made my rubric pass the eval made it reject every single open issue on the one repo I actually needed to use it on.

That taught me the eval set can only test what it was built to test. It has no way to force you to think about cases outside its own assumptions. Writing the rubric itself needed more reruns than I expected, not because the checks were badly worded, but because each fix that solved a visible failure sometimes created an invisible one I couldn't see until I ran it against something outside the eval, in this case, real candidate URLs from the actual target repo. If I'd only optimized for passing the bar, I would have shipped a rubric that looked finished but couldn't do its one job.

2.What I'd do differently:

If I started over, I'd look at real candidate issues much earlier in the process, not after the eval was already passing. I treated the eval set as the finish line, when really it was only a training ground built from established repos. Testing against actual open issues from day one, even roughly, would have surfaced the merged-PR assumption long before it cost a rerun. The eval bundles are static and safe. Real issues are messier, and that gap is exactly what the eval can't teach you on its own.

3.Why #38 fits me:

I chose #38, adding integration tests for authentication edge cases, because it lines up with the fit profile I wrote into scope.md: I'm most comfortable in Python/FastAPI on the backend, I lean toward testing work over UI work, and it fits inside the few hours a week I actually have. It's also bounded in a way I could verify myself: the missing test cases are named explicitly (expired tokens, malformed tokens, missing headers, wrong-secret tokens), so I know what "done" looks like before I even start, rather than discovering the real scope mid-PR.

4.What's still uncertain about my rubric:

My biggest worry is that the rubric is more rigid than real issues actually are. Every check I wrote resolves to a clean pass/fail with a specific threshold, which is exactly what made it gradeable and testable, but real issues don't sort into cookie-cutter categories that cleanly. An issue can be mostly well-scoped with one ambiguous sentence, or nearly abandoned but with one recent fluke commit, and my rubric has to force a binary verdict onto something that's genuinely a judgment call. I don't think this is fully fixable, since some rigidity is the price of having a rubric someone else could apply consistently, but I'd want to test it against more messy, real-world edge cases than the 20 (now plus 3 live) I've actually run it on before I'd fully trust it beyond this one assignment.
