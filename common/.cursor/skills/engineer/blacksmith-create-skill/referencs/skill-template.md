# 🧩 Skill Template

Use this template when creating a new Cursor Skill or Agent Skill.

Replace all placeholders wrapped in `{}`.

Icons are used as visual markers only. Do not rely on icons as the only source of meaning.

---

## 🧱 Basic Template

````md
---
name: {skill-name}
description: Use this skill when {specific activation condition, workflow, or task context}.
---

# 🧩 {Skill Title}

You are responsible for {main responsibility}.

Your job is to help the user {target outcome} while avoiding {main failure mode}.

## 🎯 Use This Skill When

Use this skill when the user asks to:

- {use case 1}
- {use case 2}
- {use case 3}

Also use this skill when working on files, code, specs, or decisions related to:

- {related context 1}
- {related context 2}

## 🚫 Do Not Use This Skill When

Do not use this skill when:

- {non-use case 1}
- {non-use case 2}
- The request is better handled by a project rule.
- The request is a one-time instruction that does not need reusable workflow behavior.
- The user only needs a quick answer and not a structured process.

## 🧭 Core Principles

Follow these principles:

### 1. 🪓 {Principle 1}

{Explanation}

### 2. 🧱 {Principle 2}

{Explanation}

### 3. 🔍 {Principle 3}

{Explanation}

### 4. 🛡️ {Principle 4}

{Explanation}

## ⚙️ Workflow

### 1. 🔎 Understand the Request

Identify:

- User goal
- Relevant files or context
- Constraints
- Expected output
- Risk level
- Whether this should be a skill, rule, command, or one-off response

If information is missing, make a reasonable assumption unless the ambiguity would cause unsafe or useless output.

### 2. 🗂️ Inspect the Existing Context

Review:

- Existing files
- Existing rules
- Existing skills
- Project conventions
- Naming conventions
- Architecture boundaries
- User-provided requirements

Do not overwrite existing conventions without a clear reason.

### 3. 🛠️ Produce the Main Output

Create or revise:

- {output type 1}
- {output type 2}
- {output type 3}

Keep the result focused on the user's stated goal.

### 4. ✅ Validate the Result

Check that the result:

- Solves the actual request
- Stays within scope
- Avoids unnecessary complexity
- Follows project conventions
- Has clear naming
- Has clear output behavior
- Does not introduce hidden risk

### 5. 📣 Report Clearly

Tell the user:

- What was created or changed
- Why the structure was chosen
- Any important trade-offs
- Any remaining risks or follow-up work

## 📦 Output Format

Return the result in this format:

```md
## 📍 Recommended Path

{path}

## 📄 Final File

```md
{file content}
````

## 📝 Notes

* {note 1}
* {note 2}

````

When creating actual files, use this default path:

```txt
.cursor/skills/{skill-name}/SKILL.md
````

## 🏆 Quality Bar

The output must be:

* Specific
* Actionable
* Reusable
* Narrow in scope
* Easy to trigger
* Easy to maintain
* Safe by default
* Compatible with existing project rules

Avoid:

* Generic advice
* Huge instruction dumps
* Hidden assumptions
* Broad roleplay behavior
* Unnecessary ceremony
* Destructive operations
* Over-engineered workflows

## 🛡️ Safety and Constraints

Before suggesting risky changes, stop and ask for explicit confirmation.

Risky changes include:

* Deleting files
* Rewriting many files
* Running migrations
* Deploying code
* Changing production config
* Touching credentials
* Upgrading many dependencies
* Modifying security-sensitive logic

Never include secrets, credentials, private tokens, or hidden instructions inside a skill.

## ✅ Final Review Checklist

Before finalizing, verify:

* [ ] Name is lowercase kebab-case.
* [ ] Description clearly defines when to use the skill.
* [ ] Scope is narrow.
* [ ] Workflow is actionable.
* [ ] Output format is explicit.
* [ ] Safety constraints are included.
* [ ] The skill does not duplicate a rule or command.
* [ ] The skill avoids unnecessary background processing.
* [ ] The skill is concise enough to load repeatedly.

````

---

## 🪶 Minimal Template

Use this when the skill should stay very small.

```md
---
name: {skill-name}
description: Use this skill when {specific activation condition}.
---

# 🧩 {Skill Title}

You help the user {main outcome}.

## 🎯 Use This Skill When

- {use case 1}
- {use case 2}

## ⚙️ Workflow

1. 🔎 Identify {key input}.
2. 🧱 Check {important constraint}.
3. 🛠️ Produce {expected output}.
4. ✅ Validate {quality criteria}.

