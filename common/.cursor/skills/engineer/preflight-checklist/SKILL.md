---
name: preflight-checklist
description: >
  Aviation-style preflight workflow for SDLC planning. Takes raw requirements
  or user stories, runs dispatch through before-takeoff phases, generates HTML
  spec/plan/tasks (or lite preflight-brief), ATC clearance gate, design review on
  heavy aircraft at clearance and full package at before-takeoff, outputs readiness
  incremental pushback implementation, and readiness confirmation for takeoff.
  Use when the user provides raw requirements, user stories, feature ideas, or
  asks for preflight, planning before code, or spec workflow.
disable-model-invocation: true
---

# Preflight Checklist

Orchestrates preflight through takeoff readiness. Outputs **HTML-only** artifacts under
`.cursor/preflight-report/<timestamp-story>/`.

## Do not

- Write production code **before Phase 7** (Phases 1–6 are planning only)
- Refactor unrelated modules during Pushback or Taxi
- Change architecture mid-implementation without clearance re-review
- Auto-deploy or release after Confirmation without user approval
- Write `.md` planning artifacts (HTML only)
- Dump full HTML in chat — summarize + give file paths

## Chain

```txt
preflight-checklist (this skill)
  ├── sa-spec-workflow.mdc      → spec.html, plan.html, tasks.html
  ├── sa-design-review.mdc      → clearance.html, before-takeoff.html
  └── sa-task-execution.mdc during Phase 7–8 (one task at a time)
```

Read [REFERENCE.md](REFERENCE.md) for HTML skeletons, CSS classes, scoring rubric, and per-phase templates.

## Report folder

```txt
.cursor/preflight-report/<timestamp-story>/
  index.html
  assets/preflight.css
  dispatch.html … engine-start.html … confirmation.html
```

**Folder name:** `YYYYMMDD-HHMMSS-kebab-case-slug` (e.g. `20260604-143022-customer-archive`).

Each re-run creates a **new folder**.

Copy `assets/preflight.css` from this skill into the report folder on Dispatch.

## Modes

| Mode | When | Load Sheet output | Phases skipped |
|------|------|-------------------|----------------|
| **Full** | Significant change or **Heavy/Medium aircraft** after load sheet | load-sheet.html → spec.html → plan/tasks at Pushback → code in Phase 7–8 | none |
| **Lite** | Light aircraft only (see Load Sheet rules) | load-sheet.html → preflight-brief.html | 6–9 |

Mode chosen at Dispatch is **provisional**. Load Sheet recalculates weight — upgrade Lite → Full when aircraft is not light.

Lite still runs **full Dispatch checklist** and **Walkaround checklist**. Lite collapses spec/plan/tasks into one file and skips Phases 6–9 (Clearance, Pushback, Engine Start, Taxi). Implementation after **Confirmation** is user-driven or a new preflight run if scope grows.

## Auto-run behavior

Run phases **1 → 10 → Confirmation** sequentially. Update `index.html` after each phase.

**Stop and ask the user** when:

- Dispatch checklist not 7/7
- Aircraft Status has blocking issues without valid MEL, or MEL entry incomplete
- Walkaround checklist not 7/7
- Clearance withheld (checklist incomplete, build-now unanswered, or over-design without descope)
- Clearance design review **No-Go** (Heavy aircraft only; do not create plan/tasks)
- Crew Briefing gate incomplete (Build Briefing or crew-alignment)
- Critical open questions block safe planning
- Before Takeoff finds unresolved blockers
- Big risk during Pushback/Taxi requiring re-brief (stop, update crew-briefing, re-clear)
- Engine Start checklist not 5/5 (engine flameout — do not Taxi)

**Do not stop** for non-blocking gaps — record in HTML and continue unless a hard blocker exists.

## Workflow overview

```txt
Plan → Inspect → Brief → Walk → Build → Integrate → Clear → Release prep → Monitor readiness → Confirm

Phase 1   Dispatch          dispatch.html
Phase 2   Aircraft Status   aircraft-status.html
Phase 3   Crew Briefing     crew-briefing.html
Phase 4   Walkaround        walkaround.html
Phase 5   Load Sheet        load-sheet.html + spec.html | preflight-brief.html
Phase 6   Clearance         clearance.html          (Full only)
Phase 7   Pushback          plan.html + pushback.html + incremental code (Full only)
Phase 8   Engine Start      engine-start.html       (Full only)
Phase 9   Taxi              taxi-check.html         (Full only)
Phase 10  Before Takeoff    before-takeoff.html
          Confirmation      confirmation.html
```

