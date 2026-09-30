---
name: markdown-link-check
description: Audit changed Markdown files for unlinked references (Jira tickets with known project keys, GitHub repos/PRs/commits, external file paths) and inconsistent link style, then offer fixes that follow the file's existing style. Use when a commit or PR includes .md files (git-commit, pr-create) or the user asks to check Markdown links. Not for CHANGELOG.md (owned by cloudsec-discovery-changelog-release) or for checking whether URLs resolve.
---

# Markdown Link Check

Audit changed Markdown files for linkable items that are missing hyperlinks,
and check that links follow the file's established link style.

## When to invoke

- `git-commit`: the commit includes `.md` files.
- `pr-create`: the PR changes `.md` files (report findings only; do not edit).
- The user asks to check Markdown links.

It is a quality gate, not a blocker. Report findings and ask the user whether
to fix them.

## Exclusions

- Skip `CHANGELOG.md` (any directory). Its format, including link style, is
  owned by `cloudsec-discovery-changelog-release` in the repos that use it,
  and by the repository's changelog convention elsewhere.
- Skip generated or vendored Markdown (for example under `node_modules/`,
  `vendor/`, `.terraform/`).

## Detection: what counts as "should be linked"

Scan each Markdown file for the following patterns. Ignore matches inside
fenced code blocks, inline code, and existing link syntax (`[...]`,
`[...][...]`, `[...](...)`, `<https://...>`).

### Jira tickets

Pattern — known project keys only:

```text
\b(DISC|QQ|PAAS|NET|CHIM)-[0-9]+\b
```

Link target: `https://cisco-sbg.atlassian.net/browse/<TICKET>`

Other `[A-Z]{2,10}-\d+` tokens are often not tickets (`UTF-8`, `SHA-256`,
`ISO-8601`, `CVE-2024-1234`, `RFC-001`). Do not link them automatically. List
them separately as **Possible tickets — confirm** and ask the user which, if
any, are Jira keys.

### GitHub references

| Plain-text form | Target URL |
|---|---|
| `org/repo#NNN` (PR or issue) | `https://github.com/org/repo/pull/NNN` (PRs) or `.../issues/NNN` |
| `org/repo@SHAPREFIX` (commit, ≥7 hex chars) | `https://github.com/org/repo/commit/SHA` |
| Bare `#NNN` in a repo context | current repo PR/issue URL |

### File paths in other repos

When a prose sentence names a file or directory that lives in an external
repo (e.g. `base-host/dc_config`, `dc_config/DB_FOLLOWUPS.md`), link to the
canonical GitHub tree or blob URL if it can be inferred from context.

## Link style: follow the existing convention

Match the style the file (or, for new files, the repository) already uses:

1. If the repo documents a Markdown link style (`CONTRIBUTING.md`, a style
   guide, a markdownlint config such as MD054), follow it.
2. Otherwise, if the file consistently uses one style, keep it. That is either
   reference style (`[text][ref]` with definitions at the bottom) or inline
   (`[text](url)`). Add new links in that style and do not convert existing
   ones.
3. For a new file, use the dominant style of other Markdown files in the same
   directory or repository.
4. Only when a file mixes styles, or no convention exists, suggest reference
   style (format below). Report mixed files as **Inconsistent link style** and
   let the user choose; never convert a consistent file.

Reference style example:

```markdown
See [QQ-12357] for details.

---

[QQ-12357]: https://cisco-sbg.atlassian.net/browse/QQ-12357
```

## Workflow

1. Identify changed `.md` files, then drop the exclusions above:
   ```bash
   # commit flow
   git diff --cached --name-only --diff-filter=ACMR -- '*.md'
   # PR flow
   git diff --name-only --diff-filter=ACMR <base>...HEAD -- '*.md'
   ```
   If none remain, exit silently.

2. For each file, determine its link style (section above) and run the
   detection pass.

3. Build these lists:
   - **Missing links**: linkable items in plain text with no link syntax.
   - **Possible tickets — confirm**: unknown-prefix `XXX-123` tokens.
   - **Inconsistent link style**: only for files that mix styles.

4. If every list is empty, report "All linkable references are linked" and exit.

5. Otherwise, present the findings grouped by file:
   ```text
   runbooks/foo/README.md (style: reference)
     Missing links:
       - QQ-12357 (line 4)  → https://cisco-sbg.atlassian.net/browse/QQ-12357
       - NET-11137 (line 59) → https://cisco-sbg.atlassian.net/browse/NET-11137
     Possible tickets — confirm:
       - OPS-42 (line 12)
   ```

6. In `pr-create` validation, stop here and return the findings; do not edit.
   Otherwise ask: **Fix these now? (yes / no / skip)**
   - **yes**: add missing links in the file's style, and apply only the
     style normalisation the user chose. For reference style, add definitions
     in the block format below. Re-run detection to confirm clean.
   - **no / skip**: proceed without fixing. List the unresolved findings in
     your final response to the user. Do not put them in the commit message.

## Reference definition block format

For reference-style files, append definitions at the end of the file after a
`---` rule, sorted alphabetically by ref name:

```markdown
---

[DISC-1234]: https://cisco-sbg.atlassian.net/browse/DISC-1234
[bh-pr-2690]: https://github.com/cisco-sbg/cloudsec_quadra_base-host/pull/2690
[dc_config]: https://github.com/cisco-sbg/cloudsec_quadra_base-host/tree/master/dc_config
```

If a `---` already exists at the end, append definitions after it. Do not
add a second separator. If the file already keeps its definitions elsewhere,
add new ones there.

## Guardrails

- Never modify files without the user confirming "yes".
- Never convert a file whose link style is already consistent.
- Do not link every occurrence of a ticket. Link the first mention per
  section, or wherever the prose benefits most from clickability. Do not
  repeat `[QQ-12357]` on every line.
- Do not invent URLs or Jira keys. If the target cannot be determined from
  context, flag it as "URL unknown — please provide" rather than guessing.
- Do not reformat surrounding Markdown beyond the link changes.
- This skill is advisory for non-cisco repos. Patterns and the Jira base URL
  may differ, so adapt or ask the user.
