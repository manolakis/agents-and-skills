---
name: changesets
description: Create user-facing changelog entries using changesets for versioned releases
metadata:
  version: 1.0.0
  auto-invoke:
    - Creating a commit with user-facing changes
    - Adding a changeset file
    - Documenting feature or fix for release
  references:
    - https://keepachangelog.com/en/1.1.0/
    - https://github.com/changesets/changesets
---

# Critical rules

- ALWAYS check if the project uses changesets before applying this skill
- ALWAYS create a changeset when commit type is `feat`, `fix`, or user-facing `refactor`
- ALWAYS write changeset messages for end users, NOT for developers
- ALWAYS use Keep a Changelog categories: Added, Changed, Deprecated, Removed, Fixed, Security
- ALWAYS describe WHAT changed from user perspective, not HOW it was implemented
- ALWAYS use present tense or past tense consistently (prefer past tense)
- ALWAYS keep messages concise and clear (1-3 lines per change)
- NEVER include technical implementation details in changeset messages
- NEVER duplicate commit message in changeset (they serve different audiences)
- NEVER create changeset for internal-only changes (docs, tests, chore, refactor without user impact)
- NEVER use jargon or technical terms unfamiliar to end users
- NEVER skip impact level selection (major, minor, patch)

---

# What are changesets?

Changesets are a way to manage versioning and changelogs for packages and projects. Each changeset is a small file describing a change that will appear in the changelog when the package is released.

## Key differences: Commit vs Changeset

| Aspect | Commit Message | Changeset Message |
|--------|---------------|-------------------|
| **Audience** | Developers, code reviewers | End users, product consumers |
| **Focus** | What changed in the code | What users will experience |
| **Detail** | Can be technical | Must be user-friendly |
| **Purpose** | Code history tracking | Release notes, changelog |
| **Example** | `refactor: extract retry logic into separate class` | `Fixed connection reliability issues` |

---

# Changeset structure

## File format

Changesets are markdown files in `.changeset/` directory with this structure:

```markdown
---
"package-name": patch
---

Brief description of the change from user perspective
```

## Impact levels (semver)

| Level | When to use | Examples |
|-------|-------------|----------|
| `major` | Breaking changes that affect existing usage | Removed deprecated API, Changed parameter order, Renamed core methods |
| `minor` | New features, enhancements (backward compatible) | Added new endpoint, New configuration option, Enhanced performance |
| `patch` | Bug fixes, minor improvements (backward compatible) | Fixed validation error, Corrected calculation, Improved error message |

---

# Changeset categories (Keep a Changelog)

Use these categories to classify changes for users:

| Category | Commit Types | User Impact | Example |
|----------|-------------|-------------|---------|
| **Added** | `feat` | New features or capabilities | Added retry mechanism for failed operations |
| **Changed** | `feat`, `refactor`, `perf` | Modified existing behavior | Improved dashboard loading speed |
| **Deprecated** | `feat` | Features marked for future removal | Authentication v1 API will be removed in next major version |
| **Removed** | `feat`, `refactor` | Features that no longer exist | Removed legacy export format |
| **Fixed** | `fix` | Bug fixes and corrections | Fixed incorrect totals in reports |
| **Security** | `fix`, `feat` | Security-related fixes or improvements | Resolved authentication bypass vulnerability |

---

# Decision tree: Should I create a changeset?

```
Does the project use changesets?
├─ No  -> STOP (this skill doesn't apply)
└─ Yes -> Continue

Does the change affect end users?
├─ No  -> Examples:
│         • Internal refactoring with no behavior change
│         • Test additions/updates
│         • Documentation updates
│         • Build script changes
│         • Code style changes
│         → SKIP changeset
│
└─ Yes -> Continue

What is the commit type?
├─ feat     -> CREATE changeset (likely minor or major)
├─ fix      -> CREATE changeset (patch)
├─ refactor -> Does it change user-visible behavior?
│              ├─ Yes -> CREATE changeset (patch or minor)
│              └─ No  -> SKIP changeset
├─ perf     -> Is improvement noticeable to users?
│              ├─ Yes -> CREATE changeset (patch or minor)
│              └─ No  -> SKIP changeset
└─ docs, test, chore, style, ci, build
              -> SKIP changeset (internal only)
```

