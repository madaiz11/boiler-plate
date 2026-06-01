# 🛡️ KnightGuard Reference

Lookup when assigning severity during lens pass. Main workflow: [SKILL.md](./SKILL.md).

---

## 📊 Severity by lens

Pick **one cell** per finding. Multi-lens → **highest** severity. Escalate **+1** on hot path (API, auth, payment, cron, UI render loop). **💡** = praise only, not problems.

| Lens | 🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low | 💡 Info |
| ---- | ----------- | ------- | --------- | ------ | ------- |
| 🧱 **SOLID** | God class mixing auth + billing + UI | New public API violates D/I on core; OCP — edit core for every new type on hot path | SRP drift — 2 jobs; DIP — domain imports concrete infra | Minor structure nit | Clear SRP/OCP/DIP worth copying |
| 📋 **DRY** | Same security/billing rule copy-pasted with **inconsistent** fix | Duplicated authz or validation at 2+ entry points | Same business logic duplicated ≥2 places | Repeated constant/string | Shared helper extracted well |
| 🔒 **Security** | Exploitable **without** extra preconditions (RCE, SQLi, IDOR, secret leak) | Exploitable with **common** preconditions (logged-in, guessed id, SSRF internal) | Defense gap (rate limit, verbose error, log PII) | Hardening nit | Good authz, parameterized query, safe default |
| 🧩 **Pattern** | Singleton/global mutable **security or money** state | Service locator / god object on critical path | Missing strategy causing risky `switch` growth | Over-engineering suggestion | Factory/Strategy/Adapter fits problem |
| 🪆 **Nesting** | Depth ≥7 blocking **critical** error handling (funds/auth) | Depth 5–6 callback hell on hot API / payment | Depth 5–6 warm path; mixed async styles | Depth 4 — style-only if logic clear | Clean early-return / async structure |
| 🐛 **Defect** | **Confirmed:** wrong money/state, data loss, auth bypass (graph + Read) | **Confirmed:** hot-path unhandled error / duplicate charge / stale contract breaks callers | **Confirmed:** warm-path bug; **Possible:** validation gap on hot path | **Possible** cold path; minor null guard | Callers share validation path; no drift |
| 🔁 **Loop** | **Deadly loop** on prod worker, API, or UI main thread (no exit) | Unbounded retry/poll on hot path; deadly loop confirmed | `while`/retry missing cap warm; possible cycle — exit unclear | Bounded loop but unclear exit | Retry has max + backoff + idempotent |
| 📈 **BigO** | O(2ⁿ)+ or unbounded memory on request path | O(n²)+ on hot API (nested loop orders × items) | O(n²) warm or O(n) with very large typical n | O(n) on small bounded n | Optimal approach (hash map, single pass) |
| ⚡ **Performance** | Sync block critical path; load-all OOM at prod scale | N+1 hot API; chatty RPC in request; main-thread UI block | N+1 export/report; missing pagination; no cache on heavy work | Micro-optimization | Batching, pagination, cache done right |
| 🧠 **Complexity** | >150 lines or >5 duties touching **auth/money** | >80 lines or >4 duties hot path; hidden side effect on `get*` | >50 lines or >5 branches; misleading names on changed API | Long fn but readable | Small focused fn — good example |

### Global fallback (no lens row fits)

| Icon | Use when |
| ---- | -------- |
| 🔴 | Data loss, prod-down, exploitable, deadly loop on prod |
| 🟠 | Hot-path breakage likely under real load |
| 🟡 | Real issue on warm path; fix can follow soon |
| 🔵 | Optional polish |
| 💡 | Positive only |

---

## 🔒 OWASP signals (map tag when reporting)

| Area | Code signals |
| ---- | ------------ |
| **A01 Access** | Missing authz; IDOR — client `id` trusted |
| **A02 Crypto** | Secrets in code/logs; weak algo; PII cleartext |
| **A03 Injection** | SQL/NoSQL/command from string concat; unsanitized shell |
| **A04 Design** | Client sets price/role; no rate limit on sensitive action |
| **A05 Misconfig** | `debug=true` prod; permissive CORS; default creds |
| **A06 Components** | Diff bumps dep — note version only |
| **A07 Auth** | Weak session; credential in URL |
| **A08 Integrity** | Unsigned webhooks; unsafe deserialize (`pickle`, `eval`) |
| **A09 Logging** | Secrets logged; failures swallowed, no audit |
| **A10 SSRF** | User URL fetched server-side without allowlist |

Security findings: full sentence + OWASP tag — not one-liners.

