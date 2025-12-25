# Technical Blog Post Style Guide

> **Scope**: This guide applies to **technical blog posts** (math, CS, programming). Other content types (running, food, personal) may have different conventions.

---

## Tone & Voice

- **Direct and confident** - no hedging or excessive caveats
- **Conversational but technical** - engage the reader without tutorial-speak
- **Show the work** - full notation, don't skip steps
- **Honest about limitations** - note assumptions where relevant
- **No fluff** - dive into substance quickly

## Structure

### Opening
- Immediately state why the topic is interesting (the surprising result, the paradox, the speedup)
- No generic introductions ("In this post we will...")
- Hook with the counterintuitive or remarkable aspect

### Body
1. **Problem statement** - with historical context if relevant
2. **Naive/classical approach** - establish baseline
3. **The twist** - "But what if...", "Enter...", "Here's where it gets interesting"
4. **Step-by-step walkthrough** - interleave math and code
5. **Why it works** - the key insight or proof sketch
6. **Comparisons/analysis** - tables work well here

### Closing
- Numbered takeaways (3-5 points)
- Further reading links
- References (academic papers where applicable)

## Formatting Conventions

### Punctuation
- **Dashes**: Use spaced dashes ` - ` not em dashes `—`
  - Correct: `simple to define - hereditary base notation`
  - Incorrect: `simple to define—hereditary base notation`

### Prerequisites
- **Do not include prerequisites sections** - let the content speak for itself
- If background is truly needed, weave it into the explanation naturally

### Math
- Use `$...$` for inline math
- Use `$$...$$` for display math
- Use single arrows `\to` for implications in sequences, not `\Rightarrow`
- LaTeX in descriptions should be avoided (plain text for SEO)

### Code
- Break into small chunks (3-15 lines) that illustrate specific concepts
- Each snippet should follow or precede its mathematical explanation
- Use type hints in Python
- Variable names should match mathematical notation where sensible
- Comments are minimal - prose explains intent
- No "complete implementation" dump sections at the end

### Tables
- Use for comparisons (classical vs quantum, growth rates, etc.)
- Keep concise - 3-5 rows typically sufficient

## What NOT to Do

- No emojis (unless explicitly requested)
- No "click here" link text
- No redundant section summaries
- No over-engineering explanations for simple concepts
- No apologetic language ("I'll try to explain...")
- No time estimates or schedules

## Avoiding AI-Like Patterns

These patterns are common in AI-generated text and should be avoided or rewritten.

### Phrases to avoid

| Pattern | Problem | Fix |
|---------|---------|-----|
| "Let's explore..." / "Let me explain..." | Tutorial-speak | Just explain it directly |
| "Here's the thing:" / "Here's what..." | Filler before content | Delete and start with the content |
| "The key insight is..." / "The key is..." | Overused framing | "It turns out..." or just state the insight |
| "This is the famous X" | Unnecessary hype | "This is X" |
| "Now comes the key step" | Filler | "Next" or just proceed |
| "beautifully illustrates" | AI superlative | "illustrates" or "shows" |

### Structural patterns to avoid

- **Over-parallel lists** - numbered lists where every item is `**Bold label** - explanation`. Vary the structure or convert to prose.
- **Rhetorical question + immediate answer** - "How does this work? Here's the answer:" Just explain it.
- **Label: explanation** pattern in prose - "The connection: we encode..." becomes "We encode..."
  - Exception: colons are fine before code blocks, math, or step-by-step proofs

### Code style

- **No periods at end of single-line docstrings** - `"""Convert n to base b"""` not `"""Convert n to base b."""`
- **No comma formatting in numbers** - use `{n}` not `{n:,}`, output `2352161` not `2,352,161`
- Comments in code should be terse fragments, not sentences

### Numbers in prose and math

- **No comma separators** - write `10000` not `10,000`, use `$10^{121210694}$` not `$10^{121,210,694}$`

## Example Post Structure

```markdown
---
title: "Short Punchy Title"
description: "One sentence with the key insight, under 160 chars."
pubDate: YYYY-MM-DD
---

[Hook paragraph - the surprising/counterintuitive result]

[Brief roadmap sentence - what this post covers]

## The Problem/Setup

[Historical context, problem statement]

## The Naive Approach

[Classical solution, baseline]

## The Key Insight

[Where things get interesting]

## Implementation

[Code interleaved with math]

## Why It Works

[Proof sketch or intuition]

## Comparison/Analysis

[Tables, complexity, scale]

## Conclusion

[Numbered takeaways]

## Further Reading

[Links]

## References

[Academic citations]
```

---

*This guide reflects the style established in posts like "Quantum Counterfeit Coins" and "Goodstein Sequences".*