---

# Workflow: Creating a changeset

## 1. Detect if project uses changesets

Check for these indicators:
```bash
# Check for .changeset directory
ls -la .changeset/

# Check for changesets in package.json
grep "changesets" package.json

# Check for changeset config
cat .changeset/config.json
```

If none exist → Skip this skill, project doesn't use changesets

## 2. Analyze the change for user impact

Ask yourself:
- Does this change what users can do?
- Does this fix something users experienced?
- Does this improve something users notice?
- Will users need to know about this?

If all answers are "No" → Skip changeset

## 3. Determine impact level (semver)

```
Breaking change (incompatible with previous version)?
└─ Yes -> major

New feature or enhancement (backward compatible)?
└─ Yes -> minor

Bug fix or minor improvement (backward compatible)?
└─ Yes -> patch
```

## 4. Select category from Keep a Changelog

Choose the most appropriate:
- **Added**: New capability
- **Changed**: Modified behavior
- **Deprecated**: Soon-to-be-removed feature
- **Removed**: Eliminated feature
- **Fixed**: Bug correction
- **Security**: Security-related

## 5. Write user-facing message

**Good practices:**
- Start with action verb (past tense): "Added", "Fixed", "Improved"
- Be specific but concise
- Focus on user benefit or issue resolved
- Avoid technical jargon
- Use common terminology

**Bad practices:**
- Don't mention file names, classes, or methods
- Don't include implementation details
- Don't use developer-only terms
- Don't be vague ("various improvements")

## 6. Create changeset file

Use changesets CLI - it guides you through the process:
```bash
npx changeset add
```

The CLI will:
1. Ask which packages are affected (e.g., `@tinyfwk/core`)
2. Prompt for impact level: `major`, `minor`, or `patch`
3. Request your description for end users
4. Create the file automatically with unique naming

This is the recommended approach as it ensures proper format and naming conventions.

---

# Examples: Good vs Bad

## Example 1: New Feature

### ❌ Bad (developer-focused)
```markdown
---
"@tinyfwk/core": minor
---

Added InMemoryCommandBus.java implementing CommandBus interface with
handler registration and middleware pipeline execution
```

### ✅ Good (user-focused)
```markdown
---
"@tinyfwk/core": minor
---

Added command bus with middleware support for extensible request processing
```

---

## Example 2: Bug Fix

### ❌ Bad (too technical)
```markdown
---
"@tinyfwk/api": patch
---

Fixed NullPointerException in QueryHandlerExecutor.java line 45 by adding
null check before handler.handle() invocation
```

### ✅ Good (user impact)
```markdown
---
"@tinyfwk/api": patch
---

Fixed crash when processing queries without registered handlers
```

---

## Example 3: Performance Improvement

### ❌ Bad (implementation detail)
```markdown
---
"@tinyfwk/core": patch
---

Refactored Pipeline.java to use ArrayList instead of LinkedList reducing
iteration overhead by 15ms on average
```

### ✅ Good (user benefit)
```markdown
---
"@tinyfwk/core": patch
---

Improved middleware pipeline execution speed
```

---

## Example 4: Internal Refactor (No Changeset)

### ❌ Wrong (no user impact)
```markdown
---
"@tinyfwk/core": patch
---

Extracted retry logic into separate RetryHandler class for better maintainability
```

### ✅ Correct approach
```
No changeset needed - this is internal refactoring with no user-visible changes
```

---

## Example 5: Breaking Change

### ❌ Bad (unclear impact)
```markdown
---
"@tinyfwk/api": major
---

Changed CommandBus interface signature
```

