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