**Aircraft Status vs Walkaround:** Aircraft Status checks system health and MEL (ready to fly?). Walkaround maps the codebase touch points for this feature (where to change, existing patterns) — inspect before spec/plan, do not jump to new abstractions.

**Clearance vs Pushback:** Clearance (ATC) decides *whether* to build now. Pushback leaves the gate — *implementation begins* (still on the ground, not takeoff).

**Pushback vs Engine Start:** Pushback builds incrementally. Engine Start proves the feature **starts** (minimal smoke) — not full polish.

**Engine Start vs Taxi:** Engine proves no flameout. Taxi wires integration on the ground — do not refactor the whole airport during Taxi.

---

## Phase 1 — Dispatch

**Goal:** Flight planning before code. Answer strategic questions from raw requirement only (no deep repo inspection yet).

### Seven questions (must answer in dispatch.html)

1. What problem does this feature solve?
2. Who is the user?
3. What is the success metric?
4. What is out of scope (non-goals)?
5. What dependencies exist?
6. Where might this impact existing systems? (hypothesis — confirmed in Aircraft Status)
7. What is the highest risk?

Also include **Release Strategy (initial)**: deploy type, rollback idea, environment order, release blockers.

### Dispatch Checklist

- [ ] Problem statement ชัด
- [ ] User flow หลักชัด
- [ ] Success criteria ชัด
- [ ] Non-goals ระบุแล้ว
- [ ] Dependency ระบุแล้ว
- [ ] Risk / unknown ระบุแล้ว
- [ ] Release strategy คิดไว้แล้ว

Mark each item `is-pass` or `is-fail` in HTML. Gate verdict: **7/7 required** before Aircraft Status.

Critical fail (empty problem, user flow, or success criteria) → mark **Blocked** on index.html, stop, ask user.

### Dispatch actions

1. Parse raw requirement / user story
2. Choose Full or Lite mode
3. Create report folder + copy CSS + scaffold index.html
4. Write dispatch.html with checklist gate
5. Update index.html phase 1 status

### Chat output

```md
## Dispatch Complete
Path: `.cursor/preflight-report/<id>/dispatch.html`
Checklist: X/7 — [Clear | Blocked]
Missing: ...
Next: Aircraft Status (or resolve blockers)
```

---

## Phase 2 — Aircraft Status

**Goal:** Aircraft maintenance / MEL check — is the codebase "ready to fly" for this feature?

Inspect repo health in the affected area before planning. Use codegraph/explore when available.

### Six inspections (document each in aircraft-status.html)

1. **Known bugs / issues** — open defects in affected modules
2. **API stability** — contracts, versioning, breaking-change risk
3. **Data model constraints** — schema, indexes, FK, migration limits
4. **Test coverage** — existing tests for affected paths; gaps
5. **Blocking tech debt** — debt that would block or dangerously complicate this feature
6. **Temporary workarounds** — hacks in place that this feature must respect or replace

Confirm or refute Dispatch `#impact-hypothesis`. Record repo structure and affected modules.

### Software MEL

For each **known issue accepted for release** (like aviation MEL — broken but within limits), create a `.mel-entry` card with:

## Software MEL

Known issue ที่ยอมรับให้ release ได้:

- [ ] ไม่กระทบ critical path
- [ ] มี workaround
- [ ] มี owner
- [ ] มี follow-up task
- [ ] มี monitoring หรือ guardrail

Mark each MEL item `is-pass` or `is-fail`. An accepted issue requires **5/5** on its MEL checklist.

Issues that **block the feature** and have no valid MEL → record as `#blocking-issues`, stop, ask user.

Issues with incomplete MEL (accepted but missing owner/workaround/etc.) → stop, ask user.

### Aircraft Status gate

- All six inspections documented (may be "none found" with evidence)
- No unresolved `#blocking-issues`
- Every `#mel-accepted` entry passes 5/5 Software MEL

Lite mode: run all six inspections condensed; MEL still required for any accepted known issue.

Output: `aircraft-status.html`

### Chat output

```md
## Aircraft Status Complete
Path: `.cursor/preflight-report/<id>/aircraft-status.html`
Blocking issues: [count]
MEL accepted: [count] · MEL incomplete: [count]
Next: Crew Briefing
```

