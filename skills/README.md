# Skills

Structured instructions that define how Claude should behave across a task. A skill goes beyond a one-shot prompt — it includes rules about tools, output formats, step sequences, and edge case handling.

## When to Use

Use a skill when the task requires Claude to follow a specific process, not just answer a question. If you're defining _how_ Claude should work rather than _what_ to do once, it's a skill.

## Format

Each skill is a single `.md` file. Structure it with clear sections so Claude can parse and follow it reliably.

```markdown
# Component Audit Skill

## Purpose

Audit a UI component for design system compliance, accessibility, and code quality.

## Process

1. Read the component file
2. Check naming conventions against the design system
3. Verify token usage (no hardcoded values)
4. Run accessibility checks
5. Output a structured report

## Output Format

Return a markdown table with columns: Issue, Severity, File, Line, Suggestion

## Rules

- Flag hardcoded colours, spacing, and font sizes
- Check for semantic HTML usage
- Verify ARIA attributes where interactive elements exist
- Do not modify any files — this is read-only analysis
```

## Naming

Use `kebab-case.md`. Name it after the capability, not the context.

```
component-audit.md
design-token-migration.md
content-accessibility-rewrite.md
figma-to-code-review.md
```

## Where Skills Get Used

- **Claude Projects** — paste into custom instructions
- **Claude Code** — save as a `SKILL.md` and reference it in `CLAUDE.md`
- **Conversations** — paste at the start of a chat to set Claude's behaviour

## Tips

- Be explicit about what tools or file types the skill expects.
- Include constraints (read-only, no external requests, specific output format).
- If a skill needs tool access and autonomous decision-making, it might be better as an agent in `/agents`.
