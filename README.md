# Markdown Link Check Skill

Audits staged Markdown files for unlinked Jira tickets, GitHub PR/issue/commit
references, and external file paths. Enforces reference-style link definitions
at the bottom of the document. Invoked automatically by the `git-commit` skill
when `.md` files are staged.

## Purpose

This repository contains the `markdown-link-check` agent skill. The canonical
agent instructions live in [`SKILL.md`](SKILL.md).

## Contents

- `SKILL.md`: Skill metadata and agent workflow instructions.
- `agents/openai.yaml`: OpenAI/Codex UI metadata for this skill.

## Dependencies

- MCP dependencies: None.
- Related skills: `git-commit` (invokes this skill when Markdown files are staged).

## Usage

Install or link this repository as a skill directory for an agent that supports
`SKILL.md`-based skills. Typically invoked from within the `git-commit` flow,
not directly.
