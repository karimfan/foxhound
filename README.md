# foxhound

A suite of tools that help LLM agents write better code. Each tool addresses a different gap in how LLMs understand and modify codebases.

| Tool | What it does | Directory |
|------|-------------|-----------|
| **wikigen** | Generates hierarchical wiki documentation for any codebase | `wikigen/` |
| **consult** | Identifies code experts via git history and lets agents ask them questions via Slack | `consult/` |

## Requirements

- Go 1.21+
- Git

## Quick start

```bash
# Build both tools
go build -o wikigen ./wikigen/
go build -o consult ./consult/

# Generate a wiki for a repo
ANTHROPIC_API_KEY=... ./wikigen --output /path/to/wiki /path/to/repo

# Find who knows about a file
./consult who --file path/to/file.go

# Ask an expert via Slack (requires SLACK_BOT_TOKEN)
./consult ask --file path/to/file.go --question "Is this safe to change?"
```

---

## wikigen

Generates hierarchical markdown and HTML documentation for any codebase, optimized for LLM navigation and comprehension. Traverses a repository bottom-up, sends source files to the Claude API for summarization, and produces a navigable wiki with per-directory summaries, cross-repository indexes, and git-derived analysis.

### How it works

1. Walks the directory tree in post-order (deepest directories first)
2. At each directory, reads source files and detects their language
3. Sends file contents + child directory summaries to Claude Haiku for summarization
4. Writes a `SUMMARY.md` and `SUMMARY.html` for each directory
5. Generates root-level artifacts: dependency graph, file index, symbol index, churn overlay
6. Analyzes git co-change history and generates modification recipes (1 LLM call)
7. Generates entry point traces showing end-to-end execution flows (1 LLM call)
8. Generates a root-level project overview via a separate LLM call
9. Generates `wiki.md` and `index.html` linking everything together
10. Writes `CLAUDE.md` and `AGENTS.md` skill files into the repo root
11. Saves a `.wikigen-manifest.json` for incremental runs

### Requirements

- `ANTHROPIC_API_KEY` environment variable
- Git (optional, for co-change recipes, churn overlay, and fast incremental detection)

### Usage

```bash
# Full scan
go run ./wikigen/ --output /path/to/wiki /path/to/repo

# Preview (no API calls)
go run ./wikigen/ --dry-run --output /path/to/wiki /path/to/repo

# Incremental (automatic when manifest exists)
go run ./wikigen/ --output /path/to/wiki /path/to/repo

# Force full regeneration
go run ./wikigen/ --full --output /path/to/wiki /path/to/repo

# Remove all generated files
go run ./wikigen/ --clean --output /path/to/wiki /path/to/repo

# Skip expensive features (saves 2 LLM calls)
go run ./wikigen/ --no-recipes --no-traces --output /path/to/wiki /path/to/repo

# Update skill files only (no scan, no API key needed)
go run ./wikigen/ --update-skills /path/to/repo
```

### Flags

| Flag | Description |
|------|-------------|
| `--output <dir>` | Wiki output directory (default: in-place in repo) |
| `--exclude <name>` | Exclude a directory by base name (repeatable) |
| `--no-default-excludes` | Clear default exclusions (only `.git` is still skipped) |
| `--base-url <url>` | URL prefix for source file links |
| `--full` | Force full regeneration, ignore manifest |
| `--json` | Emit line-delimited JSON progress events on stderr |
| `--dry-run` | Show what would be summarized without calling the API |
| `--clean` | Remove all generated files and manifest |
| `--update-skills` | Update CLAUDE.md and AGENTS.md without scanning |
| `--no-deps-graph` | Skip dependency graph generation (`deps-graph.md`) |
| `--no-file-index` | Skip file index generation (`file-index.md`) |
| `--no-recipes` | Skip modification recipes (`recipes.md`, saves 1 LLM call) |
| `--no-traces` | Skip entry point traces (`traces.md`, saves 1 LLM call) |
| `--no-boundaries` | Omit `## Boundary` section from directory summaries |
| `--no-test-hints` | Omit `## Testing` section from directory summaries |
| `--no-symbol-index` | Skip symbol index generation (`symbol-index.md`) |
| `--no-churn` | Skip churn overlay generation (`churn.md`) |

