# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| issue-open | the issue's state, at the top of the issue page or on the bundle's issue line | The issue is open. A closed issue is not available to take, however good it looks | required |
| repo-alive | repo-facts: archived flag, last 5 default-branch commit dates, capture date at top of bundle | Repo is not archived, and at least one of the last 5 default-branch commits is dated within 90 days of the capture date. Releases are not required: a repo that has never published one can still be alive | required |
| maintainer-merges-prs | repo-facts: last 5 default-branch commits; the repository's merged pull requests | At least one default-branch commit within 90 days is a merged PR — a "Merge pull request #N" commit, or a message ending in "(#N)". The PR author need not be an outsider; merges from org branches count. This ranks rather than gates: a project whose maintainers commit directly, or that is new enough to have merged nothing yet, is not thereby dead, and repo-alive already gates liveness | preferred |
| unclaimed | repo-facts: the "this issue" line (assignees, linked PRs); comment thread with dates | No assignee, no OPEN linked PR, and no comment within 30 days of capture claiming intent to work on it. A CLOSED linked PR does not count — that lane is free. A claim a maintainer answered by inviting anyone to try does not count | required |
| scope-settled | comment thread, issue body | Fails ONLY if the record shows a real unresolved question: an open design or product debate in the thread, a question about whether or how to do it that nobody answered, or something the issue marks TBD that you would need in order to start work. Optional extras ("consider also", "lower priority", "additional suggestions"), non-exhaustive lists ("etc."), and a short body that simply describes a bug do NOT make an issue unsettled. Judge only the required core, and never fail this for terseness | required |
| bounded-change | issue body | The required core of the issue is one coherent change with a definite end state. Several files, or several instances of the same edit, are fine. Items the issue marks optional or lower priority are excluded from this judgement. Fails only for an umbrella or tracking issue whose body is a checklist of independent tasks, or for a change that sweeps the whole codebase | required |
| wanted-by-project | issue author association, labels, comment thread, issue body | Fails only when ALL of these hold at once: the issue is a feature or enhancement request rather than a bug or docs fix, its author has no association with the project (association NONE, or a bot), and no maintainer has commented on it or applied any label. Anything a maintainer filed, labelled, or replied to passes, and so does any bug report or docs fix | required |
| policy-allows-ai-assisted-pr | repo-facts: stated contribution policy | The stated policy does not bar the PR you would actually open. Fails if it bans AI-generated code or documentation, states the repo is closed to outside contributions, or requires a CLA, RFC, or pre-approval before a PR. No stated policy passes: absence of a ban is not a ban | required |
| no-exotic-infra | issue body, repo-facts: stated build/test requirements | Nothing in the issue or the repo facts states a requirement beyond a standard laptop — no GPU, no paid external API, no physical device. Silence passes: grade on stated requirements only, never on a guess about the stack | required |
| newcomer-signposted | labels, issue body | Carries a "good first issue" or "help wanted" label, or the body walks through how to contribute | preferred |
| verification-path | issue body | The issue gives you some way to tell the change worked: acceptance criteria, a command to run, a test to add, reproduction steps, or a clear statement of the expected behaviour. A docs-only change, where maintainer review is the check, also passes | preferred |

## Verdict rule

Accept only if every required check passes. Preferred checks never change the
verdict; they only rank among accepted issues. A required check graded
`unclear` counts as a fail — except for the checks whose pass condition above
says silence passes (`policy-allows-ai-assisted-pr`, `no-exotic-infra`), which
are graded `pass` when the bundle is simply silent.

A check worded as "fails only if X" is a pass whenever X is not positively
shown in the bundle. Do not fail such a check because the issue is short, or
because you would have liked more detail.
