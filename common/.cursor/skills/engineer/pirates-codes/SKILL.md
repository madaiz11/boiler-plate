---

name: pirates-codes
description: >
Pirates Codes — author-code skill for implementing features with project alignment,
Penta design contract, test design, table-driven AAA tests first, minimal implementation,
green verification, and Pirates Manifest. Use for implementation, scaffolding,
guideline-driven coding, and approved plan-to-code execution.
---

---

# 🏴‍☠️ Pirates Codes

**Author code. Do not review.**
KnightGuard reviews. Pirates ships.

Use this skill when the user asks to:

- implement a feature
- scaffold code
- convert an approved plan into code
- add behavior with tests
- refactor behavior while preserving intent
- implement a task from `.specs/<feature-name>/tasks.md`

Do not use this skill for:

- CVE audit
- security review
- load testing
- broad repo rewrite
- pure investigation
- post-implementation review
- writing specs/plans/tasks from scratch

Use `romano-method` first if the anchor or root cause is unclear.
Use `knightguard-reviewer` after green if the user asks for review.
Use Cursor rules `spec-workflow.mdc`, `design-review.mdc`, and `task-execution.mdc` for planning, reviewing, and executing spec-driven work.

---

## Fixed Boarding Order

```txt
1. 🧭 Align + Penta
2. 🗺️ Test design
3. 🧪 Tests first
4. ⚙️ Minimal implementation
5. ✅ Green verification
6. 📜 Pirates Manifest
```

Rules:

- Never skip Align.
- Never ship production code before tests.
- Penta contract is created once during Align.
- Do not re-run Penta after implementation.
- Implementation must follow the locked contract.
- If the prompt changes materially, restart Align.
- If the task comes from `.specs/<feature-name>/tasks.md`, align only to the requested task.

---

# ⚡ Boarding Map

| Ask                             | Do                                                                         |
| ------------------------------- | -------------------------------------------------------------------------- |
| Implement / scaffold            | Full boarding                                                              |
| Approved plan → code            | Align to the specific task, then full boarding                             |
| Refactor behavior               | Re-align if architecture or behavior changes                               |
| Bug fix with known root cause   | Align to root cause, write regression test first, then implement           |
| Bug fix with unknown root cause | Use `romano-method` before Pirates                                         |
| Spec/task execution             | Use `task-execution.mdc` for task control, then Pirates for code authoring |

---

# 1. 🧭 Align

Before tests or production code, align with the project.

Check:

| Check              | Action                                                           |
| ------------------ | ---------------------------------------------------------------- |
| Clear requirement? | Restate request, input/output, acceptance                        |
| Fits project?      | Read neighboring code and existing patterns                      |
| Implementable?     | Use real modules/APIs; do not invent tables, routes, or services |
| Good approach?     | Avoid duplicate capability, wrong layer, and risky shortcuts     |

## Stop Conditions

Stop before tests/code when:

- requirement is unclear
- approach conflicts with existing architecture
- required module/API/table does not exist
- implementation would duplicate an existing capability
- risk is too high without confirmation
- task depends on missing investigation
- task should first be specified, planned, or reviewed through Cursor rules

Reply with:

```md
## Stop

### Why Stop

- ...

### Evidence

- ...

### Better Path

- ...

### Tradeoff

- ...

### Need Confirmation

- ...
```

Do not proceed to test design until alignment is safe.

---

# Penta Design Contract

Create this once during Align.

If the user changes the prompt materially, rewrite the contract.

```md
## Penta Design Contract

| Lens            | Decision                                   |
| --------------- | ------------------------------------------ |
| 🧩 Conceptual   | pattern or plain functions; where it lives |
| ⚖️ Severity     | 🔴🟠🟡🔵 tier + fail-closed rules          |
| ⚡ Performance  | hot path? Big-O budget; batch/paginate     |
| 🧱📋 SOLID+DRY  | module owner; extend vs new                |
| 📖 Simple       | API shape; comment policy                  |
| 🦀 Rust mindset | ownership, errors, no hidden mutation      |
```

## Penta Lenses

| Icon | Ask                                        | Ship when                                      |
| ---- | ------------------------------------------ | ---------------------------------------------- |
| 🧩   | Does this pattern earn its keep?           | Team can name it                               |
| ⚖️   | If wrong at 2am, is damage contained?      | Typed errors; 🔴 fails closed at boundary      |
| ⚡   | Will production-sized data hurt?           | No unbounded user-sized work without approval  |
| 🧱📋 | Does each module have one job?             | No god module; one source for constants        |
| 📖   | Can a junior understand the file quickly?  | Simple shape; no redundant comments            |
| 🦀   | Are states, errors, and mutation explicit? | No hidden mutation or unsafe input assumptions |

## Comment Policy

Allowed comments:

- complex invariant
- public docblock
- non-obvious business rule
- security-sensitive reason

Avoid comments that restate code.

## Rust Mindset In Any Language

| Idea                          | In code                                         |
| ----------------------------- | ----------------------------------------------- |
| Ownership                     | One owner per mutable state; no request globals |
| Result not panic              | Typed errors on user/network input              |
| Enums over flags              | Invalid states unrepresentable                  |
| Parse, do not merely validate | Strong types at boundaries                      |
| No hidden mutation            | `get*` must not write DB/events                 |

