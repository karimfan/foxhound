# Sprint 004 Intent: Wikigen External Output & Incremental Mode

## Seed

Expand wikigen to live outside a repo. Invoked as `wikigen <repo> <wiki>` where `<repo>` is the target codebase and `<wiki>` is the destination folder for all generated markdown. This generalizes it to run against any repo. Add incremental mode for CI/CD integration: first run scans the entire tree, subsequent runs detect changed files/folders and only regenerate affected markdown files, propagating changes up the tree.

## Context

- Sprint 003 built `docs/wikigen.go` — a 400-line single-file Go CLI using Claude Haiku API
- Current design writes SUMMARY.md files inside the source repo and wiki.md at the repo root
- Single positional arg (root dir), no change detection, full scan every time
- Hardcoded exclusions matched by directory base name (`.claude`, `.codex`, `docs`, `.git`)
- The tool already handles post-order traversal, language detection, binary/generated file skipping, `--dry-run`, `--clean`

## Recent Sprint Context

- **Sprint 001**: Rewrote sprint tracker from Python to Go (established single-file Go tooling pattern)
- **Sprint 002**: GTM strategy analysis (non-code)
- **Sprint 003**: Built wikigen.go — hierarchical codebase wiki generator with Claude API summarization

## Relevant Codebase Areas

- `docs/wikigen.go` — the tool being expanded (400 lines, Go stdlib only)
- `.gitignore` — currently excludes `wiki.md` and `**/SUMMARY.md` (will change since output is now external)

## Constraints

- Must remain a single-file Go tool with no external dependencies (stdlib only)
- Must be backwards-compatible: running with just `<repo>` should still work (output defaults to `<repo>`)
- Incremental mode must correctly propagate changes upward (changing a leaf file must regenerate its dir summary AND all ancestor summaries up to root)
- Must work in CI/CD: deterministic behavior, exit codes, machine-readable progress
- Exclusion list should be configurable (not hardcoded to leash-specific dirs)

## Success Criteria

1. `wikigen <repo> <wiki>` generates all markdown in `<wiki>` mirroring the repo directory structure
2. A manifest file tracks file hashes so subsequent runs only regenerate changed directories
3. Incremental mode correctly identifies changed files and regenerates only affected summaries + ancestors
4. The tool works in a CI/CD pipeline (e.g., GitHub Actions) triggered by push events
5. Exclusion patterns are configurable via flags rather than hardcoded

## Open Questions

1. **Manifest format**: JSON file with per-file hashes? Per-directory content hashes? Where to store it — in the wiki output folder?
2. **Change detection granularity**: Per-file hashes (precise but more storage) vs per-directory hash (simpler but may over-regenerate)?
3. **Git integration**: Should incremental mode use `git diff` to detect changes (fast, CI-friendly) vs filesystem hashing (repo-agnostic)?
4. **Link rewriting**: When output is in a separate folder, should file links in summaries point back to the source repo (e.g., GitHub URLs) or to relative paths within the wiki?
5. **Exclusion mechanism**: `--exclude` flag repeated per pattern, or a config file?
