---
name: strict-build
description: >
  Executes code changes through a strict 6-step TDD loop — trace execution tree,
  enhance-vs-new decision, tests first, minimal implementation, verify until green,
  artifact log. Use when implementing an approved Cursor plan, confirm build from
  plan mode, executing plan todos, or when the user invokes strict build or strict
  execution workflow.
---

# Strict Build Protocol

**TDD-first implementation loop — trace before change, test before code, verify before ship.**

Every response during an active build MUST print the current step header (e.g. `### [STEP 1/6: Check Existing & Trace Tree]`) before any other output.

## When this skill applies

| Trigger | Typical ask | Entry anchor |
| ------- | ----------- | ------------ |
| **Plan build** | confirm build, implement plan | `~/.cursor/plans/*.plan.md` + pending todos |
| **Strict build** | `@strict-build`, strict execution workflow | User requirement or attached plan |
| **Scoped feature** | add/fix with TDD mandate | Target symbol, file, or plan todo id |

**Plan mode:** Read the approved plan first. Run one full 6-step loop per pending todo (or per logically grouped batch). Do not deviate from approved scope without asking.

## Agent quick start

Follow all six steps in order. Never skip to production code before Step 3.

| Step | Goal | Key tool / rule |
| ---- | ---- | ---------------- |
| 1. Trace tree | Map existing topology | `user-code-review-graph/debug_issue` |
| 2. Decision | Enhance vs new | Default **Enhance Existing** |
| 3. TDD tests | Tests before prod code | Happy path + edge + error cases |
| 4. Minimal impl | Smallest diff to pass | No junk logs or redundant comments |
| 5. Verify loop | Run project test command | Fail → back to Step 4 until green |
| 6. Artifact | Summary markdown at workspace root | `YYYYMMDD_HHMMSS-<topic>.md` |

**Do not:** commit unless user asks; expand scope beyond plan; skip step headers.

## Tool allowlist

When this skill is active, prefer the tools below. Ask the user before using anything outside this list for the build task.

| Allowed | Use for |
| ------- | ------- |
| `Read`, `Grep`, `Glob`, `SemanticSearch` | Codebase inspection, locating symbols |
| `CallMcpTool` → `user-code-review-graph` / `debug_issue` | Step 1 — execution tree, call graph, sequence |
| `CallMcpTool` → `user-codegraph` | Fallback context when graph index is available |
| `Write`, `StrReplace`, `Delete` | Steps 3–4 — tests and production code |
| `Shell` | Step 5 — run tests; read project test command from package config |
| `WebFetch` | External docs when plan references APIs |

**Do NOT:** git commit/push, deploy, or broad refactors outside the current todo unless the user explicitly requests them.

---

## Instructions

### [STEP 1/6: Check Existing & Trace Tree]

- **Action:** Analyze the existing codebase and system topology before planning any changes.
- **Tool Mandate:** Use `user-code-review-graph/debug_issue` to generate or examine the execution tree, call graph, and sequence of operations related to this task.
- **Analysis:** Trace the exact data flow, parent-child component relationships, and execution order to see how the existing logic is wired.
- **Output:** Print the relevant section of the execution tree/sequence found via `user-code-review-graph/debug_issue` and list the existing files involved.

### [STEP 2/6: Decision (Enhance vs. New)]

- **Action:** Evaluate whether to refactor/extend the existing code found in Step 1 or create something new.
- **Rule:** Default to **"Enhance Existing"** to preserve the current architecture tree unless modifying it breaks SOLID principles or introduces extreme technical debt.
- **Output:** One-sentence justification based on the execution tree analysis.

### [STEP 3/6: TDD Unit Test Generation]

- **Action:** Write or update unit tests **FIRST** before touching any production code.
- **Rule:** Explicit test cases for happy path, edge cases, and error handling.
- **Output:** Show the complete test code.

### [STEP 4/6: Minimal Implementation]

- **Action:** Write the implementation code.
- **Rule:** Absolute minimum code required to make the tests pass.
- **Code style:** No redundant "what" comments. No leftover junk or console logs. Self-documenting code only.

### [STEP 5/6: Test Execution & Verification Loop]

- **Action:** Run tests using the project's test command.
- **IF TESTS FAIL:** Print the error, analyze root cause, return to **[STEP 4/6]**, fix, re-run. Repeat until green.
- **IF TESTS PASS:** Clean up temporary logs, refactor for readability if needed, proceed to Step 6.

### [STEP 6/6: Artifact Logging & Future Improvements]

- **Action:** Create a summary markdown file at the **workspace root**.
- **Filename:** `YYYYMMDD_HHMMSS-<short-topic-name>.md` (actual timestamp, clear topic — e.g. `20260601_091500-auth-token-enhancement.md`).
- **Content:** Use the artifact template below.
- **Output:** Confirm file creation and print the full preview.

## Artifact template

```markdown
# <Topic Title>

## Executive Summary
<Brief description of the task completed.>

## Modifications Log
| File | Change |
| ---- | ------ |
| `path/to/file` | One-sentence description |

## Verification Status
- [ ] All TDD tests passed (command: `<test command run>`)

## Future Improvements
1. <Concrete improvement — performance, security, scaling, or debt>
2. ...
3. ...
(3–5 items total)
```

## Examples

**Example — plan todo build**

Plan todo: *"Add retry handler to useProfilePolicies"*

1. **Step 1:** `debug_issue` on `useProfilePolicies` → lists `PageLoading.tsx`, `MyPolicySection.tsx`, `policy.service.ts`.
2. **Step 2:** Enhance Existing — hook already owns query; extend hook, do not duplicate query in parent.
3. **Step 3:** Write `useProfilePolicies.spec.ts` — success, API error, retry increments errorCount.
4. **Step 4:** Implement hook + wire section to hook output.
5. **Step 5:** `npm test -- useProfilePolicies` → green.
6. **Step 6:** Write `20260601_143022-profile-policies-retry.md` at project root; preview to user.

**Example — step header format**

```
### [STEP 3/6: TDD Unit Test Generation]

<test code and explanation>
```
