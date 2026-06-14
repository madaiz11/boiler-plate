---

name: romano-method
description: >
Romano Method — evidence-first investigation skill for research, debugging, code location,
incidents, and cross-layer behavior tracing. Uses Tier-1 anchor, desk split, binary-search
flow, expectation-vs-reality gap, wire merge, evidence gate, and confirmed delivery with
"Here we Go !!!" or a request for specific missing evidence.
disable-model-invocation: true
---

---

# 🔍 Romano Method

**No linear repo walk. No broad grep-first investigation. No conclusion without evidence.**

Use this skill when the task is about:

- debugging
- incident analysis
- locating code
- researching how behavior works
- tracing cross-service or cross-layer flows
- explaining unexpected state changes
- finding where expected behavior diverges from actual behavior

Do **not** use this skill for:

- direct implementation
- small edits
- formatting
- simple renames
- broad refactors
- feature planning unless current behavior must be investigated first

---

## Core Flow

```txt
1. 📍 Tier-1 anchor
2. 🧮 Collection expansion (only if anchor is a set)
3. 🪑 Desk split
4. ✂️ Binary-search flow per desk
5. 🔬 Expectation vs Reality → The Gap
6. 🔗 Wire merge
7. 🚀 "Here we Go !!!" or request more evidence
```

---

## Modes

| Mode        | When to use                                             |
| ----------- | ------------------------------------------------------- |
| **🪑 Desk** | Incident, cross-service, multi-layer, unclear ownership |
| **👤 Solo** | One module, one hop, one clear symbol/API/path          |

**Desk metaphor:**

- Tier-1 anchor = wire tip.
- Each desk = one boundary, layer, service, or story part.
- Main thread = editor that merges desk results before conclusion.

---

# 📍 Phase 1 — Tier-1 Anchor

Pick **one** primary anchor.

Never start by reading the whole repo.

| Mode        | Strong anchor                                                   |
| ----------- | --------------------------------------------------------------- |
| 🔥 Incident | request ID, transaction ID, correlation ID, timestamp, log line |
| 🐛 Debug    | error message, stack trace, failing test, repro step            |
| 🔎 Research | question + symbol/API/path                                      |
| 📌 Locate   | symbol, route, config key, command, component name              |

## Anchor Rules

- Pick the strongest concrete anchor.
- List what the anchor touches before deep reading.
- Prefer codegraph or semantic search before wide grep.
- Use grep only after the anchor narrows the search area.
- Do not inspect unrelated files just because they look nearby.
- If the anchor names a **collection** (`every`/`all`/`each`/`per <thing>`, a plural noun, or a glob/wildcard scope), it is **not** one anchor. Expand it in Phase 1a before desk split. Never collapse the collection word into a single abstract anchor, and never guess its members.

## Anchor Output

```md
## Anchor

- Mode:
- Primary anchor:
- Why this anchor:
- Candidate boundaries:
- Initial evidence:
```

---

# 🧮 Phase 1a — Collection Expansion (enumerate, never guess)

Run this phase **only** when the anchor is a collection: words like
`every`, `all`, `each`, `per`, a plural noun, or a glob/wildcard scope
(e.g. "every product", "all endpoints", "each tenant", "*.handler.ts").

A collection anchor is a **list of concrete member anchors** you must
discover from the project itself — not one anchor, and not a list from
memory.

## Anti-shortcut rule (important++)

The result of guessing may look right, but it skips verification and
cannot be parallelized. Do not shortcut.

- **Never** assume, recall, or invent the members.
- **Always** enumerate them from the actual project: codegraph, semantic
  search, config, registry, enum, directory listing, schema, or route table.
- Every member must carry **source evidence** (file:line, config key, dir entry).
- If you cannot enumerate from evidence, **stop** — state which source is
  missing and ask for it. Do not proceed on a guessed list.

## Enumeration steps

1. Identify the collection noun and **where its members are defined**
   (module dir, registry, enum, DB table, route map, config block).
