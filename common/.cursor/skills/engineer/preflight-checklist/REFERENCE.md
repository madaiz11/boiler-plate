# Preflight Checklist — Reference

HTML templates, section IDs, scoring rubric, and checklist pass criteria.

---

## Page skeleton

Every artifact page uses this structure:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Preflight — [Phase] — [Feature Name]</title>
  <link rel="stylesheet" href="assets/preflight.css" />
</head>
<body>
  <nav class="preflight-nav">
    <a href="index.html">Dashboard</a>
    <span class="phase-badge">Phase N · [Name]</span>
  </nav>
  <main class="preflight-doc">
    <header class="doc-header">
      <h1>[Document Title]</h1>
      <p class="meta">Flight ID: [id] · Mode: Full | Lite</p>
    </header>
    <!-- sections -->
  </main>
  <footer class="preflight-footer">
    <a href="[next].html">Next: [Phase] →</a>
  </footer>
</body>
</html>
```

---

## index.html sections

| Section ID | Content |
|------------|---------|
| `#flight-summary` | Feature name, mode, raw requirement one-liner |
| `#verdict-banner` | Ready / Not Ready / In Progress |
| `#readiness-summary` | Score ring (after Confirmation) |
| `#phase-checklist` | Phases 1–9 + Confirmation with status |
| `#quick-links` | Links to all artifacts |

Phase status classes: `is-pass`, `is-fail`, `is-skip`.

---

## dispatch.html sections

| Section ID | Content |
|------------|---------|
| `#problem-statement` | Pain, who hurts, why now |
| `#users` | Primary users / actors |
| `#success-metrics` | Measurable success criteria |
| `#non-goals` | Explicit out-of-scope |
| `#dependencies` | Teams, systems, data, APIs |
| `#impact-hypothesis` | Expected impact on existing systems (pre–aircraft-status) |
| `#top-risks` | Highest risks and unknowns |
| `#release-strategy` | Deploy type, rollback, env order, blockers |
| `#dispatch-checklist` | 7-item gate with pass/fail |
| `#dispatch-open-questions` | Questions blocking dispatch |

### Dispatch checklist pass criteria

| Item | Pass | Fail |
|------|------|------|
| Problem statement ชัด | Pain + who + why now stated | Solution-first or vague |
| User flow หลักชัด | One happy path: actor → action → outcome | No clear end-to-end flow |
| Success criteria ชัด | Verifiable metrics | Subjective only ("works", "faster") |
| Non-goals ระบุแล้ว | At least one explicit non-goal | None listed |
| Dependency ระบุแล้ว | Known deps listed or "none" justified | Unknown deps ignored |
| Risk / unknown ระบุแล้ว | At least one risk + unknown | No risks listed |
| Release strategy คิดไว้แล้ว | High-level deploy/rollback approach | No rollout thought |

Gate: **7/7 required**. Partial → stop, ask user, update dispatch.html.

---

## aircraft-status.html sections

| Section ID | Content |
|------------|---------|
| `#repo-structure` | Top-level layout summary |
| `#affected-modules` | Table: module, role, relevance to this feature |
| `#known-bugs` | Open defects / known issues in affected modules |
| `#api-stability` | API contracts, versioning, breaking-change risk |
| `#data-model-constraints` | Schema, indexes, FK, migration limits |
| `#test-coverage` | Existing tests for affected paths; gaps |
| `#blocking-tech-debt` | Tech debt that blocks or dangerously complicates feature |
| `#temporary-workarounds` | Hacks in place to respect or replace |
| `#patterns-to-reuse` | High-level conventions (detail in walkaround.html) |
| `#impact-confirmed` | Confirm/refute dispatch `#impact-hypothesis` |
| `#blocking-issues` | Issues that block feature — no valid MEL |
| `#mel-accepted` | Known issues accepted for release (MEL cards) |
| `#aircraft-status-gate` | Overall gate verdict |

### Software MEL (per accepted issue)

Each `.mel-entry` inside `#mel-accepted`:

