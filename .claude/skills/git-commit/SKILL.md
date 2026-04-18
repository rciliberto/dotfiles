---
name: git-commit
description: Execute git commit with conventional commit message analysis, intelligent staging, and message generation. Use when user asks to commit changes, create a git commit, stage files, or mentions "/commit". Automatically integrates with OneDev issue tracking via tod MCP when available. Supports auto-detecting type and scope from diff, generating conventional commit messages, and interactive overrides.
license: MIT
allowed-tools: |
  Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git branch:*),
  mcp__tod__getCurrentProject, mcp__tod__getIssue, mcp__tod__queryIssues
---

# Git Commit with Conventional Commits

Create standardized, semantic git commits using the [Conventional Commits](https://www.conventionalcommits.org/) specification. Analyze the actual diff to determine appropriate type, scope, and message.

Optionally integrates with OneDev issue tracking when `tod` is available.

## Commit Format

```
<type>[optional scope]: <description> [(KEY-N)]

[optional body]

[optional footer(s)]
```

## Commit Types

| Type       | Purpose                        |
| ---------- | ------------------------------ |
| `feat`     | New feature                    |
| `fix`      | Bug fix                        |
| `docs`     | Documentation only             |
| `style`    | Formatting/style (no logic)    |
| `refactor` | Code refactor (no feature/fix) |
| `perf`     | Performance improvement        |
| `test`     | Add/update tests               |
| `build`    | Build system/dependencies      |
| `ci`       | CI/config changes              |
| `chore`    | Maintenance/misc               |
| `revert`   | Revert commit                  |

## Breaking Changes

```
feat!: remove deprecated endpoint

# or via footer:
feat: allow config to extend other configs

BREAKING CHANGE: `extends` key behavior changed
```

---

## Workflow

### 1. Check for OneDev (optional)

Try `tod:getCurrentProject`. If it succeeds, follow the **OneDev path** below.
If it fails or is unavailable, skip to step 2 and proceed without issue tracking.

#### OneDev Path

```bash
git branch --show-current
```

Parse the branch name for a project-key pattern like `LB-123` or `INFRA-42`
(format: `type/KEY-123-some-slug`). Extract the full issue reference (e.g. `LB-123`).

- **If found** → call `tod:getIssue` to fetch the issue title and description. Use
  these to inform the commit message. Tell the user which issue was found.
- **If not found** → call `tod:queryIssues` to list open issues and let the user
  pick one, or confirm they want to proceed without an issue reference.

The issue reference goes at the **end of the subject line** in parentheses using the
full project key format — e.g. `feat(auth): add OAuth login (LB-123)`. Do not put
the issue reference in the scope slot.

#### Fix Footer — Branch Context

Check the current branch to decide whether to offer a fix footer:

**On a feature branch** (anything other than `main`, `master`, or `release/*`):
- Do NOT add a fix footer by default — the `(KEY-N)` subject reference is sufficient
  for cross-referencing individual commits.
- If the user explicitly says this is their final/closing commit, offer the footer.

**On `main`, `master`, `release/*`, or when the user indicates this is a merge/closing commit:**
- Ask: **"Does this commit close issue KEY-N?"**
- **Yes** → append a fix footer using the most semantically appropriate keyword from:
  `fix`, `fixed`, `fixes`, `fixing`, `resolve`, `resolved`, `resolves`, `resolving`,
  `close`, `closed`, `closes`, `closing` — followed by `KEY-N` (e.g. `Fixes LB-123`,
  `Resolves INFRA-42`). OneDev will link the commit to the issue's Fixing Commits tab
  and can trigger configured state transition rules.
- **No** → omit the footer.

---

### 2. Analyze Diff

```bash
git diff --staged       # prefer staged changes
git diff                # fallback if nothing staged
git status --porcelain  # get overall picture
```

Review the diff for **atomicity** — each commit should represent one logical change.
If the diff contains multiple unrelated changes (e.g. a bug fix alongside a refactor,
or changes across unrelated modules), stop and tell the user:

> "I can see multiple independent changes here — it's best to split these into
> separate commits. I'd suggest: [list the logical groupings]. Want me to stage and
> commit them one at a time?"

If the user agrees, work through each group sequentially — stage the relevant files,
commit, then move to the next. If the user wants to proceed as one commit anyway,
respect that and continue.

### 3. Stage Files (if needed)

```bash
git add path/to/file
git add *.test.*
```

**Never commit secrets** (.env, credentials, private keys).

### 4. Generate Commit Message

From the diff (and issue details if available), determine:
- **Type**: What kind of change?
- **Scope**: What area/module is affected? (optional — never use issue number here)
- **Description**: Imperative, present tense, under 72 chars
- **Issue ref**: `(KEY-N)` at end of subject (e.g. `LB-123`), if applicable

### 5. Execute Commit

```bash
# No issue reference
git commit -m "feat(scope): description"

# With issue reference only
git commit -m "feat(scope): description (KEY-N)"

# With issue reference + fix footer
git commit -m "$(cat <<'EOF'
feat(scope): description (KEY-N)

Fixes KEY-N
EOF
)"
```

---

## Best Practices

- One logical change per commit
- Present tense: "add" not "added"
- Imperative mood: "fix bug" not "fixes bug"
- Keep description under 72 characters

## Git Safety Protocol

- NEVER update git config
- NEVER run destructive commands (--force, hard reset) without explicit request
- NEVER skip hooks (--no-verify) unless user asks
- NEVER force push to main/master
- If commit fails due to hooks, fix the issue and create a NEW commit (don't amend)
