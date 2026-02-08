---
name: user-story-writing
description: Write effective user stories that capture requirements from the user's perspective. Create clear stories with detailed acceptance criteria to guide development and define done.
---

# User Story Writing Skill

This skill defines the standards and workflow for creating high-quality user stories with clear acceptance criteria and actionable tasks.

## Critical Rules

These rules are non-negotiable and must be applied to every user story.

1. **Every story must have a repository reference** — "Related to:" field is mandatory
2. **Acceptance criteria must be testable** — Each criterion must be objectively verifiable (not "looks good" or "feels right")
3. **Tasks must be 1-3 day efforts** — If a task is larger, split it into smaller subtasks
4. **Titles must be action-oriented** — Use gerunds or imperative verbs (Export, Filter, Enable, Allow)
5. **Descriptions must include context** — Answer: "What?" "Why?" and "Who benefits?"
6. **No acceptance criteria should be vague** — Replace "should be fast" with "complete within 5 minutes"
7. **Tasks must be independently implementable** — A developer should be able to pick any task without depending on another

## Workflow

Follow these steps in order when creating a user story:

### Step 1: Parse User Input
Read the user's natural language request and identify:
- What feature/capability is being requested?
- Who is the user/actor?
- Why do they need it (pain point, goal, compliance)?

### Step 2: Define Story Title
Create a concise, action-oriented title. Format: `[Verb] [Noun]` or `[Verb] [Capability]`

Examples: "Export Personal Data", "Filter by Multiple Tags", "Enable Google Sign-In"

### Step 3: Write Description
Structure the description with three parts:
1. **Capability statement**: "Enable/Allow/Permit users to..."
2. **Value/benefit**: "...to [achieve goal] / reduce [friction] / improve [metric]"
3. **Context/constraints**: Any technical requirements, compliance, or scope limits

### Step 4: Create Acceptance Criteria
Generate 4-8 criteria following this pattern:
- Start with observable user actions or system behaviors
- Use "Given/When/Then" format or numbered checklist
- Each criterion must have a single, testable assertion
- Include performance, security, or compliance criteria if applicable

### Step 5: Identify Proposed Tasks
Break the story into 5-8 tasks by ownership and dependency:
- Frontend/UI tasks
- Backend/API tasks
- Testing tasks
- Documentation tasks
- Compliance/Security tasks (if applicable)

Each task title: `[Component]: [Action]`
Each task description: 1-2 sentences with scope

### Step 6: Assign Repository
Always include "Related to:" field. If user doesn't specify, default to `agents-and-skills`.

## Decision Trees

### When to split a story into multiple stories

```
Is the story larger than 2-3 weeks of work?
├── YES → Split into multiple user stories (likely an Epic)
│   └─ Create parent story with umbrella title
│   └─ Create smaller child stories with specific features
└── NO → Keep as single story
```

### How to judge if a criterion is testable

```
Can we check this with a test (automated or manual)?
├── YES → Criterion is good
│   ├─ "The export button exists on the settings page" ✓
│   └─ "Login completes within 2 seconds" ✓
└── NO → Refine the criterion
    ├─ "The UI looks good" → "The button meets accessibility guidelines (WCAG AA)"
    └─ "The export is secure" → "Exported files are encrypted with AES-256 or higher"
```

### How to determine task size

```
Can this task be completed, tested, and merged in 1-3 days?
├── YES → Keep as single task
│   └─ Assign to owner (backend, frontend, etc.)
└── NO → Split into smaller tasks
    ├─ "Implement OAuth2" → Too large
    └─ Split into:
       ├─ "Implement OAuth2 token exchange"
       ├─ "Add OAuth2 button to UI"
       ├─ "Write OAuth2 integration tests"
```

## Examples

### Example 1: Export Personal Data

**User Input:**
"I want to export all my information from the platform so I can store it locally and use it elsewhere."

**Generated Story:**

**Title:** Export Personal Data Locally

**Description:**
Enable users to download and export all their personal information in a machine-readable format. This supports data portability, GDPR compliance, and allows users to maintain local backups. The export must be secure and delivered in standard formats (JSON, CSV).

**Acceptance Criteria:**
1. An "Export My Data" link is visible in account settings.
2. Clicking "Export My Data" initiates a download of user data in JSON or CSV format.
3. The exported file includes: profile info, preferences, activity metadata, content history.
4. Export completes within 5 minutes for datasets under 1 GB.
5. Downloaded files are encrypted or transmitted via HTTPS only.
6. User receives confirmation email with download link valid for 7 days.
7. All export requests are logged with timestamp, user ID, format, and data size.
8. The feature works across all supported browsers (Chrome, Firefox, Safari, Edge).

**Related to:** agents-and-skills

**Proposed Tasks:**
- [ ] Backend: Design export query and data aggregation logic — determine which tables/fields to include
- [ ] Backend: Implement JSON/CSV serialization — handle dates, special characters, nested objects
- [ ] Security: Add HTTPS-only transmission and encryption — implement secure token-based download links
- [ ] Backend: Create audit logging for export requests — log to compliance table
- [ ] Frontend: Add "Export My Data" button to settings — include help text and confirmation dialog
- [ ] QA: Write integration tests for export — verify all data types, formats, completeness
- [ ] Documentation: Create user guide for exported data — explain format, retention, usage
- [ ] Performance: Load test with 100MB and 1GB datasets — ensure SLA compliance

---

### Example 2: Multi-tag Filtering

**User Input:**
"Users should be able to filter by more than one tag at the same time."

**Generated Story:**

**Title:** Filter Resources by Multiple Tags

**Description:**
Allow users to apply multiple tags simultaneously when searching or browsing resources. The default behavior uses AND (all tags must match); users can toggle to OR (any tag matches). This improves precision and reduces time to find relevant items.

**Acceptance Criteria:**
1. Filter panel displays a multi-select checkbox list for tags (not single-select).
2. Results default to AND behavior: only items matching ALL selected tags appear.
3. A toggle option changes behavior to OR: items matching ANY selected tag appear.
4. Results update in real-time without page reload when tags are selected/deselected.
5. Queries with up to 5 tags complete within 500ms (p95 latency).
6. Selected tags persist in URL query string for bookmarking and sharing.
7. Users can clear all filters with a single "Reset Filters" button.
8. The feature is accessible via keyboard (Tab, Space, Arrow keys).

**Related to:** agents-and-skills

**Proposed Tasks:**
- [ ] Frontend: Build multi-select tag component — support keyboard navigation, visual feedback
- [ ] Frontend: Implement AND/OR toggle and result filtering logic — reactive updates, URL state management
- [ ] Backend: Optimize search queries for multiple tags — add database indexes on tag columns
- [ ] Backend: Implement query performance monitoring — track latency by tag count
- [ ] QA: Write component tests for tag selection — verify AND/OR logic, keyboard accessibility
- [ ] QA: Performance test with 3, 5, 10+ tag combinations — benchmark query time
- [ ] Documentation: Add filter usage guide with screenshots — explain AND vs OR behavior

---

## Notes

- User stories are a communication tool between Product and Engineering
- Acceptance criteria are the "definition of done" for a story
- Tasks should never require discussion to understand; they are self-documenting
- Avoid over-specifying implementation details in stories; focus on outcomes
- Challenge vague requests (e.g., "make it faster") with specific, measurable criteria

