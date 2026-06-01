---
name: romano-method
description: >
  Romano Method — Tier-1 anchor, desk split (boundary or story), parallel desks,
  binary-search flow, state mutation gap, wire merge, evidence gate, "Here we Go !!!"
  or request more data. Research, debug, locate code, incidents, /romano desk.
disable-model-invocation: true
---

# 🔍 Romano Method

**No linear flow-walk. No repo-wide grep.**

```
1 📍 Tier-1 anchor
1b 🪑 Desk split (2–4 sub-contexts, max 3 parallel)
2 ✂️ Binary-search flow per desk
3 🔬 Expectation vs Reality → The Gap
4b 🔗 Wire merge (main thread)
4 🚀 Here we Go !!!  OR  ask for more evidence
```

| Mode | When |
| ---- | ---- |
| **🪑 Desk** (default) | Incident, cross-service, multi-layer — use `Task` per desk |
| **👤 Solo** | One module, one hop — Phases 1–4 inline, no `Task` |

**Desk metaphor:** Tier-1 = wire tip. Each desk = one **boundary** (service/layer) or **story part** (domain slice the ask touches). Editor merges before conclusion.

---

## 📍 Phase 1 — Tier-1 anchor

Pick **one** anchor — never "read whole repo."

| Mode | Anchor |
| ---- | ------ |
| 🔥 Incident | UUID / TxID / Correlation ID → logs |
| 🔎 Research | question + symbol → `codegraph_context` |
| 🐛 Debug | error, stack, failing test, repro |
| 📌 Locate | symbol, API path, config key |

List what anchor touches → candidate desks. Don't deep-read all yet.

---

## 🪑 Phase 1b — Desk split

**One primary axis** per round. Cap **3** parallel desks.

| Axis | `kind` | Split by | Use when |
| ---- | ------ | -------- | -------- |
| **Boundary** | `boundary` | Gateway · Auth · Core · DB | Incident, cross-service |
| **Story** | `story` | **Parts this ask touches only** | "How does X work", feature build |

**Story rule:** Name parts in user language (e.g. draft · validation · persistence) — not whole domain catalog. Skip parts with no anchor link.

| Field | Content |
| ----- | ------- |
| `id` | slug: `gateway`, `draft-entity` |
| `kind` | `boundary` \| `story` |
| `label` | human name |
| `maps_to` | story: symbols, dirs, APIs |
| `hypothesis` | one sentence |
| `evidence` | logs, IDs, files |
| `depends_on` | desk `id` or `none` |

- Independent (`depends_on: none`) → launch desks in **one message**  
- Dependent → prerequisite desk first, pass report into brief  
- **Solo:** one sub-context — skip `Task`

### Desk brief (paste into `Task`)

```text
Romano desk: {id} | {kind}: {label}
Maps to: {maps_to} · Hypothesis: {hypothesis}
Anchor: {evidence} · Depends: {depends_on}

Phases 2–3 only in scope:
- Binary-search flow; verify contract at midpoint; discard clean half.
- Expectation vs Reality → The Gap (file:line).

Tools: Read, Grep, Glob, SemanticSearch, codegraph/debug_issue, read-only Shell.
No Write/git/deploy. No nested Task.

Return:
## Desk: {id} ({label})
- Midpoint checked:
- Contract: pass | fail | unknown
- Expectation vs Reality:
- The Gap:
- Evidence:
- Confidence: high | medium | low
```

---

## ✂️ Phase 2 — Flow chopping

Flow `A→B→C→D→E→F`, fail at `F`:

1. Check midpoint **C or D** — not `A` first  
2. Contract OK? → search second half. Broken? → first half  
3. Repeat — halve each step  

---

## 🔬 Phase 3 — State mutation

In suspect block: `debug_issue` (narrowed) or `codegraph_explore` + `Read`.

**The Gap** = where Expectation (code says X) ≠ Reality (log/DB shows Y).

---

## 🔗 Phase 4b — Wire merge (main only)

| Signal | Action |
| ------ | ------ |
| All `pass` until one `fail` | Bug at/after that boundary |
| Conflicting gaps | Binary-search **handoff** between desks |
| One `high`, rest `low` | Weight high |
| All `low` / `unknown` | **No** `Here we Go !!!` — request more |

Output: one merged hypothesis + owning desk(s).

---

## 🚀 Phase 4 — Delivery

**Evidence gate** (all pass):

| Check | Need |
| ----- | ---- |
| Claims | log, file:line, repro, test, or graph cite |
| Merge | no unresolved desk conflict |
| Hypothesis | **confirmed** — not plausible |
| Confidence | no `low`/`unknown` carrying conclusion |

**Pass gate** — first line **exactly:**

```text
Here we Go !!!
```

Then: root cause / trace / locate path with cites.

**Fail gate** — do **not** use opener. Give: what's known · **specific** missing items · what each unlocks.

| Mode | Confirm with |
| ---- | ------------- |
| Incident / debug | mini-sim or single test ↔ logs |
| Research / locate | path + symbol + 1–2 line cite |

---

## 🛠️ Tools (strict)

| ✅ Allowed | For |
| ---------- | --- |
| Read, Grep, Glob, SemanticSearch | code, logs |
| user-codegraph | Phases 1–3 |
| user-code-review-graph / `debug_issue` | Phase 3 narrowed |
| Task | Desk mode — Phases 2–3 per desk |
| Shell read-only | jq, logs; Phase 4 one repro/test |

| ❌ Not unless user asks |
| ----------------------- |
| Write, git, deploy, broad refactor |

**Locate solo:** `codegraph_context` + one `codegraph_explore` before wide Grep.

---

## 🧠 Mindset

| ❌ | ✅ |
| -- | -- |
| Read A→Z linear | Binary-search per desk |
| Repo grep locate | codegraph first |
| Whole domain | Story parts **this task** touches |
| Guess | Evidence or repro |
| `Here we Go !!!` early | Request missing data |
| Scattered notes | Desk schema → merge → deliver |
