# Sprint 006: Expert Consultation Tool (`wikiask`)

## Overview

`wikigen` already helps an LLM understand a repository statically: it can read summaries, indexes, traces, and churn overlays before making changes. The next gap is judgment under uncertainty. When an agent is about to modify unfamiliar code, especially code with local conventions or hidden operational context, the best source of truth is often the human who recently worked in that area.

This sprint adds `wikiask`, a second Go CLI in its own subdirectory, focused on expert consultation rather than documentation generation. The tool analyzes a target file or directory with `git blame` and `git log`, resolves likely experts to Slack identities, and sends a structured message that either asks a question or proposes a change for review. It also supports reading the reply back into an agent-friendly format so an LLM can continue work with human guidance in the loop.

The design should stay intentionally narrow for v1: Slack only, CLI only, repo-local configuration, and explicit human-invocation semantics. The goal is not to build a general workflow engine; it is to create a reliable “ask the right person” primitive that complements the wiki and can later grow into a broader suite of LLM-support tools.

## Use Cases

1. **Pre-change review request**: An agent identifies likely experts for `internal/auth/` and sends a concise proposed-change summary before editing.
2. **Targeted implementation question**: An agent asks who understands a failing subsystem and sends a question with file and wiki context.
3. **Ownership discovery**: A user runs `wikiask who-knows path/to/file.go` to see recent contributors ranked by relevance before deciding whether to consult anyone.
4. **Human-in-the-loop unblock**: An agent polls or fetches a Slack thread reply and incorporates the answer into its next step.
5. **Safe dry run**: A user previews the ranked experts, resolved Slack users, and composed message without contacting anyone.

## Architecture

```
                  ┌──────────────────────────┐
                  │   wikiask/main.go        │
                  │   CLI entry + commands   │
                  └─────────────┬────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
 ┌────────▼────────┐  ┌─────────▼─────────┐  ┌────────▼────────┐
 │ Target Resolver │  │ Git Expert Finder │  │ Context Builder │
 │ file/dir normalization │ blame + log ranking │ wiki + user input │
 └────────┬────────┘  └─────────┬─────────┘  └────────┬────────┘
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │ Identity Resolution   │
                    │ git author -> Slack   │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │ Slack Transport       │
                    │ post message/thread   │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │ Response Retrieval    │
                    │ read thread replies   │
                    └───────────────────────┘
```

### Data Flow

1. **Resolve target**: Normalize the repo path, detect whether it is a file or directory, and expand a directory into a bounded file set for analysis.
2. **Rank experts**: Use recent `git log` history plus `git blame` on representative files to score likely experts by recency, frequency, and concentration of ownership.
3. **Resolve identities**: Map git authors to Slack users via a repo-local config file, with optional email matching against Slack profile data.
4. **Build message context**: Collect the user’s question or proposal, relevant file paths, and optional wiki excerpts from the generated documentation.
5. **Send or preview**: In `--dry-run`, print the ranked experts and rendered message. Otherwise, post to Slack as a DM or channel-thread message.
6. **Persist request metadata**: Write a small local state file containing thread/channel identifiers so a later command can retrieve replies.
7. **Read replies**: Fetch thread replies, normalize them into markdown or JSON, and present them for agent consumption.

### Command Surface

The tool should expose a small, explicit command set:

```bash
wikiask who-knows [--top 5] <path>
wikiask ask --question "..." <path>
wikiask propose --summary "..." [--diff file.patch] <path>
wikiask replies <request-id>
```

- `who-knows` is analysis only; no Slack side effects.
- `ask` sends a question-oriented consultation request.
- `propose` sends a change-oriented request for validation or concerns.
- `replies` fetches and renders human responses tied to a previous request.

### Configuration Model

V1 should prefer deterministic repo-local configuration over hidden magic:

- `wikiask/config.example.yaml` documents supported keys.
- `.wikiask.yaml` at repo root stores Slack defaults and author mappings.
- `SLACK_BOT_TOKEN` comes from the environment.
- Optional config keys:
  - default destination channel
  - author-email to Slack-user mapping
  - path-specific fallback experts
  - minimum confidence threshold for auto-contact

### Slack Interaction Model

For v1, use a conservative thread-based pattern:

- Send one initial message containing target, intent, ranked attribution context, and the actual question/proposal.
- Post into a configured channel or DM the top resolved expert(s).
- Store the returned `channel` + `thread_ts` locally as a request record.
- `wikiask replies` reads the thread and emits a compact transcript.

This avoids requiring the CLI to block waiting on human replies and keeps the integration resilient to long response times.

