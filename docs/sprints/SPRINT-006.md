# Sprint 006: Expert Consultation Tool (`consult`)

## Overview

LLM agents working on unfamiliar code today have two options: read documentation (if it exists) or read source code and hope for the best. wikigen addresses the first gap by generating navigable documentation. But documentation can't capture everything — tribal knowledge, active refactoring intent, subtle constraints, and "don't touch this because..." warnings live in people's heads, not in code or docs.

This sprint builds `consult`, a CLI tool that lets an LLM agent ask human experts for help via Slack, the same way a junior developer would tap a senior on the shoulder. The tool uses `git log` and `git blame` to identify who has been most active in the code the agent is about to modify, looks up those people on Slack via their git commit email, and sends a structured message with context: what code is being changed, what the agent is planning to do, and what it's unsure about. The expert replies in a Slack thread, and the agent polls for the response asynchronously.

This is the second tool in a growing suite of LLM code-assistance tools. wikigen provides static understanding; `consult` provides dynamic, human-in-the-loop understanding. Together they cover the spectrum from "what does this code do" to "should I change this code this way."

## Use Cases

1. **Pre-change review request**: Agent is about to modify unfamiliar code. Runs `consult propose` to send the proposed diff to the most recent committers for a gut check before proceeding.
2. **Targeted question**: Agent needs to understand why a particular approach was chosen. Runs `consult ask` to ask the original author.
3. **Ownership discovery**: Agent (or human) runs `consult who` to see who knows about a file without contacting anyone. Works without Slack.
4. **Boundary-crossing change**: Agent is modifying a public API. Consults maintainers of dependent packages about the contract change.
5. **High-churn area caution**: Agent sees from churn.md that an area is actively changing. Consults active developers to avoid conflicts.

## Architecture

```
                    ┌─────────────────────────┐
                    │   consult/main.go       │
                    │   CLI entry + commands  │
                    └────────┬────────────────┘
                             │
              ┌──────────────┼──────────────────┐
              │              │                  │
     ┌────────▼───────┐ ┌───▼──────────┐ ┌────▼──────────┐
     │ git.go          │ │ slack.go     │ │ session.go    │
     │                 │ │              │ │               │
     │ - git blame     │ │ - lookupUser │ │ - create      │
     │ - git log       │ │ - postMsg    │ │ - load        │
     │ - rank experts  │ │ - getThread  │ │ - check       │
     └────────┬───────┘ └───┬──────────┘ └────┬──────────┘
              │             │                  │
              │     ┌───────▼──────────┐       │
              └────►│ message.go       │◄──────┘
                    │                  │
                    │ - format ask     │
                    │ - format propose │
                    │ - wiki excerpt   │
                    └──────────────────┘
```

### Data Flow

**Sending a consultation (`ask` or `propose`):**
```
1. Agent runs: consult ask --file path/to/file.go --question "Is it safe to ..."
2. Git analysis (git.go): git blame + git log → rank authors by recency and frequency
3. Slack lookup (slack.go): git email → Slack user ID via users.lookupByEmail
   → falls back to .consult.json overrides if lookup fails
4. Message build (message.go): format question + file context + optional diff + wiki excerpt
5. Slack send (slack.go): post message to top expert's DM
6. Session save (session.go): write .consult/{id}.json with thread info
7. Output: print session ID and who was contacted
```

**Checking for a response:**
```
1. Agent runs: consult check --session {id}
2. Load session file from .consult/
3. Slack API: fetch thread replies
4. If reply exists: print it, mark session complete
5. If no reply: print "no response yet", exit 0
```

### CLI Interface

```bash
# Identify experts for a file (no Slack needed, no side effects)
consult who --file internal/auth/handler.go

# Ask a question about a file
consult ask --file internal/auth/handler.go \
            --question "I'm adding rate limiting. Is there an existing middleware?"

# Ask about a directory
consult ask --dir internal/auth/ \
            --question "I need to add OAuth2. What should I be careful about?"

# Propose a change (include a diff)
consult propose --file internal/auth/handler.go \
                --diff "$(git diff HEAD)" \
                --question "Does this look safe? Worried about session handling."

# Check for a response
consult check --session abc123

# List recent sessions
consult sessions

# Dry run — show who would be contacted and the message, without sending
consult ask --file internal/auth/handler.go --question "..." --dry-run

# Update CLAUDE.md and AGENTS.md with consult skill instructions
consult --update-skills
```

### Expert Ranking Algorithm

