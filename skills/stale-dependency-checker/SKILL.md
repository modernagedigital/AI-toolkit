---
name: stale-dependency-checker
description: >
  Audits package.json files for outdated, deprecated, duplicated, or insecure
  dependencies and produces a risk-aware upgrade report. Use this skill whenever
  the user asks to check, audit, or review their npm/yarn/pnpm dependencies,
  mentions outdated packages, wants to know what needs updating, or is
  evaluating whether it's safe to upgrade something. Also trigger when the user
  mentions dependency security, version drift, or asks "what's out of date" in
  a JS/TS project. Covers monorepos (multiple package.json files) automatically.
tools: Read, Bash, Grep, Glob, mcp__context7__resolve-library-id, mcp__context7__query-docs
model: haiku
---

# Stale Dependency Checker

## Purpose

Scan every `package.json` in a project (including monorepo workspaces), identify dependencies that are outdated, deprecated, duplicated across packages, or flagged by `npm audit`, and present a single actionable report. The report must be pragmatic: upgrading a transitive peer dependency that would break a framework (e.g. bumping React inside a pinned Next.js range) is worse than leaving it alone. Every suggestion should weigh the real-world risk of upgrading against the risk of staying put.

## Process

### 1. Discover package files

```bash
find . -name "package.json" -not -path "*/node_modules/*" -not -path "*/.next/*" -not -path "*/dist/*"
```

If there is a root `package.json` with a `workspaces` field, note the workspace layout — duplicated or mismatched versions across workspaces are worth flagging.

### 2. Gather version data

For each `package.json`, run:

```bash
npx npm-check-updates --jsonUpgraded 2>/dev/null
```

This gives you a map of `{ "package": "latest version" }` for everything that has a newer release. If `npm-check-updates` is not installed and `npx` fails, fall back to comparing `npm view <pkg> version` for each dependency — batch them to keep it fast.

Also run:

```bash
npm audit --json 2>/dev/null
```

to collect known vulnerabilities. If the project uses `yarn` or `pnpm` (look for `yarn.lock` / `pnpm-lock.yaml`), use the equivalent command (`yarn audit --json` / `pnpm audit --json`).

### 3. Check for deprecated packages

For any package that looks stale or unfamiliar, run:

```bash
npm view <package> deprecated 2>/dev/null
```

A non-empty result means the package is officially deprecated. Note the deprecation message — it often names the replacement.

### 4. Assess upgrade risk

This is the most important step. Not every outdated package should be upgraded. Before recommending an upgrade, think about whether it could break something.

**Framework coupling** — Some packages are tightly coupled. Upgrading one without the other causes breakage. Common examples:
- `react` + `react-dom` must stay in sync
- `next` pins compatible ranges of `react`, `eslint-config-next`, etc.
- `gatsby` and its plugin ecosystem
- `@angular/*` packages must all share a major version
- CMS SDKs (`@sanity/client`, `@contentful/rich-text-react-renderer`, `storyblok-js-client`, etc.) may pin specific framework versions

When you spot a tightly-coupled group, use Context7 MCP to check the framework's current compatibility requirements:

1. `resolve-library-id` with the framework name (e.g. "next.js")
2. `query-docs` asking about peer dependency requirements or upgrade guides

If the framework's docs say "requires React 18", don't suggest React 19.

**Major vs minor/patch** — Major bumps deserve extra scrutiny. A patch or minor update is almost always safe. A major bump may have breaking changes. Mention this distinction in the report.

**Security vs freshness** — A package with a known CVE needs urgent attention regardless of breakage risk. Flag these clearly. A package that's simply a few minor versions behind with no security issues is low priority.

### 5. Detect duplicates (monorepos)

If multiple `package.json` files exist, compare dependency versions across them. Flag cases where:
- The same package appears at different major versions in different workspaces
- A dependency is listed in both root and workspace `package.json` at conflicting versions

### 6. Build the report

Present findings as a markdown table, grouped by package.json location (for monorepos). Use this structure:

```
## Dependency Audit: `<path/to/package.json>`

| Package | Current | Latest | Type | Risk | Action |
|---------|---------|--------|------|------|--------|
| lodash  | 4.17.20 | 4.17.21 | patch | low | Upgrade — minor security fix |
| react   | 18.2.0  | 19.1.0  | major | high | Hold — Next.js 14 requires React 18 |
| left-pad | 1.3.0  | — | deprecated | medium | Replace with String.prototype.padStart |
```

**Column definitions:**
- **Type**: `patch`, `minor`, `major`, `deprecated`, `vulnerable`
- **Risk**: `low`, `medium`, `high` — how likely is this upgrade to break something?
- **Action**: One of `Upgrade`, `Hold`, `Replace`, `Investigate` — with a short reason

After each table, add a **Security** section if `npm audit` found vulnerabilities, listing CVE IDs and severity.

If there are **cross-workspace duplicates**, add a final section listing mismatched versions.

### 7. Offer to apply safe upgrades

After presenting the report, offer to apply the upgrades that are marked as low-risk. Explain what you'll do before doing it:

> "I can update the following low-risk packages for you: [list]. This will modify package.json but won't run install — you can review the changes and install when ready. Want me to go ahead?"

Only modify `package.json` version strings. Do not run `npm install` or modify lock files — leave that to the user so they can verify the changes first.

For high-risk items, suggest the upgrade path but don't offer to auto-apply. For example: "Upgrading to Next.js 15 would also require React 19 and updates to your middleware — worth doing as a dedicated task."

## Rules

- Never run `npm install`, `yarn install`, or `pnpm install` — the user should do this after reviewing changes
- Never delete or modify lock files
- If `npm audit` or `npx npm-check-updates` is unavailable, fall back gracefully to manual `npm view` checks — don't error out
- When in doubt about compatibility, check Context7 docs rather than guessing
- Always flag security vulnerabilities prominently, even if the upgrade is risky
- Keep the report scannable — the table is the primary output, not paragraphs of prose
