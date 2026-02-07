---
name: security-review
description: Identify security weaknesses in code and dependencies using OWASP Top Ten as reference
metadata:
  version: 1.0.0
  auto-invoke:
    - Security review
    - Security audit
    - Checking security issues
    - Checking dependencies
    - OWASP top ten
---

# Critical rules

- ALWAYS use British English in code, comments, and documentation
- ALWAYS prefer evidence from the codebase over assumptions
- NEVER claim a dependency is safe without checking the lockfile or build manifest
- NEVER suggest disabling security controls to pass tests
- ALWAYS treat secrets and credentials as sensitive data
- ALWAYS map findings to the OWASP Top Ten where applicable
- NEVER introduce new dependencies unless explicitly requested

---

# Workflow

## General Security Review

1. **Scope the review** - Identify the target area (module, package, or change set)
2. **Scan for secrets** - Look for hard-coded credentials, tokens, or private keys
3. **Validate input handling** - Check for validation, normalisation, and boundary checks
4. **Check authorisation** - Ensure access control is enforced at each entry point
5. **Inspect data handling** - Confirm sensitive data is masked, encrypted, or minimised
6. **Review error handling** - Avoid leaking stack traces, SQL, or system details
7. **Analyse dependency risk** - Verify build manifests and lockfiles for known issues
8. **Map to OWASP** - Categorise any issues against OWASP Top Ten
9. **Recommend fixes** - Provide concrete, minimal changes and tests

## Pre-Commit Security Check

When run before committing (auto-invoke: "Creating a git commit"):

1. **Scan staged files only** - Run `git diff --staged` to identify modified code
2. **Check for secrets** - Search for credentials, API keys, tokens in staged changes
3. **Validate no injection** - Ensure no obvious SQL, command, or path injection in staged changes
4. **Verify no access control bypass** - Confirm no critical permissions logic removed
5. **List findings** - Order by severity (critical, high, medium, low)
6. **Block or warn** - Critical findings block commit; others are warnings
7. **Report to user** - Present findings with OWASP mapping and suggested fixes

---

# OWASP Top Ten Mapping (2025)

- A01:2025 - Broken Access Control
- A02:2025 - Security Misconfiguration
- A03:2025 - Software Supply Chain Failures
- A04:2025 - Cryptographic Failures
- A05:2025 - Injection
- A06:2025 - Insecure Design
- A07:2025 - Authentication Failures
- A08:2025 - Software or Data Integrity Failures
- A09:2025 - Security Logging and Alerting Failures
- A10:2025 - Mishandling of Exceptional Conditions

---

# Checks and heuristics

## Code risks
- **Injection** - Unsafe SQL, command, path, or template building
- **Input validation** - Missing constraints or normalisation for user input
- **Authorisation** - Missing permission checks on sensitive operations
- **Crypto** - Weak algorithms, improper key storage, missing encryption
- **Secrets** - Credentials in source code, configs, logs, or tests
- **Error leakage** - Stack traces or internal details exposed to callers

## Dependency risks
- **Outdated components** - Unpinned versions or obsolete libraries
- **Known vulnerabilities** - Check against advisories where possible
- **Supply chain** - Unverified integrity or suspicious sources

---

# Decision trees

## When to flag a security finding

```
Is there a realistic attacker-controlled input?
├─ YES -> Is the input used in sensitive operations?
│         ├─ YES -> Is it validated or safely encoded?
│         ├─ NO -> Flag as finding
│         └─ YES -> Likely OK, note validation
└─ NO -> No finding unless it is a design flaw
```

## When to map to OWASP A03

```
Is there a dependency?
├─ YES -> Is the version pinned and up to date?
│         ├─ NO -> Map to A03
│         └─ YES -> Check advisories before clearing risk
└─ NO -> Not applicable
```

---

# Examples

## Good: Parameterised query

- Uses placeholders or a query builder instead of string concatenation
- Input is validated before use

## Good: Explicit access control

- Permission checks occur at the boundary for each sensitive action
- Deny by default

## Bad: Hard-coded secrets

- API keys or passwords in source control
- Secrets in logs or stack traces

## Bad: Unvalidated input

- Request fields used directly in SQL, shell, or file paths
- No length or format constraints

---

# Output expectations

- **Findings** - Ordered by severity with file and line references
- **OWASP mapping** - Each finding mapped to the most relevant category
- **Fix plan** - Minimal changes with test suggestions
