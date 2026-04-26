---
name: sketching-diagrams
argument-hint: "[diagram-type] [description]"
description: |
  Interactive skill for creating Mermaid diagrams through collaborative conversation.
  Clarifies what the user wants to visualize, confirms the structure with ASCII art sketches,
  then generates accurate Mermaid code once alignment is reached.
  Use when user requests diagrams, flowcharts, sequence diagrams, ER diagrams, class diagrams,
  state diagrams, Gantt charts, mind maps, or any visualization. Triggers: "draw a diagram",
  "visualize", "make a flowchart", "sequence diagram", "ER diagram", "class diagram",
  "state machine", "Gantt chart", "Mermaid diagram", "diagram the system flow",
  "illustrate the process", "turn this into a diagram".
---

# Sketching Diagrams

Work with the user interactively to produce a Mermaid diagram. Don't generate unilaterally —
**discuss the structure, confirm with ASCII art, then generate** the code.

## Quick start

1. Say "draw a diagram" or "/sketching-diagrams [type] [description]" to invoke
2. The skill identifies the diagram type and asks 1–2 clarifying questions if needed
3. A skeleton is drawn in ASCII art — confirm or adjust the structure
4. Details (labels, conditions, styles) are filled in once the skeleton is agreed upon
5. Final Mermaid code is generated inside a fenced code block, ready to render

## Steps

### Step 1: Understand Intent

If `$ARGUMENTS` is provided, extract the diagram type and content, then proceed to Step 2.

Otherwise, use AskUserQuestion to identify what the user wants:

```javascript
AskUserQuestion({
  questions: [{
    question: "What kind of diagram do you want?",
    header: "Diagram type",
    options: [
      { label: "Flowchart",        description: "Processes, steps, decision branches" },
      { label: "Sequence diagram", description: "Time-ordered interactions, API calls" },
      { label: "State diagram",    description: "State changes, lifecycles" },
      { label: "Other",            description: "ER, class, Gantt, mind map, etc." }
    ],
    multiSelect: false
  }]
})
```

If "Other" is selected, ask for the diagram type via text. If the content is still unclear,
ask **1–2 questions at a time** to keep momentum.

### Step 2: Propose Diagram Type

Suggest the most appropriate Mermaid syntax. If multiple options fit, present them with reasoning.

| Diagram | Syntax | Best for |
|---|---|---|
| Flowchart | `flowchart` | Processes, decision branches |
| Sequence | `sequenceDiagram` | Time-ordered interactions |
| Class | `classDiagram` | Object structure, inheritance |
| ER | `erDiagram` | Database design |
| State | `stateDiagram-v2` | State changes |
| Gantt | `gantt` | Schedules, task planning |
| Mind map | `mindmap` | Idea organization |
| Pie chart | `pie` | Proportions, breakdowns |
| Git graph | `gitGraph` | Branching strategies |
| Timeline | `timeline` | Historical events, milestones |

### Step 3: Confirm Structure with ASCII Art

Refer to [reference/ascii-examples.md](reference/ascii-examples.md) for sketch styles, then draw
a skeleton and ask the user to confirm. Use real (or placeholder) node names and arrow directions.
Show it early — you don't need complete information. Update and re-confirm after each round of feedback.

### Step 4: Fill in Details (if needed)

Once the skeleton is agreed upon, confirm details as needed: labels, conditions, styles, subgraphs, etc.
Infer obvious details and keep moving. Avoid over-asking.

### Step 5: Generate Mermaid Code

````markdown
```mermaid
(code)
```
````

After generating, briefly describe the main nodes/elements and invite the user to request adjustments.
Self-review syntax keywords, arrow notation, and node syntax for the diagram type before presenting.

## Conversation Style

- Lead with concrete proposals: "How about this?" with a specific suggestion
- If the user is in a hurry, gather all required info at once and generate immediately
- Remain flexible after code generation — handle revision requests freely

## Notes

- **Language**: Always respond in the same language the user used to invoke the skill
