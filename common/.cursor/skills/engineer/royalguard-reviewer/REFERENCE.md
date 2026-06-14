# 👑 RoyalGuard Reference

Lookup when assigning severity during lens pass. Main workflow: [SKILL.md](./SKILL.md).

---

## 📊 Severity by lens

Pick **one cell** per finding. Multi-lens → **highest** severity. Escalate **+1** on hot path (API, auth, payment, cron, UI render loop). **💡** = praise only, not problems.

| Lens               | 🔴 Critical                                                                | 🟠 High                                                                                    | 🟡 Medium                                                              | 🔵 Low                                   | 💡 Info                                       |
| ------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- | ---------------------------------------- | --------------------------------------------- |
| 🧱 **SOLID**       | God class mixing auth + billing + UI                                       | New public API violates D/I on core; OCP — edit core for every new type on hot path        | SRP drift — 2 jobs; DIP — domain imports concrete infra                | Minor structure nit                      | Clear SRP/OCP/DIP worth copying               |
| 📋 **DRY**         | Same security/billing rule copy-pasted with **inconsistent** fix           | Duplicated authz or validation at 2+ entry points                                          | Same business logic duplicated ≥2 places                               | Repeated constant/string                 | Shared helper extracted well                  |
| 🔒 **Security**    | Exploitable **without** extra preconditions (RCE, SQLi, IDOR, secret leak) | Exploitable with **common** preconditions (logged-in, guessed id, SSRF internal)           | Defense gap (rate limit, verbose error, log PII)                       | Hardening nit                            | Good authz, parameterized query, safe default |
| 🧩 **Pattern**     | Singleton/global mutable **security or money** state                       | Service locator / god object on critical path                                              | Missing strategy causing risky `switch` growth                         | Over-engineering suggestion              | Factory/Strategy/Adapter fits problem         |
| 🪆 **Nesting**     | Depth ≥7 blocking **critical** error handling (funds/auth)                 | Depth 5–6 callback hell on hot API / payment                                               | Depth 5–6 warm path; mixed async styles                                | Depth 4 — style-only if logic clear      | Clean early-return / async structure          |
| 🐛 **Defect**      | **Confirmed:** wrong money/state, data loss, auth bypass (graph + Read)    | **Confirmed:** hot-path unhandled error / duplicate charge / stale contract breaks callers | **Confirmed:** warm-path bug; **Possible:** validation gap on hot path | **Possible** cold path; minor null guard | Callers share validation path; no drift       |
| 🔁 **Loop**        | **Deadly loop** on prod worker, API, or UI main thread (no exit)           | Unbounded retry/poll on hot path; deadly loop confirmed                                    | `while`/retry missing cap warm; possible cycle — exit unclear          | Bounded loop but unclear exit            | Retry has max + backoff + idempotent          |
| 📈 **BigO**        | O(2ⁿ)+ or unbounded memory on request path                                 | O(n²)+ on hot API (nested loop orders × items)                                             | O(n²) warm or O(n) with very large typical n                           | O(n) on small bounded n                  | Optimal approach (hash map, single pass)      |
| ⚡ **Performance** | Sync block critical path; load-all OOM at prod scale                       | N+1 hot API; chatty RPC in request; main-thread UI block                                   | N+1 export/report; missing pagination; no cache on heavy work          | Micro-optimization                       | Batching, pagination, cache done right        |
| 🧠 **Complexity**  | >150 lines or >5 duties touching **auth/money**                            | >80 lines or >4 duties hot path; hidden side effect on `get*`                              | >50 lines or >5 branches; misleading names on changed API             | Long fn but readable                     | Small focused fn — good example               |
| 🗃️ **DataCompat** | New code corrupts or misreads old-version data causing data loss/wrong state | No backward-compatible path and no safe skip+log on hot path; batch/request crashes on obsolete data | Partial compatibility works but important old-version shapes fail; skip exists but weak logging/observability | Safe skip+log only, no compatibility path | Old data normalized/mapped to near-new behavior (full parity not required) |
| 🤨 **Weirdness** | Weirdness on auth/money/state path masks or bypasses correct checks | Confirmed weirdness on hot path (useless caller, nested fn, no-op wrapper, intent mismatch) | **Default:** any confirmed weirdness in changed code — must fix before merge | **Not used** — do not waive weirdness as Low | Changed scope scanned; no weirdness signals |

### Global fallback (no lens row fits)

