# Sprint 003: Hierarchical Codebase Wiki Generator

## Overview

This sprint introduces a Go-based documentation generator that walks a repository bottom-up and emits a hierarchical wiki of the codebase. Each directory receives a `SUMMARY.md` that explains the purpose of its source files and child directories, and the repository root receives a `wiki.md` that acts as the navigable entry point. The goal is to make a mixed-language codebase like Leash substantially easier to understand for new contributors, reviewers, and AI agents.

The sprint should follow the project's existing tooling style: a small Go CLI with minimal dependencies, runnable with `go run`, and designed for deterministic output. Because the generated docs will be consumed as onboarding and navigation material, the implementation needs to favor clear structure, stable traversal, sensible exclusions, and summaries that say what each area does rather than merely listing file names.

## Use Cases

1. **New contributor onboarding**: A developer can open `wiki.md` and quickly understand how Leash is organized before diving into specific packages.
2. **Package-level navigation**: A contributor working in `internal/proxy` or `internal/runner` can read that directory's `SUMMARY.md` to understand responsibilities and nearby dependencies.
3. **Code review acceleration**: Reviewers can use generated summaries to orient themselves in unfamiliar parts of the tree without manually opening every file.
4. **AI agent context bootstrapping**: An agent can read top-level and directory-level summaries to build a grounded mental model of the repository before making changes.
5. **Documentation refresh**: Maintainers can re-run the generator after structural changes to keep the wiki aligned with the current codebase.

## Architecture

The generator should be implemented as a single Go CLI, likely under `docs/docs/sprints/` or a nearby tooling location consistent with `docs/sprints/tracker.go`. It accepts a target root path, recursively traverses the tree, excludes configured directories such as `.claude`, `.codex`, and `docs`, and processes directories in post-order so child summaries are always available before their parents are written.

At each directory, the tool should:

1. Enumerate eligible source and project files in a deterministic order.
2. Extract lightweight structural signals from each file based on extension and filename.
3. Incorporate any already-generated child `SUMMARY.md` content.
4. Render a markdown summary with relative links to files and child summaries.
5. Write `SUMMARY.md` for non-root directories and `wiki.md` at the root.

The initial architecture should prioritize deterministic heuristics over mandatory LLM integration. That keeps the tool runnable in CI or local environments without API credentials, preserves the single-binary workflow, and reduces variability in generated output. The design can still leave room for an optional future LLM enrichment mode, but Sprint 003 should deliver a useful baseline with static analysis and repository-aware formatting.

## Implementation Plan

### Phase 1: CLI and traversal engine (~25%)

**Files:**
- `docs/docs/sprints/wiki_generator.go` - New single-file Go CLI for wiki generation
- `Makefile` - Optional helper target for running the generator consistently

**Tasks:**
- [ ] Define CLI arguments for target root, output behavior, and exclusions
- [ ] Implement deterministic recursive traversal with post-order directory processing
- [ ] Exclude `.claude`, `.codex`, and `docs` from traversal
- [ ] Detect root vs non-root output targets (`wiki.md` vs `SUMMARY.md`)
- [ ] Ensure re-runs overwrite generated files cleanly

### Phase 2: File analysis and summarization heuristics (~35%)

**Files:**
- `docs/docs/sprints/wiki_generator.go` - File classification, metadata extraction, summary generation

**Tasks:**
- [ ] Classify supported source files by extension: `.go`, `.swift`, `.ts`, `.tsx`, `.c`, `.py`, `.sh`, `.js`
- [ ] Include key project files where useful, such as `Dockerfile`, `Makefile`, `.json`, `.yaml`, `.yml`, and `.toml`
- [ ] Extract lightweight code signals such as package names, exported types/functions, or obvious entrypoints where practical
- [ ] Produce concise prose describing likely file and directory responsibilities
- [ ] Define file size and content limits so large files do not dominate generation time

### Phase 3: Markdown rendering and link structure (~25%)

**Files:**
- `docs/docs/sprints/wiki_generator.go` - Markdown template/rendering logic
- `wiki.md` - Generated root output after running against the repository

**Tasks:**
- [ ] Render stable markdown sections for overview, files, and child directories
- [ ] Link each listed file using correct relative paths from the generated document location
- [ ] Reference child `SUMMARY.md` files from parent summaries
- [ ] Generate `wiki.md` at the repository root as the top-level entry point
- [ ] Ensure output remains readable in large directories with many files

### Phase 4: Repository rollout and validation (~15%)

**Files:**
- `docs/docs/sprints/wiki_generator.go` - Final polish and validation fixes
- `wiki.md` - Generated top-level wiki
- `internal/*/SUMMARY.md` - Generated per-directory summaries across eligible directories

**Tasks:**
- [ ] Run the generator against the Leash repository
- [ ] Review representative summaries for accuracy across Go, TypeScript, Swift, C, and script-heavy areas
- [ ] Verify excluded directories are omitted from generated output
- [ ] Confirm generated links resolve correctly from nested directories
- [ ] Document how maintainers should rerun the generator

## API Endpoints (if applicable)

Not applicable. This sprint delivers a local CLI tool and generated markdown artifacts rather than networked APIs.

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `docs/docs/sprints/wiki_generator.go` | Create | Single-file Go CLI for bottom-up wiki generation |
| `Makefile` | Modify | Add optional convenience target for running the generator |
| `wiki.md` | Create | Root-level generated wiki for the repository |
| `cmd/SUMMARY.md` | Create | Generated summary for the `cmd` tree |
| `internal/SUMMARY.md` | Create | Generated summary for the `internal` tree |
| `controlui/SUMMARY.md` | Create | Generated summary for the Control UI subtree |
| `mac-leash/SUMMARY.md` | Create | Generated summary for the macOS-specific subtree |
| `npm/SUMMARY.md` | Create | Generated summary for npm packaging assets |

## Definition of Done

- [ ] A single-file Go CLI can be run with `go run` against an arbitrary repository root
- [ ] Traversal is bottom-up so child summaries are generated before parent summaries
- [ ] `.claude`, `.codex`, and `docs` are excluded by default
- [ ] Root output is written to `wiki.md`
- [ ] Eligible subdirectories receive `SUMMARY.md` files
- [ ] Generated markdown includes working relative links to files and child summaries
- [ ] Output is useful narrative documentation, not only raw file listings
- [ ] Running the tool on Leash produces a coherent repository wiki
- [ ] The tool handles the repository's mixed-language structure without crashing
- [ ] Tests or validation checks pass for traversal and rendering behavior
- [ ] No compiler warnings or Go build errors are introduced

## Security Considerations

- Generated documentation must avoid reading excluded directories that may contain sensitive agent state or credentials.
- The tool should not execute project code, shell scripts, or build steps while summarizing files.
- Path handling should prevent writing outside the selected repository root.
- Large or unusual files should be handled defensively to avoid excessive memory use or malformed output.

## Dependencies

- Sprint 001 precedent: project tooling should prefer a small Go implementation
- Repository root structure and naming conventions described in `README.md`
- No external API dependency should be required for the initial sprint scope

## References

- `README.md`
- `docs/docs/sprints/README.md`
- `docs/docs/sprints/drafts/SPRINT-003-INTENT.md`
