# Agent System - Coordination & Routing Guide

This document defines how custom agents interact with the skills system and coordinate to provide specialized analysis and recommendations.

---

## Architecture Overview

```
User Request
     ↓
Determine Intent
     ↓
Choose Custom Agent
     ↓
┌──────────────┬──────────────┬───────────────┬──────────────┬──────────────┐
↓              ↓              ↓               ↓              ↓              ↓
@code-         @architect     @security       @tester        @commit        Main
reviewer       agent          agent           agent          guide          Agent
     ↓              ↓              ↓               ↓              ↓
Load Skills → Apply Rules → Report Findings → Escalate if needed
```

---

## Custom Agent Registry

Available custom agents are located in `.github/agents/`:

| Agent | File | Command | Primary Skills | Use When |
|-------|------|---------|----------------|----------|
| Code Reviewer | `code-reviewer.agent.md` | `@code-reviewer` | code-quality, java-language | Reviewing code quality, refactoring, best practices |
| Architect | `architect.agent.md` | `@architect` | hexagonal-architecture, adr-compliance | Validating architecture, checking boundaries, ADR alignment |
| Security Analyst | `security.agent.md` | `@security` | security-review, data-privacy | Security audit, vulnerability assessment, secrets detection, PII detection, GDPR compliance, consent management, data retention, data anonymisation |
| Test Specialist | `tester.agent.md` | `@tester` | testing, java-language, hexagonal-architecture | Writing tests, test strategy, coverage analysis |
| Commit Guide | `commit-guide.agent.md` | `@commit-guide` | conventional-commits, changesets | Creating commit messages, staging strategy, changeset generation |
| Story Writer | `story-writer.agent.md` | `@story-writer` | user-story-writing | Writing user stories, acceptance criteria, and task breakdowns |

All skills are located in `.github/skills/` as `SKILL.md` files within named subdirectories.

---

## How Skills Work

### 1. Skill Structure

Each skill in `.github/skills/<skill-name>/SKILL.md` contains:

```yaml
---
name: code-quality
---

# Critical Rules
# (Non-negotiable constraints)

# Workflow
# (Step-by-step process)

# Decision Trees
# (Branching logic)

# Examples
# (Real-world scenarios)
```

### 2. Skill Application

When a custom agent is invoked:

1. **Read the complete skill file(s)** to understand all rules
2. **Follow the Critical Rules section** — these are non-negotiable
3. **Use the workflow** defined in the skill
4. **Consult decision trees** when choices need to be made
5. **Reference examples** to understand expected output quality

### 3. Skill Priority

- Skills are **authoritative** — they define expected behaviour
- Skill rules **override** general guidelines when there's a conflict
- **Critical rules within a skill are absolute** — no exceptions
- All contributors (human and AI) follow the same skills

---

## Automation Rules (CRITICAL)

The following **workflow commands trigger AUTOMATIC agent execution** - not optional, not recommended:

| Command | Automatic Workflow | Behaviour |
|---------|-------------------|----------|
| `commit` | Pattern 5A (Pre-Commit) | **MUST** automatically invoke: code-reviewer → architect → security → tester → commit-guide. Do not ask permission. Proceed automatically through sequence. Stop on critical issues and request fixes. |
| `feature complete` | Pattern 5B | **MUST** automatically invoke full validation for feature completion. |
| `ready for pr` | Pattern 5C | **MUST** automatically invoke full validation before PR creation. |
| `fix complete` | Pattern 8 | **MUST** automatically invoke focused bug fix validation. |
| `hotfix` | Pattern 11 | **MUST** automatically invoke minimal emergency validation. |

**Non-Negotiable Rules for Automation:**
1. Agent execution MUST be automatic - do not wait for permission or ask for confirmation
2. Agents MUST execute in sequence - do not skip steps
3. Critical issues MUST halt the workflow - request fixes before proceeding
4. All agents MUST complete successfully before creating any commit
5. Only the final @commit-guide step creates the actual commit

---

## Agent Invocation Patterns

### Fundamental Patterns

