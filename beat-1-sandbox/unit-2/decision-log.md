# Unit 2 decision log — repro-check

Running record of what changed in the skill's three authored files and
why. Newest entries at the bottom. Eval runs get their own section.

## v0 — initial draft (2026-09-27)

Setup: copied `skill/` from the starter clone to
`~/.claude/skills/repro-check/`. No prior copy existed. `issue-select`
untouched.

`scope.md`: shipped with an unfilled placeholder
(`<ORG>/<PATH-REVIEW-REPO>`) rather than the filled file the header
claims. Pasted in `codepath/pathreview-ai301-fa26-s3` — the repo
holding issue #38, my Unit 1 selection. Eval mode ignores this file;
it matters only for live mode, which refuses to grade on a placeholder.

### Design decisions behind the first rubric

**Eight required checks, one preferred.** Each required check is aimed
at a failure family the eval set is built from, so that no category can
fail for lack of a check:

| Check | Aimed at |
|---|---|
| env-recorded | unfollowable-comms (no environment record at all) |
| steps-rerunnable | unfollowable-comms (private, unshared repro) |
| artifact-present | no-evidence (assertion with no run shown) |
| artifact-matches-issue | wrong-target (adjacent symptom shown as the issue's) |
| deviation-acknowledged | wrong-target (silent version/platform deviation) |
| conclusion-within-evidence | no-evidence (confidence the artifacts don't carry) |
| claim-specific-and-honest | unfollowable-comms (boilerplate, over-promising claim) |
| disclosure-when-required | disclosure (the 1-package category floor) |
| own-words (preferred) | repos asking for human-written comments |

**The disclosure check is in from the start, not after a failed run.**
The `disclosure` category holds one package, and the bar requires at
least one match in every category, so missing it fails the run no
matter how well everything else scores. Unit 1 taught this the
expensive way with the merged-PR check.

Before writing the check I read the `contribution policy:` line of all
24 bundles to find where it could over-fire. Four distinct policy kinds
exist in the set and only one gates a package:

- *no stated AI policy* (most packages) → nothing to satisfy
- *responsibility/understanding* (conda, prettier, p5.js) → no
  disclosure ask
- *human-voice* (ripgrep: comments must be human-written, AI comments
  may be hidden) → satisfied by voice, not by a disclosure line
- *scoped ask* (fd: state the tool **in the pull request**; policy
  explicitly has no ask for issue comments) → an issue-comment package
  passes
- *covering requirement* (ghostty: all AI usage in any form, naming
  issues and comments) → disclosure required, absence is a fail

A naive check ("does the repo mention AI? then require disclosure")
would wrongly reject four accepts. The pass condition is written around
the policy's *kind and scope*, not the presence of the word "AI".

**Assume AI assistance.** The evidence guide states that every graded
package is AI-assisted work, so the question is never "was AI used"
(unanswerable from a bundle, and would grade as `unclear` → fail under
my verdict rule, taking good packages down with it) but "does this repo
require saying so here, and was it said".

**An honest cannot-reproduce passes.** Two of the eight clear-accepts
are failed reproductions reported honestly. So `artifact-present` and
`artifact-matches-issue` both carry an explicit cannot-reproduce
clause: the artifact of the *attempt* satisfies them when the report
labels the result plainly and names what differed.

**No structure-shaped checks.** Nothing scores step count, headings,
length, or formatting. One accept is explicitly terse-but-complete and
two rejects are long and polished; grading shape would invert both.
`env-recorded` says "terse but complete passes" out loud for this
reason.

**Generous defaults where evidence can be legitimately absent.**
`deviation-acknowledged` passes when the issue names no target version;
`disclosure-when-required` passes when no policy covers issue comments.
Without these, `unclear`-counts-as-fail would quietly reject accepts.

**Verdict rule:** accept iff every required check passes; `unclear` on
a required check = fail; preferred checks never move the verdict.

### voice-guide.md

Five rules, each with a wrong/right pair in my own words. The house
rule — claims promise investigation only, never a fix and never a date
— is rule 1. Also encoded it as a *rubric* check
(`claim-specific-and-honest`), because eval mode ignores the voice
guide entirely, and the over-promising claim package needs to be
caught by the rubric to count.

## Harness patches (environment, not grading)

The harness could not run on Windows at all. Three bugs, none touching
grading logic; all reversible with `git checkout eval/run_eval.py`:

1. `subprocess.run(["claude", ...])` raised `FileNotFoundError`.
   Windows `CreateProcess` resolves bare names only to `.exe`, and the
   CLI is `claude.cmd`. Fixed with `shutil.which`.
2. `UnicodeEncodeError: 'charmap' codec can't encode '→'`. Python
   encoded the prompt as cp1252; the bundles themselves contain
   non-ASCII (pkg-07 has an emoji), so this hits any Windows student
   regardless of what the rubric contains. Pinned `encoding="utf-8"`.
3. The npm `claude.cmd` shim failed under cmd.exe (`claude.exe is not
   recognized`) although `claude.exe` runs fine when called directly.
   Added a `CLAUDE_BIN` env override; runs use the real executable.

Worth noting in the reflection: the harness assumed a POSIX
environment, and three environment bugs cost more wall-clock than the
rubric revision did.

## Eval runs

| Run | Scope | Result | What I changed after |
|---|---|---|---|
| smoke | `--limit 3` | 3/3 agree | nothing — plumbing check only |
| full v0 | 20 pkgs | **19/20, bar PASS**, all 5 categories matched (disclosure 1/1) | loosened `steps-rerunnable` (below) |
| canary | `--only pkg-05,pkg-18,pkg-06,pkg-20` | 4/4 agree | nothing — loosening confirmed safe |
| full v1 | 20 pkgs, `--save-run` | **20/20, bar PASS**, all 5 categories matched | nothing — this is the submitted run |

### v1 — the one revision: `steps-rerunnable` (2026-09-29)

**The disagreement.** pkg-05 (conda), gold accept, my rubric rejected.
Eight of nine checks passed; `steps-rerunnable` failed with: *"env.yml
contents described in prose ('a valid dependencies: list plus a
category: section') but never quoted inline or linked, so a stranger
cannot run the exact command against the exact input."*

**Diagnosis — the check, not the label.** I wrote the check to catch
pkg-18, where the reproduction lives in a private monorepo with an
unshared config: genuinely unrunnable by anyone else. My pass
condition said inputs must be "reproduced inline or publicly
obtainable", which conflated *not pasted verbatim* with *not
obtainable*. pkg-05's env.yml is described precisely enough that any
reader can rebuild an equivalent in seconds. The fix belonged in the
pass condition.

**The loosening.** `steps-rerunnable` now passes when inputs are shown
inline, publicly obtainable, **or characterized specifically enough to
rebuild an equivalent**, and fails only on material nobody can supply
or reconstruct, or on steps that skip the trigger.

**Canary set, per the harness's rule.** A loosening can flip a package
that already agreed, and partial runs show no bar or floor, so I
re-tested more than the disagreeing package:

- `pkg-05` — the package the change was for
- `pkg-18` — the private-monorepo package this check exists to catch;
  the direct blast radius of loosening it
- `pkg-06` — the other `unfollowable-comms` package
- `pkg-20` — already-agreeing package from `disclosure`, the set's only
  single-package category and therefore the floor's weak point

Result 4/4: pkg-05 flipped to accept, the other three held reject. The
confirming full run then went 20/20.

**What I'd have paid without the canary rule:** the flip risk was real
and would have surfaced on a $4 full run instead of an $0.80 partial.

## Where the proactive disclosure check paid off

`disclosure` scored 1/1 on the very first full run. Building the check
before the first run — instead of discovering the category floor via a
failed run, as happened with the merged-PR check in Unit 1 — meant the
v0 run already passed the bar, and the only revision needed was a
loosening rather than a new check written under time pressure.

## Live mode: the claim comment (2026-09-29)

Issue: codepath/pathreview-ai301-fa26-s3#38 (integration tests for
authentication edge cases). Posted claim:
https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902249218

Repo-side evidence gathered for the disclosure check:
`docs/CONTRIBUTING.md` (not the repo root — the root has no
CONTRIBUTING) states no AI policy and no disclosure ask, so
`disclosure-when-required` passes without a disclosure line. The only
claim-related rule is "Comment on the issue to let others know you're
working on it".

**v1 of the draft was rejected by my own rubric.** It opened "I'll add
integration tests covering the authentication edge cases mentioned
above: expired tokens, malformed tokens, missing headers, and requests
signed with the wrong secret." On an issue whose entire deliverable is
those tests, that promises the fix before any code has been read —
`claim-specific-and-honest` fails on exactly that, and it is the same
sentence as the *Wrong* line in voice-guide Rule 1. Everything else
passed.

**v2 was accepted.** Replacing the promise with the actual first
action ("getting the project running locally and checking which of the
four edge cases … the current suite already covers against
`api/middleware/auth.py`") made the comment *more* specific, not less:
dropping the promise left room for the file name and a concrete first
step. Worth keeping for the reflection — the house rule cost nothing
and bought precision.

Six checks graded `unclear` / "not yet applicable: claim-only draft"
and stayed out of the verdict, per the skill's claim-only rule. They
get exercised for the first time when the repro report is graded as a
full package.

## Live mode: the repro report (2026-09-29)

Posted repro comment:
https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5902682953

Full package (claim + repro) graded **accept**, 9/9 checks passing.
The reproduction itself and the choices behind it are written up in
`repro-decision.md`; what belongs here is what the grading loop taught.

**The first full-package grade passed but flagged a stretch.** The
report stated the result of a `tests/` search in prose without showing
its output. `conclusion-within-evidence` still passed — the
load-bearing conclusion rests on the coverage artifact, not the search
— but voice-guide Rule 2 ("I show the output, not my confidence in
it") was only partly satisfied. Added the command and its empty result
as step 3, with `echo $?` printing 1, since grep prints nothing when
it matches nothing and an empty block alone cannot be told apart from
a command that was never run.

**The rule that mattered more:** the original search had been run with
a different tool, so pasting a `grep` line that had never been
executed would have been fabricated evidence — a command that would
*probably* have produced that output. Ran the real command first and
pasted the real result. This is the same failure the four
`no-evidence` eval packages are built from, seen from the inside: it
would have been completely invisible to any reader of the comment.

**Tooling note for the reflection.** Neither posted comment could be
verified from here: web fetching reports "0 comments" even on issues
that demonstrably have them (checked against p5.js#7168, which the
eval bundle records as having 9). GitHub renders comment threads
client-side. With `gh` unavailable on this machine, posted comments
have to be verified in a browser.
