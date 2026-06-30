---
name: code-review
description: "Coordinate a delegated full multi-persona code review for a PR or local change. Use when asked to review a PR, run review agents, get code review feedback, or iterate on review feedback using all CodeReview personas: elegance, idiom, correctness, completeness, security, YAGNI, and scope. Invoking this skill is an explicit request to launch reviewer subagents."
---

# Code Review

## Overview

Use this skill to run one coordinated review wave across all focused review personas, merge their feedback, apply accepted fixes, and follow up only with the reviewer sessions that need another look.

Invoking this skill is an explicit user request for delegated multi-agent review. Do not ask for separate permission to spawn reviewer subagents, and do not collapse the personas into a single local review. If subagent tooling is unavailable, report the delegated review as blocked.

## Review Wave

Launch exactly one delegated subagent for each persona:

- `$cross-harness-skills:code-review-elegance`
- `$cross-harness-skills:code-review-idomatic`
- `$cross-harness-skills:code-review-correctness`
- `$cross-harness-skills:code-review-completeness`
- `$cross-harness-skills:code-review-security-review`
- `$cross-harness-skills:code-review-yagni`
- `$cross-harness-skills:code-review-scope`

Give each reviewer the same raw review packet: PR description, cited issue or objective, relevant diff, changed files, validation results, and any constraints from the user. Do not include your intended fix or your private diagnosis in the prompt.

Use prompts shaped like:

```text
Use $cross-harness-skills:code-review-correctness to review this PR. Focus only on your persona's facet.
Return findings first, with severity and file/line references when available.
If there are no findings, say so and mention residual risk.

Review packet:
<PR title/body, objective, diff summary, validation, relevant files>
```

Keep a mapping of persona to subagent session id. This mapping is part of the work product until review follow-up is complete.

If the current harness has no callable subagent mechanism, stop before reviewing and report that the required delegated review wave is blocked by missing tooling. Do not silently substitute a manual single-agent review for this skill.

### Capacity Fallback

If subagent tooling exists but a spawn attempt fails because capacity is exhausted, an agent-thread limit is reached, or the harness rejects the parallel fan-out, keep the review running instead of dropping personas:

1. Record which personas successfully spawned and which did not.
2. Wait for the spawned reviewers to finish.
3. Close or release finished reviewer sessions when the harness supports it.
4. Spawn the remaining personas sequentially with the same raw review packet.
5. Continue until every required persona has returned a result or the harness refuses even one-at-a-time execution.

Treat one-at-a-time refusal as a blocked delegated review and report which personas were completed before the block. Do not summarize a partial wave as complete.

## Triage

Wait for the first review wave to finish, then combine findings into one triage list.

For each finding, decide:

- `accept`: implement or document the fix.
- `reject`: explain why the finding is not applicable.
- `defer`: record as follow-up because it is valid but outside this PR.
- `needs-clarification`: ask the same reviewer session a targeted question.

Treat `CodeReviewScope` as the guardrail for the whole review. If another persona asks for broader behavior, generalized architecture, extra cleanup, or unrelated hardening, compare that request against the scope review before accepting it. Prefer the scope review when the broader request would cross lanes or expand the PR beyond its objective.

## Iteration

Apply accepted fixes in the current workspace. Do not start a new wave of review agents after making changes.

If follow-up review is requested or needed:

1. Reuse the existing subagent sessions from the first wave.
2. Message only the reviewers whose findings were fixed, rejected with uncertainty, or marked `needs-clarification`.
3. Include a concise update with the specific diff or reasoning relevant to that reviewer.
4. Do not message reviewers whose facet was unaffected.
5. Do not launch replacement reviewers unless the original session is unavailable and that reviewer is necessary to complete the task.

Example follow-up prompt:

```text
Follow-up on your CodeReviewScope finding about unrelated docs churn.
I removed the unrelated docs changes and kept only the authentication fix.
Please re-check only that scope concern against this updated diff:
<small diff or file references>
```

## Final Review Summary

Report the review outcome with:

- Persona coverage: which reviewer sessions ran.
- Accepted findings and changes made.
- Rejected or deferred findings with short rationale.
- Scope guardrail decisions, especially where broader suggestions were not taken.
- Follow-up sessions messaged, if any.
- Validation rerun after fixes.

If no actionable findings remain, say that clearly and identify any residual risk.
