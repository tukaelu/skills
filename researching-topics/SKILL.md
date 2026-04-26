---
name: researching-topics
description: >
  Collects and analyzes facts and opinions through quantitative and qualitative
  investigation, then delivers actionable insights as structured reports or
  recommendations. Use when researching market trends, technology, competitive
  analysis, or any topic requiring multi-perspective investigation.
allowed-tools:
  - WebSearch
  - WebFetch
  - Task
---

# Researching Topics

A skill for investigating and analyzing topics and problems, then delivering insights that support decision-making. Research is conducted from both quantitative and qualitative angles through iterative dialogue with the user.

## Quick start

1. Share a topic or question to investigate ("research X", "look into Y")
2. The skill asks 1–2 clarifying questions about purpose, scope, and expected output
3. Research is conducted across quantitative (data, statistics) and qualitative (opinions, cases) axes
4. An interim check-in is offered for broad topics before analysis begins
5. Findings are delivered as a structured report with sources; deeper dives continue on request

## Guiding Principles

Research is not a one-way delivery of information — it is a collaborative process driven by dialogue with the user. The goal is to understand what the user truly wants to know and deliver the most valuable insights. Focus not on listing information, but on reaching the "So what?" — the actionable implication.

---

## Step 1: Understanding the Topic and Clarifying Intent

Before beginning research, accurately understand the user's topic and intent.

### Initial Questions to Ask

1. **Purpose and background**: Why are they researching this topic? What decision or action do they want to inform?
2. **Current understanding**: What do they already know? Do they have hypotheses or assumptions?
3. **Scope of research**: What are the geographic, temporal, and subject boundaries (industry, technology, population, etc.)?
4. **Expected output**: A detailed report? A summary only? Emphasis on quantitative data? Qualitative insights and case studies?

### How to Conduct Clarifying Dialogue

When the user says "I want you to research X," don't start immediately — first deepen the question:

- Explore the underlying question behind the user's statement (e.g., "I want to know about social media success stories" → "What problem are you trying to solve?")
- If the scope is too broad, suggest narrowing it (e.g., "AI adoption in general" → "Which industry or use case would you like to focus on?")
- Surface any assumptions or blind spots the user may have early in the process

**Guideline**: Limit clarifying questions to 1–2 rounds. If the user says "go ahead and research this" or the topic is already clear, proceed to Step 2.

---

## Step 2: Conducting Research

Gather information from multiple angles based on the topic.

### Two Axes of Research

**Quantitative (numbers and data)**
- Statistical figures such as market size, growth rate, and adoption rate
- Quantitative results from research reports and studies
- Comparable benchmarks

**Qualitative (opinions and case studies)**
- Perspectives and statements from experts and practitioners
- Concrete examples and use cases
- Critical voices, concerns, and counterarguments

### Ensuring Diversity of Perspectives

To avoid biased information gathering, investigate from the following angles:
- Supporting and opposing viewpoints
- Proponents and critics (industry, policy, academia, and other stakeholders with differing positions)
- Domestic and international perspectives (when the topic has global relevance)
- Current trends and historical context

### Research Process

1. Start with broad searches to grasp the overall picture
2. Conduct deeper searches on important subtopics
3. Prioritize highly credible sources (government statistics, academic papers, industry associations, reputable media)
4. When sources conflict, verify across multiple sources
5. Note what is unknown or where information could not be found

### Interim Check-In (Optional)

For broad research or when direction needs to be confirmed, check in mid-way: "I'm currently investigating in this direction — does this angle look right to you?"

### Transition to Step 3

Move to the analysis phase when any of the following conditions are met:
- Quantitative and qualitative data for the main subtopics has been gathered
- Additional searches are no longer yielding new insights
- The user has indicated the research is sufficient

---

## Step 3: Analysis and Interpretation

Organize the gathered information and dig into the "So what?" — the actionable implication.

### Analytical Frameworks (select based on the topic)

See [reference/analysis-frameworks.md](reference/analysis-frameworks.md) for a list of available frameworks.

### Key Points for Interpretation

- Explain the context and constraints behind the numbers (e.g., "Growth rate is XX%, but the market had contracted the year before")
- Be honest about uncertainty (e.g., "This data is from [date]; the situation may have changed since then")
- Keep the focus on implications that directly inform the user's decision or action

### Transition to Step 4

Once the "So what?" of the analysis can be articulated, proceed to the insight delivery phase.

---

## Step 4: Delivering Insights

Organize the research findings and present them to the user.

**Example output (abbreviated):**

> ## Research Summary
> **Topic**: Current state of Japan's femtech market and key investment criteria
> **Key findings**:
> 1. Market size estimated at approximately ¥150–200 billion as of 2023 (Yano Research Institute estimate)
> 2. Menopause tech and B2B workplace health services are growth segments
> 3. Regulatory easing and favorable policy, but medical device certification costs pose a risk
>
> ## Analysis and Implications
> → In the short term, the B2B (corporate wellness) model offers the clearest path to monetization. A realistic strategy is to enter through the wellness space (which does not require medical device certification) while simultaneously building an evidence base.

### Standard Report Structure

```
## Research Summary
- Research topic and key questions
- Most important findings (3–5 points)

## Background and Current State
- Overview and context of the topic

## Key Findings
### [Subtopic 1]
- Quantitative data (with source citations)
- Qualitative insights

### [Subtopic 2]
...

## Analysis and Implications
- Interpretation of the data
- Implications for decision-making

## Caveats and Uncertainties
- Limitations of the research and points of caution

## Next Steps and Areas for Further Investigation
- What to investigate further
- Options to consider

## References
- [Title](URL)
- [Title](URL)
```

### Flexible Format Adjustment

Adapt the format to the user's needs:
- **Wants a detailed report**: Use the structure above
- **Wants a quick overview**: Key points in bullet form only
- **Wants to think it through together**: Present insights conversationally and deepen the discussion through dialogue

---

## Ongoing Deep Dives

Even after delivering findings, if the user says "tell me more" or "I'm curious about this point," actively dig deeper. Research is not a one-time deliverable — it is a process of progressively narrowing in on the insights the user needs through dialogue.

---

## Notes

- Always respond in the same language the user used to invoke the skill
- Always present the sources used at the end of the response as a list with title and URL (e.g., `- [Title](URL)`)
- For topics where recency matters, include the date of the information
- Be transparent when something could not be researched or information could not be found
- Clearly distinguish between confirmed facts and inferences or interpretations
- When the scope is broad and multiple subtopics need to be investigated in parallel, consider using the Task tool to launch sub-agents concurrently
