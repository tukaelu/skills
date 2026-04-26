# Skill Authoring Best Practices

## Contents

- [Core principles](#core-principles)
- [Progressive disclosure patterns](#progressive-disclosure-patterns)
- [Common patterns](#common-patterns)
- [Feedback loops](#feedback-loops)
- [Anti-patterns](#anti-patterns)
- [Testing and evaluation](#testing-and-evaluation)

---

## Core principles

### Concise is key

Claude is already smart. Only add context Claude doesn't already have. Challenge every piece of information:
- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"

**Good (≈20 tokens):**
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

**Bad (≈60 tokens):**
> PDF (Portable Document Format) files are a common file format that contains text, images, and other content. To extract text from a PDF, you'll need to use a library. There are many libraries available...

The good version assumes Claude knows what PDFs are and how libraries work.

### Degrees of freedom (expanded)

**High freedom** — use when multiple approaches are valid:
```md
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
```

**Medium freedom** — use when a preferred pattern exists:
```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
```

**Low freedom** — use when operations are fragile:
```md
## Database migration

Run exactly this script:
```bash
python scripts/migrate.py --verify --backup
```
Do not modify the command or add additional flags.
```

---

## Progressive disclosure patterns

SKILL.md is a table of contents. Claude reads additional files only when needed — no token cost until accessed.

### Pattern 1: High-level guide with references

```md
## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide  
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
```

### Pattern 2: Domain-specific organization

When a skill covers multiple domains, organize content by domain so Claude only loads what's relevant:

```
bigquery-skill/
├── SKILL.md
└── reference/
    ├── finance.md    # revenue, billing metrics
    ├── sales.md      # opportunities, pipeline
    └── product.md    # API usage, features
```

```md
## Available datasets

**Finance** → See [reference/finance.md](reference/finance.md)  
**Sales** → See [reference/sales.md](reference/sales.md)
```

### Pattern 3: Conditional details

```md
## Document editing

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)  
**For OOXML details**: See [OOXML.md](OOXML.md)
```

### Rules for references

- Keep all references **one level deep** from SKILL.md (`a.md → b.md → c.md` is bad — Claude may partial-read and miss details)
- For reference files longer than 100 lines, add a table of contents at the top

---

## Common patterns

### Template pattern

**Strict** (for API responses, data formats):
```md
ALWAYS use this exact structure:

# [Title]
## Executive summary
[One-paragraph overview]
## Key findings
- Finding 1
## Recommendations
1. Actionable recommendation
```

**Flexible** (when adaptation is useful):
```md
Here is a sensible default format — adjust as needed:
```

### Examples pattern

For output quality that depends on seeing examples, provide input/output pairs:

```md
**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```
```

### Conditional workflow pattern

```md
1. Determine the modification type:
   **Creating new content?** → Follow "Creation workflow"
   **Editing existing content?** → Follow "Editing workflow"

2. Creation workflow: ...
3. Editing workflow: ...
```

---

## Feedback loops

Use the validate → fix → repeat loop for quality-critical tasks:

```md
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues
   - Run validation again
4. **Only proceed when validation passes**
```

For complex multi-step workflows, provide a copy-paste checklist:

```md
Copy this checklist and track your progress:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
```
```

---

## Anti-patterns

### Windows-style paths

- ✓ Good: `scripts/helper.py`, `reference/guide.md`
- ✗ Avoid: `scripts\helper.py`, `reference\guide.md`

### Too many options

```md
# Bad — confusing
You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image...

# Good — default with escape hatch
Use pdfplumber for text extraction:
```python
import pdfplumber
```
For scanned PDFs requiring OCR, use pdf2image with pytesseract instead.
```

### Time-sensitive information

```md
# Bad
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.

# Good
## Current method
Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns
<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>
...
</details>
```

### Scripts that punt to Claude

```python
# Bad — just fails and lets Claude figure it out
def process_file(path):
    return open(path).read()

# Good — handles errors explicitly
def process_file(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        print(f"File {path} not found, creating default")
        with open(path, "w") as f:
            f.write("")
        return ""
```

### Magic numbers (voodoo constants)

```python
# Bad
TIMEOUT = 47
RETRIES = 5

# Good — self-documenting
REQUEST_TIMEOUT = 30  # most HTTP requests complete within 30s
MAX_RETRIES = 3       # most intermittent failures resolve by the second retry
```

---

## Testing and evaluation

### Test with all models you plan to use

- **Haiku** (fast, economical): does the skill provide enough guidance?
- **Sonnet** (balanced): is the skill clear and efficient?
- **Opus** (powerful reasoning): does the skill avoid over-explaining?

What works for Opus might need more detail for Haiku.

### Evaluation-driven development

Build evaluations **before** writing extensive documentation:

1. Run Claude on representative tasks without the skill — document specific failures
2. Create 3+ evaluation scenarios targeting those gaps
3. Establish a baseline (Claude's performance without the skill)
4. Write minimal instructions to pass evaluations
5. Iterate: run evaluations → compare against baseline → refine

**Evaluation structure:**
```json
{
  "skills": ["writing-skill"],
  "query": "Create a skill for processing PDF forms",
  "expected_behavior": [
    "Creates SKILL.md with valid frontmatter (name, description)",
    "Description is in third person and includes Use when triggers",
    "SKILL.md body is under 500 lines",
    "References are one level deep"
  ]
}
```

### Iterative refinement with Claude A/B

- **Claude A**: helps design and refine skill instructions
- **Claude B**: fresh instance that uses the skill on real tasks

Observe Claude B's behavior → bring insights back to Claude A → update → repeat.
