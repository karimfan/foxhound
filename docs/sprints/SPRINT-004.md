# Sprint 004: Wikigen External Output & Incremental Mode

## Overview

Sprint 003 delivered a working wiki generator, but it writes output directly into the source repo and performs a full scan on every invocation. This makes it unsuitable for CI/CD integration, multi-repo use, or cost-efficient incremental updates.

Sprint 004 makes three changes. First, it adds an `--output <dir>` flag so generated markdown goes into a separate wiki folder that mirrors the repo's directory structure — no source tree pollution. Second, it adds automatic incremental mode: a JSON manifest in the wiki folder tracks per-file and per-directory content hashes plus child-summary fingerprints. On subsequent runs, the tool tries `git diff` to quickly identify changed files, falling back to filesystem hash comparison when git is unavailable. Only dirty directories and their ancestors are re-summarized. Third, it replaces the hardcoded exclusion list with configurable `--exclude` flags, making the tool repo-agnostic.

The tool remains a single-file Go program using only the standard library.

## Use Cases

1. **Multi-repo documentation**: `wikigen --output /docs/leash-wiki /repos/leash` generates docs for any repo into any folder
2. **CI/CD integration**: GitHub Actions runs wikigen on push; incremental mode regenerates only affected pages (~seconds, minimal API cost)
3. **External wiki hosting**: Wiki output folder can be a separate git repo deployed to GitHub Pages
4. **Cost optimization**: Incremental mode minimizes LLM API calls — only changed directories cost money
5. **Custom exclusions**: `wikigen --exclude vendor --exclude node_modules /repos/myapp` works for any codebase

## Architecture

### Output Layout

```
  wikigen --output <wiki> [flags] <repo>

  ┌─────────────────────────┐      ┌─────────────────────────────┐
  │   Source Repo (read)     │      │   Wiki Folder (write)        │
  │                          │      │                              │
  │  cmd/                    │      │  cmd/                        │
  │    leash/                │  ──► │    leash/                    │
  │      main.go             │      │      SUMMARY.md              │
  │  internal/               │      │    SUMMARY.md                │
  │    runner/               │      │  internal/                   │
  │      *.go                │      │    runner/                   │
  │                          │      │      SUMMARY.md              │
  │                          │      │    SUMMARY.md                │
  │                          │      │  wiki.md                     │
  │                          │      │  .wikigen-manifest.json      │
  └─────────────────────────┘      └──────────────────────────────┘
```

Without `--output`, behavior is unchanged from Sprint 003 (output written in-place to repo).

### Change Detection (Dual Strategy)

**Primary: Git diff** — When the repo is a git repository:
1. Load manifest's `last_commit` field
2. Run `git diff --name-only <last_commit> HEAD` to get changed files
3. Map changed files to their parent directories → mark those directories dirty
4. Propagate dirty flag upward through all ancestors

**Fallback: Filesystem hashing** — When git is unavailable or repo is not a git repo:
1. Load manifest's per-file SHA-256 hashes
2. For each directory, compute current file hashes and compare
3. Mark mismatched directories as dirty
4. Propagate upward

### Manifest Structure

Stored at `<wiki>/.wikigen-manifest.json`:

```json
{
  "version": 1,
  "repo": "/absolute/path/to/repo",
  "last_commit": "abc123def456...",
  "generated_at": "2026-04-05T12:00:00Z",
  "excludes": [".git", ".claude", ".codex", "docs"],
  "directories": {
    "cmd/leash": {
      "files_hash": "sha256:...",
      "summary_hash": "sha256:...",
      "files": {
        "main.go": "sha256:..."
      }
    },
    "internal/runner": {
      "files_hash": "sha256:...",
      "summary_hash": "sha256:...",
      "files": {
        "container.go": "sha256:...",
        "volume.go": "sha256:..."
      }
    }
  }
}
```

Key fields:
- **`files_hash`**: SHA-256 of sorted `filename:filehash` pairs — quick directory-level comparison
- **`summary_hash`**: SHA-256 of the generated SUMMARY.md content — used by parents to detect child changes
- **`last_commit`**: Git HEAD at generation time — enables git-diff-based change detection
- **`excludes`**: The exclusion list used — if it changes, forces full rescan

### Incremental Update Flow

