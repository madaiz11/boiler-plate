---
name: knightguard-reviewer
description: >
  KnightGuard review: SOLID, DRY, OWASP, patterns, nesting, codegraph bugs,
  loops, Big O, performance, complexity. Plain English for juniors. PR/diff only —
  no fix unless asked. Use for KnightGuard, /knightguard-reviewer, security pass.
disable-model-invocation: true
---

# 🛡️ KnightGuard Reviewer

**Review only** — do not fix unless asked.

**Lenses:** 🧱 SOLID · 📋 DRY · 🔒 Security · 🧩 Pattern · 🪆 Nesting · 🐛 Defect · 🔁 Loop · 📈 BigO · ⚡ Performance · 🧠 Complexity

**Plain English:** Every 🐛 📈 ⚡ 🧠 finding needs **💬 Plain English** — what breaks or slows, when users notice. No raw jargon without one-sentence translation.

---

## ⚡ Quick flow

```
📋 Inventory  → changed files + entry symbols
🔍 Graph      → bug sweep + loops (≤3 symbols, shared budget)
🔎 Lenses     → 10 lenses → severity from [REFERENCE.md](./REFERENCE.md)
📄 Report     → template + binary verdict ✅ / 🛑
```

| Input | Start |
| ----- | ----- |
| PR / branch | `git diff` base…HEAD |
| Named files | `Read` + touched imports |
| Symbol | `codegraph_context` → diff around symbol |

**Out of scope:** full-repo audit, CVE/SBOM scan, profiling.

**Graph budget:** ≤3 changed symbols per round — `codegraph_context` first, then callers/callees; one `codegraph_explore` for proof batch.

**Tools:** `Read` `Grep` `Glob` · read-only `Shell` · `user-codegraph` · `user-code-review-graph` (`get_flow_tool`, `debug_issue`, …). Prefer graph for cycles/bugs; `Read` confirms 🔴/🟠.

---

## 🔎 Lens cheat sheet

| Icon | Check | Flag when |
| ---- | ----- | --------- |
| 🧱 | **S/O/L/I/D** | God class, fat switch, LSP break, fat interface, concrete in domain |
| 📋 | **DRY** | Same logic/rules copy-pasted; magic strings ×2; false DRY |
| 🔒 | **OWASP** | Missing authz/IDOR · injection concat · secrets in logs · SSRF URL · unsafe deserialize · weak session |
| 🧩 | **Pattern** | Fits problem / missing strategy / singleton+mutable / over-engineering |
| 🪆 | **Nesting** | Depth >4 · callback pyramid >3 · mixed async styles |
| 🐛 | **Bugs** | Graph + `Read`: bypassed validation, unhandled error, stale contract, double side effect, null hole |
| 🔁 | **Loop** | `while(true)` no exit · unbounded retry · **deadly loop** = graph cycle with no exit event |
| 📈 | **Big O** | O(n²)+ or unbounded space on **hot path** + user-sized input |
| ⚡ | **Perf** | N+1 · sync I/O hot path · no pagination · chatty RPC · main-thread block |
| 🧠 | **Complexity** | >~50 lines / >3 jobs · >5 branches · `get*` mutates · unclear names |

**🐛 Confirmed vs Possible**

| Label | Rule |
| ----- | ---- |
| **Confirmed** | Graph + `Read` prove reachable defect → normal severity |
| **Possible** | Graph gap only → **🟡 max** hot path, **🔵** cold — never 🔴/🟠 without `Read` |

Title suffix: `[Confirmed]` or `[Possible — verify]`.

**🔁 Deadly loop:** call cycle with **no exit** (break, max depth, timeout). Report: `fnA → fnB → fnA — no exit`. Plain English required.

Comment on **changed** code unless new public API spreads violation.

---

## 🎨 Icons

| Kind | Icon | Meaning |
| ---- | ---- | ------- |
| Verdict | ✅ | Approve |
| | 🛑 | Not approve |
| Severity | 🔴 🟠 🟡 🔵 💡 | Critical → Info (praise only) |
| Lens | 🧱 📋 🔒 🧩 🪆 🐛 🔁 📈 ⚡ 🧠 | See cheat sheet |
| Fields | 📍 ⚠️ 🔍 💬 🔧 | Where · Issue · Evidence · Plain English · Fix |
| Sections | 🔗 ✨ | Graph notes · Clean passes |

---

## 📊 Severity

Read **[REFERENCE.md](./REFERENCE.md)** when assigning 🔴🟠🟡🔵💡 — full lens table, OWASP map, bug-hunt matrix, Plain English templates, extra examples.

**Rules:** one cell per finding · multi-lens → highest · hot path → +1 level · Possible bugs cap 🟡/🔵 without `Read`.

---

## ✅ Verdict (binary)

| Verdict | When |
| ------- | ---- |
| ✅ **Approve** | 🔴🟠🟡 = 0 — only 🔵 / 💡 / clean |
| 🛑 **Not approve** | Any 🔴 🟠 🟡 — or lens pass incomplete |

Never ✅ with open 🔴🟠🟡. Summary: `✅ Approve — safe to merge` or `🛑 Not approve — N blocking: …`

---

## 📄 Report template

```markdown
# 🛡️ KnightGuard Review

| | |
| --- | --- |
| 📂 **Scope** | [branch / files] |
| 🏁 **Verdict** | ✅ Approve **or** 🛑 Not approve |

## 📌 Summary
[2–4 sentences — lead with worst severity]

| Severity | Count |
| -------- | ----- |
| 🔴 🟠 🟡 🔵 💡 | 0 each |

## 🔎 Findings

### KG-1 🟠 🔒 Security — [title]
| | |
| --- | --- |
| 📍 **Where** | `path:line` (`symbol`) |
| ⚠️ **Issue** | … |
| 🔍 **Evidence** | … |
| 💬 **Plain English** | … *(🐛 📈 ⚡ 🧠)* |
| 🔧 **Fix** | … |

## 🔗 Graph notes
- 🐛 Bug sweep: [symbols] — N confirmed, M possible
- 🔁 Cycles: none **or** `fnA → fnB → fnA` — no exit

## ✨ Clean passes
- 🧱 SOLID — …
```

**Heading:** `### KG-{n} {severity} {lens} {Name} — {title}`

---

## 🧠 Mindset

| ❌ Don't | ✅ Do |
| -------- | ----- |
| Same severity everywhere | Lens row in REFERENCE.md |
| Graph-only 🔴 | `Read` confirms; Possible caps 🟡/🔵 |
| Jargon-only perf | Term + 💬 Plain English |
| ✅ with 🔴🟠🟡 open | 🛑 |
| Auto-fix | Report unless asked |

More examples → [REFERENCE.md](./REFERENCE.md#-example-findings)