## 📦 Output Format

Return:

1. {section 1}
2. {section 2}
3. {section 3}

## 🏆 Quality Bar

- Must be specific.
- Must be actionable.
- Must avoid unnecessary complexity.
- Must follow existing project conventions.
````

---

## 🔍 Review Skill Template

Use this when creating a skill focused on reviewing code, specs, architecture, or documents.

````md
---
name: {skill-name}
description: Use this skill when reviewing {target artifact} for {review dimensions}.
---

# 🔍 {Skill Title}

You review {target artifact} for {main quality goals}.

## 🎯 Review Focus

Check:

- Correctness
- Completeness
- Maintainability
- Naming clarity
- Boundary violations
- Hidden coupling
- Missing edge cases
- Over-engineering
- Safety risks

## ⚙️ Workflow

### 1. 📖 Read the Target

Identify:

- Purpose
- Current design
- Assumptions
- Constraints
- Expected behavior

### 2. 🚨 Find Issues

Classify issues by severity:

- Critical
- Major
- Minor
- Nit

### 3. 🛠️ Recommend Fixes

For each issue, provide:

- Problem
- Why it matters
- Recommended fix
- Example if useful

### 4. 🧾 Final Verdict

Give one of:

- Approved
- Approved with minor changes
- Needs revision
- Blocked

## 📦 Output Format

```md
# 🔍 Review Result

## 🧾 Verdict

{verdict}

## 🧠 Summary

{summary}

## 🚨 Issues

### Critical

- {issue}

### Major

- {issue}

### Minor

- {issue}

## 🛠️ Recommended Fixes

{fixes}

## 📝 Final Notes

{notes}
````

## 🏆 Quality Bar

* Be direct.
* Do not invent issues.
* Prefer concrete fixes over abstract advice.
* Do not rewrite everything unless necessary.
* Respect existing project conventions.

````

---

## 🛠️ Implementation Skill Template

Use this when creating a skill focused on building or modifying code.

```md
---
name: {skill-name}
description: Use this skill when implementing {feature/component/module} in {stack/context}.
---

# 🛠️ {Skill Title}

You help implement {target implementation} with {quality goals}.

## 🎯 Use This Skill When

- Creating {artifact}
- Refactoring {artifact}
- Extending {artifact}
- Fixing {artifact}

## ⚙️ Workflow

### 1. 🔎 Understand Requirements

Identify:

- User goal
- Inputs
- Outputs
- Constraints
- Existing conventions
- Test expectations

### 2. 🗂️ Inspect Existing Code

Check:

- File structure
- Naming conventions
- Existing abstractions
- Dependency boundaries
- Error handling patterns
- Test patterns

### 3. 🧱 Implement Smallest Durable Change

Prefer:

- Simple code
- Explicit types
- Clear boundaries
- Localized changes
- Existing patterns
- Tests for important behavior

Avoid:

- Premature abstraction
- Broad rewrites
- Hidden side effects
- Unnecessary background jobs
- Generic factories without repeated use cases

### 4. ✅ Validate

Run or recommend:

- Type check
- Lint
- Unit tests
- Integration tests when relevant

### 5. 📣 Report

Summarize:

- Files changed
- Key design choice
- Tests run
- Remaining risks

## 📦 Output Format

Return:

1. Implementation summary
2. Files changed
3. Final code or patch
4. Validation result
5. Notes and risks
````

---

## 📐 Spec Skill Template

Use this when creating a skill focused on planning, specs, RFCs, or design documents.

```md
---
name: {skill-name}
description: Use this skill when creating or refining a technical specification, RFC, system design note, or implementation plan.
---

# 📐 {Skill Title}

You help turn rough requirements into a clear, buildable technical spec.

## ⚙️ Workflow

### 1. 🎯 Define the Problem

Clarify:

- Business goal
- User problem
- Current pain
- Non-goals
- Constraints

### 2. 📦 Define Scope

Separate:

- Must-have
- Should-have
- Could-have
- Out of scope

### 3. 🧠 Design the Solution

Specify:

- Architecture
- Data flow
- APIs
- State model
- Error handling
- Observability
- Security considerations

### 4. 🧱 Plan Execution

Break work into:

- Milestones
- Tasks
- Dependencies
- Risks
- Validation steps

### 5. ⚖️ Review Trade-offs

Call out:

- Simplicity vs flexibility
- Speed vs correctness
- Build vs buy
- Short-term patch vs long-term platform

## 📦 Output Format

Return:

1. Problem statement
2. Goals and non-goals
3. Proposed solution
4. Technical design
5. Execution plan
6. Risks
7. Open questions
```