These patterns form the foundation of agent interaction and should be understood first.

#### Pattern 1: Single-Agent Request

Use when you know exactly what you need:

```
User: @code-reviewer review this class for quality issues

Flow:
1. Agent loads: .github/skills/code-quality/SKILL.md
2. Agent loads: .github/skills/java-language/SKILL.md
3. Agent analyses code against Critical Rules
4. Agent reports findings in skill-defined format
5. Agent may suggest: "Consider @architect review for boundary validation"
```

#### Pattern 2: Sequential Multi-Agent Review

Use before merging significant features:

```
User: Please do comprehensive review before commit

Recommended sequence:
1. @code-reviewer → Code quality check
   Loads: code-quality, java-language skills

2. @architect → Architecture validation
   Loads: hexagonal-architecture, adr-compliance skills

3. @security → Security assessment
   Loads: security-review skill

4. @tester → Test coverage review
   Loads: testing, java-language, hexagonal-architecture skills

5. @commit-guide → Generate commit message
   Loads: conventional-commits skill
```

#### Pattern 3: Targeted Issue Investigation

Use for specific concerns:

```
User: @security scan for hardcoded secrets
→ Agent applies security-review skill
→ Focused on specific vulnerability type

User: @architect verify port/adapter separation
→ Agent applies hexagonal-architecture skill
→ Focused on architectural constraint
```

#### Pattern 4: Cross-Agent Escalation

Agents recommend escalation when findings suggest it:

```
@code-reviewer detects: SQL query construction from user input
→ Escalation: "Recommend @security review for SQL injection risks"

@architect detects: Domain layer importing infrastructure
→ Escalation: "Recommend @code-reviewer for design pattern issues"

@security detects: Missing input validation in endpoint
→ Escalation: "Recommend @code-reviewer for defensive coding"

@tester detects: Code is hard to test due to tight coupling
→ Escalation: "Recommend @code-reviewer for refactoring to improve testability"

@tester detects: Missing tests for critical security validation
→ Escalation: "Recommend @security review of validation logic"
```

---

## Workflow Patterns by Lifecycle Stage

Patterns organized by development stage and typical workflow contexts.

### Pre-Merge Validation Flows

Complete quality validation before committing, pushing, or creating pull requests.

#### Pattern 5: Full Quality Validation

Use before pushing commits, finishing features, or creating PR. All agents in standard sequence.

**Variant A: Pre-Commit Check (AUTOMATIC)**

```
User: commit

AUTOMATIC execution sequence (non-negotiable):
1. @code-reviewer → Code quality check on staged changes
2. @architect → Architectural boundary validation
3. @security → Security scan of modifications
4. @tester → Test coverage verification
5. @commit-guide → Generate conventional commit message

Workflow:
- Agent calls are AUTOMATIC - do not ask permission
- Each agent loads required skills and reports findings
- CRITICAL issues halt workflow → request fixes → restart
- WARNINGS can be overridden with user confirmation
- All agents must pass → @commit-guide creates commit
- Final commit is created with generated message

Result:
- All quality gates passed → Commit created
- Critical issues found → Stop, request fixes before re-running
- Warnings found → User can confirm override or fix first
```

**Variant B: Feature Development Complete (AUTOMATIC)**

```
User: feature complete

AUTOMATIC execution sequence (non-negotiable):
1. @code-reviewer → Comprehensive quality review
2. @architect → Validate no architectural violations
3. @tester → Verify test coverage and quality
4. @security → Security audit if applicable to feature
5. @commit-guide → Create feature commit message

Workflow:
- All agents execute automatically in sequence
- Critical issues halt workflow - request fixes
- All agents must pass to proceed

Result:
- All quality gates passed → Feature commit created
- Critical issues found → Stop, request fixes
```

**Variant C: Pull Request Ready (AUTOMATIC)**

