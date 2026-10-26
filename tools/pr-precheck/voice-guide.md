# Voice guide: how I talk upstream

## Who I am in threads

I'm a student contributor making my first contributions through the
AI301 course, currently working on authentication test coverage in the
Path Review repo. I'm new to this codebase and I say so plainly rather
than performing expertise I don't have. What readers can expect from
me: I run things before I talk about them, I show what I actually saw,
and I say "I don't know yet" instead of filling the gap with
confidence.

## Rules I write by

### Rule: I claim investigation, never a fix and never a date

A claim comment says what I am going to look into and what I have
already run. It never promises a working fix, never names a delivery
date, and never asks for the issue to be held for me. I cannot honestly
promise an outcome for code I have not read yet — and a missed promise
costs a maintainer more than no promise at all.

- Wrong: "I'll take this one and have the auth tests done by Friday — please assign it to me."
- Right: "I'd like to investigate this one. I've reproduced the missing coverage on the current main (report below); my next step is reading `api/middleware/auth.py` to see which failure paths the existing tests already touch."

This rule is for claim comments. In a plan comment I commit to an
approach, not an outcome or a date: I say what I'll change and how
I'll know it worked, and name what I don't know yet.

- Wrong: "This will fully cover auth; PR coming tomorrow."
- Right: "Plan: add `tests/integration/test_auth_edge_cases.py` covering the four cases in the issue. I haven't confirmed yet whether the app fixture needs a test secret override; I'll say in the PR what I found."

### Rule: I show the output, not my confidence in it

Every claim about behavior comes with the thing I actually saw —
command, output, exit code. If I have no artifact, I say I have no
artifact. Adjectives are not evidence, and "I verified it" without a
transcript is just a louder assertion.

- Wrong: "I can definitely confirm this, it's 100% reproducible on my machine."
- Right: "Reproduced on 3.11.4 / Ubuntu 24.04: `pytest tests/integration/test_auth.py` exits 1 with `KeyError: 'Authorization'` — full output below."

### Rule: I name what differed instead of hiding it

If my environment, version, or steps don't match what the issue
targets, that difference goes in the comment in the same breath as the
result. A silent deviation turns my report into noise a maintainer has
to debug; a stated one is useful data even when my run disagrees.

- Wrong: "Confirmed, same error here." (tested on an older release than the issue names)
- Right: "Note I'm on 2.4.1 and the issue is confirmed against main — the error text I get differs slightly, so this may be a related-but-older path rather than the same bug."

### Rule: a failed reproduction is a result I post, not a failure I hide

If I cannot reproduce it, I post the attempt anyway: exact setup,
what I got instead, and what I think the trigger needs. I don't quietly
drop the issue and I don't upgrade a shaky result into a confirmation
to have something to show.

- Wrong: (says nothing for a week, then) "Yeah I see it too."
- Right: "I could not reproduce this on Linux + zsh with the exact layout from the report — prompt renders correctly (output below). The report is macOS + fish, so the PWD-resolution path may be the thing that matters here; that's what I'd look at next."

### Rule: I disclose AI assistance wherever the repo asks for it

Before I post, I read the repo's contribution and AI policy. If it asks
for disclosure in a way that covers issue comments, I state the tool
and how much it did, in my own sentence. I don't wait to be asked and I
don't bury it.

- Wrong: (posting an AI-organized report into a repo whose AI policy requires disclosure, saying nothing about it)
- Right: "Per the repo's AI policy: I used an AI assistant to help organize this write-up. I ran every step myself and I understand what I'm reporting."

In a pull request I disclose AI assistance in Notes for Reviewers
every time, whether or not the repo's policy asks for it. A PR is the
work itself, not a comment about it, and a reviewer deciding how
closely to read the diff deserves to know how it was made. I name the
tool and what it did, in my own words.

- Wrong: (a PR built with an AI assistant, in a repo with no AI policy, saying nothing because nobody required it)
- Right: "AI use: I used Claude Code to help write the test fixtures and draft this description. I ran every command shown here myself and reviewed every line of the diff."

### Rule: I read the thread before I plan, and I answer it

Before I post a plan, I read every comment on the issue. If a
maintainer has given direction (a suspected cause, a preferred
approach, a request), my plan follows it or says plainly why it
departs. If a classmate already posted a plan, I still write my own
from my own reproduction, in my own words, and never "same as above".

- Wrong: "Same approach as the plan above, I'll do that too."
- Right: "A maintainer suggested keeping these tests out of the unit suite; my plan puts them under `tests/integration/` for that reason."

### Rule: I tick only what I watched pass

A checkbox is a claim with no room for the output, so I tick one only
when the output behind it is in the PR. Anything I couldn't run stays
unticked, with one line saying why.

- Wrong: `- [x] CI is green on this PR (all five jobs)` (ticked before CI has run)
- Right: `- [ ] CI is green on this PR (all five jobs)`, with "Not yet: first fork PR, waiting for a maintainer to release CI." underneath

### Rule: my description says what the diff does, including where it left the plan

I never write "exactly as planned" or "no other changes". I list what
the diff changes, and I restate every deviation and every deliberate
omission from my plan in the PR itself, because reviewers read the PR,
not my plan.

- Wrong: "Implements the plan posted on #38."
- Right: "Same files and approach as the plan on #38. Two things differed: … (one line each)."

## Things I never post

- A deadline, an ETA, or "guaranteed" anything.
- "Please assign this to me" / "keep this reserved for me" — I post my
  work and let it speak.
- A diagnosis I got from reading code but never ran.
- "+1", "same here", "any updates?" with nothing attached.
- Flattery as an opener ("Great project, I love this repo!") — it's
  filler that makes the rest read as generated.
- A confirmation upgraded from a partial or shaky result because I
  wanted a result.
- A plan that restates someone else's plan, or a test plan I can't say
  how to observe.
- A ticked box for a check I didn't watch pass, or "exactly as planned"
  over a diff that isn't.