## Implementation Plan

### Phase 1: Scaffold `wikiask` CLI and config (~18%)

**Files:**
- `wikiask/main.go` - CLI entrypoint and subcommand dispatch
- `wikiask/config.go` - Config types and loading logic
- `wikiask/config.example.yaml` - Example repo-local configuration
- `README.md` - Document multi-tool layout and `wikiask` usage

**Tasks:**
- [ ] Create `wikiask/` as a separate Go tool directory
- [ ] Implement subcommand parsing for `who-knows`, `ask`, `propose`, and `replies`
- [ ] Load `.wikiask.yaml` from repo root with clear validation errors
- [ ] Read `SLACK_BOT_TOKEN` from the environment only when a Slack command is executed
- [ ] Add `--dry-run`, `--json`, and `--repo` flags with consistent behavior across commands
- [ ] Update root `README.md` to explain the new tool and repo structure direction

### Phase 2: Git-based expert ranking engine (~24%)

**Files:**
- `wikiask/git.go` - Git command execution and parsing helpers
- `wikiask/experts.go` - Scoring and ranking model
- `wikiask/experts_test.go` - Ranking logic tests with fixture inputs

**Tasks:**
- [ ] Implement target expansion: file path passes through; directory path expands to a capped set of source files
- [ ] Parse `git log --format=...` over the target to compute recency/frequency contribution scores
- [ ] Parse `git blame --line-porcelain` for representative files to compute line ownership concentration
- [ ] Merge blame and log signals into a transparent weighted ranking formula
- [ ] Return structured expert candidates with name, email, score, evidence, and touched paths
- [ ] Ensure the ranking engine works without Slack configuration so `who-knows` is independently useful

### Phase 3: Slack identity resolution and transport (~22%)

**Files:**
- `wikiask/slack.go` - Slack API client wrapper
- `wikiask/identity.go` - Author-to-Slack resolution logic
- `wikiask/types.go` - Shared request/result structs

**Tasks:**
- [ ] Implement config-based exact mapping from git author email to Slack user ID
- [ ] Add optional Slack profile lookup by email as a fallback when mappings are absent
- [ ] Handle unresolved experts gracefully with a confidence/status field instead of hard failure
- [ ] Implement message posting to either a configured channel or direct-message target
- [ ] Capture `channel`, `ts`, recipients, and render preview text in a typed result
- [ ] Keep transport logic behind a small interface so tests can mock Slack calls

### Phase 4: Consultation message composition and request persistence (~18%)

**Files:**
- `wikiask/message.go` - Message templates and formatting
- `wikiask/state.go` - Request persistence under local state directory
- `wikiask/context.go` - Optional wiki summary lookup and truncation

**Tasks:**
- [ ] Define a standard message template for both “ask” and “propose” modes
- [ ] Include target paths, ranked-expert rationale, and the actual question/proposal text
- [ ] Optionally attach concise wiki context by reading nearby generated summaries when available
- [ ] Persist each sent request locally with a generated request ID and Slack thread metadata
- [ ] Implement `--dry-run` rendering that prints the exact structured payload without sending
- [ ] Avoid oversized Slack posts by truncating context and clearly marking omitted sections

### Phase 5: Reply retrieval and agent-facing outputs (~10%)

**Files:**
- `wikiask/replies.go` - Thread fetch and transcript normalization
- `wikiask/main.go` - `replies` command wiring
- `wikiask/replies_test.go` - Reply rendering tests

**Tasks:**
- [ ] Fetch thread replies using saved request metadata
- [ ] Render replies as readable markdown by default and machine-friendly JSON with `--json`
- [ ] Preserve author names and timestamps in the output
- [ ] Ignore the bot’s original prompt when producing the condensed answer view
- [ ] Return a non-zero exit code only for true transport/config failures, not “no reply yet”

### Phase 6: LLM skill guidance and project docs (~8%)

**Files:**
- `wikiask/AGENTS.md` - Tool-specific agent guidance
- `wikiask/CLAUDE.md` - Tool-specific guidance for Claude-based agents
- `README.md` - Usage examples and positioning relative to `wikigen`

**Tasks:**
- [ ] Write guidance that teaches when to consult `wikigen` versus when to use `wikiask`
- [ ] Define a decision rule: ask humans for ambiguity, local policy, risky changes, or missing context
- [ ] Include example commands for `who-knows`, `ask`, `propose`, and `replies`
- [ ] Explain required configuration and safe `--dry-run` usage for agents

## API Endpoints

