# Unit 4 — Test and Submit: issue #38

**GitHub username:** shreyaadani

## Pull request

Link: https://github.com/codepath/pathreview-ai301-fa26-s3/pull/267

Title: `test(api): add integration tests for auth middleware edge cases`

Base: `codepath/pathreview-ai301-fa26-s3` `main`. Head: `shreyaadani/pathreview-ai301-fa26-s3` `fix/38-auth-integration-tests` @ `71ec431` (4 commits, 3 files, +189 / -16).

## Branch

`fix/38-auth-integration-tests`, four commits (`9868159` → `71ec431`). The fourth, `71ec431 test(api): add docstrings to stub session helpers`, came from my own pr-precheck run flagging CONTRIBUTING's docstring rule.

## Run history

| # | Command | Changed before this run | Result |
|---|---|---|---|
| 1 | `--limit 3` (smoke) | First full drafts of SKILL.md, rubric, procedure, evidence guide | 2/3 partial. `pkg-02` (clear-accept) failed `decisive-test-evidence`; re-graded alone it accepted |
| — | | Defined a fix-proving item as one whose before shows the bug; "still works" items became non-regression and never decide the check | |
| 2 | Full, `--save-run` | — | 18/20, PASS, every category matched. Misses: `pkg-08`, `pkg-11` (both clear-accept, `decisive-test-evidence`) |
| — | | Accepted a specific recorded result (printed output, or a transcript note naming exactly what was observed) in place of printed output only | |
| 3 | `--only pkg-08,pkg-11,pkg-04,pkg-07,pkg-10,pkg-14,calib-04 --include-calibration` (canaries) | — | `pkg-08` now accepts; all four not-tested canaries and `calib-04` still reject. `pkg-11` now failed on the AI-disclosure wording |
| — | | AI disclosure must give what that repo's own policy asks for, instead of always requiring the tool name | |
| 4 | `--only pkg-11,pkg-20,pkg-01` (canaries) | — | 3/3: `pkg-11` accepts; both standards-wall packages still reject on `template-and-policy-met` |
| 5 (saved) | Full, `--save-run` | — | 19/20, PASS, every category matched. Only miss: `pkg-05` (clear-accept, `decisive-test-evidence`) |

## Package analysis

I'm using **pkg-05**, the one package my final saved run disagrees with. The gold label is accept; my tool rejected it on `decisive-test-evidence`.

The plan's test plan names two things to show: "re-run the issue's script, expect both `atuin` rows" and "re-run with two same-name same-key bindings, expect one row plus a warning". The evidence shows the first one properly, a real before/after table going from one `atuin` row to two. The second one only gets a sentence: "Same-key redefine prints the one-time warning." The grader's evidence line was that this item "has only the claim ... with no command or output".

My rubric defines a fix-proving item as one whose before result shows the bug, and anything that checks existing behavior still works as non-regression. The warning is neither. It isn't the issue's bug (nothing in the repro shows it broken), and it isn't old behavior being preserved; it's new behavior the fix adds. My definition has no slot for that third kind of item, so the grader had to guess, and in this run it guessed fix-proving. The same package agreed in my earlier full run, which tells me this is a boundary the grader flips on, not a settled call.

I don't think the gold label is wrong: the issue's own repro is decisively shown, which is what the plan exists to prove. The fix would be to say explicitly where new behavior introduced by the fix sits. I left it alone because loosening it risks letting through a PR that never shows a feature it promised, and 19/20 already clears the bar with every category matched.

## Check rationale

The check is `decisive-test-evidence`. As it reads in my uploaded `rubric.md`:

"Pass only if every fix-proving item on that list has, in the evidence, the actual command, named test, or exact user action AND a specific recorded result after the change, and that result differs observably from the before (shown in the evidence or in the repro evidence): an error that is gone, a value that changed, a test that now passes against the issue's input. A specific recorded result is either printed output, or a before/after transcript note that names exactly what was observed: the exact error text, message, value or listing, or a named test and its outcome. The run must exercise the changed path with the repro's own input, not a control case or a path the fix never touched. Fail if the only evidence for a fix-proving item is a non-specific claim that names no observed result ('tested locally', 'works now', 'colors work', 'tests pass', 'verified on my machine'); if the evidence runs only a control or unchanged path; or if the test plan names more fix-proving items than the evidence shows and the missing one is not disclosed (partial evidence presented as complete). A missing item passes only when the description says plainly that it was not verified and why. Secondary smoke or non-regression lines may be stated briefly; they never decide this check on their own."

I chose this one because the eval moved it more than any other check, and every miss in both of my full runs came from it. It started as the strictest reading of the activity's rule, "proof is a line in the diff or real output from a command", and that rejected two clear accepts: pkg-08, a TUI fix where the evidence is a before/after flow because there is no command to print, and pkg-11, where the results are written as `# after:` comments. I loosened it to accept a recorded result that names exactly what was seen, and kept failing claims that name nothing. I then ran every not-tested package plus calib-04 as canaries to make sure "works now" and "verified on my machine" still fail, which they did. Earlier, the smoke run had already forced me to add the fix-proving versus non-regression distinction, after pkg-02 flipped on a "successful save still clears" line. The partial-evidence clause comes straight from the activity's warning about a test plan naming two things and the evidence showing one, which is exactly how calib-04 is still rejected.

It also graded my own PR the way I'd want: it passed my evidence because all three repro steps have real before and after output in the description, and it counted the deliberate `auth.py:36` break as proof the tests can fail.

## Trade-offs

The biggest trade-off is that I softened my own activity guidance to match the gold labels. The activity said a comment describing what a command printed isn't proof; my rubric now accepts a before/after note if it names the exact observed result. That bought back pkg-08 and pkg-11, and the canaries showed vague claims still fail. What it costs is that a carefully worded annotation that was never actually run would now pass where only real printed output would have before. The check can't tell a true note from a made-up one.

A second cost is how the rubric was written. The kinds of checks come from the four failure categories and the activity guidance, but the example phrases in every pass condition ("implements the plan exactly", "tested locally", "rename pass", "commented-out earlier attempts") came from the gold notes on the scored packages. So my 19/20 probably flatters the rubric on PRs it has never seen.

`description-explains-change` has a deliberately low bar so that terse but complete PRs like pkg-11 and pkg-19 pass. The edge I found is that it asks why the PR exists, not why each change beyond the issue was needed. A PR that changed `ci.yml` with no explanation would still pass it, even though a maintainer would ask about it. I left that unfixed rather than risk rejecting terse accepts.

Grading also isn't fully stable. pkg-02 and pkg-05 each flipped between runs at the same fix-proving boundary, so my first full run's 18/20 had no margin, and the final 19/20 has one package of margin. Treating `unclear` as a fail makes the tool conservative: a PR it can't verify gets held, never waved through, which I think is the right side to err on for a pre-submit check.
