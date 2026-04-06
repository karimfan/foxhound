# Sprint 006 Intent: Expert Consultation Tool (wikiask)

## Seed

Build a tool that helps LLM agents working on unfamiliar code consult human experts via Slack. The agent uses git blame/log to identify humans who have recently modified the code it's about to change, then sends them a structured message on Slack — either proposing changes for review or asking a question. The humans most active in a codepath are likely the most knowledgeable, mirroring how junior developers ask senior developers for help.

## Context

- The project is expanding from "wikigen" (a single wiki generator) to a suite of tools that help LLMs write better code
- wikigen is tool #1 (stays in its own subfolder); this expert consultation tool is tool #2 (new subfolder)
- The wiki already provides static documentation; this tool adds a runtime interaction layer — real-time human-in-the-loop via Slack
- Each tool will have its own skill file (CLAUDE.md/AGENTS.md section or standalone) that teaches LLMs when and how to use it

## Recent Sprint Context

- **Sprint 003**: Built the core wiki generator — post-order traversal, Claude Haiku summarization
- **Sprint 004**: Added --output, incremental mode with manifest, HTML output with breadcrumbs
- **Sprint 005**: Added 9 enhanced features (deps graph, file index, symbol index, recipes, traces, boundaries, test hints, churn, HTML child nav) plus --update-skills flag

## Relevant Codebase Areas

- `wikigen.go` — existing tool (will eventually move to its own subfolder)
- `templates/CLAUDE.md`, `templates/AGENTS.md` — existing skill file templates (pattern to follow)
- Git analysis functions in wikigen.go (analyzeCoChanges, analyzeChurn) — prior art for git log/blame parsing
- The project uses single-file Go tooling convention (tracker.go precedent)

## Constraints

- Go tool, minimal external dependencies (Slack API client may be needed)
- Must work as a CLI that an LLM agent can invoke (not a daemon or server)
- Slack integration requires a bot token (environment variable)
- Git must be available in the repo being analyzed
- Should produce a skill file that teaches LLMs when to consult humans vs. just reading the wiki
- New tool lives in its own subfolder (e.g., `wikiask/`)

## Success Criteria

- An LLM agent can run the tool to identify who knows about a specific file or directory
- The tool can send a structured Slack message to identified experts
- The message includes context: what code is being modified, proposed changes or questions, relevant wiki summary
- Experts can respond in Slack and the agent can read the response
- A skill file teaches LLMs the decision framework: when to consult the wiki vs. when to ask a human

## Open Questions

1. How should the tool identify the right Slack user from a git author? (email matching? configurable mapping?)
2. Should the tool wait synchronously for a Slack response, or write a response file that the agent polls?
3. What's the minimum viable message format? (just the question + file context, or include diff previews?)
4. Should the tool also support GitHub PRs/issues as a channel, or just Slack for now?
5. How does the skill file teach the LLM the judgment call: "this is straightforward enough to just do" vs. "I should ask someone first"?
6. Should the tool have a --dry-run mode that shows who would be contacted and what the message would look like?
