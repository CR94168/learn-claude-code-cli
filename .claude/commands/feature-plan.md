---
description: Create a comprehensive implementation feature with from requirements document through extensive research
argument-hint: [requirements]
---

# Create Implementation Feature with Plan from Requirements

You are a **software architect and code generator**.  
Create a comprehensive implementation feature with plan based on initial requirements. This involves extensive research, analysis, and planning to produce a detailed roadmap for execution.

Automatically save the generated plan as `/plans/[feature-name]-[yyyy-mm-dd].md`.

---

## 🧠 Purpose
Create a **comprehensive feature implementation plan** based on initial requirements.  
The plan includes detailed analysis, roadmap, architecture, and testing strategy — and saves the result as a Markdown file in `/plans/`.
---

## Arguments
`{{args}}` = the feature description and optional context provided by the user (e.g. "Add SSO with OAuth2 for admin portal; target: v1.2; include tests").


## High-level instructions (must follow exactly)
1. **Read project context**
   - Load `CLAUDE.md` (if exists) and any docs/README, package/config files, and tests.
   - Inspect `package.json`/`requirements.txt`/`.csproj` to find dependencies and platform constraints.
   - Search repository for related modules, interfaces, controllers, services, and existing patterns.

2. **Understand `{{args}}`**
   - Restate the feature request in 3–5 sentences.
   - List assumptions and clarifying questions (if any). If a clarifying question is required to continue, ask it. Otherwise proceed with reasonable assumptions and state them.

3. **Research & analysis**
   - Identify relevant existing code modules to reuse or extend.
   - Identify external systems / third-party APIs that the feature will touch (DBs, auth providers, queues, payment gateways).
   - Check for existing tests and testing patterns to follow.
   - List compatibility / migration concerns (backwards compatibility, DB schema changes, config changes).

4. **Produce a detailed plan / roadmap (deliverable #1)**
   - Format: Markdown with clear sections:
     - **Summary** — short description of feature & goals.
     - **Scope** — what is in, what is out (non-goals).
     - **Acceptance criteria** — clear, testable outcomes.
     - **Design overview** — architecture, data model changes, API contract (sample request/response).
     - **Files to change / create** — exact paths and brief content descriptions.
     - **Task breakdown** — numbered tasks suitable to paste into an issue tracker (title, description, estimate label).
     - **Milestones** — grouping of tasks into milestones (e.g., design, backend, frontend, tests, docs, deploy).
     - **Testing plan** — unit/integration/e2e specifics and example test cases.
     - **Deployment plan** — staging rollout, smoke tests, migration steps (if DB), and rollback steps.
     - **Monitoring & alerts** — metrics, logs, thresholds to monitor after release.
     - **Security, privacy & performance considerations** — list items and mitigation steps.
     - **Dependencies & prerequisites** — libs, infra, secrets, account access.
     - **Risk assessment** — main risks and contingency plan.
   - Provide example command/endpoint usage and sample JSON if API involved.
   - Provide suggested branch name and PR title template.

5. **Create implementation plan (deliverable #2)**
   - For each task in Task breakdown, produce:
     - Short title
     - Detailed description
     - Files to modify/create (with brief content outline)
     - Estimated complexity (Low / Medium / High)
     - Test cases required

6. **Present plan for approval**
   - Output the full roadmap and a short developer checklist.
   - **Do NOT make code changes yet** unless the user/runner explicitly approves the plan or permission mode allows automatic apply.
   - Offer the user the following options (present as choices):
     - `Approve Plan` → implement all tasks automatically (if allowed).
     - `Implement Core` → implement a minimal subset (list which tasks).
     - `Iterate` → refine plan (user edits plan text then re-run).
     - `Cancel` → stop.

7. **If approved and edits are allowed**
   - Create a new branch using the suggested branch name.
   - Implement tasks incrementally, commit with clear messages (use task ID in commit).
   - Add/modify tests as specified.
   - Run the project's test suite (or the relevant subset) and attach results.
   - If any failing tests or build errors occur, stop and report back with a fix plan.


## Constraints & rules
- **Follow existing architecture** and coding style strictly. If anything new is necessary, justify it clearly and prefer minimal additions.
- **Keep changes minimal and incremental**. Prefer composition over large refactors unless the plan explicitly includes refactor tasks.
- **Do not expose secrets** in output.
- **Always show the plan before automated edits** unless CLI permission settings are configured to auto-apply.
- If multiple possible approaches exist, present 2–3 options with pros/cons and choose a recommended option.

## Example workflow (what you will actually produce)
1. Restate feature + assumptions.
2. Repo analysis summary (files found, relevant modules).
3. Proposed design + API/data model draft.
4. Full roadmap (tasks + milestones).
5. Suggested branch name and PR title.
6. Confirmation prompt with implementation options.

## Implementation details (if allowed to implement)
- Use existing naming patterns and folder paths. See CLAUDE.md
- Write tests following the repo’s test framework & style.
- When editing files, show diffs in unified diff format before applying (if interactive).

---

### Format Document Plan

```markdown
# Implementation Plan: [Feature Name]

## Overview
[Brief description of what will be implemented]

## Requirements Summary
- [Key requirement 1]
- [Key requirement 2]
- [Key requirement n]

## Research Findings
### Best Practices
- [Finding 1]
- [Finding n]

### Reference Implementations
- [Example 1 with link/location]
- [Example n with link/location]

### Technology Decisions
- [Technology choice 1 and rationale]
- [Technology choice n and rationale]

## Implementation Tasks

### Phase 1: Foundation
1. **Task Name**
   - Description: [What needs to be done]
   - Files to modify/create: [List files]
   - Dependencies: [Any prerequisites]
   - Estimated effort: [time estimate]

2. **Task Name**
   - Description: [What needs to be done]
   - Files to modify/create: [List files]
   - Dependencies: [Any prerequisites]
   - Estimated effort: [time estimate]

### Phase 2: Core Implementation
[Continue with numbered tasks...]

### Phase 3: Integration & Testing
[Continue with numbered tasks...]

## Codebase Integration Points
### Files to Modify
- `path/to/file1.js` - [What changes needed]
- `path/to/filen.py` - [What changes needed]

### New Files to Create
- `path/to/newfile1.js` - [Purpose]
- `path/to/newfilen.py` - [Purpose]

### Existing Patterns to Follow
- [Pattern 1 from codebase]
- [Pattern n from codebase]

## Technical Design

### Architecture Diagram (if applicable)
```
 