# Sprint 004: Wikigen External Output & Incremental Mode

## Overview

Sprint 004 evolves `docs/wikigen.go` from a repo-intrusive, full-rescan summarizer into a more practical documentation generator that can write output to an external directory and avoid unnecessary recomputation. The tool built in Sprint 003 already performs post-order traversal, summarizes directories via the Anthropic API, skips binary and generated files, and emits `SUMMARY.md` plus a root `wiki.md`. This sprint keeps that foundation and extends it for real repository workflows.

The two core changes are:

1. **Externalized output** so generated documentation can live outside the scanned repository tree, avoiding source pollution and simplifying adoption in CI, local previews, and multi-run comparisons.
2. **Incremental mode** so unchanged directories can reuse prior generated summaries, while changed directories and all of their ancestors are regenerated deterministically.

This sprint also formalizes exclusion handling and adds machine-readable progress so the tool is easier to integrate into automation. The implementation must remain a single-file Go program using only the standard library, and the existing `go run docs/wikigen.go <repo>` workflow must continue to behave as before unless new flags are used.

## Use Cases

- A developer wants to generate wiki output into `./tmp/wikigen` or a docs artifact directory without writing `SUMMARY.md` files into the source tree.
- CI wants to regenerate documentation quickly after a small change, updating only the affected subtree plus ancestor summaries.
- A repository outside Leash wants to use the tool with its own excluded directories instead of relying on Leash-specific hardcoded rules.
- A scripted workflow wants stable, machine-readable progress and deterministic exit behavior for logging or dashboards.
- A maintainer wants to compare outputs between two runs by pointing the tool at different output directories without modifying the repo contents.

## Architecture

Sprint 003’s core processing model remains intact: discover directories, process in post-order, build a per-directory bundle from direct files plus child summaries, then generate markdown output. Sprint 004 refines the runtime around that model.

### Output Layout

The tool will separate **scan root** from **output root**.

- **Scan root** is the repository or directory tree being analyzed.
- **Output root** is where generated `SUMMARY.md`, `wiki.md`, and incremental metadata are written.
- By default, `output_root == scan_root` to preserve backwards compatibility.
- When an explicit output directory is provided, the tool mirrors the scanned directory structure beneath that output root.

Example:

- Scan root: `/workspace/leash`
- Output root: `/tmp/leash-wiki`
- Generated summary for `/workspace/leash/internal/runner` becomes `/tmp/leash-wiki/internal/runner/SUMMARY.md`
- Root wiki becomes `/tmp/leash-wiki/wiki.md`

This preserves relative linking semantics while removing the requirement to write into the repository itself.

### Incremental Regeneration Model

Incremental mode relies on lightweight metadata persisted in the output tree. For each summarized directory, the tool records enough information to determine whether its direct inputs changed:

- directory relative path
- direct child file identities and content fingerprints
- direct child directory names
- child summary fingerprints or metadata references
- summarization-relevant configuration such as exclusions and output mode
- tool format version

A directory is considered dirty when any of the following occur:

- an eligible direct file is added, removed, renamed, or changes content
- an eligible child directory is added or removed
- a child summary changed
- summarization configuration or metadata schema changed
- the expected generated output or metadata is missing

Processing still happens in post-order. That gives a simple rule:

- leaf directories determine dirtiness from their own eligible files
- parent directories determine dirtiness from their own eligible files plus whether any child summary changed
- if a directory regenerates, all ancestors regenerate because their child-summary inputs changed

This preserves correctness without needing a global dependency graph beyond the natural directory tree.

### Metadata Strategy

To keep the tool single-file and stdlib-only, metadata should be stored as JSON in the output tree. A simple approach is a per-directory sidecar file stored adjacent to each generated summary, plus one root-level file for the root wiki.

Candidate structure:

- `internal/runner/SUMMARY.md`
- `internal/runner/.wikigen.json`
- `wiki.md`
- `.wikigen-root.json`

Each metadata file records:

- schema version
- source directory relative path
- source file fingerprints
- child directory list
- child summary fingerprints
- generated summary fingerprint
- generation timestamp or run identifier if needed for diagnostics

