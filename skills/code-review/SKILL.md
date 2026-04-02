---
name: code-review
description: >
  Performs a multi-agent code review of the current branch's changes.
  Use when the user asks for a code review, PR review, diff review,
  or asks to check their changes for bugs and compliance issues.
  Launches parallel sub-agents at different model tiers (haiku, sonnet, opus)
  to review for CLAUDE.md compliance, bugs, and code problems, then validates
  and filters issues for a high-signal report.
---

# Code Review Skill

## Purpose

Orchestrate a multi-agent code review that catches real bugs and compliance violations while minimising false positives. This skill coordinates 6 specialised agents across 3 model tiers to review, validate, and report on branch changes.

## Installation

This skill ships with its own agents in the `agents/` subdirectory. Before use, install them into the target project's `.claude/agents/` directory:

```bash
# From the target project root
mkdir -p .claude/commands .claude/agents

# Install the skill as a slash command
cp path/to/AI-toolkit/skills/code-review/SKILL.md .claude/commands/code-review.md

# Install the agents
cp path/to/AI-toolkit/skills/code-review/agents/*.md .claude/agents/

# Or symlink everything for automatic updates
ln -s path/to/AI-toolkit/skills/code-review/SKILL.md .claude/commands/code-review.md
ln -s path/to/AI-toolkit/skills/code-review/agents/*.md .claude/agents/
```

Once installed, run `/code-review` in Claude Code to start a review.

### Required agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `review-file-discovery` | haiku | Find relevant CLAUDE.md/AGENTS.md files |
| `review-diff-summarizer` | sonnet | Summarise the diff |
| `review-compliance-checker` | sonnet | Audit CLAUDE.md/AGENTS.md compliance |
| `review-bug-scanner` | opus | Diff-only bug scan |
| `review-code-problem-detector` | opus | Security, logic, and correctness issues |
| `review-issue-validator` | sonnet/opus | Validate each flagged issue |

All agent definitions live in `skills/code-review/agents/` alongside this skill.

## Prerequisites

Before starting the review process, gather the raw materials.

### Get the diff

```bash
# Detect the default branch
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
if [ -z "$DEFAULT_BRANCH" ]; then DEFAULT_BRANCH="main"; fi

# Get the merge base
MERGE_BASE=$(git merge-base origin/$DEFAULT_BRANCH HEAD)

# Committed changes on this branch
git diff $MERGE_BASE HEAD

# Uncommitted changes (staged + unstaged)
git diff HEAD
```

Combine both outputs — the first shows all committed branch changes, the second shows work in progress. Together they form the **full diff** used throughout this review.

### Get changed file list

```bash
git diff --name-only $MERGE_BASE HEAD
git diff --name-only HEAD
```

Combine and deduplicate these into the **changed files list**.

### Get PR context (optional)

```bash
gh pr view --json title,body --jq '"\(.title)\n\n\(.body)"' 2>/dev/null
```

If this succeeds, capture the PR title and description. If it fails (no PR exists), proceed without it — this is not a blocker.

---

## Process

### Step 1 — File discovery + PR context + diff summary (parallel)

Launch these three actions **in parallel** (single message, multiple tool calls):

1. **Launch `review-file-discovery` agent** (haiku)
   - Pass it the changed files list
   - It returns paths to all relevant CLAUDE.md and AGENTS.md files

2. **Get PR context** (main session, Bash)
   - Run the `gh pr view` command above
   - If no PR exists, set PR context to empty

3. **Launch `review-diff-summarizer` agent** (sonnet)
   - Pass it the full diff
   - It returns a prose summary of the changes

### Step 2 — Read CLAUDE.md contents

After step 1 completes, read the contents of every CLAUDE.md and AGENTS.md file returned by the file-discovery agent. These contents are needed for step 3.

### Step 3 — Four parallel review agents

Launch all four agents **in parallel** in a single message:

