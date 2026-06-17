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

## Install

### Claude Code

```bash
claude plugin marketplace add eric-tramel/cross-harness-skills
claude plugin install cross-harness-skills@eric-cross-harness-skills
```

For local testing from this checkout:

```bash
claude plugin marketplace add .
claude plugin install cross-harness-skills@eric-cross-harness-skills
```

### Codex

This repo includes a Codex marketplace manifest at `.agents/plugins/marketplace.json` and a plugin manifest at `plugins/cross-harness-skills/.codex-plugin/plugin.json`.

For local testing from this checkout:

```bash
codex plugin marketplace add .
```

For remote install:

```bash
codex plugin marketplace add eric-tramel/cross-harness-skills
```

After adding the marketplace, install or enable `cross-harness-skills` from the Codex plugin UI.

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
