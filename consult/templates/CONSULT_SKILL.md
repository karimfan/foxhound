## Consulting Human Experts

This project has the `consult` tool. Use it to ask human experts for help via Slack when you're unsure about code you're modifying.

### Decision Framework

Before consulting a human, work through these layers in order:

1. **Wiki sufficient?** Check the project wiki (`wiki.md`, `SUMMARY.md` files, `recipes.md`, `deps-graph.md`) for architecture, patterns, and dependencies. If the answer is documented, use it.
2. **Source sufficient?** Read the source code, git history, and inline comments. If the code and its history make the intent clear, proceed without consulting.
3. **Consult a human.** When neither wiki nor source resolves the question, use `consult` to reach the right person.

### When to Consult

- **High-churn unfamiliar code** — files with many recent authors and no clear owner signal evolving conventions that may not be documented.
- **Public API changes** — modifications to exported interfaces, REST endpoints, or shared contracts can break downstream consumers in ways the code alone won't reveal.
- **Code with warning comments** — `// HACK`, `// XXX`, `// DO NOT CHANGE`, `// FRAGILE` indicate tribal knowledge that the original author should confirm.
- **Tribal knowledge** — deployment procedures, feature flags, rollout sequences, or environment-specific behaviors that live in people's heads.
- **You're unsure and the cost of being wrong is high** — if reverting the change would be painful, ask first.

### When NOT to Consult

- **Tests** — adding or updating tests for existing behavior.
- **Typos and formatting** — mechanical fixes with no semantic risk.
- **Well-documented code** — changes to code where intent is clear from docs, comments, and naming.
- **Routine refactoring** — renaming, extracting functions, or restructuring when behavior is preserved and tests pass.

### Step-by-step workflow

**Step 1: Preview experts (no Slack needed)**

```bash
consult who --file path/to/file.go
```

This shows who knows the code best. Use it to decide whether consultation is worthwhile. If the top expert has a low score or hasn't committed recently, the code may not have a clear owner.

**Step 2: Send a question or proposal**

To ask a question:
```bash
consult ask --file path/to/file.go \
    --question "Is the token refresh intentionally skipped for service accounts?"
```

To propose a change and get feedback on a diff:
```bash
consult propose --file path/to/file.go \
    --diff "$(git diff HEAD)" \
    --question "Does this migration look safe?"
```

Both commands will output a **session ID**. Save it — you need it to check for the response.

Example output:
```
Contacted Alice Smith (alice@company.com) via Slack
Session ID: a1b2c3d4
Check for a reply with: consult check --session a1b2c3d4
```

**Step 3: Continue other work while waiting**

The expert will reply in Slack. Humans may take minutes or hours. Don't block — continue with other tasks that don't depend on the answer.

**Step 4: Poll for the response**

```bash
consult check --session a1b2c3d4
```

If the expert has replied:
```
Response received for session a1b2c3d4:
Yes, use the middleware in pkg/middleware/ratelimit.go — it already handles per-user limiting.
```

If not yet:
```
No response yet.
```

Poll periodically. If no response after a reasonable time (e.g., 30 minutes for urgent questions, end of day for non-urgent), proceed with your best judgment and note the unanswered consultation in the PR description.

**Step 5: List all sessions**

```bash
consult sessions
```

Shows all consultations with their status (pending/complete).

### Preview without sending

Use `--dry-run` to see who would be contacted and what the message would look like, without actually sending anything:

```bash
consult ask --dry-run --file path/to/file.go \
    --question "Is this safe to change?"
```

No Slack token is needed for `--dry-run` or `consult who`.

### Tips for Good Questions

- **State what you already know.** "I read the wiki entry on X and the git history shows Y, but I'm unclear on Z." This respects the expert's time.
- **Be specific about the file and line range.** Not "how does auth work?" but "In `auth/middleware.go:45-60`, is the token refresh intentionally skipped for service accounts?"
- **Explain the change you're considering.** Describe the proposed change so the expert can flag risks you don't see.
- **One question per consult.** Multiple unrelated questions in one message reduce response quality. Split them into separate sessions.
