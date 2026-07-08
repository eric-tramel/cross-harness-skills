---
name: code-review-agent-audit
description: Audit and improve a code-review persona or review-coordinator skill using historical agent sessions. Use when asked to evaluate whether a review agent is useful, noisy, overreaching, missing meaningful findings, or to update a review skill or agent definition based on observed Moraine/session-history behavior.
---

# Code Review Agent Audit

## Overview

Use this skill to evaluate how a code-review agent actually behaves across past runs, judge its quality with a consistent rubric, identify soft spots in the persona or coordinator prompt, and patch the skill definition so future reviews improve.

Prefer real behavior over theoretical prompt critique. Use session-history tools when available, especially Moraine, then update the canonical source skill rather than only an installed cache copy.

## Identify The Target

1. Identify whether the target is a reviewer persona, such as `code-review-yagni`, or a coordinator skill, such as `code-review`.
2. Locate and read the target `SKILL.md` completely before judging behavior.
3. Read adjacent definitions when they affect behavior:
   - For a persona, read the review coordinator prompt that launches it.
   - For a coordinator, read each reviewer persona it delegates to.
   - Read `agents/openai.yaml` when UI/default prompt wording may need to change.
4. Prefer the authoritative source repository over plugin caches. If only a cache copy exists, say the update is ephemeral unless the user explicitly wants cache-only edits.

## Gather Historical Evidence

Use session history to find real target runs. Moraine is the preferred source when available; if the harness exposes tools with different names, use the equivalent session search and open tools. If no session-history tool is available, report that behavioral evidence is blocked and limit the task to a source-only prompt review.

Moraine search is BM25 keyword search. Start broad, then narrow:

- Exact skill ids: `code-review-yagni`, `$cross-harness-skills:code-review-yagni`, `CodeReviewYAGNI`.
- Persona labels: `YAGNI reviewer`, `Reviewer YAGNI`, `No YAGNI findings`.
- Finding shapes: `P2`, `P3`, `findings`, `follow-up on your`, `accepted and patched`, `re-check`.
- Coordinator context: `review wave`, `triage`, `accepted`, `rejected`, `deferred`, `all reviewer follow-ups`.

Classify opened records before judging them:

- `direct-review`: the target agent is performing the review.
- `follow-up`: the same target re-checks a finding.
- `coordinator-disposition`: a parent session records whether findings were accepted, rejected, deferred, or fixed.
- `irrelevant`: search hit only mentions the target but is not evidence of behavior.

Review every direct target run that is feasible to inspect. If the history is too large, use a stratified sample across time, repositories, finding/no-finding outcomes, and accepted/rejected dispositions; state the sampling limit clearly.

For each direct run, capture:

- target skill/persona and session id,
- objective or issue boundary the reviewer was given,
- findings or no-finding result,
- whether the reviewer inspected relevant source/context,
- later disposition from coordinator or follow-up sessions,
- observed effect on final code, tests, docs, or PR outcome,
- quality notes and residual uncertainty.

Do not expose secrets or long transcript excerpts. Summarize evidence and keep session ids opaque unless the user asks for raw handles.

## Rubric

Judge the target across these facets:

- Boundary grounding: Did it check the PR objective, linked issue, acceptance criteria, and repository direction before judging?
- Signal: Were findings concrete, line-referenced, and actionable?
- Meaningful impact: Did accepted findings improve final code, implementation, docs, tests, safety, or review clarity?
- False positives: Did it recommend deleting required behavior, plausible robustness, or repo-consistent patterns?
- False negatives: Did it repeatedly miss issues that were squarely within its persona and later caught elsewhere?
- Severity calibration: Were P2/P3/P4 levels proportional to blast radius and requiredness?
- Persona fit: Did it stay in its facet rather than duplicating another reviewer or drifting into unrelated review?
- No-finding quality: Did clean results explain the boundary checked instead of only saying "no findings"?
- Follow-up quality: Did it re-check narrowly and accept/revise its own position when new evidence arrived?
- Output hygiene: Were findings concise, evidence-backed, and easy for a coordinator to triage?

Use simple verdicts:

- `strong`: repeatedly catches meaningful issues or makes high-quality no-finding calls with low false-positive rate.
- `mixed`: useful signal but recurring overreach, weak evidence, noisy severity, or inconsistent boundaries.
- `weak`: mostly generic, unactionable, duplicative, or harmful relative to coordinator effort.

## Diagnose Soft Spots

Map observed behavior to prompt changes. Common patterns:

- Flags required behavior as overengineering: require reading issue/acceptance criteria before findings and add explicit non-goals.
- Treats "not in the incident" as "not needed": distinguish reported incident from plausible production states.
- Deletes plausible robustness: list live-state categories that are not YAGNI, such as partial migrations, retries, transient I/O, malformed external input, and corrupted local data.
- Emits vague no-findings: require the reviewer to name the public surface, objective, or acceptance boundary it checked.
- Produces low-severity noise: add severity thresholds and route uncertain simplifications to P4 conscious-choice notes.
- Duplicates another persona: narrow the focus section and add non-goals for neighboring facets.
- Misses public surface creep: call out config, CLI, schema, durable data, exported APIs, hooks, commands, documented workflows, and compatibility promises as expensive surfaces.
- Lacks downstream awareness: require the reviewer to state the simpler present-tense alternative and the concrete cost of keeping the extra surface.
- Makes broad fixes from narrow findings: add scope guardrails and instruct coordinators to compare broader suggestions against the scope persona.

## Update The Skill

Before editing, state the intended changes briefly. Then patch the canonical source:

- Update `SKILL.md` first; it is the portable behavior.
- Update `agents/openai.yaml` if the display name, short description, or default prompt no longer matches the revised behavior.
- Update a coordinator skill only when the review packet, triage, or follow-up workflow caused the target's recurring problem.
- Keep instructions portable across Codex, Claude Code, and compatible harnesses. Put harness-specific details in separate sections.
- Do not add scripts or references unless the workflow truly needs reusable code or long reference material.
- Do not edit unrelated reviewer personas unless their definitions directly caused the audited target's behavior.
- If installed cache copies also need to change for immediate local effect, mirror the source after validation and say which cache copies were synchronized.

Use imperative, concise wording. Preserve the target skill's triggering description while adding enough "when to use" detail for future discovery.

## Validation

Run the validation available in the repository or harness. Typical checks:

```bash
git diff --check
```

When Codex skill or plugin validator helpers are available, run the skill validator against the edited skill directory and the plugin validator against the changed plugin package.

For Claude Code packaging, run these when the CLI is available:

```bash
claude plugin validate .
claude plugin validate plugins/cross-harness-skills
```

If validation cannot run, report the reason and what was checked instead.

## Final Report

Report:

- target reviewed and number/type of sessions inspected,
- verdict and strongest evidence,
- recurring soft spots,
- skill and agent-definition changes made,
- validation results,
- any cache synchronization or residual risk.