```
User: ready for pr

AUTOMATIC execution sequence (non-negotiable):
1. @code-reviewer → Final quality review
2. @architect → Final architectural validation
3. @security → Final security audit
4. @tester → Final test coverage check
5. @commit-guide → Verify commit messages

Workflow:
- All agents execute automatically in sequence
- Critical issues halt workflow - request fixes before PR
- All must pass for PR readiness confirmation

Result:
- All quality gates passed → Ready for pull request
- Critical issues found → Stop, request fixes before PR
```

---

### Specialized Domain Reviews

Reviews where a particular domain takes priority due to context sensitivity.

#### Pattern 6: Security-First Review

Use for sensitive changes (authentication, payments, PII):

```
User: security audit

Recommended sequence:
1. @security → Comprehensive security analysis
   - Vulnerability scanning
   - Input validation review
   - Data protection verification
2. @code-reviewer → Defensive coding patterns
   - Error handling
   - Input sanitization
   - Exception safety
3. @tester → Security test coverage
   - Attack vector testing
   - Edge case coverage
4. @architect → Layer isolation verification
   - No data leakage between layers
   - Security boundary enforcement

Result:
- Security-critical code thoroughly reviewed
- Defense in depth validated
- Ready for production
```

#### Pattern 7: Architecture Decision Review

Use when making significant architectural changes or ADR updates:

```
User: architecture change

Recommended sequence:
1. @architect → Primary architectural validation
   - ADR alignment
   - Design pattern correctness
   - Boundary definitions
2. @security → Security implications review
   - New attack surfaces?
   - Data flow security?
3. @code-reviewer → Implementation quality
   - Code patterns align with architecture
   - Proper separation of concerns
4. @tester → Testability assessment
   - New components testable?
   - Isolation for unit testing?

Result:
- Architectural change thoroughly reviewed
- Security and quality implications understood
- ADR documentation updated if needed
```

---

### Focused Quality Reviews

Targeted reviews without full validation, focusing on specific aspects. No final commit generation.

#### Pattern 8: Bug Fix Validation

Use for quick, focused bug fix reviews:

```
User: fix complete

Recommended sequence:
1. @code-reviewer → Verify fix quality and correctness
2. @tester → Regression testing and fix coverage
3. @security → Confirm no new vulnerabilities introduced
4. @commit-guide → Generate fix commit message

Result:
- Fix validated
- No regressions introduced
- Ready to backport if needed
```

#### Pattern 9: Refactoring Review

Use for quality improvements without new functionality:

```
User: refactor complete

Recommended sequence:
1. @code-reviewer → Code quality improvement validation
   - Better readability
   - Reduced complexity
   - Improved maintainability
2. @architect → No architectural changes verification
   - Boundaries unchanged
   - Dependencies valid
3. @tester → Test coverage preservation
   - Coverage maintained or improved
   - All tests still passing

Result:
- Code quality improved without side effects
- Ready to merge safely
```

#### Pattern 10: Test Coverage Analysis

Use to identify and address coverage gaps:

```
User: check coverage

Recommended sequence:
1. @tester → Coverage analysis
   - Identify uncovered code
   - Analyze why coverage is low
   - Recommend test strategy
2. @code-reviewer → Testability assessment
   - Is code structure preventing testing?
   - Suggest refactoring if needed
   - Defensive coding patterns missing?
3. @architect → Design assessment
   - Layer isolation preventing unit tests?
   - Tight coupling issues?
   - Architecture supporting testability?

Result:
- Clear understanding of coverage gaps
- Root cause analysis
- Path to improvement identified
```

---

### Emergency/Specialized Scenarios

Quick or context-specific reviews for non-standard situations.

#### Pattern 11: Emergency Hotfix

Use for production-critical fixes requiring minimal review time:

```
User: hotfix

Recommended sequence (minimal but essential):
1. @code-reviewer → Critical quality check
   - Verify fix correctness
   - No obvious bugs
2. @security → Security check (if applicable)
   - No new vulnerabilities
   - Safe for production
3. @commit-guide → Generate hotfix commit

Result:
- Critical fix validated
- Ready for emergency deployment
- Follow-up improvements can wait
```

#### Pattern 12: Dependency Update Review

