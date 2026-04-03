# False Positive Exclusion Criteria

Use this list when evaluating and filtering code review issues. If an issue matches any of these criteria, it is a false positive and should **not** be flagged.

## Exclude These

- **Pre-existing issues** — Problems that existed before the change. Only flag issues introduced by the current diff.
- **Correct code that looks wrong** — Something that appears to be a bug but is actually correct behaviour when you understand the full context.
- **Pedantic nitpicks** — Issues that a senior engineer would not flag during review. If it doesn't affect correctness, security, or an explicit project rule, skip it.
- **Linter-catchable issues** — Problems that a linter, formatter, or type checker will catch automatically. Do not duplicate their work. Do not run the linter to verify.
- **General code quality concerns** — Lack of test coverage, missing documentation, naming preferences, or general security hardening — unless explicitly required by a CLAUDE.md or AGENTS.md rule.
- **Silenced rules** — Issues covered by CLAUDE.md or AGENTS.md but explicitly silenced in the code via inline comments (e.g. `// eslint-disable`, `# noqa`, `@SuppressWarnings`).
- **Subjective concerns** — Style preferences, "suggestions", or "you might want to consider" items. Only objective, verifiable issues qualify.
- **Speculative issues** — Anything that "might" be a problem or "could potentially" cause issues. If you are not certain, do not flag it.

## The Standard

Every flagged issue must be one of:
1. An **objective bug** that will cause incorrect behaviour at runtime
2. A **clear, unambiguous CLAUDE.md/AGENTS.md violation** where you can quote the exact rule being broken

If an issue does not meet this standard, it does not belong in the review.
