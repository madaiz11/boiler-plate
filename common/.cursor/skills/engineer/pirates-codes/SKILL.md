---
name: pirates-codes
description: >
  Align to project (stop if misaligned), Penta + Rust contract once at Align,
  test design, table-driven AAA tests first, minimal impl, green run, Manifest.
  Use for implement features, Pirates Codes, Penta Course, guideline-driven code.
---

# 🏴‍☠️ Pirates Codes

**Author code** — not review. KnightGuard reviews; Pirates **ships**.

**Fixed order:** 🧭 Align + Penta **once** → 🗺️ test design → 🧪 tests first → ⚙️ minimal code → ✅ green → 📜 Manifest. **No Penta re-pass after implement.**

Never skip Align. Never ship prod code before tests.

---

## ⚡ Boarding

```
1 🧭 Align + Penta  → fit OR stop + better path; lock contract
2 🗺️ Test design    → happy + rules + errors from contract
3 🧪 Unit tests     → table-driven AAA; fail until impl exists
4 ⚙️ Implement      → minimal; follow contract only
5 ✅ Green           → all unit tests pass
6 📜 Deliver         → code + Manifest (copy contract)
```

| Ask | Do |
| --- | --- |
| Implement / scaffold | Full boarding |
| Refactor behavior | Re-Align + Penta if architecture changes |
| Plan → code | Penta contract per todo at Align |

**Out of scope:** CVE audit, load tests, full-repo rewrite.

---

## 1 🧭 Align

| Check | Action |
| ----- | ------ |
| Clear? | Restate req, I/O, acceptance |
| Fits project? | `Read` neighbors / `codegraph_context` |
| Implementable? | Real modules/APIs — no invented tables |
| Good approach? | No duplicate capability, wrong layer, 🔴 risk |

**🛑 Stop** (no tests/code) when misaligned, blocked, or bad approach. Reply with:

1. **Why stop** — cite `file:line` if possible  
2. **Better path** — extend X, use Y, smaller Z  
3. **Tradeoff**  
4. **Confirm** before step 2  

**Penta contract** (user-visible, rewrite if prompt changes):

```markdown
## Penta design contract (Align)

| Lens | Decision |
| ---- | -------- |
| 🧩 Conceptual | pattern or plain fns; where it lives |
| ⚖️ Severity | 🔴🟠🟡🔵 tier + fail-closed rules |
| ⚡ Performance | hot path? Big-O budget; batch/paginate |
| 🧱📋 SOLID+DRY | module owner; extend vs new |
| 📖 Simple | API shape; comment policy |
| 🦀 Rust mindset | ownership, errors, no hidden mutation |
```

### Penta lenses (Align only)

| Icon | Ask | Ship when |
| ---- | --- | --------- |
| 🧩 | Pattern earns keep? One primary or none | Team can name it |
| ⚖️ | If wrong at 2am — contained? | Typed errors; 🔴 fail closed at boundary |
| ⚡ | Prod-sized data hurt? | No unbounded work on user size without approval |
| 🧱📋 | One job per module; stable DRY only | No god module; one source for constants |
| 📖 | Junior gets file in one sentence? | ≤50 lines, ≤4 nest, ≤5 params; **no redundant comments** |

**Comments:** only complex invariant or **public** docblock (purpose, params, errors). Never restate code.

### 🦀 Rust mindset (any language)

| Idea | In code |
| ---- | ------- |
| Ownership | One owner per mutable state; no request globals |
| Result not panic | Typed errors on user/network input |
| Enums over flags | Invalid states unrepresentable |
| Parse don't validate | Strong types at boundary |
| No hidden mutation | `get*` doesn't write DB/events |

**Reject:** `any` through domain · unwrap on user input · global mutable singleton · clone in tight loops

---

## 2 🗺️ Test design

| Bucket | Cover |
| ------ | ----- |
| ✅ Happy | Each required outcome |
| 📏 Rules | Branch per rule, boundaries, state |
| ❌ Errors | Every ⚖️ Severity error from contract |
| 🔀 Edge | empty, null, max, duplicate, idempotency |

Output: numbered list `case | input | expected` before test files.

---

## 3 🧪 Tests first

**AAA** per subtest: Arrange → Act → Assert (one outcome).

**Table-driven** — match project style (`Read` `*_test.*`):

```go
for _, tt := range []struct{ name string; wantErr error }{...} {
    t.Run(tt.name, func(t *testing.T) { /* AAA */ })
}
```

Ship when tests exist and **fail for missing impl**, not syntax errors.

---

## 4 ⚙️ Implement · 5 ✅ Green

| Rule | Detail |
| ---- | ------ |
| Scope | Tests + prompt only |
| Contract | Align decisions — **no lens re-audit** |
| Extend | Existing symbols over new files |
| Green | Project test cmd — 100% touched suite before deliver |

Fail → fix impl; wrong test → update design, re-run from step 2.

---

## 6 📜 Deliver

**Pirates Manifest** (≤16 lines; Penta rows **copy** contract — do not re-derive):

```markdown
## Pirates Manifest

| Boarding | Status |
| -------- | ------ |
| ✅ Align | [where it lives / stopped → path] |
| ✅ Penta | [locked at Align — not re-checked] |
| ✅ Tests | [N cases] |
| ✅ Green | [cmd + pass] |

| Lens | From contract |
| ---- | ------------- |
| 🧩 ⚖️ ⚡ 🧱📋 📖 | [copy rows] |

**Rust mindset:** [copy] · **Drift:** [none | one line]
```

---

## ✅ Pre-delivery checklist

```
Align: fits project · contract written · user confirmed if stopped
Boarding: test map · tests before prod · minimal impl · green seen
Deliver: Manifest copies contract · no post-impl Penta
```

---

## 📜 Code rules

1. Align + contract → tests → prod → green  
2. Minimal diff; match neighbors  
3. Security: authz at handler; parameterized queries; secrets from env  
4. KnightGuard only if user asks  

**Related:** `strict-build` (plan TDD) · `romano-method` (can't find anchor) · `knightguard-reviewer` (post-green review)