### ✅ Good (clear migration path)
```markdown
---
"@tinyfwk/api": major
---

Changed command handler registration to require explicit error handling.
Existing handlers must implement error handling or wrap with default handler.
```

---

# Multiple changes in one changeset

When multiple related changes go together:

```markdown
---
"@tinyfwk/core": minor
---

- Added support for async middleware execution
- Improved error messages in command validation
- Fixed memory leak in query handler cache
```

Use bullet points for clarity. Each item should still be user-focused.

---

# Cross-package changesets

For monorepos affecting multiple packages:

```markdown
---
"@tinyfwk/core": minor
"@tinyfwk/api": minor
---

Added distributed tracing support across all packages
```

---

# Validation checklist

Before finalizing a changeset, verify:

- [ ] Project uses changesets (`.changeset/` directory exists)
- [ ] Change has user-facing impact
- [ ] Commit type is `feat`, `fix`, or user-impacting `refactor`/`perf`
- [ ] Impact level (major/minor/patch) is correct
- [ ] Message is written for end users, not developers
- [ ] Message is clear and specific
- [ ] No technical jargon or implementation details
- [ ] File is in `.changeset/` directory
- [ ] File follows the correct format

---

# Integration with commit workflow

When creating a commit that needs a changeset:

```
1. Write code changes
   ↓
2. Stage files (git add)
   ↓
3. Create changeset (npx changeset add OR manual file)
   ↓
4. Stage changeset file (git add .changeset/*)
   ↓
5. Create commit with conventional commit message
   ↓
6. Both commit and changeset are included in same commit
```

**Important**: The changeset file should be committed alongside the code changes.

---

# Common pitfalls

## 1. Writing for developers instead of users

❌ "Refactored UserService to use dependency injection"
✅ "Improved application startup time"

## 2. Being too vague

❌ "Various improvements and fixes"
✅ "Fixed timeout errors when saving large documents"

## 3. Including too much technical detail

❌ "Modified SQL query in UserRepository.findByEmail() to add index hint"
✅ "Improved user search performance"

## 4. Creating changeset for internal changes

❌ Creating changeset for test additions
✅ Skip changeset for test-only changes

## 5. Wrong impact level

❌ Marking breaking change as `patch`
✅ Breaking changes MUST be `major`

## 6. Forgetting to stage changeset

❌ Commit code without changeset file
✅ Always include `.changeset/*.md` in the commit

---

# FAQ

## Q: Do I need a changeset for every commit?

**A**: No. Only commits with user-facing changes need changesets. Typically `feat`, `fix`, and some `refactor`/`perf` commits.

## Q: What if my commit has both new feature and bug fix?

**A**: Create one changeset with multiple bullet points, or create two separate changesets. Choose based on clarity for users.

## Q: Should refactoring always have a changeset?

**A**: Only if the refactoring changes user-visible behavior or performance. Pure internal refactoring = no changeset.

## Q: How do I name changeset files?

**A**: Use the changesets CLI (`npx changeset add`) which generates unique names automatically. Manual naming: use timestamp or random string to avoid conflicts.

## Q: Can I edit a changeset after committing?

**A**: Yes, but only before the package is released. After release, changesets are consumed into CHANGELOG.md.

## Q: What about documentation changes?

**A**: If documentation change helps users understand a feature better, consider a changeset categorized as "Changed" or "Added". Pure internal docs = no changeset.

---

# Related skills

- **conventional-commits**: Defines commit message structure (developer audience)
- **code-quality**: Ensures code changes meet quality standards
- **testing**: Verifies changes work correctly

---

# Summary

**Remember:**
1. Changesets are for **users**, commits are for **developers**
2. Only create changesets for **user-facing changes**
3. Follow **Keep a Changelog** categories and principles
4. Choose correct **impact level** (major/minor/patch)
5. Write **clear, jargon-free** messages
6. **Commit changeset file** with your code changes

**When in doubt**: Ask "Would a user care about this change?" If yes, create changeset. If no, skip it.