```html
<article class="mel-entry">
  <h3>MEL-001: [issue title]</h3>
  <p class="mel-description">...</p>
  <ul class="checklist mel-checklist">
    <li class="checklist-item is-pass">ไม่กระทบ critical path</li>
    <li class="checklist-item is-pass">มี workaround</li>
    <li class="checklist-item is-pass">มี owner</li>
    <li class="checklist-item is-pass">มี follow-up task</li>
    <li class="checklist-item is-pass">มี monitoring หรือ guardrail</li>
  </ul>
  <p class="mel-verdict verdict--ready">MEL valid — 5/5</p>
</article>
```

### Software MEL pass criteria

| Item | Pass | Fail |
|------|------|------|
| ไม่กระทบ critical path | Issue off main user/data flow for this feature | Blocks or corrupts critical path |
| มี workaround | Documented workaround exists | No workaround |
| มี owner | Named owner or team | Unowned |
| มี follow-up task | Linked task ID or ticket | No follow-up planned |
| มี monitoring หรือ guardrail | Alert, flag, guard, or test in place | Silent failure possible |

Each accepted issue requires **5/5**. Incomplete MEL → stop, ask user.

### Blocking vs MEL

| Type | Action |
|------|--------|
| Blocks feature, no MEL possible | `#blocking-issues` → stop, ask user |
| Known issue, acceptable with MEL | `#mel-accepted` with 5/5 checklist |
| Unknown / not inspected | Document in inspection section — do not skip |

---

## crew-briefing.html sections

### Build Briefing

| Section ID | Field |
|------------|-------|
| `#build-briefing` | Container for mini brief |
| `#brief-goal` | Goal |
| `#brief-main-flow` | Main flow |
| `#brief-edge-cases` | Edge cases |
| `#brief-technical-approach` | Technical approach |
| `#brief-risk` | Risk |
| `#brief-test-strategy` | Test strategy |
| `#brief-rollback` | Rollback |
| `#brief-owner` | Owner |

Example structure:

```html
<section id="build-briefing">
  <h2>Build Briefing</h2>
  <dl class="brief-fields">
    <dt>Goal</dt><dd id="brief-goal">...</dd>
    <dt>Main flow</dt><dd id="brief-main-flow">...</dd>
    <dt>Edge cases</dt><dd id="brief-edge-cases">...</dd>
    <dt>Technical approach</dt><dd id="brief-technical-approach">...</dd>
    <dt>Risk</dt><dd id="brief-risk">...</dd>
    <dt>Test strategy</dt><dd id="brief-test-strategy">...</dd>
    <dt>Rollback</dt><dd id="brief-rollback">...</dd>
    <dt>Owner</dt><dd id="brief-owner">...</dd>
  </dl>
</section>
```

Sources: Dispatch (`#problem-statement`, `#success-metrics`, `#release-strategy`), Aircraft Status (MEL, blockers, constraints), raw requirement.

### Crew alignment

| Section ID | Content |
|------------|---------|
| `#crew-alignment` | Container |
| `#expected-behavior` | What the system should do when the feature works |
| `#emergency-case` | Failure modes, fallback, stop conditions |
| `#decision-authority` | Who decides on conflict (named role/person) |
| `#definition-of-done` | Verifiable done criteria for this feature |

### Supporting sections

| Section ID | Content |
|------------|---------|
| `#assumptions` | Documented assumptions carried into spec |
| `#open-questions` | Unresolved items blocking alignment |
| `#integration-hints` | Cross-module touch points from aircraft status |
| `#crew-briefing-gate` | Pass/fail — all fields + alignment complete |

### Crew Briefing pass criteria

| Item | Pass | Fail |
|------|------|------|
| Goal | Clear, matches Dispatch problem | Vague or solution-only |
| Main flow | End-to-end path documented | Missing steps |
| Edge cases | At least key edges listed | None mentioned |
| Technical approach | High-level how, grounded in repo | Hand-wavy or ignores aircraft status |
| Risk | Top risks including MEL/blockers | Empty |
| Test strategy | What to test, not just "add tests" | Generic only |
| Rollback | Aligns with Dispatch release strategy | Missing |
| Owner | Named owner or explicit TBD + blocker | Unassigned with no plan |
| Expected behavior | Concrete expected outcomes | Ambiguous |
| Emergency case | Documented failure/fallback | Not addressed |
| Decision authority | Named decider | "The team" only |
| Definition of done | Verifiable checklist | Subjective only |