| Icon | Use when                                               |
| ---- | ------------------------------------------------------ |
| 🔴   | Data loss, prod-down, exploitable, deadly loop on prod |
| 🟠   | Hot-path breakage likely under real load               |
| 🟡   | Real issue on warm path; fix can follow soon           |
| 🔵   | Optional polish                                        |
| 💡   | Positive only                                          |

---

## 🔒 OWASP signals (map tag when reporting)

| Area               | Code signals                                              |
| ------------------ | --------------------------------------------------------- |
| **A01 Access**     | Missing authz; IDOR — client `id` trusted                 |
| **A02 Crypto**     | Secrets in code/logs; weak algo; PII cleartext            |
| **A03 Injection**  | SQL/NoSQL/command from string concat; unsanitized shell   |
| **A04 Design**     | Client sets price/role; no rate limit on sensitive action |
| **A05 Misconfig**  | `debug=true` prod; permissive CORS; default creds         |
| **A06 Components** | Diff bumps dep — note version only                        |
| **A07 Auth**       | Weak session; credential in URL                           |
| **A08 Integrity**  | Unsigned webhooks; unsafe deserialize (`pickle`, `eval`)  |
| **A09 Logging**    | Secrets logged; failures swallowed, no audit              |
| **A10 SSRF**       | User URL fetched server-side without allowlist            |

Security findings: full sentence + OWASP tag — not one-liners.

---

## 🐛 Bug hunt (graph → Read)

| Bug class              | Graph signal                                                 | Read confirms                 |
| ---------------------- | ------------------------------------------------------------ | ----------------------------- |
| Bypassed validation    | Caller skips `validate*` / `authorize*` other paths use      | Caller body omits check       |
| Missing error handling | Callee throws/returns error; caller no catch on all paths    | Unhandled rejection           |
| Wrong order (TOCTOU)   | `mutate*` before `check*` / `lock*`                          | State changed before auth     |
| Stale contract         | `codegraph_impact` lists callers; return/nullability changed | Callers assume old shape      |
| Double side effect     | Multiple callers invoke `save*` / `charge*` for one action   | Duplicate charge/event        |
| Null hole              | Optional callee; callers chain property access               | No `?.` / guard on hot caller |
| Leaked trust           | Internal DB/HTTP helper now reachable from new public caller | Was private only              |
| Inconsistent branch    | One caller passes `userId`, one client `id` only             | IDOR / wrong tenant           |
| Hidden mutation        | `get*` / `fetch*` calls `update*` / `write*` in callees      | Side effect in read API       |
| Re-entrancy            | Back-edge in callers∪callees; handler re-fires event         | Sync re-trigger no guard      |

---

## 🗃️ DataCompat (Migration / Backward Compatible)

Use this lens when changes affect:

- persisted schema/data shape
- payload versioning/contracts
- serializer/deserializer behavior
- migrations, backfills, import/export pipelines

Expected handling levels:

- Best case: old/obsolete versions are normalized or mapped so behavior is close to new-version data (not required to be 100% identical)
- Acceptable fallback: explicit backward-compatible read path for old versions
- Minimum baseline: obsolete/unreadable data is skipped safely and emits structured warning/error logs based on team agreement

Fail conditions to flag:

- decode/parse failure on old data crashes request, worker, or batch loop
- unknown data version is silently dropped with no observability
- migration path mutates data into ambiguous or lossy shape without guardrails
- fallback path exists but is not deterministic/replay-safe for retry workflows

Plain English template:

- "Old data from previous versions can still be used, or is skipped safely with logs, so one bad record does not break the whole flow."

---

## 📈 Big O — Plain English templates

| Pattern                        | Complexity           | 💬 Plain English                                                |
| ------------------------------ | -------------------- | --------------------------------------------------------------- |
| Single pass                    | O(n)                 | "Double the list → roughly double the time."                    |
| Nested loop same collection    | O(n²)                | "Each item scans all items — 10× items ≈ 100× slower."          |
| Halving each step              | O(log n)             | "Huge lists stay fast — cut problem in half each step."         |
| Sort inside loop               | O(n log n) per outer | "Re-sort repeatedly — cost stacks as data grows."               |
| Unbounded cache keyed by input | O(n) space           | "Memory grows with every unique request — RAM risk under load." |

Skip O(n) nitpick on fixed arrays ≤10. Focus on scale product actually hits.

---

## ⚡ Performance — Plain English angles

