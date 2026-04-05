# Sprint 004: Wikigen External Output & Incremental Mode

## Overview

Sprint 003 delivered a working wiki generator that reads a codebase and produces LLM-powered markdown summaries. However, it writes output files directly into the source repository and performs a full scan on every invocation — making it unsuitable for CI/CD integration or use across multiple repos.

This sprint makes two fundamental changes. First, it separates input from output: `wikigen <repo> <wiki>` reads from `<repo>` and writes all generated markdown into a `<wiki>` folder that mirrors the repo's directory structure. This means wikigen can live as a standalone tool, generating documentation for any repository without polluting its source tree. Second, it adds incremental mode: a JSON manifest stored in the wiki folder tracks per-file content hashes. On subsequent runs, only directories with changed files are re-summarized, and changes propagate upward through ancestor directories to the root `wiki.md`. This makes it fast enough for CI/CD pipelines triggered on every push.

Additionally, the hardcoded exclusion list is replaced with a configurable `--exclude` flag, making the tool truly repo-agnostic.

## Use Cases

1. **Multi-repo documentation**: A single wikigen binary generates docs for multiple repos into separate wiki folders
2. **CI/CD integration**: GitHub Actions runs wikigen on push, incrementally updating only the affected wiki pages (~seconds instead of minutes)
3. **External wiki hosting**: Wiki output folder can be a separate git repo, deployed to GitHub Pages or a static site
4. **Repo-agnostic tooling**: Any team can run wikigen against their codebase with custom exclusion patterns
5. **Cost optimization**: Incremental mode minimizes LLM API calls — only changed directories cost money

## Architecture

```
  wikigen <repo> <wiki> [--exclude pattern...]

  ┌─────────────────────────┐      ┌─────────────────────────┐
  │   Source Repo (read)     │      │   Wiki Folder (write)    │
  │                          │      │                          │
  │  cmd/                    │      │  cmd/                    │
  │    leash/                │  ──► │    leash/                │
  │      main.go             │      │      SUMMARY.md          │
  │  internal/               │      │    SUMMARY.md            │
  │    runner/               │      │  internal/               │
  │      *.go                │      │    runner/               │
  │    proxy/                │      │      SUMMARY.md          │
  │      *.go                │      │    proxy/                │
  │                          │      │      SUMMARY.md          │
  │                          │      │    SUMMARY.md            │
  │                          │      │  wiki.md                 │
  │                          │      │  .wikigen-manifest.json  │
  └─────────────────────────┘      └─────────────────────────┘
```

### Manifest Structure

```json
{
  "version": 1,
  "repo": "/absolute/path/to/repo",
  "generated_at": "2026-04-05T12:00:00Z",
  "directories": {
    "cmd/leash": {
      "files_hash": "sha256:abc123...",
      "files": {
        "main.go": "sha256:def456..."
      }
    },
    "internal/runner": {
      "files_hash": "sha256:...",
      "files": {
        "container.go": "sha256:...",
        "volume.go": "sha256:..."
      }
    }
  }
}
```

- **Per-file hashes**: SHA-256 of file content (after applying the same filters: skip binary, skip generated, truncate at 500 lines)
- **Per-directory hash** (`files_hash`): SHA-256 of sorted concatenation of `filename:hash` pairs — used for quick directory-level comparison
- **Stored in wiki folder**: `.wikigen-manifest.json` lives alongside the generated markdown

### Incremental Update Flow

```
1. Load manifest from <wiki>/.wikigen-manifest.json
   └─ If missing → full scan (first run)

2. Walk <repo> directories (post-order, same as before)

3. For each directory:
   a. Compute current files_hash
   b. Compare with manifest
   c. If unchanged → skip (use existing SUMMARY.md)
   d. If changed → mark as dirty

4. Propagate: any dirty dir marks all ancestors as dirty

5. For each dirty dir (post-order):
   a. Read files + child SUMMARY.md from wiki folder
   b. Summarize via Claude API
   c. Write SUMMARY.md to wiki folder

6. If any dir was dirty → regenerate wiki.md

7. Write updated manifest
```

### Link Strategy

Since output is in a separate folder, file links in summaries use **relative paths back to the source repo**. The tool accepts an optional `--base-url` flag for generating links to a remote source (e.g., `--base-url https://github.com/org/repo/blob/main`). Without it, links are relative paths assuming the wiki folder is at the same level as the repo.

## Implementation Plan

### Phase 1: Two-Arg CLI & Output Separation (~25%)

**Files:**
- `docs/wikigen.go` - Modify existing tool

**Tasks:**
- [ ] Change CLI to accept two positional args: `<repo>` `<wiki>` (wiki defaults to repo if omitted for backwards compat)
- [ ] Add `--exclude` flag (repeatable): patterns to exclude by base name (replaces hardcoded `excludeDirs`)
- [ ] Add `--base-url` flag: optional URL prefix for source file links
- [ ] Default exclusions: `.git` and dot-prefixed dirs always excluded; other exclusions via `--exclude`
- [ ] Create wiki output directory structure: mirror repo dirs, create missing dirs as needed
- [ ] Change all write operations to target wiki folder instead of source repo
- [ ] Update `buildBundle` to read source files from repo but read child SUMMARY.md from wiki folder
- [ ] Update `buildWiki` to generate links appropriate to the output structure
- [ ] Update `--clean` to operate on the wiki folder
- [ ] Update `printUsage` with new CLI syntax

