# Sprint 003: Hierarchical Codebase Wiki Generator

## Overview

Leash has grown to ~20+ packages across Go, Swift, TypeScript, C, and shell scripts, but has no auto-generated navigational documentation. New contributors (human or AI) must manually explore the directory tree to understand what each module does.

This sprint builds a Go CLI tool that traverses the codebase bottom-up, reads source files in each directory, generates a concise markdown summary of what the code does with links to individual files, and chains these summaries upward into a top-level `wiki.md`. The tool uses an LLM (Claude API) to produce meaningful descriptions rather than mechanical file listings. It follows the project's established pattern of single-file Go tooling (Sprint 001 precedent with `tracker.go`).

The result is a navigable, hierarchical wiki that can be regenerated at any time to stay current with the codebase.

## Use Cases

1. **Onboarding navigation**: New contributors get a top-level `wiki.md` that links into per-directory summaries, providing a guided tour of the codebase
2. **AI agent context**: AI coding agents can read `wiki.md` and relevant `SUMMARY.md` files to quickly understand project structure without reading every file
3. **Code review context**: When reviewing changes to a package, the `SUMMARY.md` provides context about the package's purpose and related files
4. **Architecture documentation**: The generated hierarchy serves as living architecture documentation that stays in sync with the code
5. **Re-generation on demand**: Running the tool produces a fresh snapshot — useful after major refactors or new module additions

## Architecture

```
                    ┌──────────────────┐
                    │   wikigen.go     │
                    │   (CLI entry)    │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  Directory Walker │
                    │  (bottom-up BFS)  │
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
     │    LLM Summarizer           │
     │  (Claude API via stdin/     │
     │   subprocess or HTTP)       │
     └────────────┬────────────────┘
                  │
     ┌────────────▼────────────────┐
     │    Markdown Generator       │
     │  (SUMMARY.md per dir,       │
     │   wiki.md at root)          │
     └─────────────────────────────┘
```

### Data Flow

1. **Walk phase**: Collect all directories (excluding `.claude`, `.codex`, `docs`, `.git`, hidden dirs), sort by depth (deepest first)
2. **Read phase**: For each directory (leaf-to-root order), read source files, detect language, collect content
3. **Summarize phase**: Send file contents + child `SUMMARY.md` contents to LLM, get back a structured summary
4. **Write phase**: Write `SUMMARY.md` to the directory, then move to parent
5. **Finalize phase**: Generate `wiki.md` at root from all top-level child summaries

### LLM Integration Options

**Option A: Claude API (HTTP)**
- Direct HTTP calls to the Anthropic API using Go's `net/http`
- Requires `ANTHROPIC_API_KEY` environment variable
- Pro: No external dependencies; fast; works in CI
- Con: Requires API key; costs money per run

**Option B: Subprocess to `claude` CLI**
- Shell out to the Claude Code CLI: `claude -p "summarize this code..."`
- Pro: Uses existing auth; no API key management
- Con: Slower; requires Claude Code installed

**Recommendation**: Option A (direct API) with Option B as a fallback flag (`--use-cli`). The tool should work standalone.

## Implementation Plan

### Phase 1: Core CLI & Directory Walking (~25%)

**Files:**
- `tools/wikigen.go` - Single-file Go CLI

**Tasks:**
- [ ] Create `package main` with CLI argument parsing (root dir, exclude patterns, output filename)
- [ ] Implement directory walker that collects all dirs sorted by depth (deepest first)
- [ ] Implement exclusion filter: skip `.claude`, `.codex`, `docs`, `.git`, and any dot-prefixed dirs
- [ ] Implement file grouping: for each directory, collect source files by extension
- [ ] Implement language detection by file extension mapping:
  - `.go` → Go
  - `.swift` → Swift
  - `.ts`, `.tsx` → TypeScript
  - `.js`, `.jsx` → JavaScript
  - `.c`, `.h` → C
  - `.py` → Python
  - `.sh` → Shell
  - `Makefile` → Make
  - `Dockerfile*` → Docker
  - `.yaml`, `.yml` → YAML
  - `.toml` → TOML
  - `.json` → JSON (skip `package-lock.json`, `pnpm-lock.yaml`)

### Phase 2: File Reading & Content Preparation (~15%)

**Files:**
- `tools/wikigen.go` - Same file

**Tasks:**
- [ ] Implement file reader with size limits (skip files > 100KB, truncate at 500 lines for LLM input)
- [ ] Skip binary files (detect by null byte in first 512 bytes)
- [ ] Skip generated files (detect by common headers: "Code generated", "DO NOT EDIT", auto-generated markers)
- [ ] Skip lock files (`pnpm-lock.yaml`, `package-lock.json`, `go.sum`)
- [ ] Build per-directory content bundle: `{dir_path, files: [{name, language, content}], child_summaries: []string}`

### Phase 3: LLM Summarization (~30%)

**Files:**
- `tools/wikigen.go` - Same file

**Tasks:**
- [ ] Implement Anthropic API client (HTTP POST to `https://api.anthropic.com/v1/messages`)
- [ ] Design the summarization prompt:
  ```
  You are documenting a codebase. Given the source files in a directory,
  write a concise markdown summary:
  1. A 1-2 sentence overview of what this directory/package does
  2. A bullet list of key files with one-line descriptions
  3. Key types, functions, or interfaces defined here
  4. How this package relates to its children (if child summaries provided)
  
  Keep it concise. Use relative links to files.
  ```
