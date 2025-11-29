# Technical Blog Post Style Guide

> **Scope**: This guide applies to **technical blog posts** (math, CS, programming). Other content types (running, food, personal) may have different conventions.

---

## Tone & Voice

- **Direct and confident** - no hedging or excessive caveats
- **Conversational but technical** - "Here's the connection:", "Watch the ordinals:", "Let me explain"
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
