# Sprint 006: Expert Consultation Tool (`consult`)

## Overview

LLM agents working on unfamiliar code today have two options: read documentation (if it exists) or read source code and hope for the best. wikigen addresses the first gap by generating navigable documentation. But documentation can't capture everything — tribal knowledge, active refactoring intent, subtle constraints, and "don't touch this because..." warnings live in people's heads, not in code or docs.

This sprint builds `consult`, a CLI tool that lets an LLM agent ask human experts for help via Slack, the same way a junior developer would tap a senior on the shoulder. The tool uses `git log` and `git blame` to identify who has been most active in the code the agent is about to modify, looks up those people on Slack via their git email, and sends a structured message with context: what code is being changed, what the agent is planning to do, and what it's unsure about. The expert replies in a Slack thread, and the agent polls for the response asynchronously.

This is the second tool in a growing suite of LLM code-assistance tools. wikigen provides static understanding; `consult` provides dynamic, human-in-the-loop understanding. Together they cover the spectrum from "what does this code do" to "should I change this code this way."

## Use Cases

1. **Unfamiliar codepath modification**: Agent is about to modify a file it hasn't seen before. It runs `consult` to ask the most recent committer whether the proposed change is safe.
2. **High-churn area caution**: Agent sees from `churn.md` that an area is actively changing. Before adding more changes, it consults the active developers to avoid conflicts.
3. **Boundary-crossing change**: Agent is about to modify a "public" package that other packages depend on. It consults the maintainers of dependent packages about the contract change.
4. **Missing context**: The wiki summary says "This module handles X" but the agent needs to understand *why* a particular approach was chosen. It asks the original author.
5. **Pre-review sanity check**: Before creating a PR, the agent sends a proposed diff to the most relevant expert for a quick gut check.

## Architecture

```
                    ┌─────────────────────────┐
                    │   consult CLI           │
                    │   (single Go binary)    │
                    └────────┬────────────────┘
                             │
              ┌──────────────┼──────────────────┐
              │              │                  │
     ┌────────▼───────┐ ┌───▼──────────┐ ┌────▼──────────┐
     │ Git Analyzer    │ │ Slack Client │ │ Session       │
     │                 │ │              │ │ Manager       │
     │ - git blame     │ │ - lookupUser │ │ - write JSON  │
     │ - git log       │ │ - postMsg    │ │ - poll thread │
     │ - rank authors  │ │ - getThread  │ │ - read reply  │
     └────────┬───────┘ └───┬──────────┘ └────┬──────────┘
              │             │                  │
              │     ┌───────▼──────────┐       │
              └────►│ Message Builder  │◄──────┘
                    │                  │
                    │ - format context │
                    │ - include diff   │
                    │ - include wiki   │
                    └──────────────────┘
```

### Data Flow

**Sending a consultation:**
```
1. Agent runs: consult ask --file path/to/file.go --question "Is it safe to ..."
2. Git analysis: git blame + git log → rank authors by recency and frequency
3. Slack lookup: git email → Slack user ID via users.lookupByEmail API
4. Message build: format question + file context + optional diff + wiki excerpt
5. Slack send: post message to top expert's DM (or channel if configured)
6. Session save: write .consult-session-{id}.json with thread info
7. Output: print session ID and who was contacted
```

**Checking for a response:**
```
1. Agent runs: consult check --session {id}
2. Load session file
3. Slack API: fetch thread replies
4. If reply exists: print it, mark session complete
5. If no reply: print "no response yet", exit 0
```

### CLI Interface

```bash
# Ask a question about a file
consult ask --file internal/auth/handler.go \
            --question "I'm adding rate limiting here. Is there an existing middleware I should use instead?"

# Ask about a directory
consult ask --dir internal/auth/ \
            --question "I need to add OAuth2 support. What should I be careful about?"

# Propose a change (include a diff)
consult ask --file internal/auth/handler.go \
            --diff "$(git diff HEAD)" \
            --question "Does this change look safe? I'm worried about the session handling."

# Check for a response
consult check --session abc123

# Dry run — show who would be contacted and what the message looks like
consult ask --file internal/auth/handler.go \
            --question "..." --dry-run

# List recent sessions
consult sessions

# Identify experts for a file (without sending a message)
consult who --file internal/auth/handler.go
```

### Session File Format

