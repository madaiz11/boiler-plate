---
name: romano-method
description: >
Romano Method — evidence-first investigation skill for research, debugging, code location,
incidents, and cross-layer behavior tracing. Uses Tier-1 anchor, desk split, binary-search
flow (success + error paths, color-coded), expectation-vs-reality gap, wire merge, adversarial
recap loop, evidence gate, HTML export, and confirmed delivery with "Here we Go !!!" or a
request for specific missing evidence. Responses use /caveman (default full) to stay terse.
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

## Communication — `/caveman`

All Romano responses use **caveman mode** unless user says `stop caveman` or `normal mode`.

| Command | Effect |
| ------- | ------ |
| `/caveman` | Activate full (default) |
| `/caveman lite` | Tight sentences, keep articles |
| `/caveman full` | Drop filler, fragments OK |
| `/caveman ultra` | Max compression |

**Rules during Romano:**

- Terse delivery. Technical substance stay. Fluff die.
- No tool-call narration. No decorative tables/emoji in final answer (skill doc emoji OK).
- Code blocks, file:line, error strings, API names — exact, unchanged.
- Match user language; compress style only.
- Drop caveman only for: security warnings, irreversible actions, ambiguous multi-step sequences.
- Resume caveman after clear part done.

Code/commits/HTML artifact content: write normal. Only user-facing investigation reply = caveman.

---

## Core Flow

```txt
1. 📍 Tier-1 anchor
2. 🧮 Collection expansion (only if anchor is a set)
3. 🪑 Desk split
4. ✂️ Binary-search flow per desk — success path + error paths, color-coded
5. 🔬 Expectation vs Reality → The Gap
6. 🔗 Wire merge
7. 🔄 Adversarial recap loop (edge cases → re-process if open)
8. 📄 HTML export
9. 🚀 "Here we Go !!!" or request more evidence
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
- Binary-search the flow — success path AND error paths.
- Color-code every edge.
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
- Flow diagram (success + error, color-coded):
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

For every flow, map **two parallel tracks**:

1. **Success path** — happy path from entry to expected outcome.
2. **Error paths** — every known failure branch, early exit, throw, HTTP error, rollback, timeout, retry exhaustion.

Do not ship success-only diagram. Error paths are mandatory.

## Example

```txt
Success:
A ──green──▶ B ──green──▶ C ──green──▶ D ──green──▶ E

Errors from same entry:
A ──red──▶ B ──orange──▶ ERR_VALIDATION (400)
A ──green──▶ B ──red──▶ ERR_AUTH (401)
B ──orange──▶ ERR_TIMEOUT (504)
C ──red──▶ ERR_DB_ROLLBACK
```

If failure appears at `F` on success path:

1. Check midpoint `C` or `D`.
2. If midpoint contract passes, search second half.
3. If midpoint contract fails, search first half.
4. Repeat until failing transition isolated.
5. Cross-check: does active error path explain observed failure? If not, binary-search error branch too.

## Color Legend (required on every flow)

Apply color to **each edge** (arrow/transition), not just nodes.

| Color | Hex | Meaning | Use when |
| ----- | --- | ------- | -------- |
| 🟢 green | `#22c55e` | success / pass / expected OK | Contract pass, happy transition |
| 🔴 red | `#ef4444` | hard fail / throw / 5xx / crash | Unhandled error, fatal branch |
| 🟠 orange | `#f97316` | soft fail / 4xx / validation / business reject | Expected error response |
| 🟡 yellow | `#eab308` | unknown / not verified | Contract `unknown`, missing evidence |
| 🔵 blue | `#3b82f6` | boundary / handoff / external call | Service hop, API, queue, DB |
| ⚫ gray | `#6b7280` | skipped / dead / not reached | Discarded half, unreachable in repro |

In markdown/mermaid replies, use `style` or `linkStyle` with hex. In HTML export, inline `stroke` / `color` with same hex.

## Flow Rules

- Do not walk A→F linearly unless flow tiny.
- Check contracts at boundaries — success AND error contracts.
- Discard clean halves.
- Prefer file-level, symbol-level, log-level, or test-level evidence.
- Mark contract status `unknown` if evidence insufficient.
- Every external call gets at least one error edge (timeout, non-2xx, parse fail).
- If code has `try/catch`, map catch path as orange or red edge.

## Desk Result Format

```md
## Desk Result: <id>

### Flow — Success
<color-coded diagram>

### Flow — Errors
<color-coded diagram — all branches>

- Midpoint checked:
- Contract:
  - Expected:
  - Actual:
  - Result: pass | fail | unknown
- Active path (success or error):
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
- Path type: success | error
- Edge color at gap:
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
- If gap suspected but not confirmed, say what evidence missing.
- If expectation unclear, investigate contract before blaming implementation.
- Check gap on error path too — observed 500 may be wrong branch, not success-path bug.

---

# 🔗 Phase 4b — Wire Merge

Merge desk results only after desks return evidence.

| Signal                              | Action                                             |
| ----------------------------------- | -------------------------------------------------- |
| All desks pass until one fails      | Bug at or after failing boundary                   |
| Error path explains symptom         | Weight error-path gap over success-path guess      |
| Conflicting gaps                    | Binary-search handoff between desks                |
| One high-confidence gap, others low | Weight high-confidence gap                         |
| All low or unknown                  | Do not conclude; request specific missing evidence |

## Merge Output

```md
## Wire Merge