2. List every member with evidence:

```md
## Collection: <noun>

- Source of truth: <file / registry / config / dir>
- Members:
  - <member> — <file:line | key | dir entry>
  - <member> — <file:line | key | dir entry>
- Enumeration confidence: high | medium | low
- Missing / uncertain members:
```

3. Each enumerated member becomes **its own desk/anchor** in Phase 1b.

## Why enumerate (multitask)

One member = one independent desk. Independent desks run in **parallel**
(Phase 1b parallel rule), so a real enumerated list unlocks multitask.
A guessed or collapsed "all products" anchor cannot be parallelized and
silently hides members you never checked.

---

# 🪑 Phase 1b — Desk Split

Split into **1–3 desks**.

A desk is one focused investigation context.

Use one split axis per round.

| Axis           | Use when                                | Example desks                  |
| -------------- | --------------------------------------- | ------------------------------ |
| **Boundary**   | cross-service, multi-layer, incidents   | gateway, auth, core, database  |
| **Story**      | feature behavior or domain flow         | draft, validation, persistence |
| **Runtime**    | operational issue                       | client, API, worker, queue     |
| **Collection** | anchor is a set (Phase 1a enumerated)   | product-a, product-b, product-c |

## Desk Rules

- Maximum 3 desks per round.
- Name story desks in user language.
- Skip desks with no anchor link.
- Independent desks can be investigated in parallel if the tool/runtime supports it.
- If parallel execution is not available, investigate desks sequentially but keep their reports separate.
- Dependent desks must wait for prerequisite evidence.
- Solo mode skips desk split.
- A **collection anchor** (Phase 1a) maps to **one desk per enumerated member**. If members exceed the per-round cap, run them in batches — in parallel when the runtime supports it. Never drop or merge members just to fit the 3-desk cap, and never substitute a guessed member for an enumerated one.

## Desk Schema

```md
## Desk: <id> — <label>

- Kind: boundary | story | runtime
- Maps to:
- Hypothesis:
- Evidence:
- Depends on:
```

## Desk Brief

Use this structure when delegating or isolating a sub-investigation:

```text
Romano desk: {id} | {kind}: {label}
Maps to: {maps_to}
Hypothesis: {hypothesis}
Anchor: {evidence}
Depends on: {depends_on}

Scope:
- Phases 2–3 only.
- Binary-search the flow.
- Verify contract at midpoint.
- Discard clean half.
- Identify Expectation vs Reality → The Gap.

Allowed:
- Read relevant files.
- Search targeted symbols.
- Use codegraph or semantic search.
- Use read-only shell/log commands if needed.

Not allowed:
- Write code.
- Refactor.
- Commit.
- Deploy.
- Start nested investigations beyond this desk scope.

Return:
## Desk Result: {id} ({label})
- Midpoint checked:
- Contract:
  - Expected:
  - Actual:
  - Result: pass | fail | unknown
- The Gap:
- Evidence:
- Confidence: high | medium | low
```

---

# ✂️ Phase 2 — Binary-Search Flow

For a flow:

```txt
A → B → C → D → E → F
```

If failure appears at `F`:

1. Check midpoint `C` or `D`.
2. If the midpoint contract passes, search the second half.
3. If the midpoint contract fails, search the first half.
4. Repeat until the failing transition is isolated.

## Flow Rules

- Do not walk A→F linearly unless the flow is tiny.
- Check contracts at boundaries.
- Discard clean halves.
- Prefer file-level, symbol-level, log-level, or test-level evidence.
- Mark contract status as `unknown` if evidence is insufficient.

## Desk Result Format

```md
## Desk Result: <id>

- Midpoint checked:
- Contract:
  - Expected:
  - Actual:
  - Result: pass | fail | unknown
- Next narrowed range:
- Evidence:
- Confidence: high | medium | low
```

---

# 🔬 Phase 3 — Expectation vs Reality

The Gap is the smallest known place where expected behavior differs from actual behavior.

