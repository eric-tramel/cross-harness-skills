---
name: cross-harness-skill-authoring
description: Use when creating, reviewing, or adapting an agent skill that should remain portable across Codex, Claude Code, and other compatible harnesses.
---

# Cross-Harness Skill Authoring

Use this skill when a skill should travel between agent harnesses without depending on one runtime's private assumptions.

## Workflow

1. Treat `SKILL.md` as the canonical portable behavior.
2. Keep the skill directory name stable, lowercase, and kebab-case.
3. Put harness-specific launch, install, validation, or tool notes in separate sections instead of mixing them into the core workflow.
4. Use paths relative to the plugin or skill directory. Installed plugins are commonly copied into cache directories, so references outside the plugin may break after installation.
5. Prefer instructions that describe intent, inputs, outputs, and verification over instructions tied to one chat UI or model provider.
6. When a harness requires metadata, add only the fields that harness accepts and keep the meaning aligned with the canonical skill.

## Compatibility Checklist

- The skill lives at `skills/<skill-name>/SKILL.md`.
- Frontmatter includes a concise `description`.
- Optional frontmatter fields are additive and do not change the skill's meaning for another harness.
- External assets, scripts, and references live inside the plugin directory.
- Codex metadata points `skills` at `./skills/` in `.codex-plugin/plugin.json`.
- Claude Code metadata points `skills` at `./skills/` in `.claude-plugin/plugin.json`.
- Marketplace entries point at the plugin directory with a relative `./plugins/<plugin-name>` source.

## Validation

For Codex packaging, run the Codex plugin validator against the plugin directory.

For Claude Code packaging, run `claude plugin validate .` at the marketplace root and against the plugin directory when the Claude CLI is available.
