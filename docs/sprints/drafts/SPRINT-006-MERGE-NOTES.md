# Sprint 006 Merge Notes

## Claude Draft Strengths
- Strong problem framing and use cases
- Clean async polling model (session files + check command)
- Good Slack message format with Block Kit
- Expert ranking algorithm well-specified with weighted scoring
- Skill file design with decision framework
- Dry-run mode spec

## Codex Draft Strengths
- Multi-file module layout (git.go, slack.go, identity.go, etc.) — better for a durable tool
- `propose` as a distinct subcommand from `ask` — clearer intent separation
- Config-based identity mapping as primary, Slack lookup as fallback — more realistic for real orgs
- State directory instead of CWD for session files
- Tests specified for ranking logic and reply formatting
- `--json` output mode for agent consumption

## Valid Critiques Accepted

1. **Tool name**: The user chose "consult" in the interview. Keeping it. Codex's preference for "wikiask" was based on the intent doc which was written before the interview.

2. **Multi-file layout**: Accepted. A second durable tool should not repeat the wikigen monolith pattern. Split into focused files: main.go, git.go, slack.go, session.go, message.go.

3. **Identity resolution needs config fallback**: Accepted. The user chose "Slack email lookup" in the interview, but Codex is right that real orgs often have email mismatches. Use Slack email lookup as primary (per user's choice), but support an optional `.consult.json` config for overrides.

4. **Session files shouldn't litter CWD**: Accepted. Store in `.consult/` directory under the repo root.

5. **`propose` as distinct subcommand**: Accepted. Different from `ask` — propose includes a diff and frames the message differently.

## Critiques Rejected

1. **Rename to wikiask**: Rejected. User explicitly chose "consult" in the interview.

2. **Config-based mapping as primary**: Partially rejected. User chose Slack email lookup as primary. But we'll add config override support.

3. **config.yaml**: Rejected in favor of simpler `.consult.json`. The project uses JSON elsewhere (manifest) and doesn't import a YAML library.

4. **Tests in Phase 2**: Partially rejected for MVP scope. The single-file Go convention in this project means no _test.go files exist yet. We'll add them if scope allows but won't block the sprint on test infrastructure.

## Interview Refinements Applied
- Tool name: `consult`
- Identity: Slack email lookup primary, config override secondary
- Response mode: Async with polling
- Channels: Slack only for MVP

## Final Decisions

1. **Tool name**: `consult`
2. **Directory**: `consult/`
3. **File layout**: Multi-file Go module (main.go, git.go, slack.go, session.go, message.go)
4. **Identity**: Slack `users.lookupByEmail` primary, optional `.consult.json` for overrides
5. **Session storage**: `.consult/` directory under repo root
6. **Subcommands**: `who`, `ask`, `propose`, `check`, `sessions`
7. **Skill file**: Embedded template with `--update-skills` flag, same pattern as wikigen
