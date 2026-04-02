# Prompts

Standalone instructions you paste into a conversation. No tools, no memory, no special setup — text in, text out.

## When to Use

Use a prompt when you need a repeatable instruction for a single task. If you find yourself typing the same thing into Claude over and over, it belongs here.

## Format

Each prompt is a single `.md` file. Start with a brief comment block describing what it does, then the prompt body.

```markdown
<!--
  What: Reviews a component for design system compliance
  When: After building or updating any UI component
-->

Review the following component and check it follows our design system conventions...
```

## Naming

Use `kebab-case.md` and be descriptive enough that you can find it by scanning filenames.

```
review-design-system-compliance.md
rewrite-copy-for-accessibility.md
generate-alt-text.md
summarise-meeting-notes.md
```

## Tips

- Keep prompts single-purpose. If a prompt tries to do three things, split it into three files.
- Include example input/output in the comment block if the prompt's behaviour isn't obvious.
- If a prompt keeps growing with rules and edge cases, it's probably a skill — move it to `/skills`.