| Signal                 | 💬 Plain English                                          |
| ---------------------- | --------------------------------------------------------- |
| N+1 queries            | "One DB trip per row — pages with many rows stall."       |
| Sync I/O hot path      | "Server waits on disk/network — users wait on spinner."   |
| No pagination          | "Load everything — slow first paint, heavy mobile data."  |
| No cache/memo          | "Same heavy calc every click though answer unchanged."    |
| Large clone in loop    | "Copy big objects repeatedly — CPU spikes under traffic." |
| Missing index          | "DB reads every row — worse as table grows."              |
| Chatty micro-calls     | "Many round-trips instead of one — latency adds up."      |
| Main-thread block (UI) | "Heavy work on UI thread — page freezes until done."      |

---

## 📝 Example findings

### Confirmed defect

```markdown
### RG-1 🟠 🐛 Defect — Validation bypass [Confirmed]

|                      |                                                                         |
| -------------------- | ----------------------------------------------------------------------- |
| 📍 **Where**         | `api/webhook.ts:28` (`handleStripeEvent`)                               |
| ⚠️ **Issue**         | Calls `applySubscription` without `validateStripeSignature`.            |
| 🔍 **Evidence**      | codegraph: no validator in callees; Read: `handleHttp` validates first. |
| 💬 **Plain English** | Fake payment event could change a plan — this door skips the ID check.  |
| 🔧 **Fix**           | Validate before `applySubscription`; match `handleHttp`.                |
```

### Possible defect

```markdown
### RG-2 🟡 🐛 Defect — Nullable return not guarded [Possible — verify]

|                      |                                                                                 |
| -------------------- | ------------------------------------------------------------------------------- |
| 📍 **Where**         | `services/user.ts:55` (`findUserById`)                                          |
| ⚠️ **Issue**         | Return now nullable; 2 of 4 callers still chain `.email`.                       |
| 🔍 **Evidence**      | codegraph_impact: `sendDigest`, `exportRow`. **Verify:** null check after call. |
| 💬 **Plain English** | Missing user → crash when sending email.                                        |
| 🔧 **Fix**           | Guard at each caller or keep non-null contract with explicit throw.             |
```

### Big O · Performance · Loop

```markdown
### RG-3 🟠 📈 BigO — Nested filter on orders

| 💬 **Plain English** | 100 orders × 50 items → thousands of comparisons — list slows as history grows. |

### RG-4 🟡 ⚡ Performance — N+1 load users

| 💬 **Plain English** | 500 rows → 500 DB trips — exports time out on real data. |

### RG-5 🟠 🔁 Loop — Deadly loop (no exit)

| 💬 **Plain English** | Worker spins forever waiting — never moves to next job. |
| 🔍 **Evidence** | 🔁 `pollUntilReady → fetchStatus → pollUntilReady` — no break on 404. |
```

---

## 🤨 Appendix — Weirdness (จุดเอ๊ะ — important++)

**Zero-tolerance lens.** Mandatory on every review. Confirmed weirdness in the diff **blocks merge** (minimum 🟡). No 🔵 Low findings for this lens.

Use this appendix when code looks suspicious or purposeless even if it compiles and tests pass.

### Severity guide for Weirdness

| Lens             | 🔴 Critical | 🟠 High | 🟡 Medium (default) | 🔵 Low | 💡 Info |
| ---------------- | ----------- | ------- | ------------------- | ------ | ------- |
| 🤨 **Weirdness** | Masks/breaks auth, money, or state correctness | Hot-path weirdness (see signals below) | Any confirmed weirdness in **changed** code | **Not used** | No weirdness in changed scope after full scan |

**Default every confirmed 🤨 finding to 🟡.** Escalate to 🟠/🔴 per main table + hot-path +1 rule. Never approve with open 🤨 findings.

### Common "ทำไปทำไม" signals

| Signal | Why it is weird | Typical action |
| ------ | --------------- | -------------- |
| **Nested function** — inner `function`/`const fn` inside outer when inner does not use outer locals | extra scope layer; harder to test/reuse; often copy-paste | hoist to module scope or extract named helper in same file |
| **Useless caller** — `a(...args) => b(...args)` (sync/async) with identical signature and no added behavior | indirection with two names to maintain; drift risk | call `b` directly or give `a` a real job (validate, map errors, enforce policy) |
| `import type { AType } ...` then `export type { AType }` with no local use | adds indirection without behavior | inline `export type { AType } from ...` or justify as API barrel |
| Wrapper function only forwards args and return | no extra policy/validation/translation | remove wrapper or add real responsibility |
| Constant-guard dead branch (`if (false)`) in production path | code is unreachable and misleading | delete dead path or move to feature flag |
| Duplicate mapping/transformation with same output | extra code, unclear ownership | keep one source of truth |
| Name says read (`get*`) but mutates state | intent mismatch and hidden side effect | rename or split read/write responsibilities |