```
For a given file or directory:

1. git log --format="%ae %H" --since=1y -- <path>
   → All authors who've committed to this path in the last year

2. git log --format="%ae %H" --since=90d -- <path>
   → Recent authors (weighted higher)

3. For files: git blame --porcelain <file>
   → Who wrote each current line

4. Score = (commits_last_90d * 3) + (commits_last_year * 1) + (blame_lines * 0.5)
   → Weighted toward recency > volume > current ownership

5. Sort by score descending, take top 3

6. For each: Slack lookup via users.lookupByEmail
   → Falls back to .consult.json if configured
```

### Identity Resolution

**Primary**: Slack `users.lookupByEmail` API — zero config, works when git email matches Slack email.

**Fallback**: Optional `.consult.json` at repo root for overrides:
```json
{
  "user_map": {
    "alice-old@personal.com": "U12345",
    "bob@contractor.io": "U67890"
  },
  "default_channel": "C_TEAM_CHANNEL"
}
```

The config is optional. If neither lookup nor config resolves a user, the tool reports who it found in git but couldn't map to Slack, and suggests adding them to `.consult.json`.

### Session Storage

Sessions are stored in `.consult/` under the repo root (not CWD):

```
.consult/
  abc123.json
  def456.json
```

Each session file:
```json
{
  "id": "abc123",
  "created_at": "2026-04-06T10:30:00Z",
  "file": "internal/auth/handler.go",
  "question": "Is it safe to add rate limiting here?",
  "type": "ask",
  "experts": [
    {"name": "Alice", "email": "alice@co.com", "slack_id": "U123", "commits_90d": 12, "commits_1y": 47, "score": 83.5}
  ],
  "slack_channel": "D456",
  "slack_thread_ts": "1712345678.000100",
  "status": "pending",
  "response": null
}
```

### Slack Message Format

```
🔍 *Code Consultation Request*

*File:* `internal/auth/handler.go`
*Type:* Question
*From:* LLM Agent (via consult)

*Question:*
I'm adding rate limiting to the login handler. Is there an existing middleware I should use instead of implementing it inline?

*Why you:* You've made 12 commits to this file in the last 90 days (47 in the last year).

*File summary (from wiki):*
> HTTP request handlers for authentication. Handles login, logout, session validation.

━━━
_Reply in this thread. The agent will poll for your response._
```

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SLACK_BOT_TOKEN` | For `ask`/`propose`/`check` | Bot token with `chat:write`, `users:read.email`, `im:write` scopes |

Not required for `who`, `sessions`, or `--dry-run`.

## Implementation Plan

### Phase 1: Project Structure & CLI Skeleton (~12%)

**Files:**
- `consult/main.go` — CLI entry point, subcommand dispatch, flag parsing

**Tasks:**
- [ ] Create `consult/` directory
- [ ] Implement subcommand parsing: `who`, `ask`, `propose`, `check`, `sessions`
- [ ] Implement shared flags: `--file`, `--dir`, `--question`, `--diff`, `--dry-run`, `--session`, `--update-skills`
- [ ] Add `--help` with usage examples for each subcommand
- [ ] Validate required args per subcommand (e.g., `ask` requires `--file` or `--dir` + `--question`)

### Phase 2: Git Analysis Engine (~20%)

**Files:**
- `consult/git.go` — git command execution, author scoring, expert ranking

**Tasks:**
- [ ] Implement `expert` struct: `Name, Email, SlackID, Commits90d, Commits1y, BlameLines, Score, LastCommit`
- [ ] Implement `analyzeExperts(repoRoot, targetPath string) ([]expert, error)`:
  - Run `git log --format="%ae" --since=1y -- <path>` for yearly commit counts
  - Run `git log --format="%ae" --since=90d -- <path>` for recent commit counts
  - For files: run `git blame --porcelain <file>` to count lines per author
  - Compute score: `(commits_90d * 3) + (commits_1y * 1) + (blame_lines * 0.5)`
  - Deduplicate by email, sort by score desc, return top 3
- [ ] Implement `getLastCommitDate(repoRoot, email, path string) string`
- [ ] Handle: non-git repos (error), files with no history (empty result), directories (aggregate across files)
- [ ] For directories: analyze up to 20 most recently modified files, merge author scores

### Phase 3: Slack Integration (~22%)

**Files:**
- `consult/slack.go` — Slack API client (HTTP, no SDK dependency)

**Tasks:**
- [ ] Implement `lookupSlackUser(token, email string) (string, string, error)` → (userID, displayName, error)
  - POST `https://slack.com/api/users.lookupByEmail?email=<email>`
  - Parse JSON response
  - Cache results in memory for the session
