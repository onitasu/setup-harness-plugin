---
name: setup-harness
description: Set up a project-specific Claude Code harness. Use when the user asks to initialize, design, or improve a coding-agent harness with CLAUDE.md, agents, skills, rules, settings, checkpoints, and Codex double-check workflows.
---

# Setup Harness

Use this skill to build a project-specific Claude Code harness.

The canonical workflow for this plugin is stored in `../../commands/setup-harness.md`.
Before acting, read that file and follow it as the source of truth.

## Invocation Notes

- For Claude Code users, this plugin also exposes the legacy slash command `/setup-harness`.
- For Codex users, treat this skill as the entrypoint and adapt Claude-specific files into Codex equivalents only when the user asks for Codex harness support.
- Do not skip the discovery and design phases unless the user explicitly asks for a narrowly scoped update to an existing harness.

## Output Expectations

When implementing a harness, produce concrete files and validation steps, not only a proposal.
When only reviewing a harness, return prioritized findings and recommended file changes.