Use when updating frameworks, libraries, or dependencies:

```
User: review dependencies

Recommended sequence:
1. @security → Vulnerability assessment
   - Known CVEs in new versions?
   - License compatibility?
2. @code-reviewer → Breaking changes analysis
   - API changes requiring code updates?
   - Deprecated features to address?
3. @tester → Regression testing
   - Integration issues?
   - Behavior changes?

Result:
- Dependencies validated
- Breaking changes understood
- Safe to merge
```

---

## Custom Agent Responsibilities

### What Agents MUST Do

✅ **Load relevant skills** on invocation:
```
Each agent knows which skills to load from .github/skills/
Code Reviewer → loads code-quality, java-language
Architect → loads hexagonal-architecture, adr-compliance
Security → loads security-review, data-privacy
Tester → loads testing, java-language, hexagonal-architecture
Commit Guide → loads conventional-commits
```

✅ **Apply Critical Rules** exactly as defined:
- Rules are non-negotiable
- No exceptions or workarounds
- Explain when a Critical Rule is violated

✅ **Follow the skill workflow**:
- Use the defined process steps
- Consult decision trees for branching
- Reference examples for quality standards

✅ **Report findings in standard format**:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (nice to have)
- Remediation advice

✅ **Suggest escalation** when needed:
- "Recommend @security review for..."
- "Recommend @architect review for..."
- "Recommend @code-reviewer review for..."
- "Recommend @tester for test coverage..."

### What Agents MUST NOT Do

❌ **Modify skill files** — they are source of truth
❌ **Ignore Critical Rules** — operate outside their authority
❌ **Skip workflow steps** — the process matters
❌ **Create custom reporting formats** — use skill-defined format
❌ **Operate outside their domain** — stay in scope

---

## Skill Usage Guidelines

### For Custom Agents

Before analyzing anything:

1. **Identify your role**: code-reviewer, architect, security, or commit-guide
2. **Load your skills**: Check which `.github/skills/` files apply to you
3. **Read completely**: Study the entire SKILL.md, not just summaries
4. **Understand Critical Rules**: These are your guardrails
5. **Follow the workflow**: Step through the defined process
6. **Use decision trees**: Make consistent choices
7. **Reference examples**: Match the quality shown in examples

### For Users

To get best results:

✅ **DO:**
- Invoke agents with `@agent-name` syntax
- Provide context and file references
- Review agent recommendations carefully
- Ask agents to explain Critical Rule violations
- Recommend escalation to other agents
- Give feedback on agent effectiveness

❌ **DON'T:**
- Try to invoke agents without `@`
- Expect agents to work outside their domain
- Ignore Critical Rule violations
- Assume agents coordinate automatically
- Expect agents to modify code (they review only)

---

## Integration with Development Workflow

### Pre-Commit Quality Checklist

Before pushing commits:

```
1. Code Quality
   @code-reviewer review staged changes for quality

2. Architecture Validation
   @architect verify no boundary violations

3. Security Assessment
   @security scan staged changes

4. Test Coverage
   @tester verify tests exist and are adequate

5. Commit Message
   @commit-guide create conventional commit message
```

### Feature Development

During feature development:

```
• Ask @code-reviewer as you write
• Consult @architect on design decisions
• Keep @security in mind for sensitive operations
• Ask @tester to write tests for new functionality
```

### Before Creating Pull Request

```
@code-reviewer comprehensive quality review
@architect full architectural validation
@security complete security audit
@tester verify test coverage and quality
@commit-guide finalize commit message
```

### Bug Fixes

```
@code-reviewer verify fix quality
@tester write tests to prevent regression
@security confirm no new vulnerabilities
@commit-guide create fix commit
```

---

## Agent Coordination Protocol

### Context Passing

When requesting an agent's help, provide:

```
- Task description (what do you want?)
- Relevant files/code (show the code)
- Language/framework (Java, TypeScript, etc.)
- Constraints (what matters most?)
- Related context (what else is relevant?)
```

### Response Format

Agents always respond with:

```
Agent: [name]
Skills Applied: [list]

Findings:
  Critical Issues: [list with fix recommendations]
  Warnings: [list with improvement suggestions]
  Suggestions: [list with nice-to-haves]

Escalation: [if needed] "Recommend @agent-name review for..."
```

### Interpreting Results

- **Critical Issues** → Must be fixed before merge
- **Warnings** → Should be addressed soon
- **Suggestions** → Consider for next iteration
- **Escalation** → Seek specialist agent if recommended

---

## Skill Application Examples

### Example 1: Code Quality Review

```
User: @code-reviewer review UserService.java

Agent process:
1. Load .github/skills/code-quality/SKILL.md
2. Load .github/skills/java-language/SKILL.md
3. Analyze code against Critical Rules:
   - Method length ≤ 20 lines
   - Single Responsibility Principle
   - Immutability where applicable
   - Proper encapsulation
4. Report findings with remediation
5. Suggest @architect if architectural issues found
```

### Example 2: Architecture Validation

```
User: @architect validate this new domain service

Agent process:
1. Load .github/skills/hexagonal-architecture/SKILL.md
2. Load .github/skills/adr-compliance/SKILL.md
3. Check Critical Rules:
   - Domain layer isolation
   - Port/adapter separation
   - No cross-layer imports
   - ADR alignment
4. Report boundary violations
5. Suggest ADR updates if needed
6. Recommend @code-reviewer if design pattern issues
```

### Example 3: Security Audit

```
User: @security scan PaymentService.java

Agent process:
1. Load .github/skills/security-review/SKILL.md
2. Apply Critical Rules:
   - No hardcoded secrets
   - Input validation
   - Secure data handling
   - Error handling (no info leaks)
3. Map findings to OWASP Top 10
4. Provide remediation with severity levels
5. Recommend @code-reviewer for structural improvements
```

### Example 4: Test Creation

```
User: @tester write tests for UserService.java

Agent process:
1. Load .github/skills/testing/SKILL.md
2. Load .github/skills/java-language/SKILL.md
3. Load .github/skills/hexagonal-architecture/SKILL.md
4. Analyze code to test:
   - Identify public methods
   - Determine layer (domain/application/infrastructure)
   - Plan test cases (happy path, edge cases, errors)
5. Apply Critical Rules:
   - AAA pattern (Arrange-Act-Assert)
   - Naming: methodName_scenario_expectedBehavior
   - Mock strategy based on layer
   - Test independence
6. Generate test code with builders and proper assertions
7. Report coverage and suggest @architect if testability issues
```

### Example 5: Commit Guidance

```
User: @commit-guide help me create a commit

Agent process:
1. Load .github/skills/conventional-commits/SKILL.md
2. Check if project uses changesets:
   - Detect .changeset/ directory
   - Load .github/skills/changesets/SKILL.md if present
3. Gather context:
   - What changed
   - Why it changed
   - Related issues
   - User impact (if changesets used)
4. Follow skill workflow:
   - Determine commit type
   - Write clear summary
   - Add detailed body if needed
   - Reference issues
5. Determine if changeset needed:
   - Check if change affects end users
   - If feat/fix/user-facing-refactor → suggest changeset
6. Generate conventional commit message
7. Generate changeset file content (if applicable)
8. Provide explanation of choices
```

---

## Future Enhancements

As VS Code and GitHub Copilot capabilities evolve, this system can grow to include:

- [ ] Automatic agent suggestion based on file context
- [ ] Parallel agent consultation for complex reviews
- [ ] Agent state sharing for coordinated analysis
- [ ] Integration with Git hooks for pre-push validation
- [ ] CI/CD integration with automated agent runs
- [ ] Agent performance metrics and team feedback loops

---

## Notes

- **Agents are tools**, not replacements for developer judgment
- **Skills are constraints**, ensuring consistency across all work
- **Coordination is currently manual**, giving developers full control
- **This system complements** Git hooks, CI/CD, and code review processes
- **All contributors** (human and AI) follow the same skills and rules

---