Gate: all Build Briefing fields + all four alignment items pass. Partial → stop, ask user.

Footer link: `walkaround.html`.

---

## walkaround.html sections

**Goal:** Map codebase touch points before spec/plan. Prevent jumping to new abstractions when project patterns already exist.

### Codebase Walkaround checklist

```html
<section id="codebase-walkaround">
  <h2>Codebase Walkaround</h2>
  <ul class="checklist">
    <li class="checklist-item is-pass">Entry point อยู่ไหน</li>
    <li class="checklist-item is-pass">Data flow ไปทางไหน</li>
    <li class="checklist-item is-pass">State อยู่ที่ไหน</li>
    <li class="checklist-item is-pass">API contract คืออะไร</li>
    <li class="checklist-item is-pass">Error handling เดิมทำยังไง</li>
    <li class="checklist-item is-pass">Pattern เดิมใน project คืออะไร</li>
    <li class="checklist-item is-pass">Test เดิมอยู่ตรงไหน</li>
  </ul>
</section>
```

### Detail sections (one per checklist item)

| Section ID | Checklist item | Document |
|------------|----------------|----------|
| `#entry-points` | Entry point อยู่ไหน | Files, routes, handlers, CLI commands |
| `#data-flow` | Data flow ไปทางไหน | Request → service → store → response path |
| `#state-location` | State อยู่ที่ไหน | DB, cache, session, client state, config |
| `#api-contracts` | API contract คืออะไร | Shapes, headers, versioning, consumers |
| `#error-handling` | Error handling เดิมทำยังไง | Patterns, middleware, error types |
| `#patterns-to-reuse` | Pattern เดิมใน project คืออะไร | Naming, layers, DI, modules to extend — **not reinvent** |
| `#existing-tests` | Test เดิมอยู่ตรงไหน | Test files, fixtures, coverage for affected paths |
| `#walkaround-gate` | Gate verdict | 7/7 required |

### Walkaround pass criteria

| Item | Pass | Fail |
|------|------|------|
| Entry point อยู่ไหน | Concrete paths/symbols for this feature | Vague "backend" / "frontend" only |
| Data flow ไปทางไหน | Traced path with file/module names | Missing steps |
| State อยู่ที่ไหน | State stores identified | Unknown state ownership |
| API contract คืออะไร | Contract documented or N/A justified | Assumed contract |
| Error handling เดิมทำยังไง | Existing pattern named | Will invent new pattern without reason |
| Pattern เดิมใน project คืออะไร | At least one pattern to reuse cited | No patterns — new abstraction planned |
| Test เดิมอยู่ตรงไหน | Test locations listed or gap noted | Tests not located |

Gate: **7/7 required** before Load Sheet. Partial → stop, ask user.

Footer link: `load-sheet.html` (Load Sheet).

---

## load-sheet.html sections

**Goal:** Weight and balance — scope and impact before formal spec.

### Feature Load Sheet

```html
<section id="feature-load-sheet">
  <h2>Feature Load Sheet</h2>
  <table class="req-table load-sheet-table">
    <thead>
      <tr><th>Area</th><th>Impact</th><th>Notes</th></tr>
    </thead>
    <tbody>
      <tr><td>UI</td><td><span class="impact impact--low">Low</span></td><td>...</td></tr>
      <tr><td>API</td><td><span class="impact impact--medium">Medium</span></td><td>...</td></tr>
      <tr><td>Database</td><td><span class="impact impact--high">High</span></td><td>...</td></tr>
      <tr><td>Auth / Permission</td><td>...</td><td>...</td></tr>
      <tr><td>Existing user flow</td><td>...</td><td>...</td></tr>
      <tr><td>Migration</td><td><span class="impact impact--yes">Yes</span></td><td>...</td></tr>
      <tr><td>Rollback difficulty</td><td><span class="impact impact--hard">Hard</span></td><td>...</td></tr>
    </tbody>
  </table>
</section>
```