```
1. Load manifest from <wiki>/.wikigen-manifest.json
   ├─ Missing → full scan (first run)
   └─ Present & excludes changed → full scan

2. Detect changes:
   ├─ Git available → git diff last_commit..HEAD → set of changed files
   └─ No git → compare per-file hashes against manifest

3. Map changed files to directories → mark dirty

4. Walk directories post-order:
   For each dir:
   ├─ Dir is dirty → regenerate (call LLM, write SUMMARY.md)
   ├─ Any child's summary_hash differs from manifest → mark dirty, regenerate
   └─ Clean → skip, load existing SUMMARY.md for parent use

5. If any dir regenerated → regenerate wiki.md

6. Write updated manifest (atomic: write temp file, rename)

7. Print stats: "Regenerated 3/68 directories (4 files changed)"
```

### Link Strategy

- **With `--base-url`**: File references become clickable URLs: `[main.go](https://github.com/org/repo/blob/main/cmd/leash/main.go)`
- **Without `--base-url`**: Filenames appear as inline code without links (avoids broken relative paths when wiki is external)
- Links between SUMMARY.md files within the wiki folder use relative paths (always valid since structure is mirrored)

## Implementation Plan

### Phase 1: CLI Refactor & Output Separation (~20%)

**Files:**
- `docs/wikigen.go` - Modify existing tool

**Tasks:**
- [ ] Add `--output <dir>` flag; default to scan root when omitted (backwards compat)
- [ ] Add `--exclude <pattern>` flag (repeatable); additive to defaults
- [ ] Add `--no-default-excludes` flag to start with empty exclusion list
- [ ] Add `--base-url <url>` flag for source file links
- [ ] Add `--full` flag to force full scan even when manifest exists
- [ ] Add `--json` flag for line-delimited JSON progress events on stderr
- [ ] Refactor path handling: separate `repoRoot` and `wikiRoot` throughout
- [ ] Create mirrored directory structure in wiki folder as needed
- [ ] Change all write operations to target wiki folder
- [ ] Update `buildBundle` to read source from repo, read child SUMMARY.md from wiki folder
- [ ] Update `printUsage` with new CLI syntax

### Phase 2: Manifest & Hashing (~20%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Define manifest JSON structs with schema version
- [ ] Implement SHA-256 hashing of file content (after read filters: skip binary, skip generated, truncate)
- [ ] Implement directory hash: SHA-256 of sorted `filename:filehash` pairs
- [ ] Implement summary hash: SHA-256 of generated SUMMARY.md content
- [ ] Implement manifest load from `<wiki>/.wikigen-manifest.json`
- [ ] Implement manifest save (atomic: write to temp file, rename)
- [ ] Handle corrupt/unparseable manifest: warn and fall back to full scan
- [ ] Handle version mismatch: force full scan
- [ ] Handle exclusion list mismatch: force full scan

### Phase 3: Git-Based Change Detection (~15%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Detect if repo root is a git repository (check for `.git` directory)
- [ ] Read current HEAD commit hash via `git rev-parse HEAD`
- [ ] Run `git diff --name-only <last_commit> HEAD` to get changed file paths
- [ ] Map changed file paths to their parent directories
- [ ] Filter changed files through the same eligibility rules (extension, skip patterns)
- [ ] If git commands fail: log warning, fall back to hash-based detection
- [ ] Store current HEAD in manifest after successful run

### Phase 4: Incremental Execution & Propagation (~20%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] On run: load manifest, detect changes (git or hash), build dirty set
- [ ] Mark directories dirty if: files changed, directory is new, directory was removed from repo
- [ ] Propagate dirty flag upward: any dirty child → parent is dirty (up to root)
- [ ] Also mark parent dirty if child's `summary_hash` differs from manifest
- [ ] In main loop: skip clean directories (load existing SUMMARY.md from wiki folder for parent use)
- [ ] For dirty directories: re-summarize via Claude API, write SUMMARY.md, compute new summary_hash
- [ ] Handle deleted directories: remove SUMMARY.md from wiki folder, remove from manifest
- [ ] If `--full` flag: skip all change detection, regenerate everything
- [ ] If no directories dirty: print "No changes detected", exit 0
- [ ] Regenerate wiki.md if any directory was dirty

### Phase 5: Progress Output & Link Rewriting (~10%)

**Files:**
- `docs/wikigen.go` - Same file

**Tasks:**
- [ ] Default progress: human-readable on stderr (same style as Sprint 003, add skip/regen indicators)
- [ ] `--json` mode: emit line-delimited JSON events: `{"event":"scan","dir":"internal/runner","status":"dirty","files_changed":2}`
- [ ] Events: `scan` (per dir), `skip` (unchanged), `regenerate` (dirty), `write` (file written), `done` (final stats)
- [ ] Post-process LLM output for links: when `--base-url` set, rewrite `[file](./file)` to `[file](base_url/dir/file)`
- [ ] When no `--base-url`: replace `[file](./file)` with `` `file` `` (inline code, no link)
- [ ] Print final stats: "Regenerated N/M directories (K files changed)"
- [ ] Exit codes: 0 = success, 1 = error, 2 = partial failure (some dirs failed but others succeeded)

