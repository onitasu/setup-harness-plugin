# Setup Harness Plugin

Set up a project-specific Claude Code harness with agents, skills, rules, settings, checkpoints, and Codex double-check workflows.

This repository is both:

- a Claude Code/Codex plugin root
- a marketplace root for installing the `setup-harness` plugin from GitHub

## Install

Claude Code:

```bash
claude plugin marketplace add onitasu/setup-harness-plugin
claude plugin install setup-harness@setup-harness
```

Codex:

```bash
codex plugin marketplace add onitasu/setup-harness-plugin
```

After adding the marketplace, enable or install `setup-harness` from the Codex plugin UI.

## Usage

Claude Code:

```text
/setup-harness
```

Codex:

```text
$setup-harness Set up a coding-agent harness for this repository.
```

## Components

- `skills/setup-harness/SKILL.md`: primary plugin skill entrypoint.
- `skills/setup-harness/references/setup-harness-workflow.md`: full harness setup workflow.
- `.claude-plugin/plugin.json`: Claude Code plugin manifest.
- `.codex-plugin/plugin.json`: Codex plugin manifest.
- `.claude-plugin/marketplace.json`: marketplace manifest for GitHub install.

## Development

Validate the plugin:

```bash
claude plugin validate --strict .
python3 ~/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

## License

MIT
