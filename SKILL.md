---
name: markdown-link-check
description: Audit Markdown files for unlinked references (Jira tickets, GitHub repos/PRs/commits, file paths) and enforce reference-style link definitions at the bottom. Invoke when committing Markdown files.
---

# Markdown Link Check

Audit staged Markdown files for linkable items that are missing hyperlinks,
and verify that existing links use reference style with definitions at the
bottom of the document.

## When to invoke

Run this skill whenever a commit includes `.md` files. It is a pre-commit
quality gate, not a blocker — report findings and ask the user whether to fix
before committing.

## Detection: what counts as "should be linked"

Scan each Markdown file for the following patterns. Each is linkable and
should use a reference-style link `[text][ref]` or `[text]` (auto-ref).

### Jira tickets

Pattern: `[A-Z]{2,10}-\d+` that appears as plain text (not already inside
`[...]` or `[...][...]` or `[...](...)` syntax).

Common prefixes in this environment: `QQ`, `DISC`, `PAAS`, `NET`, `CHIM`.

Link target: `https://cisco-sbg.atlassian.net/browse/<TICKET>`

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

## Enforcement: reference-style links

All links in the audited file **must** use reference style:

```markdown
<!-- inline text -->
See [QQ-12357] for details.

<!-- at the bottom of the file, after a --- separator -->
[QQ-12357]: https://cisco-sbg.atlassian.net/browse/QQ-12357
```

Inline links (`[text](url)`) should be converted to reference style.

## Workflow

1. Identify staged `.md` files:
   ```bash
   git diff --cached --name-only --diff-filter=ACM | grep '\.md$'
   ```
   If none, exit silently.

2. For each file, read it and run the detection pass.

3. Build two lists:
   - **Missing links** — linkable items in plain text with no surrounding link syntax.
   - **Inline links** — `[text](url)` forms that should be reference style.

4. If both lists are empty, report "All linkable references are linked ✓" and exit.

5. Otherwise, present the findings grouped by file:
   ```
   runbooks/foo/README.md
     Missing links:
       - QQ-12357 (line 4)  → https://cisco-sbg.atlassian.net/browse/QQ-12357
       - NET-11137 (line 59) → https://cisco-sbg.atlassian.net/browse/NET-11137
     Inline links to convert:
       - [cloudsec_quadra_base-host#2690](https://github.com/...) (line 3)
   ```

6. Ask: **Fix these now before committing? (yes / no / skip)**
   - **yes** — apply all fixes: convert inline links to reference style, add
     missing reference-style links, append definition block after `---`
     separator at end of file. Then re-run detection to confirm clean.
   - **no / skip** — proceed to commit without fixing. Note findings in commit
     message or as a follow-up comment.

## Reference definition block format

Append definitions at the end of the file after a `---` rule, sorted
alphabetically by ref name:

```markdown
---

[DISC-1234]: https://cisco-sbg.atlassian.net/browse/DISC-1234
[bh-pr-2690]: https://github.com/cisco-sbg/cloudsec_quadra_base-host/pull/2690
[dc_config]: https://github.com/cisco-sbg/cloudsec_quadra_base-host/tree/master/dc_config
```

If a `---` already exists at the end, append definitions after it. Do not
add a second separator.

## Guardrails

- Never modify files without the user confirming "yes".
- Do not link every occurrence of a ticket — link on first mention per section,
  or wherever the prose benefits most from clickability. Do not spam `[QQ-12357]`
  on every line.
- Do not invent URLs. If the target cannot be determined from context, flag it
  as "URL unknown — please provide" rather than guessing.
- Do not reformat surrounding Markdown beyond the link changes.
- This skill is advisory for non-cisco repos — patterns and Jira base URL may
  differ. Adapt accordingly or ask the user.
