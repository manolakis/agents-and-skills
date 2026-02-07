---
name: conventional-commits
description: Create professional git commits following conventional commits format
metadata:
  version: 1.0.0
  auto-invoke:
    - Creating a git commit
    - Committing changes
---

# Critical rules

- ALWAYS use conventional commits format: `type(scope): description`
- ALWAYS keep the first line under 72 characters
- ALWAYS run `git diff --staged` as the FIRST step when committing
- ALWAYS auto-stage all modified files if no files are currently staged
- ALWAYS ask for user confirmation before committing
- ALWAYS analyze ONLY staged files when drafting commit message
- ALWAYS validate commit message against staged changes before presenting
- ALWAYS use `chore(skills)` for changes to agents and skills
- ALWAYS reserve `feat` and `fix` for user-facing product changes only
- NEVER call get_changed_files or other tools before running `git diff --staged`
- NEVER draft message based on get_changed_files output - it includes unstaged changes
- NEVER stage files manually except when auto-staging due to empty staging area
- NEVER include file names, line numbers or implementation details in the commit message
- NEVER include implementation details in the commit description (title)
- NEVER be overly specific (avoid counts like "6 subsections", "3 files", etc.)
- NEVER use `-n` flag unless user explicitly requests it
- NEVER use `git push --force` or `git push -f` (destructive, rewrites history)
- NEVER proactively offer to commit, wait for user to explicitly request it
- NEVER add unrelated changes to the same commit

---

# Commits

## Commit format

```
type(scope): description

- key change 1
- key change 2
- ...
```

## Commit types

| Type | When |
|------|----------|
| `feat` | **User-facing** new features (visible to end users) |
| `fix` | **User-facing** bug fixes (impacts end users) |
| `docs` | Documentation changes only (guides, READMEs, user documentation) |
| `style` | Code style changes without affecting functionality (formatting, missing semicolons, etc.) |
| `refactor` | Code refactoring without changing functionality |
| `test` | Adding or updating tests |
| `perf` | Performance improvements visible to users |
| `ci` | Continuous integration and CI/CD changes (GitHub Actions, workflows) |
| `chore` | Maintenance, tooling, build scripts, skills, agents (internal) |
| `build` | Build system configuration |

## Scopes

| Scope | When | Type to use |
|-------|------|-------------|
| `skills` | Changes in `.github/skills/` or `.github/agents/` | Always `chore(skills)` |
| *none* | Changes in CI/CD workflows and GitHub Actions | Use `ci:` (no scope) |

| *none* | Changes in `docs/` directory | Use `docs:` (no scope) |

**Key distinction:**
- `feat(scope)`: New features visible to end users
- `fix(scope)`: Bug fixes impacting end users
- `chore(skills)`: Changes to agents/skills (internal tooling)
- `ci:` - Changes to CI/CD (no scope needed)
- `docs`: Changes to user documentation
- Never use `feat` or `fix` for internal/tooling changes

---

# Good vs Bad commit examples

## Title line

```
# GOOD - Concise and clear
feat(api): add provider connection retry logic
fix(validation): resolve dashboard loading state
chore(skills): add new changeset skill for releases
docs: update installation guide
ci: add GitHub workflow for automated testing

# BAD - too specific or verbose
feat: add provider connection retry logic with exponential backoff and jitter (3 retries max)
fix: fix the bug in dashboard component on line 45
feat(skills): add new changeset skill for releases  ← WRONG: skills = chore, not feat
docs: update installation guide covering the use of devcontainers cloning in a named volume
```

## Body (bullet points)

```
# GOOD - High level changes
- Add retry mechanism for failed connections
- Document task composition patterns
- Expand configuration reference

# BAD - too detailed
- Add retry mechanism with max_retries=3, backoff=True, jitter=True
- Add 6 subsections covering chain, group, chord
- Update lines 45-67 in Dashboard.java
```

---

# Workflow

1. Check staged files
  ```bash
  git diff --staged
  ```
  - If no files are staged, auto-stage all modified files:
    ```bash
    git add -A
    ```
  - Then re-check staged files

2. Draft commit message
  - Choose appropriate type and scope analyzing staged files
  - Write concise title (<72 characters)
  - Add 2-5 bullet points for significant changes if required (see decision tree)

3. Validate commit message against staged changes
  - Verify every change mentioned in the message exists in staged files
  - Verify no staged files are missing from the message
  - If mismatch or doubt exists, redraft from step 2

4. Present to user for confirmation
  - Show files to be committed
  - Show processed message
  - Wait for explicit confirmation

5. Execute commit
  ```bash
  git commit -m "$(cat <<'EOF'
  type(scope): description

  - Change 1
  - Change 2
  EOF
  )"
  ```

---

# Decision trees

## Choose commit type

```
Does this change affect end users (functionality, UI, performance)?
├─ No  -> Continue to internal changes section
└─ Yes -> Continue to next question

Is this a new feature users will see?
├─ Yes -> feat(scope): description
└─ No  -> Continue

Is this a bug fix users will experience?
├─ Yes -> fix(scope): description
└─ No  -> Continue

=== FOR INTERNAL CHANGES (NOT USER-FACING) ===

Changes in .github/skills/ or .github/agents/?
└─ YES -> chore(skills): description

Changes in CI/GitHub workflows or GitHub Actions?
└─ YES -> ci: description (no scope needed)

Changes in user documentation (docs/ directory)?
└─ YES -> docs: description (no scope needed)

Changes in tests?
└─ YES -> test(scope): description

Code refactoring (no behavior change)?
└─ YES -> refactor(scope): description

Performance improvements (internal)?
└─ YES -> perf(scope): description

Dependency updates, build config, other maintenance?
└─ YES -> chore(scope): description
```

## Format the message

```
Single file changed?
├─ Yes -> May omit body, title only
└─ No  -> Include body with key changes

Multiple scopes affected?
├─ Yes -> Omit scope: `feat: description`
└─ No  -> Include scope: `feat(scope): description`
```

---

# Commands

```bash
# Check the changes to commit
git diff --stat HEAD

# Single line commit
git commit -m "type(scope): description"

# Multi-line commit
git commit -m "$(cat <<'EOF'
type(scope): description

- Change 1
- Change 2
- ...
EOF
)"

# Amend last commit with the same message
git commit --amend --no-edit

# Amend last commit changing the message
git commit --amend -m "new message"
```
