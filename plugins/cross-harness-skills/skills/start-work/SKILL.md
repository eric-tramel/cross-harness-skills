---
name: start-work
description: Start repository development work safely. Use when an agent takes on new code, docs, test, CI, packaging, or release work; when it must create or verify an isolated branch/worktree; when sibling agent collisions are possible; or before editing files for a contributor task.
---

# Start Work

## Overview

Use this workflow to start a contributor task with the same orientation every time: verify the checkout, isolate the branch, read governing instructions, and choose the right validation path before editing.

Using `$cross-harness-skills:code-review` in this workflow is an explicit user request for that skill's delegated reviewer subagents. You have explicit permission to spawn the `$cross-harness-skills:code-review` review wave when the follow-on workflow reaches that step.

## Follow-On Workflow

After the start checklist and branch/worktree setup, run the rest of the contributor workflow in this order:

1. Use `$cross-harness-skills:crystallize` to turn rough or ambiguous input into a ready-to-implement local plan.
2. Implement the scoped change from that plan or, when the task is already concrete, from the user's explicit request.
3. Use `$cross-harness-skills:code-review` and iterate fixes until the review is clean or every remaining item is explicitly rejected or deferred with rationale.
4. Use `$cross-harness-skills:author-pr` to create a PR when the branch is ready.

## Start Checklist

Run these from the repository root unless the current harness already supplied a trusted workspace root:

```bash
pwd
git rev-parse --show-toplevel
git status --short --branch
git worktree list --porcelain
find .. \( -name AGENTS.md -o -name CLAUDE.md -o -name GEMINI.md \) -print
```

If the user refers to previous work, another agent, an unexplained decision, a branch you cannot see, or a failure from another session, use available session-history or agent-history tools before making assumptions.

## Branch And Worktree

Use an isolated branch for development work. Choose an appropriate branch prefix (e.g. `feat/`, `bug/`, etc.)

If the harness already placed you in an isolated worktree, create or verify a branch there:

```bash
git switch -c codex/<short-task-name>
```

If you are in the main checkout and need to start a new worktree, create one with a task-specific name:

```bash
git worktree add ../worktrees/<short-task-name> -b codex/<short-task-name> main
```

When spawned into a temporary agent worktree, compare your `HEAD` with any sibling worktree whose branch name matches the task. If the sibling has newer in-progress code, treat that sibling as authoritative unless the user says otherwise.

## Validation Description

Favor narrative description of what you tested instead of bullet points.
