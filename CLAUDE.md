# Claude Code Instructions

## Project Structure

- **`.docs/`** - Internal documentation (style guides, codebase notes). See `BLOG_STYLE.md` for blog writing conventions.
- **`.plans/`** - Master plans and implementation plans. Naming convention:
  - Master plans: `###-MP.md` (e.g., `001-MP.md`, `002-MP.md`)
  - Implementation plans: `###-IP.md` (e.g., `001-IP.md`, `002-IP.md`)

## Blog Posts

When writing technical blog posts:

- **No prerequisites sections** - don't add "Prerequisites:" or similar preambles listing what readers should know
- **Spaced dashes** - use ` - ` not em dashes `—` (e.g., "simple to define - not complex")
- **No "Complete Code" sections** - code should be interleaved throughout, not dumped at the end
- **Single arrows in math** - use `\to` not `\Rightarrow` for sequence steps
- **Avoid AI patterns** - no "Let's explore", "Here's the thing:", "The key insight is...", etc.

See `.docs/BLOG_STYLE.md` for the full technical blog style guide (including detailed AI pattern avoidance).
