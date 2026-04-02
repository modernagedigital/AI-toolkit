# Agents

Claude Code subagent definitions. Each file is a self-contained agent that Claude Code can delegate tasks to automatically. Subagents run in their own context window, do their work independently, and return a summary to the main session.

## When to Use

Use an agent when the task requires autonomy — Claude needs to explore, make decisions, use tools, and report back without you guiding each step. If a skill defines _how_ to work, an agent defines _a worker_.

## Format

Each agent is a single `.md` file with YAML frontmatter and a system prompt body.

```markdown
---
name: design-token-auditor
description: Audits design token usage across components, flags hardcoded values
tools: Read, Grep, Glob
model: sonnet
---

You are a design systems specialist. Audit the codebase for:

- Hardcoded colour, spacing, and font-size values that should use tokens
- Inconsistent token naming patterns
- Unused or duplicate token definitions

## Output

Return a structured report with:

- File path and line number for each issue
- The hardcoded value found
- Suggested token replacement

## Rules

- Read-only — do not modify any files
- Skip node_modules, dist, and build directories
- Group findings by severity: critical, warning, info
```

## Frontmatter Reference

| Field             | Required | Description                                                                                                               |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `name`            | Yes      | Unique identifier, used to invoke the agent by name                                                                       |
| `description`     | Yes      | Tells Claude Code when to delegate to this agent                                                                          |
| `tools`           | No       | Allowlist of tools (e.g. `Read, Grep, Glob, Bash, Edit, Write`). Defaults to inheriting all tools from the parent session |
| `disallowedTools` | No       | Denylist — inherit everything except these                                                                                |
| `model`           | No       | `sonnet`, `opus`, `haiku`, or a full model ID. Defaults to the parent session's model                                     |

## Installation

**Project-level** — copy or symlink into your project:

```
.claude/agents/design-token-auditor.md
```

**Global** — available across all projects:

```
~/.claude/agents/design-token-auditor.md
```

Project-level agents override global ones if there's a naming conflict.

## Naming

Use `kebab-case.md`. Name it after the agent's role.

```
design-token-auditor.md
component-accessibility-checker.md
test-coverage-reporter.md
codebase-explorer.md
```

## Tips

- Write clear `description` fields — this is how Claude Code decides whether to delegate to the agent. Be specific about the trigger conditions.
- Apply the principle of least privilege with `tools`. A research agent only needs `Read, Grep, Glob`. Don't give `Write` or `Bash` unless the agent needs them.
- Use `model: haiku` or `model: sonnet` for focused, repetitive tasks to save cost. Reserve `opus` for complex reasoning.
- Keep the system prompt focused on one domain. If an agent is doing too many things, split it.
- You can invoke any agent explicitly: _"Use the design-token-auditor agent to check this repo."_