```json
{
  "id": "abc123",
  "created_at": "2026-04-06T10:30:00Z",
  "file": "internal/auth/handler.go",
  "question": "Is it safe to add rate limiting here?",
  "experts": [
    {"name": "Alice", "email": "alice@co.com", "slack_id": "U123", "commits": 47, "last_commit": "2026-04-01"}
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
*From:* LLM Agent (via consult)

*Question:*
I'm adding rate limiting to the login handler. Is there an existing middleware I should use instead of implementing it inline?

*Context:*
You've made 47 commits to this file in the last year, most recently on Apr 1.

*File summary (from wiki):*
> HTTP request handlers for authentication. Handles login, logout, session validation. Imported by api/routes.go.

━━━
_Reply in this thread. The agent will check for your response._
```

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SLACK_BOT_TOKEN` | Yes | Slack bot token with `chat:write`, `users:read.email`, `im:write` scopes |
| `SLACK_DEFAULT_CHANNEL` | No | Default channel to post to instead of DMs |

### Expert Ranking Algorithm

```
For a given file or directory:

1. git log --format="%ae %H" --since=1y -- <path>
   → List all authors who've committed to this path in the last year

2. git blame --porcelain <file>
   → For files: who wrote each current line

3. Score = (commits_last_90d * 3) + (commits_last_year * 1) + (blame_lines * 0.5)
   → Weighted toward recent activity (recency > volume > current ownership)

4. Sort by score descending, take top 3

5. For each: git log --format="%ae" → Slack lookup via users.lookupByEmail
```

## Implementation Plan

### Phase 1: Project Structure & CLI Skeleton (~15%)

**Files:**
- `consult/consult.go` — main CLI entry point and argument parsing

**Tasks:**
- [ ] Create `consult/` directory
- [ ] Implement CLI argument parsing: `ask`, `check`, `who`, `sessions` subcommands
- [ ] Implement `--file`, `--dir`, `--question`, `--diff`, `--dry-run`, `--session` flags
- [ ] Add `--help` with usage examples
- [ ] Implement `printUsage()` for each subcommand

### Phase 2: Git Analysis Engine (~20%)

**Files:**
- `consult/consult.go` — git analysis functions

**Tasks:**
- [ ] Implement `analyzeAuthors(repoRoot, path string) ([]expert, error)`:
  - Run `git log --format="%ae %H" --since=1y -- <path>` to get commit authors
  - Run `git log --format="%ae %H" --since=90d -- <path>` for recent weighting
  - For files: run `git blame --porcelain <file>` to get current line ownership
  - Score authors: `(commits_90d * 3) + (commits_1y * 1) + (blame_lines * 0.5)`
  - Return top 3 sorted by score
- [ ] Implement `expert` struct: `Name, Email, SlackID, Commits, LastCommit, Score`
- [ ] Handle non-git repos gracefully (error message, not panic)
- [ ] Handle files with no git history (new files)

### Phase 3: Slack Integration (~25%)

**Files:**
- `consult/consult.go` — Slack API client functions

**Tasks:**
- [ ] Implement `lookupSlackUser(token, email string) (slackUser, error)`:
  - HTTP POST to `https://slack.com/api/users.lookupByEmail`
  - Parse response JSON for user ID, display name
  - Cache lookups in memory for the session
- [ ] Implement `sendMessage(token string, msg slackMessage) (channelID, threadTS string, error)`:
  - HTTP POST to `https://slack.com/api/chat.postMessage`
  - Support both DM (open conversation first via `conversations.open`) and channel
  - Use Block Kit for formatted message
- [ ] Implement `getThreadReplies(token, channelID, threadTS string) ([]reply, error)`:
  - HTTP GET `https://slack.com/api/conversations.replies`
  - Filter out the bot's own message
  - Return human replies
- [ ] Error handling: invalid token, user not found, rate limiting

### Phase 4: Message Builder (~10%)

**Files:**
- `consult/consult.go` — message formatting

**Tasks:**
- [ ] Implement `buildMessage(question string, experts []expert, fileContext, wikiSummary, diff string) slackMessage`:
  - Format as Slack Block Kit blocks (section, context, divider)
  - Include: file path, question, expert context (why they were chosen), wiki summary excerpt, optional diff (truncated to 20 lines)
- [ ] Implement `loadWikiSummary(wikiRoot, filePath string) string`:
  - Find the SUMMARY.md for the directory containing the file
  - Extract the Overview section
  - Return as a one-paragraph excerpt
- [ ] Implement `--dry-run` output: print the formatted message to stderr instead of sending

### Phase 5: Session Management (~15%)

**Files:**
- `consult/consult.go` — session file handling

**Tasks:**
- [ ] Implement `createSession(question, file string, experts []expert, channelID, threadTS string) session`:
  - Generate short random ID (8 hex chars)
  - Write `.consult-session-{id}.json` to current directory
- [ ] Implement `loadSession(sessionID string) (session, error)`:
  - Find and read the session file
  - Parse JSON
- [ ] Implement `checkSession(token string, sess session) (string, error)`:
  - Call getThreadReplies
  - If reply exists: update session file with response, mark complete
  - Return response text or "no response yet"
- [ ] Implement `listSessions()`:
  - Glob `.consult-session-*.json` in current directory
  - Print table: ID, file, status, age, expert contacted

