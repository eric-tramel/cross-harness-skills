# Cross-Harness Skills

Marketplace-style repository for portable agent skills that can be shared across Codex, Claude Code, and compatible local harnesses.

The repository is organized as a marketplace catalog with one initial plugin:

```text
.
├── .agents/plugins/marketplace.json
├── .claude-plugin/marketplace.json
└── plugins/
    └── cross-harness-skills/
        ├── .codex-plugin/plugin.json
        ├── .claude-plugin/plugin.json
        └── skills/
```

## Skill Inventory

- `start-work`: start repository work with branch/worktree isolation and local guidance checks.
- `crystallize`: turn rough work into an implementation-ready local plan.
- `code-review`: coordinate delegated multi-persona review.
- `code-review-elegance`: review for minimal design and simplification.
- `code-review-idomatic`: review for language and repository idioms.
- `code-review-correctness`: review for bugs, regressions, and edge cases.
- `code-review-completeness`: review against stated objectives.
- `code-review-security-review`: review for security risk.
- `code-review-yagni`: review for overengineering.
- `code-review-scope`: review for scope control.
- `author-pr`: create ready-for-review GitHub pull requests.
- `release`: cut and verify repository releases.
- `cross-harness-skill-authoring`: create or adapt portable skills.

## Install

This marketplace is hosted in the public GitHub repository
`eric-tramel/cross-harness-skills`, so no private repository access is required
to install it.

### Claude Code

Install the marketplace and then install the plugin from that marketplace:

```bash
claude plugin marketplace add eric-tramel/cross-harness-skills
claude plugin install cross-harness-skills@eric-cross-harness-skills
```

Verify the install:

```bash
claude plugin list
claude plugin details cross-harness-skills
```

Update later:

```bash
claude plugin marketplace update eric-cross-harness-skills
claude plugin update cross-harness-skills
```

For local testing from a checkout of this repository:

```bash
claude plugin marketplace add .
claude plugin install cross-harness-skills@eric-cross-harness-skills
```

### Codex

This repo includes a Codex marketplace manifest at `.agents/plugins/marketplace.json` and a plugin manifest at `plugins/cross-harness-skills/.codex-plugin/plugin.json`.

Add the marketplace from GitHub:

```bash
codex plugin marketplace add eric-tramel/cross-harness-skills
```

After adding the marketplace, open the Codex plugin UI, find `Cross Harness Skills` in the `Eric Cross-Harness Skills` marketplace, and install or enable `cross-harness-skills`.

Update later:

```bash
codex plugin marketplace upgrade eric-cross-harness-skills
```

For local testing from a checkout of this repository:

```bash
codex plugin marketplace add .
```

Then install or enable `cross-harness-skills` from the Codex plugin UI. Start a new Codex thread after installing or updating so the refreshed skill metadata is loaded.

## Add A Skill

Create a new skill directory under `plugins/cross-harness-skills/skills/`:

```text
plugins/cross-harness-skills/skills/<skill-name>/SKILL.md
```

Use lowercase kebab-case for `<skill-name>`. Keep the core workflow portable, and put harness-specific details in clearly labeled sections or companion reference files.

## Validate

Codex plugin validation:

```bash
python3 /Users/eric/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py plugins/cross-harness-skills
```

Claude Code validation, when the CLI is available:

```bash
claude plugin validate .
claude plugin validate plugins/cross-harness-skills
```

## Release Notes

When publishing a release, update the `version` field in both plugin manifests:

- `plugins/cross-harness-skills/.codex-plugin/plugin.json`
- `plugins/cross-harness-skills/.claude-plugin/plugin.json`