The metadata must not be required for non-incremental runs. If absent, the tool simply falls back to regeneration.

### Configurable Exclusions

Sprint 003 hardcodes exclusions such as `.claude`, `.codex`, `docs`, and `.git`. Sprint 004 replaces that with configurable exclusion rules while keeping the current behavior as the default profile.

The minimum design should support repeatable CLI flags such as:

- `--exclude docs`
- `--exclude .git`
- `--exclude node_modules`

And optionally a comma-separated variant for convenience. Matching should remain simple and deterministic:

- exact base-name directory exclusions
- dot-prefixed directories still skipped by default unless intentionally changed

This is sufficient for the current problem without introducing glob engines or external config formats.

### Progress & Automation

Current output is human-readable progress on stderr. Sprint 004 should preserve that while adding a structured option suitable for CI.

Recommended behavior:

- default stderr progress remains concise and readable
- optional machine-readable mode emits line-delimited JSON events to stderr
- exit codes clearly distinguish success from configuration or runtime failure

Example event types:

- directory scanned
- directory skipped as unchanged
- directory regenerated
- summary write complete
- root wiki write complete

## Implementation Plan

### Phase 1: CLI Refactor for Source/Output Separation (~20%)

Goal: extend the existing single-file CLI without breaking the current invocation shape.

- [ ] Add `--output <dir>` flag; default to the scan root when omitted
- [ ] Preserve positional root-directory argument and current `--dry-run`, `--clean`, `--help` behavior
- [ ] Refactor path handling so scan paths and write paths are computed separately
- [ ] Implement mirrored output path generation for per-directory `SUMMARY.md` files under the output root
- [ ] Update root `wiki.md` generation to write under the output root instead of always the scan root
- [ ] Ensure relative markdown links are still correct when output root differs from scan root
- [ ] Validate that `--output` can point outside the scanned repository tree

### Phase 2: Metadata & Incremental Dirty Checking (~30%)

Goal: determine whether a directory can reuse existing generated output safely.

- [ ] Define per-directory metadata schema in JSON with explicit schema versioning
- [ ] Compute deterministic fingerprints for eligible direct file inputs
- [ ] Record child directory names and child summary fingerprints in metadata
- [ ] Load prior metadata from the output tree when present
- [ ] Mark a directory dirty when direct files, child directories, child summaries, config inputs, or schema version differ
- [ ] Fall back to regeneration when metadata or generated markdown is missing or unreadable
- [ ] Keep non-incremental execution path simple: if incremental mode is disabled, ignore cached metadata and regenerate everything

### Phase 3: Incremental Execution & Upward Propagation (~20%)

Goal: ensure changes in leaf directories force correct regeneration through ancestors.

- [ ] Add `--incremental` flag to enable metadata reuse and dirty checking
- [ ] Preserve post-order traversal so child regeneration status is known before evaluating each parent
- [ ] Reuse existing `SUMMARY.md` content when a directory is unchanged in incremental mode
- [ ] Regenerate a directory whenever any child summary fingerprint changed
- [ ] Ensure root `wiki.md` regenerates whenever any included top-level summary changes
- [ ] Report whether each directory was regenerated or reused

### Phase 4: Exclusions, Cleaning, and Output Robustness (~15%)

Goal: make the tool more portable and predictable across repositories.

- [ ] Replace hardcoded named exclusions with configurable repeatable `--exclude` flags
- [ ] Keep current default exclusions for backwards compatibility: `.claude`, `.codex`, `docs`, `.git`
- [ ] Continue skipping dot-prefixed directories by default during traversal
- [ ] Update `--clean` so it removes generated files from the configured output root, including metadata sidecars
- [ ] Ensure clean mode never deletes source-repo files outside generated outputs
- [ ] Handle output directory creation automatically for nested summaries

### Phase 5: Structured Progress, Validation, and Rollout (~15%)

Goal: make the new behavior safe to operate locally and in CI.

