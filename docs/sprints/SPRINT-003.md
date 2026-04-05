# Sprint 003: Hierarchical Codebase Wiki Generator

## Overview

Leash has grown to ~20+ packages spanning Go, Swift, TypeScript, C, and shell scripts, but has no auto-generated navigational documentation. New contributors and AI agents must manually explore the directory tree to understand what each module does.

This sprint builds a Go CLI tool that performs a post-order traversal of the codebase, reads source files in each directory, sends them to the Claude API for summarization, and generates a hierarchical set of markdown documents. Each directory gets a `SUMMARY.md` explaining what its code does with links to individual files and child directories. The root gets a `wiki.md` that serves as the navigable entry point. All generated files are gitignored.

The tool follows the project's established pattern of single-file Go tooling (Sprint 001 precedent with `tracker.go`) and lives at `docs/wikigen.go`.

## Use Cases

1. **New contributor onboarding**: Open `wiki.md` to understand how Leash is organized before diving into specific packages
2. **AI agent context bootstrapping**: An agent reads `wiki.md` and relevant `SUMMARY.md` files to build a grounded mental model of the repository
3. **Package-level navigation**: A contributor working in `internal/proxy` reads that directory's `SUMMARY.md` to understand responsibilities and nearby dependencies
4. **Code review acceleration**: Reviewers orient themselves in unfamiliar parts of the tree without manually opening every file
5. **Documentation refresh**: Re-run the generator after structural changes to keep the wiki aligned with the current codebase

## Architecture

```
                    ┌──────────────────┐
                    │  docs/wikigen.go │
                    │   (CLI entry)    │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  Post-Order      │
                    │  Directory Walker │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───┐  ┌──────▼─────┐  ┌────▼────────┐
     │ File Reader │  │  Language  │  │   Exclude   │
     │ & Grouper   │  │  Detector  │  │   Filter    │
     └────────┬───┘  └──────┬─────┘  └─────────────┘
              │              │
     ┌────────▼──────────────▼─────┐
     │    Claude API Summarizer    │
     │  (Haiku via HTTP POST)      │
     └────────────┬────────────────┘
                  │
     ┌────────────▼────────────────┐
     │    Markdown Generator       │
     │  (SUMMARY.md per dir,       │
     │   wiki.md at root)          │
     └─────────────────────────────┘
```

### Data Flow

1. **Walk phase**: Collect all directories (excluding `.claude`, `.codex`, `docs`, `.git`, dot-prefixed dirs). Sort by depth — deepest first (post-order).
2. **Read phase**: For each directory, read source and config files, detect language, collect content. Skip binary files, generated files, lock files, and files > 100KB. Truncate at 500 lines for LLM input.
3. **Summarize phase**: Send file contents + any child `SUMMARY.md` contents to Claude API (Haiku). Receive structured markdown summary.
4. **Write phase**: Write `SUMMARY.md` to the directory, then process the next directory up.
5. **Finalize phase**: Generate `wiki.md` at root from all top-level child summaries.

### LLM Integration

Direct HTTP calls to the Anthropic Messages API (`https://api.anthropic.com/v1/messages`) using Go's `net/http`. Requires `ANTHROPIC_API_KEY` environment variable. Uses Claude Haiku for speed and cost (~$0.25 per full codebase run).

## Implementation Plan

### Phase 1: CLI & Post-Order Directory Walker (~25%)

**Files:**
- `docs/wikigen.go` - Single-file Go CLI

**Tasks:**
- [ ] Create `package main` with CLI argument parsing: positional root dir, `--dry-run`, `--clean` flags
- [ ] Implement post-order directory traversal: collect all dirs, sort by depth (deepest first)
- [ ] Implement exclusion filter: skip `.claude`, `.codex`, `docs`, `.git`, and dot-prefixed dirs
- [ ] Implement file grouping: for each directory, collect eligible files sorted by name
- [ ] Implement language detection by file extension:
  - `.go` → Go, `.swift` → Swift, `.ts`/`.tsx` → TypeScript, `.js`/`.jsx` → JavaScript
  - `.c`/`.h` → C, `.py` → Python, `.sh` → Shell
  - `Makefile` → Make, `Dockerfile*` → Docker
  - `.yaml`/`.yml` → YAML, `.toml` → TOML, `.json` → JSON
- [ ] Skip lock files (`pnpm-lock.yaml`, `package-lock.json`, `go.sum`)

### Phase 2: File Reading & Content Preparation (~15%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Implement file reader with size limits: skip files > 100KB, truncate at 500 lines for LLM input
- [ ] Skip binary files (detect by null byte in first 512 bytes)
- [ ] Skip generated files (detect common headers: "Code generated", "DO NOT EDIT")
- [ ] Build per-directory content bundle: `{dir_path, files: [{name, language, content}], child_summaries: []string}`