- [ ] Implement `openDM(token, userID string) (string, error)` → channelID
  - POST `https://slack.com/api/conversations.open` with user ID
- [ ] Implement `postMessage(token, channel, text string, blocks []block) (string, error)` → threadTS
  - POST `https://slack.com/api/chat.postMessage`
- [ ] Implement `getThreadReplies(token, channel, threadTS string) ([]reply, error)`
  - GET `https://slack.com/api/conversations.replies`
  - Filter out bot's own message
- [ ] Implement identity fallback: if `lookupSlackUser` fails, check `.consult.json` user_map
- [ ] Error handling: invalid token (clear message naming required scopes), user not found, rate limiting

### Phase 4: Message Builder (~12%)

**Files:**
- `consult/message.go` — message formatting and wiki integration

**Tasks:**
- [ ] Implement `buildAskMessage(question string, experts []expert, filePath, wikiExcerpt string) (text string, blocks []block)`
- [ ] Implement `buildProposeMessage(question, diff string, experts []expert, filePath, wikiExcerpt string) (text string, blocks []block)`
  - Truncate diff to 20 lines with "full diff omitted" note
- [ ] Implement `loadWikiExcerpt(wikiRoot, filePath string) string`
  - Find SUMMARY.md for the directory containing the file
  - Extract Overview section via string parsing
  - Return empty string if wiki not available (not an error)
- [ ] Format messages using Slack Block Kit (section, context, divider blocks)
- [ ] `--dry-run` prints the formatted message to stderr

### Phase 5: Session Management (~12%)

**Files:**
- `consult/session.go` — session file CRUD

**Tasks:**
- [ ] Implement `createSession(...)  session` — generate 8-char hex ID, write to `.consult/{id}.json`
- [ ] Implement `loadSession(repoRoot, sessionID string) (session, error)` — read and parse
- [ ] Implement `checkSession(token string, sess session) (string, bool, error)` — poll Slack thread, return (response, hasReply, error)
- [ ] Implement `updateSession(repoRoot string, sess session) error` — update status and response
- [ ] Implement `listSessions(repoRoot string) ([]session, error)` — glob `.consult/*.json`, parse, print table
- [ ] Create `.consult/` directory on first write

### Phase 6: Subcommand Wiring (~8%)

**Files:**
- `consult/main.go` — wire subcommands to functions

**Tasks:**
- [ ] Wire `who`: analyzeExperts → print table (name, email, commits 90d/1y, blame lines, score)
- [ ] Wire `ask`: analyzeExperts → lookupSlack → buildAskMessage → postMessage → createSession → print session ID
- [ ] Wire `propose`: same as ask but with buildProposeMessage + diff
- [ ] Wire `check`: loadSession → checkSession → print response or "no response yet"
- [ ] Wire `sessions`: listSessions → print table (ID, file, type, status, age, expert)
- [ ] Wire `--update-skills`: writeSkillFiles → exit

### Phase 7: Skill File (~10%)

**Files:**
- `consult/templates/CONSULT_SKILL.md` — embedded skill template

**Tasks:**
- [ ] Write skill content with three-layer decision framework:
  - **Layer 1 — Wiki sufficient**: Code is well-documented, change is straightforward → just use the wiki
  - **Layer 2 — Source sufficient**: You can understand the code by reading it → no need to consult
  - **Layer 3 — Consult a human**: Unfamiliar code + risky change, public API modification, complex warnings/comments, tribal knowledge needed
- [ ] Include signal checklist for when to consult:
  - Modifying code you haven't seen before in a high-churn area
  - Changing a public API that other packages depend on
  - Code has comments like "DON'T CHANGE", "HACK", "WORKAROUND"
  - The wiki summary says "internal" but you're about to import it externally
  - You're unsure if your approach is the right one
- [ ] Include signal checklist for when NOT to consult:
  - Adding tests, fixing typos, formatting changes
  - Well-documented code with clear patterns
  - Changes within a directory marked "internal" that nothing depends on
- [ ] Include example workflows: `consult who` → assess → `consult ask` → `consult check`
- [ ] Embed template with `//go:embed`
- [ ] Implement `writeSkillFiles()` following wikigen's pattern (marker comments for safe updates)