Slack API usage is external rather than repo-hosted, but these methods shape the integration:

| Service | Method | Purpose |
|---------|--------|---------|
| Slack Web API | `users.lookupByEmail` | Resolve git author email to Slack user |
| Slack Web API | `chat.postMessage` | Send consultation request |
| Slack Web API | `conversations.replies` | Retrieve thread responses |
| Git CLI | `git log` | Gather recent contribution history |
| Git CLI | `git blame --line-porcelain` | Estimate file-level expertise |

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `wikiask/main.go` | Create | New CLI entrypoint and command dispatch |
| `wikiask/config.go` | Create | Load and validate repo-local config |
| `wikiask/git.go` | Create | Encapsulate git command execution |
| `wikiask/experts.go` | Create | Score and rank likely experts |
| `wikiask/identity.go` | Create | Resolve git authors to Slack identities |
| `wikiask/slack.go` | Create | Slack API wrapper for posting and reading |
| `wikiask/message.go` | Create | Render structured consultation messages |
| `wikiask/context.go` | Create | Pull nearby wiki summaries into prompts |
| `wikiask/state.go` | Create | Persist request records for later reply fetch |
| `wikiask/replies.go` | Create | Normalize Slack thread replies |
| `wikiask/types.go` | Create | Shared domain models |
| `wikiask/experts_test.go` | Create | Ranking logic tests |
| `wikiask/replies_test.go` | Create | Reply formatting tests |
| `wikiask/config.example.yaml` | Create | Example configuration template |
| `wikiask/AGENTS.md` | Create | Agent usage guidance |
| `wikiask/CLAUDE.md` | Create | Claude-facing tool guidance |
| `README.md` | Modify | Describe the new tool and usage |

## Definition of Done

- [ ] `wikiask who-knows <path>` ranks experts from git history without Slack access
- [ ] `wikiask ask ...` and `wikiask propose ...` can render a preview with `--dry-run`
- [ ] Slack-backed commands resolve at least one recipient via config mapping or email lookup
- [ ] Sent requests persist a request ID and enough metadata for later reply retrieval
- [ ] `wikiask replies <request-id>` returns a readable transcript or a clear “no replies yet” result
- [ ] Tool-specific `AGENTS.md` and `CLAUDE.md` explain when and how to use `wikiask`
- [ ] `README.md` reflects the repo’s multi-tool direction
- [ ] Tests cover expert ranking, config loading, and reply formatting
- [ ] `go test ./...` passes
- [ ] No compiler warnings or unchecked critical errors

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Git authors do not map cleanly to Slack users | High | High | Require explicit config mappings first, then use email lookup as fallback |
| Expert ranking feels arbitrary or wrong | Medium | High | Expose evidence and scoring components in output and dry-run mode |
| Slack API setup is brittle for first-time users | Medium | Medium | Keep required scopes narrow, document config clearly, and fail with actionable errors |
| Large directories make blame analysis slow | Medium | Medium | Cap analyzed files, sample representative files, and show when limits were applied |
| Agents overuse human consultation for trivial tasks | Medium | Medium | Encode judgment guidance in skill files and preserve `who-knows` as a non-contacting step |
| Reply retrieval state becomes inconsistent | Low | Medium | Store simple local JSON records with versioned schema and explicit request IDs |

## Security Considerations

- Keep Slack tokens in environment variables only; never persist them to config or request state.
- Treat Slack user IDs, emails, and consultation transcripts as sensitive operational metadata.
- Make `--dry-run` the safest default path for agents experimenting with the tool.
- Sanitize message composition to avoid accidentally posting full secrets, large diffs, or unrelated file contents.
- Prefer least-privilege Slack scopes and document them explicitly.

## Dependencies

- Sprint 003 for the original `wikigen` foundation and project patterns
- Sprint 004 for multi-output and manifest-style CLI evolution
- Sprint 005 for skill-file generation concepts and git-analysis precedent
- Slack Web API access with a bot token and appropriate scopes
- Git installed and available in the analyzed repository

## Open Questions

1. Should v1 post into a shared team channel, direct messages, or support both from the start?
2. Should unresolved author mappings block sending, or should the tool fall back to a configured channel with attribution text only?
3. How much wiki context is enough before Slack messages become noisy?
4. Should the request state live under the analyzed repo (for transparency) or under a user cache directory (for privacy)?
5. Do we want request templates optimized for human readability only, or also for machine parsing of replies?

## References

- `docs/sprints/drafts/SPRINT-006-INTENT.md`
- `docs/sprints/README.md`
- `README.md`
- `wikigen.go`
