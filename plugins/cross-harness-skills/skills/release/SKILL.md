---
name: release
description: Cut and publish repository releases from a version argument such as /release X.Y.Z or a request to use $cross-harness-skills:release. Use when Codex is asked to run a release process, bump release-managed versions, create and merge a release PR, push a vX.Y.Z tag, write GitHub release notes for users, verify release automation, and confirm published artifacts or package registry entries. Start or continue a Codex goal for the release.
---

# Release

Run releases end to end for the current repository. A `/release X.Y.Z` invocation means publish `vX.Y.Z`, not just prepare a plan, unless the user explicitly asks only for release planning.

## Goal Contract

Make the release a Codex goal before doing release work when goal tools are available.

- If `create_goal` is available and there is no active goal, call it with: `Cut <repo> vX.Y.Z, including version bump PR, merged code, annotated repo tag, GitHub release notes, release workflow verification, and artifact or package verification.`
- If a goal already exists, continue inside it and keep it current with `update_plan`.
- Do not call `update_goal(status="complete")` until the release evidence exists: merged PR or explicitly approved direct commit, pushed tag, successful release automation when applicable, GitHub release body/assets when applicable, and package/artifact verification when applicable.

## Preconditions

Normalize the argument first:

- `X.Y.Z` and `vX.Y.Z` both mean `VERSION=X.Y.Z` and `TAG=vX.Y.Z`.
- Refuse ambiguous input, missing versions, invalid version formats for the repo, or a target older than the latest stable release.

Before editing:

1. Read active repository guidance such as `AGENTS.md`, `CLAUDE.md`, release docs, contributing docs, and workflow comments.
2. Use available session-history or agent-history tools if prior release context may matter. Start broad, then narrow with keywords such as `release process`, `tag`, `GitHub release`, package names, workflow names, and the previous version.
3. Inspect release automation under `.github/workflows/`, package manifests, changelog files, install docs, and any existing release scripts.
4. Verify tooling:
   - `gh auth status`
   - `git fetch origin --prune --tags`
   - `gh repo view --json nameWithOwner,defaultBranchRef,url`
5. Prove the target is unused:
   - `git tag --list "$TAG"` returns nothing.
   - `gh release view "$TAG"` fails with not found when the repo uses GitHub releases.
   - Registry checks for the package ecosystem fail with not found when the repo publishes packages.

## Branch And Context

Do release edits in a dedicated branch or worktree from fresh `origin/<default-branch>` unless the user explicitly says otherwise.

```bash
git fetch origin --prune --tags
git worktree add -b "codex/release-$TAG" "../worktrees/release-$TAG" origin/main
```

If the default branch is not `main`, substitute the value reported by `gh repo view`.

Gather changes since the previous stable release:

```bash
prev_tag="$(gh release list --limit 20 --json tagName,isDraft,isPrerelease -q '[.[] | select(.isDraft==false and .isPrerelease==false) | .tagName] | .[0]')"
git log --oneline --decorate "$prev_tag"..HEAD
gh pr list --state merged --base main --limit 50 --json number,title,url,mergedAt,body
```

Read PR bodies for user-visible changes. Release notes should explain what a user can do or what is fixed, not just repeat commit titles.

## Version Bump

Prefer the repository's existing release or version-bump script. If no script exists, identify release-managed files from repository conventions before editing.

Common release-managed files include:

- language package manifests such as `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, gemspecs, or plugin manifests,
- lockfiles when the package manager records local package versions,
- changelog or release notes files,
- install docs with explicit version examples,
- workflow examples that pin a release tag.

After bumping, inspect the diff carefully:

- Confirm every changed file is release-related.
- Confirm all version-bearing files agree on `VERSION`.
- Confirm generated lockfile changes are expected for the ecosystem.
- Stop if unrelated source, formatting, dependency, or generated churn appears.

## Validation

Always run the repository's cheapest high-signal checks first, then broaden according to release risk.

Baseline checks usually include:

```bash
git diff --check
```

Then run the repo's normal format, lint, type, build, and test commands from its docs or CI config. For package releases, also run the build command that produces the release artifact when feasible.

For stack-facing changes since the previous tag, run the repository's normal integration, end-to-end, smoke, or staging validation. If those checks require credentials or external systems that are not available, report the skipped check and the reason in the release PR.

## Release PR

Create a focused release commit:

```bash
git status --short
git add <release-managed-files>
git commit -m "chore(release): cut $TAG"
git push -u origin "codex/release-$TAG"
```

Open a ready-for-review PR to the default branch titled `chore(release): cut $TAG`. The PR body must include:

- what version was bumped,
- a concise user-facing summary of changes since `prev_tag`,
- validation commands and outcomes,
- operational impact: what pushing the tag or merging the PR will publish.

Wait for required checks. Merge only when checks pass or the user explicitly accepts the remaining risk. Use the repository's normal merge style, then fetch the default branch and verify the merge commit:

```bash
gh pr view <number> --json state,mergedAt,mergeCommit,url
git fetch origin <default-branch> --tags
git log --oneline -1 origin/<default-branch>
```

Stop before tagging if the PR is not merged into the default branch.

## Tag And Automation

Tag the merge commit on the default branch with an annotated tag unless the repository's release process requires a different tag form:

```bash
merge_sha="$(gh pr view <number> --json mergeCommit -q .mergeCommit.oid)"
git tag -a "$TAG" "$merge_sha" -m "$TAG"
git push origin "$TAG"
```

If a tag-triggered workflow exists, wait for it and verify the result:

```bash
gh run list --event push --branch "$TAG" --limit 5
gh run watch <run-id> --exit-status
gh run view <run-id> --json status,conclusion,url,name,event,headBranch,headSha
```

If the repository uses manual release workflows, run only the documented dispatch command and record its inputs.

## GitHub Release Notes

Edit the GitHub release body after release automation has finished uploading or generating assets, because automation can overwrite notes while assets are created.

Write for users:

- Open with what changed and why it matters.
- Group changes by user-visible area.
- Include upgrade notes only when action is required.
- Link PRs or issues the first time they are mentioned.
- Keep exactly one full changelog or compare link.
- Avoid implementation-first summaries unless the audience is maintainers.

Publish with:

```bash
gh release edit "$TAG" --notes-file /tmp/release-notes.md
```

Verify:

```bash
gh release view "$TAG" --json tagName,name,url,body,assets,publishedAt,targetCommitish
```

## Artifact And Registry Verification

Verify every public output that the release claims to publish.

Examples:

- GitHub release assets: expected filenames, checksums, and count.
- npm: package version appears in `npm view`.
- PyPI: version endpoint and simple index show the new files.
- crates.io: crate version page or API shows the new version.
- Docker: image tag exists and can be pulled or inspected.
- Homebrew or installer channels: formula or install script points at the new version.

For installable CLIs or libraries, run a smoke install in a clean temp environment when feasible and record the exact command and output.

## Stop Conditions

Stop and report clearly if:

- target tag, GitHub release, package version, or artifact already exists,
- `gh` is unavailable or unauthenticated,
- version bump diff touches unexpected files,
- validation fails,
- the release PR cannot be merged,
- the tag workflow fails,
- release assets are missing or checksums do not match,
- package registry metadata does not publish after reasonable retries.

If a tag has already been pushed, do not delete or recreate it without explicit user instruction.

## Final Report

End with concrete public evidence:

- merged release PR URL,
- tag and tagged commit,
- release workflow run URL and conclusion, if applicable,
- GitHub release URL and asset count, if applicable,
- package registry or artifact URL and file count, if applicable,
- smoke install output, if applicable,
- concise release notes summary or a link to the full notes.