---

## Phase 3 — Crew Briefing

**Goal:** Pilot / cabin crew briefing — align everyone before hands-on work. Merge Dispatch + Aircraft Status into one shared understanding.

Output: `crew-briefing.html`

### Build Briefing (required fields)

## Build Briefing

- Goal:
- Main flow:
- Edge cases:
- Technical approach:
- Risk:
- Test strategy:
- Rollback:
- Owner:

Render each field as a section in HTML (see REFERENCE.md). Pull content from Dispatch, Aircraft Status, and raw requirement. Do not invent owners — use named owner or mark open question.

### Crew alignment (everyone must understand)

Document in `#crew-alignment`:

1. **Expected behavior** — what the system should do when the feature works
2. **Emergency case** — what breaks, fallback, or stop condition
3. **Decision authority** — who decides if there is conflict (role/name, not vague "team")
4. **Definition of done** — verifiable done criteria for this feature

### Crew Briefing gate

- All eight Build Briefing fields filled (or explicitly N/A with reason)
- All four crew-alignment items answered
- Unresolved critical questions → stop, ask user

Lite mode: same fields, condensed prose.

### Chat output

```md
## Crew Briefing Complete
Path: `.cursor/preflight-report/<id>/crew-briefing.html`
Owner: [name or TBD]
Definition of done: [one line]
Open alignment gaps: ...
Next: Walkaround
```

---

## Phase 4 — Walkaround

**Goal:** Pilot walkaround — inspect the codebase before changing it. Do not jump into implementation or invent new abstractions until existing patterns are mapped.

Use codegraph/explore when available. Scope inspection to modules affected by this feature (from Crew Briefing and Aircraft Status).

### Codebase Walkaround

- [ ] Entry point อยู่ไหน
- [ ] Data flow ไปทางไหน
- [ ] State อยู่ที่ไหน
- [ ] API contract คืออะไร
- [ ] Error handling เดิมทำยังไง
- [ ] Pattern เดิมใน project คืออะไร
- [ ] Test เดิมอยู่ตรงไหน

Mark each item `is-pass` or `is-fail` in HTML. Gate: **7/7 required** before Load Sheet.

For each checklist item, document concrete file paths, symbols, or modules — not generic descriptions.

### Anti-pattern to prevent

Do not propose new abstractions in later phases unless walkaround shows no existing pattern fits. Record reused patterns in `#patterns-to-reuse`.

Output: `walkaround.html`

Lite mode: same 7 items, condensed paths.

### Chat output

```md
## Walkaround Complete
Path: `.cursor/preflight-report/<id>/walkaround.html`
Checklist: X/7 — [Clear | Blocked]
Patterns to reuse: ...
Next: Load Sheet
```

---

## Phase 5 — Load Sheet

**Goal:** Weight and balance — calculate feature "weight" and scope impact before takeoff.

Sources: `crew-briefing.html`, `walkaround.html`, `aircraft-status.html`.

### Step 1 — Feature Load Sheet (always first)

Write `load-sheet.html` with impact table:

## Feature Load Sheet

| Area | Impact |
|---|---|
| UI | Low / Medium / High |
| API | Low / Medium / High |
| Database | Low / Medium / High |
| Auth / Permission | Low / Medium / High |
| Existing user flow | Low / Medium / High |
| Migration | Yes / No |
| Rollback difficulty | Easy / Medium / Hard |

Rate each area from walkaround and crew briefing evidence — not gut feel.

Document `#aircraft-classification`: **Light** | **Medium** | **Heavy** (see REFERENCE.md).

**Heavy aircraft rules (not a "small task"):**

- Any area rated **High**
- **Migration Yes** plus Medium+ in Database, Auth/Permission, or API
- **Rollback Hard**
- Two or more of {Database, Auth/Permission, API, Existing user flow} rated **Medium** or higher

Example: UI looks small but touches DB migration + permission + billing → **Heavy** — needs full runway (Full mode, plan, clearance, taxi).

**Mode upgrade:** If Dispatch chose Lite but Load Sheet classifies **Medium** or **Heavy** → upgrade to **Full**, update `index.html` mode, proceed with full phase path.

Lite allowed only when classification is **Light** AND migration is **No** AND rollback is **Easy**.

### Step 2 — Formal spec (after load sheet)

Follow **sa-spec-workflow.mdc**; render as HTML per REFERENCE.md.

