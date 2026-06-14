# Skill Quality Checklist

Use this checklist before finalizing any Cursor Skill or Agent Skill.

A skill is only considered ready when it is narrow, triggerable, operational, composable, context-efficient, and safe.

---

## 1. Identity Check

* [ ] Skill name is written in lowercase kebab-case.
* [ ] Skill name describes the actual job clearly.
* [ ] Skill name is not too generic.
* [ ] Skill name does not conflict with an existing skill, rule, or command.
* [ ] Folder name matches the skill name.

Good examples:

```txt
blacksmith-create-skill
nestjs-module-review
api-contract-review
incident-analysis
spec-workflow
task-execution
```

Bad examples:

```txt
helper
dev-skill
good-code
reviewer
assistant-mode
```

---

## 2. Trigger Check

The `description` field must clearly tell the agent when to use the skill.

* [ ] Description starts with an activation-oriented phrase such as `Use this skill when...`
* [ ] Description names the target workflow.
* [ ] Description includes enough context to avoid accidental activation.
* [ ] Description avoids vague wording.
* [ ] Description does not claim the skill should always be used.

Bad:

```yaml
description: Helps write better code.
```

Good:

```yaml
description: Use this skill when reviewing a NestJS module, service, provider, or controller for clean architecture boundaries, dependency direction, and maintainability issues.
```

---

## 3. Scope Check

A good skill does one thing well.

* [ ] Skill has one main responsibility.
* [ ] Skill does not combine unrelated workflows.
* [ ] Skill does not duplicate an existing rule unless intentional.
* [ ] Skill does not replace project-level rules.
* [ ] Skill does not contain one-time project context.
* [ ] Skill can be reused across multiple tasks.

Ask:

```txt
Would this still be useful next month?
Would this still be useful in another project?
Can the agent explain when this skill should not be used?
```

If the answer is no, it may be a prompt, note, rule, or command instead of a skill.

---

## 4. Workflow Check

The skill must tell the agent what to do, not just what to care about.

* [ ] Workflow is step-by-step.
* [ ] Steps are written as direct instructions.
* [ ] The agent knows what inputs to inspect.
* [ ] The agent knows what outputs to produce.
* [ ] The workflow includes decision points where needed.
* [ ] The workflow avoids vague advice like “make it clean” without defining what clean means.

Weak:

```md
Make sure the code is good and follows best practices.
```

Strong:

```md
Review module boundaries. Identify dependencies that point from domain logic into infrastructure or presentation layers. Recommend a concrete refactor when a boundary is violated.
```

---

## 5. Output Check

The user should know what they will get.

* [ ] Output format is explicit.
* [ ] Required sections are listed.
* [ ] Optional sections are clearly marked.
* [ ] The skill says whether to output code, markdown, diffs, review notes, commands, or files.
* [ ] The skill avoids overly long default output.
* [ ] The skill encourages concise but complete results.

Example:

```md
## Output Format

Return:

1. Summary
2. Issues found
3. Recommended changes
4. Final revised file
5. Follow-up risks
```

---

## 6. Quality Bar Check

The skill should define what “good” means.

* [ ] Quality criteria are concrete.
* [ ] The skill defines unacceptable output.
* [ ] The skill includes domain-specific standards.
* [ ] The skill prefers maintainability over cleverness.
* [ ] The skill requires explicit trade-offs when relevant.
* [ ] The skill prevents over-engineering.

Useful quality prompts:

```txt
Is this simpler than the alternative?
Can another developer maintain it?
Does this introduce hidden coupling?
Does this create background processing that is not necessary?
Does this solve the user's actual problem?
```

---

## 7. Safety Check

The skill must not create hidden risk.

* [ ] No hidden destructive commands.
* [ ] No automatic deletion, migration, deployment, or production write action without explicit user approval.
* [ ] No credential exposure.
* [ ] No instruction to bypass security checks.
* [ ] No prompt injection behavior.
* [ ] No broad permissions unless justified.
* [ ] Risky operations require confirmation.

Risky operations include:

```txt
rm -rf
database migration
production deployment
credential rotation
mass file rewrite
dependency upgrade across the whole repo
network call with secrets
```

---

## 8. Context Efficiency Check

A skill should not dump unnecessary context into every run.

* [ ] `SKILL.md` is concise.
* [ ] Long examples are moved to `references/`.
* [ ] Templates are moved to `references/`.
* [ ] Repeated checklists are moved to `references/`.
* [ ] The main skill file links to supporting files only when needed.
* [ ] The skill avoids large essays.

Rule of thumb:

```txt
SKILL.md should contain the core operating behavior.
references/ should contain optional depth.
```

---

## 9. Composition Check

The skill should cooperate with rules, commands, and other skills.

* [ ] The skill does not override global project rules.
* [ ] The skill explains when to defer to project conventions.
* [ ] The skill can be combined with review, spec, or implementation skills.
* [ ] The skill does not assume a specific stack unless stack-specific by design.
* [ ] The skill avoids fighting with formatter, linter, or architecture rules.

---

## 10. Anti Over-Design Check

The skill should not encourage needless complexity.

* [ ] No unnecessary background processing.
* [ ] No excessive abstractions.
* [ ] No premature plugin architecture.
* [ ] No huge framework when a small workflow is enough.
* [ ] No generic factory/helper unless the repeated use case is proven.
* [ ] No automation that hides important user decisions.

Ask:

```txt
What is the smallest durable version of this skill?
What would Grandpa Ive remove?
```

---

## Final Gate

Before shipping, answer these questions:

```txt
1. What exact situation triggers this skill?
2. What exact result should the agent produce?
3. What should the skill refuse or avoid?
4. What makes this skill better than a normal prompt?
5. What part of this belongs in a rule instead?
6. What part of this belongs in a command instead?
7. What can be removed without reducing usefulness?
```

If any answer is unclear, revise the skill before using it.