Impact values:

| Area | Allowed values |
|------|----------------|
| UI, API, Database, Auth / Permission, Existing user flow | Low / Medium / High |
| Migration | Yes / No |
| Rollback difficulty | Easy / Medium / Hard |

Each row **Notes** column: evidence from walkaround / aircraft status (paths, modules, contracts).

### Aircraft classification

| Section ID | Content |
|------------|---------|
| `#aircraft-classification` | Light \| Medium \| Heavy banner |
| `#classification-rationale` | Why this class — cite load sheet rows |
| `#runway-requirements` | What Full-mode phases / extra prep needed |
| `#mode-decision` | Full or Lite; note if upgraded from Dispatch Lite |
| `#load-sheet-gate` | Pass — proceed to spec |

### Classification rules

**Heavy** (Full mode required — long runway):

- Any of UI, API, Database, Auth/Permission, Existing user flow = **High**
- **Migration Yes** AND any of Database, Auth/Permission, API ≥ **Medium**
- **Rollback Hard**
- Two or more of {Database, Auth/Permission, API, Existing user flow} ≥ **Medium**

**Medium** (Full mode required):

- Does not meet Heavy rules
- At least one **Medium** impact OR Migration Yes OR Rollback Medium

**Light** (Lite allowed):

- All impact areas **Low**
- Migration **No**
- Rollback **Easy**

If Dispatch chose Lite but classification is Medium or Heavy → **upgrade to Full** on `#mode-decision` and index.html.

Footer link: `spec.html` or `preflight-brief.html`.

---

## spec.html sections (sa-spec-workflow)

| Section ID | Heading |
|------------|---------|
| `#goal` | Goal |
| `#background` | Background |
| `#user-stories` | User Stories |
| `#functional-requirements` | Functional Requirements |
| `#non-functional-requirements` | Non-Functional Requirements |
| `#business-rules` | Business Rules |
| `#edge-cases` | Edge Cases |
| `#out-of-scope` | Out of Scope |
| `#success-criteria` | Success Criteria |
| `#assumptions` | Assumptions |
| `#open-questions` | Open Questions |

Use `<table class="req-table">` for requirements where appropriate.

---

## plan.html sections (sa-spec-workflow)

| Section ID | Heading |
|------------|---------|
| `#summary` | Summary |
| `#current-architecture` | Current Architecture |
| `#proposed-architecture` | Proposed Architecture |
| `#affected-modules` | Affected Modules |
| `#api-changes` | API Changes |
| `#data-model` | Data Model / Database Changes |
| `#ui-changes` | UI Changes |
| `#state-management` | State Management Changes |
| `#security` | Security / Permission Considerations |
| `#error-handling` | Error Handling |
| `#observability` | Observability / Logging |
| `#testing-strategy` | Testing Strategy |
| `#migration-rollout` | Migration / Rollout Plan |
| `#risks` | Risks |
| `#alternatives` | Alternatives Considered |
| `#open-technical-questions` | Open Technical Questions |

---

## tasks.html structure

```html
<section id="task-list">
  <article class="task-card" id="t001">
    <div class="task-card-header">T001: [title]</div>
    <div class="task-card-body">
      <dl>
        <dt>Scope</dt><dd>...</dd>
        <dt>Files</dt><dd>...</dd>
        <dt>Verification</dt><dd>...</dd>
        <dt>Risk</dt><dd>...</dd>
        <dt>Status</dt><dd><span class="checklist-item is-fail">Pending</span></dd>
      </dl>
    </div>
  </article>
</section>
```

Mark completed tasks with `is-pass` on status.

Created at **start of Phase 7 Pushback** (before first code line).

---

## pushback.html (Phase 7 — implementation log)

**Goal:** Leave the gate — track incremental build on the ground (not takeoff).

### Incremental steps (required order)

| Step ID | Step | Status |
|---------|------|--------|
| `#step-scaffold` | scaffold | Pending / Complete |
| `#step-core-logic` | core logic | Pending / Complete |
| `#step-integration` | integration | Pending / Complete |
| `#step-ui` | UI | Pending / Complete |
| `#step-error-states` | error states | Pending / Complete |
| `#step-tests` | tests | Pending / Complete |
| `#step-cleanup` | cleanup | Pending / Complete |