### Phase 8: Validation (~4%)

**Tasks:**
- [ ] `go vet ./consult/` passes
- [ ] `go build ./consult/` compiles
- [ ] `consult --help` shows all subcommands and flags
- [ ] `consult who --file wikigen.go` works in this repo
- [ ] `consult ask --dry-run --file wikigen.go --question "test"` shows message without sending
- [ ] `consult sessions` works with no sessions (empty output)
- [ ] Missing `SLACK_BOT_TOKEN` gives clear error on `ask` but not on `who` or `--dry-run`

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `consult/main.go` | Create | CLI entry point, subcommand dispatch, flag parsing, skill file writing |
| `consult/git.go` | Create | Git blame/log analysis, expert scoring and ranking |
| `consult/slack.go` | Create | Slack API client (lookupByEmail, postMessage, getThreadReplies) |
| `consult/message.go` | Create | Message formatting (ask/propose), wiki excerpt loading |
| `consult/session.go` | Create | Session file CRUD in .consult/ directory |
| `consult/templates/CONSULT_SKILL.md` | Create | Skill template teaching LLMs when and how to consult humans |

## Definition of Done

- [ ] `consult who --file <path>` identifies top 3 experts from git history
- [ ] `consult ask --file <path> --question "..."` sends a Slack DM to the top expert
- [ ] `consult propose --file <path> --diff "..." --question "..."` sends a diff review request
- [ ] `consult ask --dry-run` shows the message without sending (no SLACK_BOT_TOKEN needed)
- [ ] `consult check --session <id>` polls for and returns Slack replies
- [ ] `consult sessions` lists all consultation sessions with status
- [ ] Session files stored in `.consult/` under repo root
- [ ] Slack messages include file context, question, expert rationale, and wiki excerpt
- [ ] Expert ranking weights: recent commits (3x) > total commits (1x) > blame lines (0.5x)
- [ ] Identity resolution: Slack email lookup primary, `.consult.json` fallback
- [ ] Skill file teaches LLMs the three-layer decision framework
- [ ] `consult --update-skills` writes skill section to CLAUDE.md/AGENTS.md
- [ ] `go vet` and `go build` pass
- [ ] Missing env vars produce clear error messages with required scopes
- [ ] Non-git repos produce clear error messages
- [ ] Wiki integration is optional — tool works without wikigen output

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Git email doesn't match Slack email | High | High | `consult who` previews without Slack; `.consult.json` for manual mapping; clear error suggesting config |
| Expert doesn't respond | High | Medium | Async model means agent isn't blocked; can proceed after timeout |
| Slack rate limiting | Low | Low | Single message per consultation; no bulk operations |
| Bot token scope insufficient | Medium | Medium | Clear error naming required scopes (`chat:write`, `users:read.email`, `im:write`) |
| Message too long (large diffs) | Medium | Low | Truncate diffs to 20 lines |
| Agent over-consults | Medium | Medium | Skill file teaches judgment framework; `who` for preview without sending |
| Expert annoyed by bot messages | Medium | Medium | Messages explain why they were chosen; one message per consultation |
| Ranking identifies wrong expert | Medium | High | Expose scoring breakdown in `who` output; transparent formula |

## Security Considerations

- Slack bot token stored in environment variable, never logged or persisted
- Git commands are read-only (log, blame)
- Session files contain email addresses — stored locally under `.consult/`, gitignored
- Diffs in messages may contain sensitive code — same exposure as normal code review
- No credentials or secrets should be included in question or diff context
- `.consult.json` may contain Slack user IDs — not secret but should be treated as internal

## Dependencies

- Go stdlib (no external dependencies — Slack API via raw HTTP)
- Git (required)
- Slack workspace with bot installed (required for `ask`/`propose`/`check`, not for `who`/`--dry-run`)
- wikigen output (optional, for wiki excerpt enrichment)
- Sprint 005 (pattern for skill file templates, git analysis precedent)

## References

- Intent: `docs/sprints/drafts/SPRINT-006-INTENT.md`
- Claude draft: `docs/sprints/drafts/SPRINT-006-CLAUDE-DRAFT.md`
- Codex draft: `docs/sprints/drafts/SPRINT-006-CODEX-DRAFT.md`
- Codex critique: `docs/sprints/drafts/SPRINT-006-CLAUDE-DRAFT-CODEX-CRITIQUE.md`
- Merge notes: `docs/sprints/drafts/SPRINT-006-MERGE-NOTES.md`