### Required finding fields for Weirdness

Keep standard finding format and include:

- `❓ Why this exists` — expected intent or **"unknown intent"** (unknown → treat as must-fix)
- `🔍 Evidence` — exact code snippet or call path
- `💬 Plain English` — why this blocks safe maintenance or misleads the next developer
- `🔧 Fix` — **required concrete change** (remove, inline, hoist, rename, single export path) — not “add a comment and keep as-is”

### Plain English templates for Weirdness

| Pattern | 💬 Plain English |
| ------- | ---------------- |
| Nested function (no closure need) | "This helper is defined inside another function but does not use variables from outside, so it is harder to find and test for no benefit." |
| Useless caller `a → b` | "Function A only calls B with the same arguments, so every change must be checked in two places even though one call would do the same thing." |
| Redundant import-export alias | "This line only re-exports a type without adding behavior, so it adds one more place to maintain for no user benefit." |
| No-op wrapper | "This wrapper does the same thing as the original call, so future changes can drift in two places." |
| Intent mismatch | "The function name says read, but it writes data, so developers may call it in places where side effects are unsafe." |

---

## 📁 HTML report export

Write before the review session ends. Path is always under the **reviewed code repo** git root:

```txt
<git-root>/.cursor/code-review-report/<YYYYMMDD-HHMMSS-story-slug>/
  index.html
  assets/royalguard.css
```

Copy CSS from `skills/royalguard-reviewer/assets/royalguard.css` in the skill package.

### Page skeleton (`index.html`)

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>RoyalGuard — [story slug]</title>
  <link rel="stylesheet" href="assets/royalguard.css" />
</head>
<body>
  <nav class="rg-nav">
    <span class="rg-badge">RoyalGuard Review</span>
    <span>[timestamp-story]</span>
  </nav>
  <main class="rg-doc">
    <header class="doc-header">
      <h1>👑 RoyalGuard Review</h1>
      <p class="meta">Scope: … · Repo: … · Generated: …</p>
    </header>
    <!-- sections — see IDs below -->
  </main>
  <footer class="rg-footer">RoyalGuard — review only, no auto-fix</footer>
</body>
</html>
```

### `index.html` sections

| Section ID | Content |
|------------|---------|
| `#verdict-banner` | `verdict--approve` or `verdict--not-approve` + one-line verdict text |
| `#summary` | 2–4 sentence summary; review limits |
| `#severity-counts` | `.count-grid` with `.count-card` per severity (critical/high/medium/low/info) |
| `#inventory` | `.meta-table`: scope, changed files, entry symbols, public API, hot paths, limits |
| `#findings` | One `.finding-card` per RG-n (classes: `sev-critical`, `sev-high`, `sev-medium`, `sev-low`, `sev-info`) |
| `#graph-notes` | Bug sweep symbols; cycle chains or "none" |
| `#clean-passes` | `.clean-list` of lenses with no material issues |

### Finding card structure

```html
<article class="finding-card sev-high" id="rg-1">
  <h3>RG-1 🟠 🔒 Security — [title]</h3>
  <table class="field-table">
    <tr><th>Where</th><td><code>path:line</code> (<code>symbol</code>)</td></tr>
    <tr><th>Issue</th><td>...</td></tr>
    <tr><th>Evidence</th><td>...</td></tr>
    <tr><th>Why this exists</th><td>...</td></tr>
    <tr><th>Plain English</th><td>...</td></tr>
    <tr><th>Fix</th><td>...</td></tr>
  </table>
</article>
```

Include **Plain English** row for 🐛 📈 ⚡ 🧠 🗃️ findings. OWASP tag in `<h3>` when applicable.

When there are no findings, `#findings` contains `<p>No blocking findings.</p>`.

### Verdict banner classes

| Verdict | Class |
|---------|-------|
| ✅ Approve | `verdict-banner verdict--approve` |
| 🛑 Not approve | `verdict-banner verdict--not-approve` |

---

## 🔗 Workflow boundary

RoyalGuard reviews code, diffs, changed files, and changed symbols.

Use:

- `romano-method` when investigation/root cause is unclear.
- `pirates-codes` when implementing fixes.
- `design-review.mdc` when reviewing specs, plans, migrations, or system design documents.