Each step section documents:

- Files changed
- Verification run
- Notes / risks found

```html
<section id="incremental-track">
  <h2>Pushback — Incremental Build</h2>
  <article class="pushback-step" id="step-scaffold">
    <h3>1. Scaffold</h3>
    <p class="step-status"><span class="checklist-item is-pass">Complete</span></p>
    <dl>
      <dt>Files</dt><dd>...</dd>
      <dt>Verification</dt><dd>...</dd>
    </dl>
  </article>
  <!-- repeat for each step -->
</section>
```

### Pushback principles (document in HTML)

| Section ID | Content |
|------------|---------|
| `#pushback-principles` | No airport-wide refactor; no mid-flight architecture change |
| `#risks-found` | Risks discovered during build |
| `#rebrief-required` | Yes/No — if big risk, link to updated crew-briefing.html |
| `#architecture-change` | None / Described — if change, document re-clearance |
| `#pushback-summary` | Steps complete X/7 |

### Re-brief trigger

Stop implementation and return to **Crew Briefing** when:

- Scope changes materially
- Architecture must change
- Unacceptable risk appears
- Decision authority conflict

Record in `#rebrief-required` and `#risks-found`.

Footer link: `engine-start.html`.

---

## engine-start.html (Phase 8 — prove engine starts)

**Goal:** Minimum smoke test — feature turns on; engine does not flameout. **Not** full polish.

### Engine Start checklist

```html
<section id="engine-start">
  <h2>Engine Start</h2>
  <ul class="checklist">
    <li class="checklist-item is-pass">Local run ได้</li>
    <li class="checklist-item is-pass">Happy path ทำงาน</li>
    <li class="checklist-item is-pass">Core API connect ได้</li>
    <li class="checklist-item is-pass">Data shape ถูก</li>
    <li class="checklist-item is-pass">Error แรก ๆ ถูกจับได้</li>
  </ul>
</section>
```

### Detail sections

| Section ID | Checklist item | Document |
|------------|----------------|----------|
| `#local-run` | Local run ได้ | Start command, env, port, success signal |
| `#happy-path` | Happy path ทำงาน | Steps exercised, expected vs actual |
| `#core-api` | Core API connect ได้ | Endpoint, auth, response status |
| `#data-shape` | Data shape ถูก | Sample payload matches contract / schema |
| `#basic-errors` | Error แรก ๆ ถูกจับได้ | One invalid input or failure mode handled |
| `#not-polished` | Explicitly deferred | What was NOT done yet (OK for this phase) |
| `#engine-start-gate` | Verdict | Running \| Flameout |

### Pass criteria

| Item | Pass | Fail |
|------|------|------|
| Local run ได้ | App/service starts locally without crash | Cannot start or build fails |
| Happy path ทำงาน | Primary user flow completes once | Happy path broken |
| Core API connect ได้ | Critical integration responds | Core API unreachable or wrong |
| Data shape ถูก | Request/response matches expected shape | Wrong schema / missing fields |
| Error แรก ๆ ถูกจับได้ | At least one error path handled visibly | Silent failure or unhandled throw |

Gate: **5/5** before Taxi. **Flameout** → stop, return to Pushback fixes or re-brief.

### Principles (in HTML)

```html
<section id="engine-start-principles">
  <p>Not full polish. Prove the engine starts — no flameout.</p>
</section>
```

Footer link: `taxi-check.html`.

---

## preflight-brief.html (Lite mode)

```html
<section id="preflight-brief">
  <section id="load-sheet-summary">
    <h2>Load Sheet Summary</h2>
    <p>Aircraft: <strong>Light</strong> · Link to <a href="load-sheet.html">full load sheet</a></p>
  </section>
  <details class="accordion-panel" open>
    <summary>Spec (condensed)</summary>
    <div class="panel-body">...</div>
  </details>
  <details class="accordion-panel">
    <summary>Plan (condensed)</summary>
    <div class="panel-body">...</div>
  </details>
  <details class="accordion-panel">
    <summary>Tasks (condensed)</summary>
    <div class="panel-body">...</div>
  </details>
</section>
```