```md
## The Gap

- Location:
- Expectation:
- Reality:
- Evidence:
- Why this matters:
- Confidence:
```

## Evidence May Include

- file path and line reference
- stack trace
- log line
- failing test
- reproduction step
- API response
- database value
- config value
- codegraph relationship
- verified runtime behavior

## Gap Rules

- Do not claim root cause from plausibility alone.
- Do not use low-confidence evidence as final proof.
- If the gap is suspected but not confirmed, say what evidence is missing.
- If expectation is unclear, investigate the contract before blaming implementation.

---

# 🔗 Phase 4b — Wire Merge

Merge desk results only after desks return evidence.

| Signal                              | Action                                             |
| ----------------------------------- | -------------------------------------------------- |
| All desks pass until one fails      | Bug is at or after the failing boundary            |
| Conflicting gaps                    | Binary-search the handoff between desks            |
| One high-confidence gap, others low | Weight the high-confidence gap                     |
| All low or unknown                  | Do not conclude; request specific missing evidence |

## Merge Output

```md
## Wire Merge

- Confirmed clean areas:
- Suspect area:
- Owning desk:
- Conflicts:
- Remaining unknowns:
```

---

# 🚀 Phase 4 — Delivery

Use the evidence gate before final answer.

## Evidence Gate

All must pass:

| Check      | Requirement                                                       |
| ---------- | ----------------------------------------------------------------- |
| Claims     | backed by log, file:line, repro, test, graph, or runtime evidence |
| Merge      | no unresolved desk conflict affects conclusion                    |
| Hypothesis | confirmed, not merely plausible                                   |
| Confidence | no low/unknown evidence carrying the conclusion                   |
| Scope      | no unrelated assumptions                                          |

---

## Confirmed Delivery

If the evidence gate passes, first line must be exactly:

```text
Here we Go !!!
```

Then respond with:

```md
## Root Cause

## Trace

## Evidence

## Fix Direction

## Verification
```

---

## Not Confirmed Delivery

If the evidence gate does not pass, do **not** use `Here we Go !!!`.

Respond with:

```md
## Known So Far

## Most Likely Hypothesis

## Missing Evidence

## What Each Missing Item Unlocks

## Next Investigation Step
```

---

# 🛠️ Tooling Guidance

Prefer this order:

1. codegraph or semantic search for architecture/symbol context
2. targeted file read
3. targeted grep
4. read-only shell for logs, tests, jq, or structured output
5. one focused repro/test only when needed

## Allowed During Investigation

- Read
- targeted search
- semantic search
- codegraph tools
- read-only shell
- logs
- tests for reproduction or verification

## Not Allowed Unless User Explicitly Asks

- writing production code
- broad refactor
- git commit
- deploy
- modifying external systems
- changing database state
- deleting files

---

# 🧠 Mindset

| Avoid                              | Prefer                                             |
| ---------------------------------- | -------------------------------------------------- |
| Read A→Z linear                    | Binary-search per desk                             |
| Broad repo grep first              | Anchor → codegraph/semantic search → targeted grep |
| Whole domain catalog               | Only story parts touched by the ask                |
| Treat "all X" as one vague anchor  | Phase 1a: enumerate members from evidence, one desk each |
| Guess collection members from memory | Read project (codegraph/config/registry) with file:line |
| Plausible guessing                 | Evidence or repro                                  |
| Early conclusion                   | Evidence gate                                      |
| Scattered notes                    | Desk schema → merge → deliver                      |
| Refactor while debugging           | Diagnose first, fix later                          |

---

# Integration With Planning Skills

Use Romano Method before planning when current behavior is unclear.

Recommended flow:

```txt
Romano Method → confirmed current behavior/root cause
Spec workflow → plan the change
Task execution → implement one task at a time
Design review → review risky plan before build
```

Do not let Romano Method replace:

- feature specification
- implementation planning
- design review
- task execution

Romano Method owns investigation only.
