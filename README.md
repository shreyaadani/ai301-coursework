# AI Eval Harness for Open-Source Triage

Custom [Claude Code](https://docs.claude.com/en/docs/claude-code) skills that help contributors work on real open-source projects more effectively, built as part of **CodePath AI301: Advanced AI Open-Source Capstone**.

AI301 is the capstone of CodePath's Applied AI Engineering pathway, where students contribute to large, real-world open-source codebases using Claude Code. This repo holds the tooling I'm building to make that contribution workflow faster and more reliable.

> Status: in progress (Fall 2026)

## The problem

Contributing to an unfamiliar open-source project starts with two judgment calls:

1. **Is this issue actually a good first contribution?** Many issues labeled "good first issue" are stale, underspecified, or sit in repos where outside PRs rarely land.
2. **Is my report ready to post?** Claiming an issue or posting a reproduction that is vague or missing steps wastes maintainers' time and hurts your credibility.

Both are easy to get wrong at scale. This project turns them into rubric-based checks an LLM can run consistently, then measures whether those checks actually agree with human judgment.

## What's in here

| Skill | What it does |
|---|---|
| **Issue grader** | Scores a GitHub issue against a rubric for good-first-contribution fit |
| **`repro-check`** | Reviews a claim or reproduction report before it's posted: are the steps rerunnable, is the evidence there, can a stranger actually follow it |

```
.
├── beat-1-sandbox/   # experiments and early skill iterations
└── tools/            # skills and evaluation tooling
```

## How it's evaluated

An LLM grader is only useful if it agrees with a careful human. Every rubric runs against a **hand-labeled gold dataset**, and **every disagreement gets reviewed**. Each one turns out to be either:

- a **rubric bug**: the check measures the wrong thing, so the rubric gets fixed, or
- a **labeling miss**: the gold label was wrong, so the dataset gets fixed.

### Example 1: a check that was confidently wrong

The original "is this repo actively maintained?" check used recently **merged pull requests** as evidence. On one repo, it rejected *every single issue*.

The disagreements showed why: that project's staff push directly to main instead of merging PRs, so an active repo looked dead. I rewrote the check around **commit activity**, which reflects maintenance regardless of a team's workflow.

### Example 2: one check, two different failures

The `repro-check` skill scored **19/20 agreement with gold** on 20 hand-labeled packages (clear-accept, disclosure, no-evidence, unfollowable-comms, wrong-target), above the 18/20 pass bar.

The one miss: a conda package that gold marked *accept*, but the `steps-rerunnable` check rejected. The report described its `env.yml` in prose instead of pasting it, so the check flagged it as unreproducible.

Comparing it against a package that *correctly* failed the same check made the problem clear. That report depended on a private company monorepo no outsider could ever access. The check was conflating **"not pasted verbatim"** with **"not obtainable by a stranger."** A config described clearly enough to rebuild in thirty seconds is a different problem from one nobody can reach.

I rewrote the check to fail only on genuine unobtainability, re-ran a small canary (the two contrasting packages plus controls) to confirm the fix, then reconfirmed the full run.

### Harness fixes

Separately from grading logic, I fixed three Windows-specific bugs in the eval harness:

- `subprocess` couldn't resolve the `claude` binary (fixed with `shutil.which`)
- stdin crashed on a non-ASCII character in one test package (pinned input to UTF-8)
- the npm `claude.cmd` shim failed under `cmd.exe` while `claude.exe` worked (added a `CLAUDE_BIN` override)

I kept these documented apart from rubric changes, since they were environment bugs, not grading bugs.

**The takeaway from both examples:** an aggregate score like 19/20 hides exactly the bugs worth finding. They only show up when you read each disagreement's evidence closely.

## What I'm learning

- Writing rubrics specific enough that an LLM applies them consistently
- Treating LLM output as something to **measure**, not trust by default
- Diagnosing *why* a grader disagrees with gold, not just whether it does
- Working inside large, unfamiliar codebases with AI-assisted workflows

## About

Built by [Shreyaa Dani](https://shreyaadani.github.io) · [LinkedIn](https://linkedin.com/in/shreyaadani)