---

## clearance.html (Phase 6 — ATC decision gate)

**Goal:** Decide whether to proceed to Pushback. Lean by default; full design review only for **Heavy** aircraft.

### Clearance checklist

```html
<section id="atc-clearance">
  <h2>Clearance</h2>
  <ul class="checklist">
    <li class="checklist-item is-pass">Product direction accepted</li>
    <li class="checklist-item is-pass">Technical approach accepted</li>
    <li class="checklist-item is-pass">Risk accepted</li>
    <li class="checklist-item is-pass">Release window accepted</li>
    <li class="checklist-item is-pass">Rollback path accepted</li>
  </ul>
</section>
```

| Item | Pass | Fail |
|------|------|------|
| Product direction accepted | Aligns with crew-briefing goal; stakeholders aligned | Direction conflicts spec or dispatch |
| Technical approach accepted | Matches walkaround patterns; load sheet feasible | Approach ignores existing patterns or MEL |
| Risk accepted | Risks named and acceptable | Unacceptable risk unmitigated |
| Release window accepted | Window known or N/A with reason | Cannot ship in any planned window |
| Rollback path accepted | Rollback from load sheet / crew-briefing viable | Rollback Hard with no plan |

Gate: **5/5** before Cleared for Pushback.

### Build now decision

| Section ID | Content |
|------------|---------|
| `#build-now-decision` | Answer: build now \| over-design \| defer \| unclear |
| `#build-now-rationale` | Evidence from spec + load sheet |

| Answer | Verdict impact |
|--------|----------------|
| build now | Eligible for Cleared if 5/5 |
| over-design | **Withheld** — do not Pushback |
| defer | **Withheld** |
| unclear | Stop, ask user |

### Clearance verdict

| Section ID | Content |
|------------|---------|
| `#clearance-verdict` | Cleared for Pushback \| Cleared with conditions \| Withheld |
| `#clearance-conditions` | Required fixes before/during Pushback (if conditions) |
| `#withhold-reason` | Why Pushback blocked (if withheld) |

Banner mapping:

- Cleared for Pushback → `verdict--ready`
- Cleared with conditions → `verdict--go-with-changes`
- Withheld → `verdict--no-go`

Footer link: `plan.html` only when Cleared; otherwise `confirmation.html` or hold.

### Design review (Heavy aircraft only)

When `load-sheet.html` classifies **Heavy**, append sections from **sa-design-review.mdc** below the ATC checklist:

| Section ID | Heading |
|------------|---------|
| `#design-review-summary` | Summary |
| `#missing-requirements` | Missing Requirements |
| `#ambiguities` | Ambiguities |
| `#architecture-risks` | Architecture Risks |
| `#api-contract-risks` | API / Contract Risks |
| `#data-migration-risks` | Data / Migration Risks |
| `#security-risks` | Security Risks |
| `#error-handling-gaps` | Error Handling Gaps |
| `#observability-gaps` | Observability Gaps |
| `#test-gaps` | Test Gaps |
| `#rollout-risks` | Rollout / Rollback Risks |
| `#over-engineering` | Over-Engineering Concerns |
| `#recommended-changes` | Recommended Changes |
| `#design-review-go-no-go` | Go / No-Go for design (Heavy only) |

If design review **No-Go** on Heavy aircraft → **Withheld** even if ATC checklist 5/5.

Light/Medium aircraft: omit design review sections — lean clearance is sufficient.

---

## before-takeoff.html (Phase 10 — sa-design-review)

Required sections (each finding as `.review-finding` with severity class):

| Section ID | Heading |
|------------|---------|
| `#summary` | Summary |
| `#missing-requirements` | Missing Requirements |
| `#ambiguities` | Ambiguities |
| `#architecture-risks` | Architecture Risks |
| `#api-contract-risks` | API / Contract Risks |
| `#data-migration-risks` | Data / Migration Risks |
| `#security-risks` | Security Risks |
| `#error-handling-gaps` | Error Handling Gaps |
| `#observability-gaps` | Observability Gaps |
| `#test-gaps` | Test Gaps |
| `#rollout-risks` | Rollout / Rollback Risks |
| `#over-engineering` | Over-Engineering Concerns |
| `#recommended-changes` | Recommended Changes |
| `#go-no-go` | Go / No-Go verdict banner |