### Phase 2: Manifest & Hashing (~25%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Define manifest JSON struct: version, repo path, generated_at, per-directory entries with files_hash and per-file hashes
- [ ] Implement SHA-256 hashing of file content (after applying read filters: skip binary, skip generated, truncate)
- [ ] Implement directory hash: sorted concatenation of `filename:filehash` pairs, then SHA-256
- [ ] Implement manifest load from `<wiki>/.wikigen-manifest.json`
- [ ] Implement manifest save after generation completes
- [ ] Handle manifest version mismatch: if version differs, force full scan

### Phase 3: Incremental Change Detection (~25%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] On run: load manifest, compute current directory hashes, compare
- [ ] Mark directories as dirty if: files_hash changed, directory is new, directory was removed
- [ ] Propagate dirty flag upward: if any child is dirty, parent is dirty (up to root)
- [ ] In the main loop: skip non-dirty directories (reuse existing SUMMARY.md from wiki folder)
- [ ] For dirty directories: re-read child SUMMARY.md from wiki folder (children may have just been regenerated)
- [ ] Handle deleted directories: remove their SUMMARY.md from wiki folder, remove from manifest
- [ ] If no directories are dirty: print "No changes detected" and exit early
- [ ] Always regenerate wiki.md if any directory was dirty
- [ ] Add `--full` flag to force full scan even when manifest exists
- [ ] Print stats: "Regenerated 3/68 directories (4 files changed)"

### Phase 4: Link Rewriting & Output Polish (~15%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] When `--base-url` is set: file links become `[filename](https://base-url/path/to/filename)` 
- [ ] When `--base-url` is not set: file links use relative paths from wiki SUMMARY.md to source repo files
- [ ] Update the LLM system prompt to generate links using a placeholder that gets post-processed
- [ ] Actually — simpler approach: post-process the LLM output to rewrite `[file](./file)` links to the correct paths based on output mode
- [ ] Ensure wiki.md table of contents links work within the wiki folder structure
- [ ] Add generated timestamp to wiki.md footer

### Phase 5: Validation & Testing (~10%)

**Tasks:**
- [ ] Run `go vet docs/wikigen.go`
- [ ] Test backwards compat: `wikigen <repo>` still works (output in repo)
- [ ] Test two-arg mode: `wikigen <repo> <wiki>` creates wiki folder with mirrored structure
- [ ] Test incremental: run twice, verify second run detects no changes
- [ ] Test incremental: modify a file, verify only its dir + ancestors regenerated
- [ ] Test `--exclude`: verify custom exclusions work
- [ ] Test `--full`: verify it forces full scan
- [ ] Test `--clean` on wiki folder
- [ ] Test `--dry-run` with incremental (should show what would change)
- [ ] Verify manifest is written correctly after full and incremental runs
- [ ] Test deleted directory handling

## API Endpoints (if applicable)

Not applicable.

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `docs/wikigen.go` | Modify | Add two-arg CLI, external output, manifest, incremental mode |
| `.gitignore` | Modify | Remove `wiki.md` and `**/SUMMARY.md` entries (output is now external) |

## Definition of Done

- [ ] `wikigen <repo> <wiki>` generates all markdown in the wiki folder, not the source repo
- [ ] `wikigen <repo>` still works (backwards compat, output in repo)
- [ ] `.wikigen-manifest.json` tracks per-file SHA-256 hashes
- [ ] Second run with no changes: prints "No changes detected", makes zero API calls
- [ ] Modifying a leaf file: only regenerates that dir + ancestors (not the whole tree)
- [ ] `--exclude` flag replaces hardcoded exclusion list
- [ ] `--full` flag forces full regeneration
- [ ] `--base-url` generates remote source links
- [ ] `--dry-run` shows what would change without calling API
- [ ] `--clean` operates on wiki folder
- [ ] `go vet` passes
- [ ] Deleted directories are cleaned up from wiki folder and manifest
- [ ] Stats output: shows how many dirs regenerated vs total
- [ ] No external Go dependencies

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Hash computation slows down large repos | Low | Low | SHA-256 is fast; only hashing files that pass filters (~eligible source files) |
| Manifest corruption | Low | High | If manifest is unparseable, fall back to full scan with a warning |
| Race condition in CI (concurrent runs) | Medium | Medium | Manifest write is atomic (write to temp file, rename); concurrent runs will just re-do some work |
| Link rewriting breaks LLM output | Medium | Medium | Post-process links rather than modifying the prompt; clear regex pattern for `[text](./path)` |
| Propagation misses an ancestor | Low | High | Test with nested changes; propagation is a simple upward walk from dirty dir to root |
| Backwards compat regression | Low | Medium | Test single-arg mode explicitly |

## Security Considerations

- API key still from environment variable, never logged
- Manifest contains file hashes but not file content — safe to commit
- No writes to source repo when wiki folder is specified
- `--base-url` is user-controlled input used in markdown links — no injection risk since markdown is static
- Wiki folder writes scoped to the specified path; no path traversal

## Dependencies

- Sprint 003 (completed) — existing wikigen.go codebase
- Go standard library (`crypto/sha256`, `encoding/json`, `net/http`)
- Anthropic API access

## References

- `docs/wikigen.go` — current implementation from Sprint 003
- `docs/docs/sprints/SPRINT-003.md` — original design