### Phase 3: Claude API Summarization (~30%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Implement Anthropic Messages API client (HTTP POST, `anthropic-version: 2023-06-01`)
- [ ] Read `ANTHROPIC_API_KEY` from environment; error with clear message if unset
- [ ] Use `claude-haiku-4-5-20251001` model
- [ ] Design summarization prompt:
  ```
  You are documenting a codebase. Given the source files in a directory,
  write a concise markdown summary:
  1. A 1-2 sentence overview of what this directory/package does
  2. A bullet list of key files with one-line descriptions
  3. Key types, functions, or interfaces defined here
  4. How this package relates to its children (if child summaries provided)
  Keep it concise. Use relative links to files.
  ```
- [ ] Handle directories with only config files (shorter summary)
- [ ] Handle directories with child summaries but no source files (aggregation-only summary)
- [ ] Implement `--dry-run` flag: print what would be summarized without calling API
- [ ] Print progress to stderr: `Summarizing internal/runner/ (14 files, 3 children)...`

### Phase 4: Markdown Generation & Output (~20%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Generate `SUMMARY.md` per directory with:
  - Auto-generated notice: `<!-- Generated by wikigen. Do not edit manually. -->`
  - Header: `# {directory name}`
  - LLM-generated overview and file descriptions
  - Child directory links: `- [{childdir}/](./{childdir}/SUMMARY.md) — child overview`
- [ ] Generate `wiki.md` at root with:
  - Project title
  - Top-level overview
  - Tree-style table of contents linking to all `SUMMARY.md` files
  - Per-module sections with top-level child summaries inlined
  - Auto-generated notice header
- [ ] Implement `--clean` flag: remove all generated `SUMMARY.md` and `wiki.md` files

### Phase 5: Validation & Repository Rollout (~10%)

**Tasks:**
- [ ] Run `go vet docs/wikigen.go`
- [ ] Run the generator against the Leash codebase: `ANTHROPIC_API_KEY=... go run docs/wikigen.go .`
- [ ] Verify excluded directories have no `SUMMARY.md`
- [ ] Verify all markdown links are valid relative paths
- [ ] Verify `wiki.md` has entries for all expected top-level modules (`cmd`, `internal`, `build`, `controlui`, `e2e`, `mac-leash`, `npm`)
- [ ] Review representative summaries for accuracy across Go, TypeScript, Swift, C, and script-heavy areas
- [ ] Test `--dry-run` flag (no API calls, no files written)
- [ ] Test `--clean` flag (removes all generated files)
- [ ] Test with `ANTHROPIC_API_KEY` unset (should error gracefully with clear message)

## API Endpoints (if applicable)

Not applicable. This sprint delivers a local CLI tool and generated markdown artifacts.

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `docs/wikigen.go` | Create | Single-file Go CLI for hierarchical wiki generation |
| `.gitignore` | Modify | Add `wiki.md` and `**/SUMMARY.md` patterns |
| `wiki.md` | Generated | Root-level codebase navigation document |
| `*/SUMMARY.md` | Generated | Per-directory summary documents |

## Definition of Done

- [ ] `go run docs/wikigen.go .` produces `wiki.md` and `SUMMARY.md` files throughout the codebase
- [ ] `go vet docs/wikigen.go` passes
- [ ] Post-order traversal: child summaries generated before parent summaries
- [ ] Excluded directories (`.claude`, `.codex`, `docs`, `.git`) have no generated files
- [ ] All markdown links are valid relative paths
- [ ] Summaries describe what the code does, not just list files (manual review)
- [ ] `wiki.md` provides a navigable overview of the entire project
- [ ] `--dry-run` works without API calls or file writes
- [ ] `--clean` removes all generated `SUMMARY.md` and `wiki.md` files
- [ ] Tool errors gracefully when `ANTHROPIC_API_KEY` is unset
- [ ] Progress output to stderr shows directory being processed
- [ ] Generated files include `<!-- Generated by wikigen -->` header
- [ ] `.gitignore` updated to exclude generated files
- [ ] No compiler warnings or Go build errors
- [ ] No external Go dependencies (stdlib only, `net/http` for API)

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM hallucinations in summaries | Medium | Medium | Prompt includes actual file content for grounding; manual review step |
| API cost for large codebases | Low | Low | Leash has ~25 dirs; ~$0.25/run with Haiku |
| Large files overwhelm context window | Medium | Medium | 500-line truncation; skip files >100KB |
| Generated markdown conflicts with existing files | Low | High | `<!-- Generated by wikigen -->` header; `--clean` for removal |
| Stale summaries after code changes | Medium | Low | Tool designed to regenerate on demand; gitignored so no stale tracking |
| API unavailable or key expired | Low | Medium | Clear error message; `--dry-run` for testing traversal logic without API |

## Security Considerations

- API key read from environment variable, never hardcoded or logged
- File I/O scoped to the target directory tree; no writes outside root
- No execution of discovered files — read-only analysis
- Binary files and lock files skipped defensively
- Generated markdown is static content with no executable components

## Dependencies

- Go standard library (no external modules)
- Anthropic API access (`ANTHROPIC_API_KEY` environment variable)
- Sprint 001 (completed) — established Go tooling pattern

## References

- `README.md` — project overview and structure
- `docs/docs/sprints/README.md` — sprint conventions and template
- `docs/docs/sprints/drafts/SPRINT-003-INTENT.md` — sprint intent document
- `docs/docs/sprints/SPRINT-001.md` — Go tooling precedent
