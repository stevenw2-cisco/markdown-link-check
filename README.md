# Markdown Link Check Skill

Audits changed Markdown files for unlinked Jira tickets (known project keys
only), GitHub PR/issue/commit references, and external file paths. Follows each
file's or repository's existing link style instead of forcing one, and skips
`CHANGELOG.md`. Invoked by `git-commit` when `.md` files are staged and by
`pr-create` (report only) when a PR changes Markdown.

## Purpose

This repository contains the `markdown-link-check` agent skill. The canonical
agent instructions live in [`SKILL.md`](SKILL.md).

## Contents

- `SKILL.md`: Skill metadata and agent workflow instructions.
- `agents/openai.yaml`: agent UI metadata for this skill.

## Dependencies

- MCP dependencies: None.
- Related skills: `git-commit` (invokes this skill when Markdown files are staged), `pr-create` (report-only), `cloudsec-discovery-changelog-release` (owns `CHANGELOG.md` format).

## Usage

Install or link this repository as a skill directory for an agent that supports
`SKILL.md`-based skills. Typically invoked from within the `git-commit` flow,
not directly.