- [ ] Add optional machine-readable progress mode (for example `--progress=json`)
- [ ] Keep human-readable stderr progress as the default mode
- [ ] Emit deterministic statuses for regenerated vs reused directories
- [ ] Run `go vet docs/wikigen.go`
- [ ] Validate legacy behavior: `go run docs/wikigen.go .` still writes into the repo root/output-in-place
- [ ] Validate external output mode: `go run docs/wikigen.go --output /tmp/wikigen .`
- [ ] Validate incremental mode by changing a leaf file and confirming regeneration of the leaf, its ancestors, and root wiki only
- [ ] Validate exclusion overrides against a test repository shape or representative local tree
- [ ] Validate `--clean` for both in-place output and external output

## API Endpoints (if applicable)

Not applicable. `docs/wikigen.go` remains a local CLI and does not expose a network API.

## Files Summary

- `docs/wikigen.go` — refactor CLI flags, output path handling, exclusion config, metadata persistence, incremental dirty checking, and structured progress
- `docs/docs/sprints/drafts/SPRINT-004-CODEX-DRAFT.md` — Sprint 004 draft based on the intent, current implementation, and sprint template
- Generated output artifacts (design target, not committed source): external or in-place `SUMMARY.md`, `wiki.md`, and metadata sidecars such as `.wikigen.json`

## Definition of Done

- [ ] `go run docs/wikigen.go .` remains backwards-compatible and writes output in place
- [ ] `go run docs/wikigen.go --output /tmp/wikigen .` writes all generated files under `/tmp/wikigen`
- [ ] `--incremental` skips unchanged directories and reuses prior generated summaries
- [ ] Changing an eligible file in a leaf directory regenerates that directory, each ancestor directory, and the root `wiki.md`
- [ ] Unchanged sibling subtrees are not regenerated in incremental mode
- [ ] Missing or corrupt metadata causes safe regeneration rather than incorrect reuse
- [ ] `--clean` removes generated markdown and metadata from the configured output root only
- [ ] Exclusions are configurable via CLI while preserving the current default excluded directories
- [ ] Relative links inside generated markdown remain valid when using an external output root
- [ ] `go vet docs/wikigen.go` passes
- [ ] Tool output clearly indicates whether directories were regenerated or reused
- [ ] Machine-readable progress mode emits deterministic, parseable events
- [ ] No external Go dependencies are introduced
- [ ] No compiler warnings or Go build errors

## Risks & Mitigations

- **Incorrect cache hits**: If fingerprints miss an input, summaries may go stale. Mitigation: include direct file content fingerprints, child summary fingerprints, exclusion config, and schema version in dirty checks.
- **Broken relative links in external output mode**: Moving output outside the repo can invalidate assumptions. Mitigation: derive links from mirrored output paths only and validate representative paths.
- **Overly aggressive cleaning**: External output support increases the risk of deleting the wrong tree. Mitigation: scope all clean operations strictly to generated filenames beneath the configured output root.
- **Incremental complexity in a single file**: Additional state handling can make the tool harder to maintain. Mitigation: keep the data model small, use clear helper structs/functions, and preserve the existing post-order flow.
- **Configurable exclusions weakening defaults**: Users may accidentally include heavy or irrelevant directories. Mitigation: preserve current defaults unless explicitly overridden and print effective exclusions in verbose/debug output if needed.

## Security Considerations

- API keys continue to be read from environment variables only and must never be logged
- Metadata files must contain fingerprints and generation state only, not raw source file contents
- File writes must remain confined to the configured output root
- Incremental mode must never execute repository files; it performs read-only analysis plus markdown/metadata writes
- Clean mode must remove only known generated files and metadata sidecars
- Structured progress output must avoid leaking sensitive source contents

## Dependencies

- Go standard library only
- Anthropic API access via `ANTHROPIC_API_KEY`
- Sprint 003 (completed) — established current `wikigen` traversal, summarization, and output flow

## References

- `README.md`
- `docs/docs/sprints/README.md`
- `docs/docs/sprints/SPRINT-003.md`
- `docs/docs/sprints/drafts/SPRINT-004-INTENT.md`
- `docs/wikigen.go`
