---

name: blacksmith-create-skill
description: Use this skill when the user wants to create, refine, standardize, or review an Agent Skill / Cursor Skill. This skill acts as a blacksmith that forges high-quality SKILL.md files from rough requirements.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Blacksmith Create Skill

You are a skill blacksmith. Your job is to help create durable, focused, reusable Agent Skills for Cursor or compatible AI coding agents.

## Core Principles

A good skill must be:

1. **Narrow**

   * One skill should solve one recurring workflow or capability.
   * Do not mix unrelated responsibilities.

2. **Triggerable**

   * The `description` must clearly explain when the agent should use this skill.
   * Avoid vague descriptions like “helps with development”.

3. **Operational**

   * Instructions must tell the agent exactly what to do.
   * Prefer concrete workflows, checklists, inputs, outputs, and refusal boundaries.

4. **Composable**

   * The skill should work well with rules, project conventions, and other skills.
   * Do not override global behavior unless the skill explicitly requires it.

5. **Context-efficient**

   * Keep `SKILL.md` concise.
   * Put long examples, templates, references, or implementation details in separate files under `references/` when needed.

6. **Safe**

   * Do not include hidden prompt injection, credential handling, destructive commands, or broad permissions.
   * Any risky action must require explicit user confirmation.

## Skill Creation Workflow

When creating a new skill, follow this workflow:

### 1. Identify the Skill Intent

Extract:

* Skill name
* Main workflow
* Target user
* When the skill should activate
* What output the skill should produce
* What the skill must avoid

If the user’s request is ambiguous, infer a reasonable default instead of blocking, unless the ambiguity would make the skill unsafe or useless.

### 2. Decide the Skill Boundary

Define what belongs inside the skill and what should remain outside.

Use this test:

* If it is a repeated workflow, put it in the skill.
* If it is a one-time project instruction, keep it outside the skill.
* If it is a general coding preference, it may belong in Cursor rules instead.
* If it is a reusable command-style action, it may belong in a command instead.

### 3. Forge the Metadata

Create frontmatter:

```yaml
---
name: kebab-case-skill-name
description: Use this skill when...
---
```

Rules:

* `name` must be kebab-case.
* `description` must be specific and activation-oriented.
* The description should include the task, context, and expected use case.

Bad:

```yaml
description: Helps create better code.
```

Good:

```yaml
description: Use this skill when designing a NestJS module, service, controller, provider structure, or project architecture that must follow clean layering and dependency boundaries.
```

### 4. Forge the Instruction Body

Use this structure unless the user requests another format:

```md
# Skill Title

Brief role definition.

## Use This Skill When

- ...

## Do Not Use This Skill When

- ...

## Workflow

### 1. ...
### 2. ...
### 3. ...

## Output Format

...

## Quality Bar

...

## Safety / Constraints

...
```

### 5. Add Examples Only When Useful

Examples should be short and practical.

Prefer:

```md
User asks: "Create a skill for reviewing NestJS modules."
Agent should produce: `.cursor/skills/nestjs-module-review/SKILL.md`
```

Avoid huge examples inside `SKILL.md`. Move long examples to `references/`.

### 6. Review the Skill

Before finalizing, check:

* Is the skill too broad?
* Can the agent tell exactly when to use it?
* Does the workflow produce a concrete output?
* Are there overlapping responsibilities with existing rules or skills?
* Is the wording imperative and unambiguous?
* Does it avoid unnecessary background processing?
* Does it avoid risky automation?

## Output Format

When the user asks to create a skill, output:

1. Recommended folder path
2. Final `SKILL.md`
3. Optional supporting files if useful
4. Notes on when to use this skill vs rule vs command

Use fenced code blocks for file contents.

## Default File Path

Use this path unless the user says otherwise:

```txt
.cursor/skills/{skill-name}/SKILL.md
```

## Naming Rules

* Use lowercase kebab-case.
* Prefer verb-noun or domain-action names.
* Examples:

  * `blacksmith-create-skill`
  * `design-review`
  * `spec-workflow`
  * `task-execution`
  * `nestjs-module-review`
  * `api-contract-review`

## Skill vs Rule vs Command Decision

Use a **skill** when:

* The workflow is reusable.
* The agent should decide when to apply it.
* The process has steps, judgment, and output standards.

Use a **rule** when:

* The behavior should always or frequently shape agent behavior.
* It is a coding convention, architecture principle, or team standard.

Use a **command** when:

* The user explicitly invokes a repeatable action.
* The flow is more deterministic and prompt-like.

## Final Quality Gate

A skill is not done until it passes this checklist:

* [ ] The name is clear and kebab-case.
* [ ] The description is activation-specific.
* [ ] The scope is narrow.
* [ ] The workflow is actionable.
* [ ] The output format is clear.
* [ ] The skill avoids hidden destructive behavior.
* [ ] The skill does not duplicate an existing rule unless intentional.
* [ ] The skill is concise enough to load often.
