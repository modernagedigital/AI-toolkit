# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal library of reusable AI assets — prompt templates, Claude Code skills, and subagent definitions. This is a content repo (markdown files), not a software project. There is no build system, test suite, or package manager.

## Structure

- `/prompts/` — Standalone one-shot instructions (plain `.md`, no frontmatter)
- `/skills/` — Multi-step behavioural instructions with process definitions, output formats, and rules (plain `.md`)
- `/agents/` — Claude Code subagent definitions (`.md` with YAML frontmatter: `name`, `description`, `tools`, `model`)
- `/assets/` — Images and other static files

## Conventions

- One asset per file, `kebab-case.md` naming
- Each file starts with a brief description of what it does and when to use it
- Prompts: use an HTML comment block (`<!-- What: ... When: ... -->`) at the top
- Skills: use markdown sections (`## Purpose`, `## Process`, `## Output Format`, `## Rules`)
- Agents: YAML frontmatter (`name`, `description`, `tools`, `model`) followed by a system prompt body
- If a prompt grows complex with rules and edge cases, it should become a skill
- If a skill needs tool access and autonomous decision-making, it should become an agent

## Agent Frontmatter Spec

```yaml
---
name: kebab-case-identifier
description: When Claude Code should delegate to this agent
tools: Read, Grep, Glob        # optional, defaults to all parent tools
model: sonnet                   # optional: sonnet | opus | haiku | full model ID
---
```

## When Adding New Assets

- Apply least-privilege for agent `tools` — research agents only need `Read, Grep, Glob`
- Use `model: haiku` or `model: sonnet` for focused tasks; reserve `opus` for complex reasoning
- Write specific `description` fields for agents — this drives automatic delegation