**Full:** write `spec.html` with all spec sections. Reflect load sheet impacts in requirements and risks.

**Lite (Light aircraft only):** write `preflight-brief.html` — accordion panels for Spec / Plan / Tasks (condensed). Include load sheet summary at top.

Do not write production code.

### Chat output

```md
## Load Sheet Complete
Path: `.cursor/preflight-report/<id>/load-sheet.html`
Aircraft: Light | Medium | Heavy
Mode: Full | Lite (upgraded from Lite if applicable)
High-impact areas: ...
Migration: Yes/No · Rollback: Easy/Medium/Hard
Spec: `.cursor/preflight-report/<id>/spec.html` (or preflight-brief.html)
Next: Clearance (Full) | Before Takeoff (Lite)
```

---

## Phase 6 — Clearance (Full only)

**Goal:** ATC clearance — decision gate before Pushback (plan). Not always a large process.

Inputs: `load-sheet.html`, `spec.html`, `crew-briefing.html`.

Output: `clearance.html`

### Clearance

- [ ] Product direction accepted
- [ ] Technical approach accepted
- [ ] Risk accepted
- [ ] Release window accepted
- [ ] Rollback path accepted

Mark each `is-pass` or `is-fail`. Gate: **5/5 required** for clearance.

### Build now vs over-design

Document in `#build-now-decision`:

**สิ่งนี้ควร build ตอนนี้จริงไหม หรือเป็น over-design?**

| Answer | Action |
|--------|--------|
| **Build now** | Proceed if checklist 5/5 |
| **Over-design** | **Withhold clearance** — do not Pushback; descope or stop |
| **Defer** | **Withhold clearance** — not ready to build |
| **Unclear** | Stop, ask user — do not Pushback |

If the question cannot be answered with evidence from load sheet and spec → **withhold clearance**.

### Process depth (by aircraft class from load-sheet)

| Class | Clearance content |
|-------|-------------------|
| **Light** | Checklist + build-now only (lean) |
| **Medium** | Checklist + build-now only (lean) |
| **Heavy** | Checklist + build-now + **sa-design-review.mdc** on load-sheet + spec (append review sections in clearance.html) |

Lean clearance is valid — do not inflate process for small aircraft.

### Clearance verdict

| Verdict | Meaning | Next |
|---------|---------|------|
| **Cleared for Pushback** | 5/5 + build now | Phase 7 Pushback |
| **Cleared with conditions** | 5/5 + minor fixes listed | Pushback after conditions noted |
| **Withheld** | Checklist fail, over-design, defer, or unclear | No plan.html — Confirmation Not Ready |

Use banner classes: `verdict--ready`, `verdict--go-with-changes`, `verdict--no-go` (withheld).

**Withheld → stop.** Do not enter Pushback (no plan, tasks, or code).

### Chat output

```md
## Clearance Complete
Path: `.cursor/preflight-report/<id>/clearance.html`
Verdict: Cleared for Pushback | Cleared with conditions | Withheld
Checklist: X/5
Build now: Yes | Over-design | Defer | Unclear
Next: Pushback (or resolve withholding)
```

---

## Phase 7 — Pushback (Full only)

**Goal:** Leave the gate — **start implementation**. You are on the ground, not in flight (not released/deployed yet).

**Requires Cleared for Pushback** from Phase 6.

### Step A — Plan and tasks (before first code line)

1. Write `plan.html` per **sa-spec-workflow.mdc** (reuse `walkaround.html` `#patterns-to-reuse`)
2. Write `tasks.html` — ordered task cards; map incremental steps to tasks where possible

### Step B — Incremental implementation order

Implement in this order (update `pushback.html` after each step):

```txt
scaffold → core logic → integration → UI → error states → tests → cleanup
```

Track each step in `pushback.html` with status, files touched, and verification run.

Use **sa-task-execution.mdc** discipline: smallest safe change, one logical slice at a time, do not batch unrelated edits.

### Pushback principles

- **Do not refactor the whole airport** — no unrelated module rewrites
- **Do not change architecture mid-way** unless unavoidable; if unavoidable, stop and re-run **Crew Briefing** (Phase 3) + seek clearance again
- **Big risk discovered** → stop implementation, update `crew-briefing.html` / open questions, ask user before continuing
- Reuse existing patterns from walkaround — no new abstraction without evidence

Output: `plan.html`, `tasks.html`, `pushback.html`, and production code changes.

### Chat output

