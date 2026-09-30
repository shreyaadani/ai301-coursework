# Evidence guide: where proof lives in a reproduction package

Every rubric check names evidence. This file says where that evidence
sits in a package and what it looks like when it is good enough.

A note that applies to every family below: in an eval bundle the
sections are literal headings — `## Repo facts`, `## Issue`,
`## Thread highlights`, `## Candidate claim comment`, `## Candidate
repro report`. In live mode the same five things live on the issue
page (`## Issue` = the issue body, `## Thread highlights` = the
comments, `## Repo facts` = the repo's README/CONTRIBUTING and issue
template) and in the student's own draft files.

## Environment

**Where it lives.** In a bundle: the opening `Environment:` line of
the `## Candidate repro report`, and nowhere else — if that line is
missing, the environment record is missing, even when version strings
appear later inside an artifact. Read it against two things: the
version and platform named in the `## Issue` body (often in the first
lines, e.g. "p5.js version: 1.9.4, 1.10.0. Browser: any…"), and the
`bug reports:` bullet in `## Repo facts`, which lists what that repo's
template asks reporters to state. In live mode: the student's draft
report, against the issue body and the repo's bug-report template.

**What good looks like.** The version or build of the software under
test is named, plus the platform facts this particular issue depends
on — the ones the issue itself or the repo's template treats as
load-bearing. A Windows-only issue needs the OS and the driver; a
shell-rendering issue needs the shell; a browser issue needs the
browser and version. One dense line ("yq 4.53.3, macOS 14.6") is a
sufficient record; a paragraph that never names a version is not.
Absence of any environment record is a fail even when the rest of the
report is strong — a stranger cannot place the attempt.

## Steps

**Where it lives.** The numbered or prose steps in the `## Candidate
repro report`, plus every input those steps reference: config files,
command lines, source snippets, sample data. In live mode: the draft
report, read alongside anything it links to.

**What good looks like.** A reader on a different machine can run it.
The starting state is stated (a fresh config, an isolated instance, a
specific file), the trigger is an exact command or action rather than
a summary of one, and the materials the artifact came from are in the
package. The failure mode to watch for: a reproduction that happens
inside something private — an internal monorepo, a config the report
mentions but never shows — where every step is literally unrunnable by
anyone else. Publicly obtainable inputs (a CDN script tag, a release
binary, a documented flag) are fine; unshared ones are not.

## Behavior shown

**Where it lives.** The fenced output blocks, logs, measurements, and
described screenshots inside the `## Candidate repro report`. Read
them against the specific symptom named in the `## Issue` body — the
error text, the error class, the exit code, the visible outcome — and
against any maintainer narrowing in `## Thread highlights`.

**What good looks like.** The artifact shows the issue's behavior, not
a neighbor of it. Compare the artifact to the issue at the level of
what actually failed: a graceful argument-validation error exiting 1
is not the capacity-overflow panic exiting 101; a compile error from an
unbound variable is not an invalid-path-expression error; garbled
escape output with the terminal still running is not a crash. An
artifact that shows only that the tool launched — a version banner, a
session list, a window that opened — shows nothing about the bug.

A control run is the strongest form of this evidence: the same steps
with one variable changed, demonstrating the behavior appears and
disappears with the trigger. Not required, but when present it usually
settles the check.

## Honesty

**Where it lives.** The `Expected:` / `Actual:` lines and any
concluding or diagnostic sentence in the `## Candidate repro report`,
read against the artifacts in that same report. Also the confidence
language in the `## Candidate claim comment`.

**What good looks like.** The words claim exactly what the artifacts
carry, and no more. Three shapes pass: a confirmation whose output
matches the issue; an honest cannot-reproduce that says so plainly,
shows what the attempt produced instead, and names what likely differed
(the environment, the data distribution, the configuration); and a
partial result labeled as partial. What fails is confidence with
nothing behind it — "guaranteed reproducible", "I verified this race
condition", a root cause asserted from reading rather than running —
and expected/actual written backwards from what was shown.

An evidenced cannot-reproduce is a **pass**, not a failure of the
package. It is a real result honestly reported, and it is often more
useful to a maintainer than another unverified confirmation.

## Comms

**Where it lives.** The `## Candidate claim comment`, read against the
issue it would be posted on, and against the `contribution policy:`
and `bug reports:` bullets in `## Repo facts`. In live mode: the
student's draft comment, the repo's CONTRIBUTING.md and any AI policy
file, and `scope.md`'s house rules.

**What good looks like — the claim.** It could only have been written
about this issue: it names what the author reproduced, on what
version, or what they will read next. It promises investigation only.
Boilerplate that would fit any issue in any repo fails, as does a bare
"+1, any updates?", as does any promise of a fix, a date, or a
reserved assignment.

**What good looks like — the policy.** Read the contribution-policy
bullet for what it actually requires, because these differ in kind and
only one kind gates the package:

- *No stated AI policy* → nothing to satisfy; the check passes.
- *Responsibility / understanding rules* ("you are responsible for
  what you submit", "only submit code you understand", "fully
  AI-generated contributions are not accepted") → no disclosure ask;
  passes without a disclosure statement.
- *Human-voice rules* ("comments to maintainers must be written by
  humans in their own words", "AI-generated comments may be hidden")
  → satisfied by a comment that reads as a specific person writing,
  not by a disclosure line.
- *Scoped disclosure asks* ("state the tool and extent of its use **in
  the pull request**", with no ask for issue comments) → the scope is
  part of the rule; an issue-comment package passes.
- *Covering disclosure requirements* ("all AI usage in any form must be
  disclosed", policy naming issues and comments explicitly) → a
  comment must state the AI assistance and its extent. If none does,
  the package fails, however good the reproduction is.

Treat every package graded here as AI-assisted work. The question is
never "was AI used?" — assume it was — but "does this repo require
that to be said here, and was it said?" A disclosure that names the
tool and the extent ("I used an AI assistant to help organize this
report; I ran and verified every step myself") satisfies a covering
policy; a report that silently omits it does not.
