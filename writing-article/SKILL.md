---
name: writing-article
description: >
  Analyzes article structure as an information dependency graph, then rewrites
  each section for clarity, narrative flow, and cognitive accessibility.
  Use when given an article, blog post, essay, or technical document to improve,
  restructure, or rewrite.
allowed-tools:
  - Read
  - Edit
  - Write
  - AskUserQuestion
---

# Article Writing

Map article structure using dependency and narrative analysis, then rewrite
section by section to sharpen clarity and flow.

## Quick start

1. Share the article (paste text or provide a file path)
2. State the target reader and goal (or let the skill infer them)
3. The skill proposes a section map — confirm or adjust
4. Sections are rewritten one at a time, each requiring your approval
5. A final holistic review is delivered at the end

## Step 1: Understand the Article

Read the full article, then establish context if not obvious:
- [ ] Who is the target reader?
- [ ] What should they *think, feel, or do* after reading?
- [ ] Any constraints (tone, length, style guide)?

## Step 2: Structure Analysis

Model the article as an **information dependency graph (DAG)** — some concepts
must be established before others can land.

For each section, write one sentence: the **core message** it must deliver.

Then evaluate:
- [ ] Does the current order respect information dependencies?
- [ ] Where does narrative tension build and release? (problem → empathy → solution → action)
- [ ] Are any sections redundant, orphaned, or missing?

Present a proposed structure:

```
Section map:
① Introduction — Hook + problem framing
② Background — defines "X"  [required by ③ and ④]
③ Current State — what's broken  [depends on ②]
④ Solution — the approach  [depends on ③]
⑤ Conclusion — call to action  [depends on all]
```

When proposing reordering, explain the dependency reason:
> "Section B must precede Section C because C assumes the reader already knows X."

**Wait for user confirmation before proceeding.**

## Step 3: Section Rewrite

For each confirmed section, rewrite following these rules:

**Paragraph rules**
- One idea per paragraph — state it in the opening sentence
- Max 240 characters per paragraph — split if over
- Active voice by default; passive only when the agent matters less than the action
- Cut throat-clearing openers ("In this section, we will...")

**Transition rules**
- End each section with a sentence that closes the loop or points forward
- Start each section (after the first) with a sentence that connects to the previous

**Cognitive load check** — after rewriting, flag any paragraph that:
- Introduces more than one new term
- Assumes knowledge not yet established in the article

Show the rewritten section and wait for approval before moving to the next.

## Step 4: Final Review

After all sections are approved, do a holistic pass:
- [ ] Does the opening sentence create enough pull to continue reading?
- [ ] Does each section's core message still land clearly?
- [ ] Does the ending deliver on the promise made in the introduction?

Deliver the final article with a brief note on the key changes made.

---

## Notes

- Always respond in the same language the user used to invoke the skill
- If the article is in a file, use Read to load it; otherwise ask the user to paste it
- Never silently skip a section — if a section needs no changes, briefly acknowledge it