```md
## Pushback Complete
Path: `.cursor/preflight-report/<id>/pushback.html`
Incremental steps: X/7 complete
Plan: plan.html · Tasks: tasks.html
Changed files: ...
Next: Engine Start
```

---

## Phase 8 — Engine Start (Full only)

**Goal:** Start engine — prove the feature **turns on** at minimum power before Taxi. Not full polish; prove the engine does not flameout.

After Phase 7 Pushback, run smoke verification and document in `engine-start.html`.

### Engine Start

- [ ] Local run ได้
- [ ] Happy path ทำงาน
- [ ] Core API connect ได้
- [ ] Data shape ถูก
- [ ] Error แรก ๆ ถูกจับได้

Mark each `is-pass` or `is-fail`. Gate: **5/5 required** before Taxi.

For each item record: command run, evidence (log, screenshot path, test output), and what was **not** polished yet (explicitly out of scope for this phase).

### Principles

- **Not full polish** — UI perfection, edge cases, and performance tuning come later (Taxi / Before Takeoff)
- **Smallest proof** — narrowest path that proves the feature is alive
- **Flameout** → stop, fix on ground (Pushback) or re-brief if architectural; do not enter Taxi

Optional: update `tasks.html` status for tasks proven by happy-path smoke.

Output: `engine-start.html`

### Chat output

```md
## Engine Start Complete
Path: `.cursor/preflight-report/<id>/engine-start.html`
Checklist: X/5 — [Running | Flameout]
Proven: local run, happy path, ...
Not polished (deferred): ...
Next: Taxi
```

---

## Phase 9 — Taxi / Integration (Full only)

**Goal:** Taxi on the ground — integration check before takeoff. **Not** a refactor phase.

Cross-check implementation against walkaround, aircraft-status, and plan.

Output: `taxi-check.html`:

- Task dependency order (ASCII diagram in `<pre class="diagram">`)
- Module cross-check vs walkaround entry points and patterns
- Cross-check vs aircraft-status MEL and blockers
- External API / DB migration order
- Feature flag / rollout dependencies
- Pre-takeoff integration checklist

### Taxi principles

- **Do not refactor the whole airport during Taxi** — integration fixes only, minimal scope
- **Do not change architecture** — if required, stop and return to Crew Briefing + Clearance
- Fix integration defects; defer nice-to-have refactors to a separate preflight

---

## Phase 10 — Before Takeoff

Apply **sa-design-review.mdc** to full package:

- Full: load-sheet.html + spec.html + plan.html + tasks.html + pushback.html + engine-start.html + taxi-check.html
- Lite: load-sheet.html + preflight-brief.html

Output: `before-takeoff.html` with Go / Go with changes / No-Go.

---

## Confirmation

Synthesize clearance + pushback + before-takeoff + all open items into **takeoff readiness** (ready to release/deploy/merge — not "start coding").

Output: `confirmation.html`:

- Verdict: **Ready** | **Not Ready**
- Readiness score: 0–100% (weighted rubric in REFERENCE.md)
- Score breakdown table with bars
- Blockers (must fix before takeoff)
- Gaps (non-blocking)
- Open questions
- Suggested next step

**Ready:** score ≥ 85% AND zero blockers.

Update index.html with final verdict, score, and links.

### Chat output

```md
## Preflight Confirmation
Verdict: Ready for Takeoff | Not Ready
Score: XX%
Dashboard: `.cursor/preflight-report/<id>/index.html`
Confirmation: `.cursor/preflight-report/<id>/confirmation.html`
Blockers: ...
Suggested next step: ...
```

If Ready → suggest PR, deploy, or release per plan (user approval required).
If Not Ready → list blockers; fix on ground (Pushback/Taxi) or re-brief if scope/architecture changed.

---

## index.html dashboard

Maintain throughout the run:

- Flight ID, feature name, mode (Full/Lite)
- Phase checklist 1–10 + Confirmation with `is-pass` / `is-fail` / `is-skip`
- Readiness gauge (after Confirmation)
- Verdict banner
- Links to all artifacts

---

## HTML rules

1. Valid HTML5, `lang="en"`
2. Link `assets/preflight.css` on every page
3. Nav back to index.html on every page
4. Use semantic sections matching rule templates
5. Use CSS classes from REFERENCE.md
6. Footer link to next phase where applicable

See [REFERENCE.md](REFERENCE.md) for page skeleton and section IDs.