#### Agent 1: Compliance checker A (sonnet)
- Use the `review-compliance-checker` agent
- Pass: full diff, CLAUDE.md/AGENTS.md contents, PR context
- Assign it the **first half** of the changed files list
- It returns a list of compliance violations

#### Agent 2: Compliance checker B (sonnet)
- Use the `review-compliance-checker` agent
- Pass: full diff, CLAUDE.md/AGENTS.md contents, PR context
- Assign it the **second half** of the changed files list
- It returns a list of compliance violations

#### Agent 3: Bug scanner (opus)
- Use the `review-bug-scanner` agent
- Pass: **only** the full diff and PR context — no CLAUDE.md contents, no file access
- It returns a list of bugs found from the diff alone

#### Agent 4: Code problem detector (opus)
- Use the `review-code-problem-detector` agent
- Pass: full diff and PR context
- It returns a list of security, logic, and correctness issues

### Step 4 — Collect and deduplicate issues

Gather all issues from the four review agents. Deduplicate any issues that flag the same file + line + problem. Combine into a single **issues list**.

If the issues list is empty, skip to Step 7 (final summary with "No issues found").

### Step 5 — Validate issues (parallel)

For each issue in the issues list, launch a `review-issue-validator` agent **in parallel** (batch all validators into a single message):

- **For bug and logic/correctness issues:** launch with `model: opus`
- **For compliance issues:** launch with `model: sonnet` (default)

Each validator receives:
- The issue description (file, line, category, reason)
- The PR title and description
- Instruction to read the code and confirm or reject the issue

Each returns `validated: true` or `validated: false` with justification.

### Step 6 — Filter

Remove any issues where the validator returned `validated: false`.

Also review the remaining issues against the exclusion criteria in `skills/code-review/references/false-positive-guide.md` and remove any that match.

The surviving issues are the **final issues list**.

### Step 7 — Post comments (if PR exists)

If a PR exists and the final issues list is not empty, post inline review comments using the GitHub API:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews \
  -f event=COMMENT \
  -f body="Code review: found N issue(s)" \
  -f 'comments=[{"path":"<file>","line":<line>,"body":"**[<category>]** <description>"}]'
```

If no PR exists, skip this step — the report in step 8 is the output.

### Step 8 — Final summary

Output a numbered list of validated issues in this format:

```markdown
## Code Review Results

**Branch:** <branch name>
**Files reviewed:** <count>
**Issues found:** <count>

---

### #1 <Short title>

<Description of the issue>

**File:** <path>
**Line:** <line number>
**Category:** <bug | compliance | security | logic | correctness>

---

### #2 <Short title>

...
```

If no issues were found:

```markdown
## Code Review Results

**Branch:** <branch name>
**Files reviewed:** <count>
**Issues found:** 0

No issues found. The changes look good.
```

---

## Output Format

The final output is a markdown report as shown in Step 8. Each issue includes a description, file location, and category. If a PR exists, inline comments are also posted.

---

## Rules

- **HIGH SIGNAL ONLY.** Every flagged issue must be either an objective bug or a clear CLAUDE.md violation with the exact rule quoted. Nothing else qualifies.
- **Only the main session posts comments.** All sub-agents return their findings to you — they never post comments themselves.
- **Do not use AskUserQuestion.** Complete the entire review without user intervention.
- **Use gh CLI for GitHub interactions.** Do not use web fetch for GitHub API calls.
- **Cite sources.** For CLAUDE.md violations, include the path to the CLAUDE.md file and quote the rule. For bugs, include the specific code and reasoning.
- **Handle missing PR gracefully.** If no PR exists, skip PR context gathering and comment posting. Output the report to the terminal instead.
- **Handle large diffs.** If the combined diff exceeds 50,000 characters, note this in the summary and prioritise reviewing the most critical files (source code over config, application code over tests).
- **Do not mention the orchestration process.** The user sees the final report, not the internal agent coordination. Do not describe which agents were launched or how results were combined.
- **Respect the false-positive guide.** Always apply the exclusion criteria in `references/false-positive-guide.md` before including an issue in the final report.