### Phase 6: Subcommand Wiring & the `who` Command (~5%)

**Files:**
- `consult/consult.go` — subcommand dispatch

**Tasks:**
- [ ] Wire `ask` subcommand: analyzeAuthors → lookupSlack → buildMessage → sendMessage → createSession
- [ ] Wire `check` subcommand: loadSession → checkSession → print
- [ ] Wire `sessions` subcommand: listSessions
- [ ] Implement `who` subcommand: analyzeAuthors → print expert ranking (no Slack needed)
- [ ] `who` output: table with name, email, commits (90d), commits (1y), blame lines, score

### Phase 7: Skill File (~10%)

**Files:**
- `consult/templates/CONSULT_SKILL.md` — skill template for LLMs

**Tasks:**
- [ ] Write skill content teaching LLMs when to use `consult`:
  - Decision framework: wiki sufficient → just use wiki. Wiki + source sufficient → don't consult. Unfamiliar code + risky change → consult.
  - Signals that should trigger consultation: modifying public API, high-churn area, code with complex comments/warnings, code you don't fully understand
  - Signals that should NOT trigger consultation: straightforward changes, well-documented code, test-only changes, formatting
  - How to formulate good questions (specific, include context, propose a plan)
  - How to use `consult who` to preview before asking
  - How to poll with `consult check`
- [ ] Embed template in binary (go:embed)
- [ ] Add `consult --update-skills` flag to write the skill section to CLAUDE.md/AGENTS.md in the repo

### Phase 8: Validation (~5%)

**Tasks:**
- [ ] `go vet consult/consult.go` passes
- [ ] `go build consult/consult.go` compiles
- [ ] `consult --help` shows all subcommands
- [ ] `consult who --file <file>` works in this repo
- [ ] `consult ask --dry-run --file <file> --question "test"` shows formatted message without sending
- [ ] `consult sessions` works with no sessions (empty output)
- [ ] Missing `SLACK_BOT_TOKEN` gives clear error message on `ask` (but not on `who` or `--dry-run`)

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `consult/consult.go` | Create | Full CLI tool: git analysis, Slack integration, session management |
| `consult/templates/CONSULT_SKILL.md` | Create | Skill template teaching LLMs when and how to consult humans |

## Definition of Done

- [ ] `consult who --file <path>` identifies top 3 experts from git history
- [ ] `consult ask --file <path> --question "..."` sends a Slack DM to the top expert
- [ ] `consult ask --dry-run` shows the message without sending
- [ ] `consult check --session <id>` polls for and returns Slack replies
- [ ] `consult sessions` lists all consultation sessions
- [ ] Session files track state across invocations
- [ ] Slack messages include file context, question, expert rationale, and wiki excerpt
- [ ] Expert ranking weights recent commits > total commits > blame ownership
- [ ] Skill file teaches LLMs the judgment framework for when to consult
- [ ] `consult --update-skills` writes skill section to CLAUDE.md/AGENTS.md
- [ ] `go vet` and `go build` pass
- [ ] Missing env vars produce clear error messages
- [ ] Non-git repos produce clear error messages
- [ ] `--dry-run` works without `SLACK_BOT_TOKEN`

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Git email doesn't match Slack email | High | High | `consult who` lets agent preview before asking; clear error when lookup fails; future: config file fallback |
| Expert doesn't respond | High | Medium | Async polling model means agent isn't blocked; can timeout and proceed |
| Slack rate limiting | Medium | Low | Single message per consultation; no bulk operations |
| Bot token scope insufficient | Medium | Medium | Clear error messages naming required scopes |
| Message too long (large diffs) | Medium | Low | Truncate diffs to 20 lines with "full diff available at ..." |
| Agent over-consults (asks too often) | Medium | Medium | Skill file teaches judgment framework; `who` command for preview without sending |
| Expert annoyed by bot messages | Medium | Medium | Messages include clear context on why they were chosen; one message per consultation, not spam |

## Security Considerations

- Slack bot token stored in environment variable, never logged or included in session files
- Git commands are read-only (log, blame)
- Session files may contain email addresses — stored locally, not transmitted beyond Slack
- Diffs included in messages may contain sensitive code — same exposure as a normal code review
- No credentials or secrets should be included in the question or diff context

## Dependencies

- Go stdlib + Slack Web API (HTTP, no SDK dependency)
- Git (required, for blame/log analysis)
- Slack workspace with a bot installed (required for `ask`/`check`, not for `who`/`--dry-run`)
- wikigen output (optional, for wiki summary excerpts in messages)

## Open Questions

1. Should `consult ask` support contacting multiple experts at once (group DM or individual DMs)?
2. Should session files be stored in a `.consult/` directory instead of littering the working directory?
3. Should there be a `consult respond` subcommand for testing without Slack (simulating a response)?