---

## 🐛 Bug hunt (graph → Read)

| Bug class | Graph signal | Read confirms |
| --------- | ------------ | ------------- |
| Bypassed validation | Caller skips `validate*` / `authorize*` other paths use | Caller body omits check |
| Missing error handling | Callee throws/returns error; caller no catch on all paths | Unhandled rejection |
| Wrong order (TOCTOU) | `mutate*` before `check*` / `lock*` | State changed before auth |
| Stale contract | `codegraph_impact` lists callers; return/nullability changed | Callers assume old shape |
| Double side effect | Multiple callers invoke `save*` / `charge*` for one action | Duplicate charge/event |
| Null hole | Optional callee; callers chain property access | No `?.` / guard on hot caller |
| Leaked trust | Internal DB/HTTP helper now reachable from new public caller | Was private only |
| Inconsistent branch | One caller passes `userId`, one client `id` only | IDOR / wrong tenant |
| Hidden mutation | `get*` / `fetch*` calls `update*` / `write*` in callees | Side effect in read API |
| Re-entrancy | Back-edge in callers∪callees; handler re-fires event | Sync re-trigger no guard |

---

## 📈 Big O — Plain English templates

| Pattern | Complexity | 💬 Plain English |
| ------- | ---------- | ---------------- |
| Single pass | O(n) | "Double the list → roughly double the time." |
| Nested loop same collection | O(n²) | "Each item scans all items — 10× items ≈ 100× slower." |
| Halving each step | O(log n) | "Huge lists stay fast — cut problem in half each step." |
| Sort inside loop | O(n log n) per outer | "Re-sort repeatedly — cost stacks as data grows." |
| Unbounded cache keyed by input | O(n) space | "Memory grows with every unique request — RAM risk under load." |

Skip O(n) nitpick on fixed arrays ≤10. Focus on scale product actually hits.

---

## ⚡ Performance — Plain English angles

| Signal | 💬 Plain English |
| ------ | ---------------- |
| N+1 queries | "One DB trip per row — pages with many rows stall." |
| Sync I/O hot path | "Server waits on disk/network — users wait on spinner." |
| No pagination | "Load everything — slow first paint, heavy mobile data." |
| No cache/memo | "Same heavy calc every click though answer unchanged." |
| Large clone in loop | "Copy big objects repeatedly — CPU spikes under traffic." |
| Missing index | "DB reads every row — worse as table grows." |
| Chatty micro-calls | "Many round-trips instead of one — latency adds up." |
| Main-thread block (UI) | "Heavy work on UI thread — page freezes until done." |

---

## 📝 Example findings

### Confirmed defect

```markdown
### KG-1 🟠 🐛 Defect — Validation bypass [Confirmed]
| | |
| --- | --- |
| 📍 **Where** | `api/webhook.ts:28` (`handleStripeEvent`) |
| ⚠️ **Issue** | Calls `applySubscription` without `validateStripeSignature`. |
| 🔍 **Evidence** | codegraph: no validator in callees; Read: `handleHttp` validates first. |
| 💬 **Plain English** | Fake payment event could change a plan — this door skips the ID check. |
| 🔧 **Fix** | Validate before `applySubscription`; match `handleHttp`. |
```

### Possible defect

```markdown
### KG-2 🟡 🐛 Defect — Nullable return not guarded [Possible — verify]
| | |
| --- | --- |
| 📍 **Where** | `services/user.ts:55` (`findUserById`) |
| ⚠️ **Issue** | Return now nullable; 2 of 4 callers still chain `.email`. |
| 🔍 **Evidence** | codegraph_impact: `sendDigest`, `exportRow`. **Verify:** null check after call. |
| 💬 **Plain English** | Missing user → crash when sending email. |
| 🔧 **Fix** | Guard at each caller or keep non-null contract with explicit throw. |
```

### Big O · Performance · Loop

```markdown
### KG-3 🟠 📈 BigO — Nested filter on orders
| 💬 **Plain English** | 100 orders × 50 items → thousands of comparisons — list slows as history grows. |

### KG-4 🟡 ⚡ Performance — N+1 load users
| 💬 **Plain English** | 500 rows → 500 DB trips — exports time out on real data. |

### KG-5 🟠 🔁 Loop — Deadly loop (no exit)
| 💬 **Plain English** | Worker spins forever waiting — never moves to next job. |
| 🔍 **Evidence** | 🔁 `pollUntilReady → fetchStatus → pollUntilReady` — no break on 404. |
```