### Phase 6: Validation & Testing (~10%)

**Tasks:**
- [ ] Run `go vet docs/wikigen.go`
- [ ] Test backwards compat: `go run docs/wikigen.go .` writes output in-place (same as Sprint 003)
- [ ] Test external output: `go run docs/wikigen.go --output /tmp/wiki .` creates mirrored structure
- [ ] Test full scan: first run with no manifest generates everything
- [ ] Test incremental: run twice with no changes → "No changes detected", zero API calls
- [ ] Test incremental: modify a leaf file → only that dir + ancestors regenerated
- [ ] Test git-based detection: verify git diff is used when available
- [ ] Test hash fallback: run against a non-git directory → hash-based detection works
- [ ] Test `--exclude`: custom exclusions work, defaults preserved
- [ ] Test `--no-default-excludes`: only `.git` excluded
- [ ] Test `--full`: forces full regeneration even with manifest
- [ ] Test `--clean` on wiki folder: removes all generated files and manifest
- [ ] Test `--json`: verify valid JSON events on stderr
- [ ] Test `--base-url`: verify links in generated markdown
- [ ] Test deleted directory: remove a dir from repo, run → SUMMARY.md removed from wiki
- [ ] Test corrupt manifest: garbled JSON → warning + full scan
- [ ] Test `--dry-run` with incremental: shows what would change

## API Endpoints (if applicable)

Not applicable.

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `docs/wikigen.go` | Modify | Add --output, --exclude, --base-url, --full, --json flags; manifest system; git diff + hash-based change detection; incremental execution; link rewriting |

## Definition of Done

- [ ] `go run docs/wikigen.go .` remains backwards-compatible (in-place output)
- [ ] `go run docs/wikigen.go --output /tmp/wiki .` generates all markdown under `/tmp/wiki`
- [ ] `.wikigen-manifest.json` tracks per-file hashes, directory hashes, summary hashes, and last git commit
- [ ] Second run with no changes: zero API calls, quick exit
- [ ] Modifying a leaf file: only that directory + ancestors re-summarized
- [ ] Git diff used when available; hash comparison when not
- [ ] `--exclude` adds to default exclusions; `--no-default-excludes` clears defaults
- [ ] `--base-url` generates remote source links in summaries
- [ ] `--full` forces complete regeneration
- [ ] `--json` emits line-delimited JSON events on stderr
- [ ] `--dry-run` shows what would change without API calls
- [ ] `--clean` removes generated files and manifest from wiki folder
- [ ] Deleted source directories → removed from wiki folder and manifest
- [ ] Corrupt manifest → warning + full scan (safe fallback)
- [ ] `go vet` passes, no external Go dependencies
- [ ] Stats output shows regenerated/total directories

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Git diff misses renames/moves | Low | Medium | Hash fallback catches what git diff misses; `--full` available |
| Hash computation slow on large repos | Low | Low | Only hashing eligible source files (small subset); used as fallback only |
| Manifest corruption | Low | High | Unparseable manifest → warning + full scan (safe fallback) |
| Concurrent CI runs race on manifest | Medium | Medium | Atomic write (temp + rename); concurrent runs just re-do some work |
| Link rewriting breaks LLM output | Medium | Medium | Post-process with clear regex; test with representative outputs |
| Child summary propagation incorrect | Low | High | Manifest tracks summary_hash per directory; parent checks child hashes before skipping |

## Security Considerations

- API key from environment variable, never logged or included in manifest
- Manifest contains file hashes and metadata only, not file content
- No writes to source repo when `--output` is specified
- Wiki folder writes scoped to the specified path
- Git commands executed read-only (`rev-parse`, `diff --name-only`)
- `--base-url` is user input used in markdown links — static content, no injection risk

## Dependencies

- Go standard library (`crypto/sha256`, `encoding/json`, `net/http`, `os/exec` for git)
- Anthropic API access (`ANTHROPIC_API_KEY`)
- Sprint 003 (completed) — existing wikigen.go

## References

- `docs/wikigen.go` — current implementation
- `docs/docs/sprints/SPRINT-003.md` — original design
- `docs/docs/sprints/drafts/SPRINT-004-INTENT.md` — sprint intent