Go / No-Go banner classes:

- Go → `verdict--ready`
- Go with changes → `verdict--go-with-changes`
- No-Go → `verdict--no-go`

---

## taxi-check.html sections

| Section ID | Content |
|------------|---------|
| `#dependency-order` | ASCII diagram in `<pre class="diagram">` |
| `#module-crosscheck` | Tasks vs walkaround entry points, patterns, and data flow |
| `#mel-crosscheck` | Tasks vs aircraft-status MEL and blockers |
| `#external-deps` | API / DB / third-party order |
| `#rollout-deps` | Feature flags, migration order |
| `#taxi-checklist` | Pre-takeoff integration checklist |
| `#taxi-principles` | No whole-airport refactor; integration-only fixes |

### Taxi principles

- Do not refactor unrelated modules during Taxi
- Do not change architecture — return to Crew Briefing + Clearance if needed
- Integration and wiring fixes only; defer broad refactors to a new preflight

---

## confirmation.html sections

| Section ID | Content |
|------------|---------|
| `#verdict` | Ready for Takeoff / Not Ready banner |
| `#readiness-score` | Ring + percentage |
| `#score-breakdown` | Weighted table with score bars |
| `#blockers` | Critical items — `.severity--critical` cards |
| `#gaps` | Non-blocking — `.severity--warning` cards |
| `#open-questions` | `.severity--info` cards |
| `#next-step` | Suggested action |

---

## Readiness scoring rubric

Weighted checklist. Score each area 0–100, then compute weighted total.

| Area | Weight | Score guidance |
|------|--------|----------------|
| Requirements clarity | 25% | Load sheet + crew-briefing + spec/brief complete; aircraft class matches scope |
| Architecture fit | 20% | Plan reuses walkaround patterns; aircraft status clear; no unjustified new abstractions |
| Task implementability | 20% | Pushback complete; Engine Start 5/5 smoke (engine running) |
| Security & permissions | 15% | Auth, validation, sensitive data addressed |
| Testing & verification | 10% | Test strategy specific, not generic |
| Rollout / rollback | 10% | Clearance rollback accepted; migration/rollback plan exists |

**Ready for Takeoff:** total ≥ 85% AND zero blockers (implementation on ground complete; OK to release/deploy/merge per user approval).

**Blocker examples:**

- Clearance **Withheld** (checklist, build-now, or over-design)
- Clearance design review **No-Go** (Heavy aircraft)
- Before Takeoff **No-Go**
- Pushback incomplete (incremental steps not done)
- Engine Start checklist not 5/5 (flameout)
- Architecture changed during Pushback/Taxi without re-clearance
- Blocking issue in aircraft-status without valid MEL
- MEL accepted issue with incomplete 5/5 checklist
- Walkaround checklist incomplete (7/7 required)
- Heavy aircraft proceeding in Lite mode without upgrade
- Load sheet impact ratings contradict walkaround evidence
- Crew Briefing gate incomplete (missing owner, definition of done, or alignment item)
- Dispatch checklist incomplete
- Unresolved critical open question
- Missing success criteria
- Breaking API change without migration plan
- Security gap flagged critical in review

**Gap examples (non-blocking):**

- Optional observability improvements
- Nice-to-have test coverage
- Minor ambiguity with documented assumption

---

## Lite vs Full triggers

**Lite** when change matches sa-spec-workflow exclusions:

- typo fixes
- small CSS changes
- simple renames
- one-line bug fixes

**Full** for everything else (features, API, DB, auth, multi-file, refactor, migration).

When uncertain, choose Full.

---

## Copy CSS on Dispatch

On Phase 1, copy from skill template:

```txt
Source: .cursor/skills/preflight-checklist/assets/preflight.css
Target: .cursor/preflight-report/<id>/assets/preflight.css
```

If skill path unavailable in target project, inline minimal styles from preflight.css into each page as fallback.
