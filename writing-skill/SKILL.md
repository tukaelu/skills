---
name: writing-skill
description: Designs and implements Claude agent skills (SKILL.md, reference files, utility scripts) following official best practices. Use when user wants to create, write, build, update, or improve a skill; when user mentions SKILL.md; or when designing Claude agent capabilities.
allowed-tools:
  - Bash
  - Glob
  - Grep
  - Read
  - Edit
  - Write
---

# Writing Skills

## Workflow

### Step 1: Gather requirements

Ask the user:
- What task or domain should the skill cover?
- What specific use cases should it handle?
- Does it need executable scripts, or just instructions?
- Any reference materials (docs, schemas, examples) to include?

### Step 2: Inspect the environment

Before writing any files, understand the existing layout:

```bash
ls <skills-root>/
```

Read one representative `SKILL.md` to confirm naming conventions, frontmatter fields, and style. Mirror those conventions exactly.

### Step 3: Draft the skill

```
skill-name/
├── SKILL.md           # Required. Keep under 500 lines.
├── reference/         # Add when content is domain-specific or rarely needed
│   └── topic.md
└── scripts/           # Add for deterministic, reusable operations
    └── helper.py
```

**SKILL.md template:**

```md
---
name: skill-name
description: What it does. Use when [specific keywords / contexts / file types].
---

# Skill Name

## Quick start

[Minimal working example]

## Workflows

[Step-by-step checklist for complex tasks]

## Advanced features

See [topic.md](reference/topic.md)
```

Self-check against the review checklist before presenting to the user.

### Step 4: Review with user

- Does this cover your use cases?
- Anything missing or unclear?
- Should any section be more or less detailed?

Iterate until the user is satisfied.

---

## Naming rules

| Constraint | Rule |
|---|---|
| Format | Lowercase letters, numbers, hyphens only |
| Max length | 64 characters |
| Preferred style | Gerund form: `processing-pdfs`, `writing-skills` |
| Forbidden | Reserved words: `anthropic-*`, `claude-*`; XML tags |

## Description rules

- Max 1024 characters; no XML tags; **always write in third person**
- First sentence: what it does. Second sentence: `Use when [specific triggers]`

**Good:** `"Analyzes Excel spreadsheets and creates pivot tables. Use when analyzing .xlsx files, tabular data, or spreadsheets."`  
**Bad:** `"Helps with documents."` — gives no context for skill selection

## Degrees of freedom

Match instruction specificity to task fragility:

- **High** (text instructions): multiple valid approaches, context-dependent heuristics
- **Medium** (templates with parameters): preferred pattern, some variation acceptable
- **Low** (exact commands/scripts): fragile sequences requiring strict consistency

## Review checklist

- [ ] Description is specific, third-person, includes key terms and triggers
- [ ] SKILL.md body under 500 lines
- [ ] Domain-specific or rarely-needed content split into `reference/` files
- [ ] No time-sensitive information (or isolated in an "old patterns" section)
- [ ] Consistent terminology throughout (one term per concept)
- [ ] Concrete examples included
- [ ] All references are one level deep from SKILL.md
- [ ] Complex workflows include copy-paste checklists
- [ ] Scripts solve problems explicitly — no punting edge cases to Claude
- [ ] Forward slashes in all file paths (no Windows-style backslashes)
- [ ] Required packages listed when scripts have dependencies

**For detailed guidance on patterns, anti-patterns, and evaluation:**  
See [reference/BEST-PRACTICES.md](reference/BEST-PRACTICES.md)

## Notes

- Always respond in the same language the user used to invoke the skill