- [ ] Implement rate limiting (respect Anthropic API rate limits)
- [ ] Implement retry with exponential backoff for API errors
- [ ] Implement `--dry-run` flag that prints what would be summarized without calling the API
- [ ] Implement `--use-cli` flag to use `claude -p` subprocess instead of API
- [ ] Handle directories with only config files (shorter summary)
- [ ] Handle directories with child summaries but no source files (aggregation-only summary)

### Phase 4: Markdown Generation & Output (~20%)

**Files:**
- `tools/wikigen.go` - Same file

**Tasks:**
- [ ] Generate `SUMMARY.md` per directory with:
  - Header: `# {directory name}`
  - LLM-generated overview paragraph
  - File listing with relative links: `- [{filename}](./{filename}) — description`
  - Child directory links: `- [{childdir}/](./childdir/SUMMARY.md) — child overview`
  - Auto-generated notice at top: `<!-- Generated by wikigen. Do not edit manually. -->`
- [ ] Generate `wiki.md` at root with:
  - Project title from `README.md` or directory name
  - Top-level overview
  - Tree-style table of contents linking to all `SUMMARY.md` files
  - Per-module section with the top-level child summaries inlined
- [ ] Implement `--clean` flag to remove all generated `SUMMARY.md` files
- [ ] Print progress to stderr: `Summarizing internal/runner/ (14 files, 3 children)...`

### Phase 5: Testing & Verification (~10%)

**Tasks:**
- [ ] Run `go vet tools/wikigen.go`
- [ ] Test against leash codebase: `go run tools/wikigen.go .`
- [ ] Verify all links in generated markdown are valid (relative paths resolve)
- [ ] Verify excluded directories have no `SUMMARY.md`
- [ ] Verify `wiki.md` has entries for all expected top-level modules
- [ ] Manual review: are summaries accurate and useful?
- [ ] Test `--dry-run` flag
- [ ] Test `--clean` flag
- [ ] Test with `ANTHROPIC_API_KEY` unset (should error gracefully)

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `tools/wikigen.go` | Create | Single-file Go CLI for hierarchical wiki generation |
| `wiki.md` | Generated | Top-level codebase navigation document |
| `*/SUMMARY.md` | Generated | Per-directory summary documents (one per traversed dir) |
| `.gitignore` | Modify | Add `SUMMARY.md` and `wiki.md` if we don't want them tracked (TBD) |

## Definition of Done

- [ ] `go run tools/wikigen.go .` produces `wiki.md` and `SUMMARY.md` files throughout the codebase
- [ ] `go vet tools/wikigen.go` passes
- [ ] Excluded directories (`.claude`, `.codex`, `docs`, `.git`) have no generated files
- [ ] All markdown links are valid relative paths
- [ ] Summaries are accurate descriptions of what the code does (manual review)
- [ ] `wiki.md` provides a navigable overview of the entire project
- [ ] `--dry-run` works without API calls
- [ ] `--clean` removes all generated files
- [ ] Tool handles errors gracefully (missing API key, API failures, permission errors)
- [ ] Progress output to stderr shows what's happening
- [ ] No external Go dependencies (stdlib + net/http for API)

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM hallucinations in summaries | Medium | Medium | Include file names/content in prompt so LLM grounds descriptions in actual code; manual review step |
| API cost for large codebases | Low | Low | Leash has ~25 dirs; at ~$0.01/dir that's ~$0.25 per run |
| Rate limiting on API | Low | Medium | Built-in rate limiting and retry logic |
| Large files overwhelm context window | Medium | Medium | 500-line truncation; skip files >100KB |
| Generated markdown conflicts with existing files | Low | High | Use `<!-- Generated by wikigen -->` header; `--clean` for removal |
| Stale summaries after code changes | Medium | Low | Tool is designed to regenerate; could add git-hook integration later |

## Security Considerations

- API key read from environment variable, never hardcoded or logged
- File I/O scoped to the target directory tree
- No execution of discovered files — read-only analysis
- LLM prompt injection risk is minimal (source code is the input, not user-controlled text)
- Generated markdown is static content, no executable components
- `--dry-run` available for safe preview

## Dependencies

- Go standard library (no external modules)
- Anthropic API access (`ANTHROPIC_API_KEY` environment variable)
- Claude Code CLI (optional, for `--use-cli` mode)

## Open Questions

1. **Should generated files be git-tracked?** Pro: available without running the tool. Con: noise in diffs. Recommendation: track `wiki.md` but gitignore `SUMMARY.md` files.
2. **Model selection**: Which Claude model for summaries? Haiku for speed/cost, Sonnet for quality? Recommend Haiku with `--model` flag override.
3. **Incremental mode**: Worth implementing file-hash-based caching to skip unchanged directories? Defer to a follow-up sprint.
4. **Config file inclusion**: Should `Dockerfile`, `Makefile`, `.yaml` configs be summarized alongside source? Recommend yes — they're part of the module's purpose.
