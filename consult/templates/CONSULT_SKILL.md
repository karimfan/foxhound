## Consulting Human Experts

### Decision Framework

Before consulting a human, work through these layers in order:

1. **Wiki sufficient?** Check the project wiki (`docs/`, `SUMMARY.md` files, HTML docs) for architectural decisions, API contracts, and design rationale. If the answer is documented, use it.
2. **Source sufficient?** Read the source code, git history, and inline comments. If the code and its history make the intent clear, proceed without consulting.
3. **Consult a human.** When neither wiki nor source resolves the question, use `consult` to reach the right person.

### When to Consult

- **High-churn unfamiliar code** — files with many recent authors and no clear owner signal evolving conventions that may not be documented.
- **Public API changes** — modifications to exported interfaces, REST endpoints, or shared contracts can break downstream consumers in ways the code alone won't reveal.
- **Code with warning comments** — `// HACK`, `// XXX`, `// DO NOT CHANGE`, `// FRAGILE` indicate tribal knowledge that the original author should confirm.
- **Tribal knowledge** — deployment procedures, feature flags, rollout sequences, or environment-specific behaviors that live in people's heads.

### When NOT to Consult

- **Tests** — adding or updating tests for existing behavior.
- **Typos and formatting** — mechanical fixes with no semantic risk.
- **Well-documented code** — changes to code where intent is clear from docs, comments, and naming.
- **Routine refactoring** — renaming, extracting functions, or restructuring when behavior is preserved and tests pass.

### How to Use

```bash
# 1. Identify who knows this code best
consult who --file path/to/file.go

# 2. Assess whether you truly need to ask (re-check wiki/source first)

# 3. Ask a question
consult ask --file path/to/file.go -q "Is the heading parser intentionally recursive?"

# 4. Or propose a change with a diff
consult propose --file path/to/file.go --diff changes.patch -q "Does this migration look safe?"

# 5. Check for a reply
consult check --session <session-id>

# 6. List all active sessions
consult sessions
```

### Tips for Good Questions

- **State what you already know.** "I read the wiki entry on X and the git history shows Y, but I'm unclear on Z." This respects the expert's time and avoids redundant explanation.
- **Be specific about the file and line range.** Vague questions like "how does auth work?" waste time. Prefer "In `auth/middleware.go:45-60`, is the token refresh intentionally skipped for service accounts?"
- **Explain the change you're considering.** If you're about to modify something, describe the proposed change so the expert can flag risks you might not see.
- **One question per consult.** Multiple unrelated questions in one message reduce response quality. Split them into separate sessions.