Reject:

- `any` through domain logic
- unwrap/throw on user input without boundary handling
- global mutable singleton
- clone/copy in tight loops without reason
- hidden write inside read-looking function

---

# 2. 🗺️ Test Design

Before writing tests, produce a test map.

Cover:

| Bucket    | Cover                                       |
| --------- | ------------------------------------------- |
| ✅ Happy  | each required outcome                       |
| 📏 Rules  | branch per business rule, boundaries, state |
| ❌ Errors | every Severity error from contract          |
| 🔀 Edge   | empty, null, max, duplicate, idempotency    |

Output numbered cases:

```md
## Test Design

1. case:
   - input:
   - expected:

2. case:
   - input:
   - expected:
```

Rules:

- Tests must come from the contract.
- Do not write implementation before test design.
- Do not create meaningless tests only to satisfy process.
- Match existing project test style.
- If the task came from `tasks.md`, tests must verify the requested task only.

---

# 3. 🧪 Tests First

Write tests before production code.

Use AAA per test:

```txt
Arrange → Act → Assert
```

Prefer table-driven tests when the language/project style supports it.

Example shape:

```go
for _, tt := range []struct {
    name string
    wantErr error
}{
    // cases
} {
    t.Run(tt.name, func(t *testing.T) {
        // Arrange
        // Act
        // Assert
    })
}
```

Ship tests when:

- test files exist
- tests target required behavior
- tests fail because implementation is missing or incomplete
- tests do not fail from syntax/import mistakes

If existing project style is different, match the project.

---

# 4. ⚙️ Minimal Implementation

Implement only what the tests and contract require.

Rules:

| Rule              | Detail                                          |
| ----------------- | ----------------------------------------------- |
| Scope             | Only prompt + tests                             |
| Contract          | Follow Align decisions                          |
| Existing patterns | Extend existing symbols before adding new files |
| Diff size         | Keep minimal                                    |
| Comments          | Only when policy allows                         |
| Risk              | Do not widen behavior without approval          |

Do not:

- re-audit Penta after coding
- add optional architecture
- refactor unrelated code
- change contracts silently
- add broad abstractions prematurely
- implement future tasks from `tasks.md` unless explicitly requested

If the implementation reveals the test design is wrong:

1. stop
2. update test design
3. update tests
4. continue implementation

---

# 5. ✅ Green Verification

Run the smallest relevant verification.

Prefer project commands.

Examples:

```txt
npm test
npm run test
npm run typecheck
npm run lint
go test ./...
pytest
```

Rules:

- Inspect project scripts before inventing commands.
- Touched unit suite must pass before delivery.
- If verification cannot be run, say why.
- Do not claim green if tests were not run.
- If tests fail, fix implementation first.
- If tests reveal wrong expectation, return to test design.

---

# 6. 📜 Deliver

End with Pirates Manifest.

Manifest must copy the Penta contract decisions.
Do not re-derive Penta after implementation.

```md
## Pirates Manifest

| Boarding | Status                             |
| -------- | ---------------------------------- |
| ✅ Align | [where it lives / stopped → path]  |
| ✅ Penta | [locked at Align — not re-checked] |
| ✅ Tests | [N cases]                          |
| ✅ Green | [cmd + pass / not run + reason]    |

| Lens            | From Contract        |
| --------------- | -------------------- |
| 🧩 Conceptual   | [copy from contract] |
| ⚖️ Severity     | [copy from contract] |
| ⚡ Performance  | [copy from contract] |
| 🧱📋 SOLID+DRY  | [copy from contract] |
| 📖 Simple       | [copy from contract] |
| 🦀 Rust mindset | [copy from contract] |

**Drift:** [none | one line]
```

Keep Manifest concise.

---

# Pre-Delivery Checklist

```txt
Align:
- fits project
- contract written
- stopped if misaligned

Boarding:
- test design written
- tests before production code
- implementation minimal
- verification done or honestly marked not run

Deliver:
- Manifest included
- Penta copied from contract
- no post-implementation Penta re-audit
```

---

# Code Rules

1. Align + Penta contract before tests.
2. Tests before production code.
3. Minimal diff.
4. Match neighboring style.
5. Security basics:
   - authorization at boundary
   - parameterized queries
   - secrets from environment
   - no sensitive logs

6. Do not review your own shipped code unless the user asks.
7. Use KnightGuard for post-green review.

---

# Related

| Type        | Name                   | Use                                                      |
| ----------- | ---------------------- | -------------------------------------------------------- |
| Skill       | `romano-method`        | investigate first when anchor/root cause is unclear      |
| Skill       | `knightguard-reviewer` | post-green review                                        |
| Cursor Rule | `spec-workflow.mdc`    | create specs/plans/tasks before implementation           |
| Cursor Rule | `design-review.mdc`    | review specs, plans, migrations, and technical proposals |
| Cursor Rule | `task-execution.mdc`   | execute approved task lists one task at a time           |