- Confirmed clean areas:
- Suspect area:
- Active path: success | error | both
- Owning desk:
- Conflicts:
- Remaining unknowns:
```

---

# 🔄 Phase 4c — Adversarial Recap Loop

Run **before** evidence gate and HTML export. Attack own answer.

## Steps

1. **Recap** — 3–5 bullet summary of conclusion, gap, evidence, fix direction.
2. **Edge-case probe** — list scenarios that could break conclusion:
   - null / empty / boundary values
   - race / retry / duplicate request
   - partial failure / rollback
   - wrong error path taken
   - env/config drift
   - permission / tenant / feature-flag variants
3. **Adversarial questions** — write questions a skeptical user or reviewer would ask:
   - "What if X happens before Y?"
   - "How you know not red herring from adjacent service?"
   - "Evidence cover error path or only success?"
4. **Answer each question** with evidence or mark `unanswered`.
5. **Re-process rule:**
   - Any `unanswered` or new gap found → **do not deliver**.
   - Return to Phase 2–4 for affected desk only.
   - Re-run recap loop after new evidence.
   - Max 3 recap rounds; if still open, deliver as Not Confirmed with open questions listed.

## Recap Output (internal, include in HTML)

```md
## Recap

- Summary:
- Edge cases checked:
- Adversarial Q&A:
  - Q: ... → A: ... | unanswered
- Open loops:
- Re-process needed: yes | no
- Round: 1 | 2 | 3
```

---

# 📄 Phase 4d — HTML Export

Export every completed investigation (confirmed or not confirmed) as standalone HTML.

## Path

```
.cursor/docs/romano-method/<YYYYMMDD-HHmmss>-<slug>/report.html
```

`<slug>` = short kebab-case from anchor (e.g. `auth-401-middleware`).

## Required sections in HTML

1. Title + anchor + mode + timestamp
2. Color legend (table with hex swatches)
3. Success flow diagram (SVG or mermaid rendered, colored edges)
4. Error flow diagram(s) (colored edges)
5. Desk results summary
6. Wire merge
7. The Gap
8. Recap + adversarial Q&A
9. Final delivery (Root Cause / Missing Evidence)
10. Evidence appendix (file:line, logs, snippets)

## HTML style rules

- Self-contained single file (inline CSS, no external deps).
- Dark-friendly readable theme.
- Edge colors use Phase 2 hex values inline on SVG `<line>` / `<path>` or styled mermaid.
- Node boxes: neutral bg; edge color carries status.
- Collapsible sections for long evidence blocks.

## Export trigger

- **Confirmed** → export before `Here we Go !!!` reply.
- **Not confirmed** → export before Missing Evidence reply.
- Mention path in final caveman reply: `HTML: .cursor/docs/romano-method/.../report.html`

---

# 🚀 Phase 5 — Delivery

Use evidence gate before final answer.

## Evidence Gate

All must pass:

| Check      | Requirement                                                       |
| ---------- | ----------------------------------------------------------------- |
| Claims     | backed by log, file:line, repro, test, graph, or runtime evidence |
| Flows      | success + error paths drawn, every edge color-coded               |
| Merge      | no unresolved desk conflict affects conclusion                    |
| Hypothesis | confirmed, not merely plausible                                   |
| Confidence | no low/unknown evidence carrying conclusion                       |
| Scope      | no unrelated assumptions                                          |
| Recap      | adversarial loop complete, no open unanswered questions           |
| Export     | HTML report written to `.cursor/docs/romano-method/...`           |

---

## Confirmed Delivery

If evidence gate passes, first line must be exactly:

```text
Here we Go !!!
```

Then respond (caveman style) with:

```md
## Root Cause

## Trace

## Flow (success + error, color legend)

## Evidence

## Fix Direction

## Verification

## HTML
```

---

## Not Confirmed Delivery

If evidence gate does not pass, do **not** use `Here we Go !!!`.

Respond (caveman style) with:

```md
## Known So Far

## Most Likely Hypothesis

## Missing Evidence

## What Each Missing Item Unlocks

## Open Adversarial Questions

## Next Investigation Step

## HTML
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
- Write HTML report to `.cursor/docs/romano-method/`

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
| Success-only flow                  | Success + error paths, color per edge              |
| Broad repo grep first              | Anchor → codegraph/semantic search → targeted grep |
| Whole domain catalog               | Only story parts touched by ask                    |
| Treat "all X" as one vague anchor  | Phase 1a: enumerate members from evidence, one desk each |
| Guess collection members from memory | Read project (codegraph/config/registry) with file:line |
| Plausible guessing                 | Evidence or repro                                  |
| Early conclusion                   | Evidence gate + adversarial recap                  |
| Wordy delivery                     | /caveman terse, substance intact                   |
| Scattered notes                    | Desk schema → merge → recap → HTML → deliver       |
| Refactor while debugging           | Diagnose first, fix later                          |

---

# Integration With Planning Skills

Use Romano Method before planning when current behavior unclear.

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