### Generated artifacts

All on by default. Each can be disabled with its `--no-*` flag.

| Artifact | Description |
|----------|-------------|
| `wiki.md` / `index.html` | Root overview with architecture, navigation guide, and links to all artifacts |
| `{dir}/SUMMARY.md` / `.html` | Per-directory summary with key files, types, dependencies, boundary, testing |
| `deps-graph.md` | Directory dependency adjacency list with boundary annotations |
| `file-index.md` | Flat file lookup table (name, directory, language, description) |
| `symbol-index.md` | Key types, functions, interfaces and their locations |
| `recipes.md` | Git co-change derived modification patterns |
| `traces.md` | Entry point execution traces |
| `churn.md` | Recent activity by directory (90 days) |

### LLM cost

For a repo with 50 directories on a full scan: ~52 LLM calls (50 dirs + 1 root + 1 recipes). Incremental runs only regenerate changed directories.

### Incremental mode

wikigen saves `.wikigen-manifest.json` and uses `git diff` (or hash comparison) to detect changes. Only dirty directories and their ancestors are re-summarized.

### Supported languages

Go, Swift, TypeScript, JavaScript, C, Python, Shell, Make, Docker, YAML, TOML, JSON.

---

## consult

Identifies code experts via git blame/log analysis and lets LLM agents ask them questions via Slack. The tool finds who has been most active in the code being modified and sends them a structured message with context.

### Requirements

- `SLACK_BOT_TOKEN` environment variable (for `ask`/`propose`/`check` commands)
  - Required scopes: `chat:write`, `users:read.email`, `im:write`
- Git
- No Slack token needed for `who`, `sessions`, or `--dry-run`

### Usage

```bash
# Find who knows about a file (no Slack needed)
go run ./consult/ who --file internal/auth/handler.go

# Ask a question via Slack
go run ./consult/ ask --file internal/auth/handler.go \
    --question "Is there a rate limit middleware I should use?"

# Propose a change with a diff
go run ./consult/ propose --file internal/auth/handler.go \
    --diff "$(git diff HEAD)" \
    --question "Does this look safe?"

# Preview without sending (no token needed)
go run ./consult/ ask --dry-run --file internal/auth/handler.go \
    --question "test question"

# Check for a response
go run ./consult/ check --session abc123

# List all consultation sessions
go run ./consult/ sessions

# Update CLAUDE.md/AGENTS.md with consult skill
go run ./consult/ --update-skills
```

### Expert ranking

Experts are scored by: `(commits_90d * 3) + (commits_1y * 1) + (blame_lines * 0.5)`. Weights favor recent activity over total history over current line ownership. Top 3 experts are returned.

### Identity resolution

1. **Primary**: Slack `users.lookupByEmail` — zero config when git email matches Slack email
2. **Fallback**: Optional `.consult.json` at repo root for manual overrides:

```json
{
  "user_map": {
    "alice-old@personal.com": "U12345"
  },
  "default_channel": "C_TEAM_CHANNEL"
}
```

### Session management

Consultations are async. Sessions are stored in `.consult/` under the repo root. Poll with `consult check --session <id>`.

---

## Project structure

```
foxhound/
  wikigen/
    wikigen.go         # Wiki generator (single-file Go tool)
    templates/         # Embedded skill file templates
  consult/
    main.go            # CLI entry point and subcommand dispatch
    git.go             # Git blame/log analysis, expert ranking
    slack.go           # Slack API client, identity resolution
    message.go         # Message formatting, wiki excerpt loading
    session.go         # Async session management
    templates/         # Embedded skill file template
  docs/
    sprints/           # Sprint planning documents
  go.mod               # Shared Go module
  README.md            # This file
```

## CI/CD integration

Both tools are designed for CI. wikigen supports `--json` for machine-readable progress:

```bash
go run ./wikigen/ --json --output ./wiki /path/to/repo 2>progress.jsonl
```

Exit codes: `0` = success, `1` = error, `2` = partial failure.
