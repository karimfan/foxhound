# Sprint 004 Merge Notes

## Claude Draft Strengths
- Clear architecture diagram showing scan root vs output root separation
- Detailed manifest JSON schema with per-file and per-directory hashes
- Comprehensive incremental update flow (7-step algorithm)
- Good deletion handling (remove orphaned SUMMARY.md and manifest entries)
- `--base-url` flag for GitHub-compatible links (validated by interview)

## Codex Draft Strengths
- Proposed `--output` flag instead of positional args (lower friction, better backwards compat)
- Emphasized child-summary fingerprints as first-class metadata input for correct propagation
- Called out machine-readable progress as a missing first-class deliverable
- Correctly noted `.gitignore` shouldn't change since in-place mode is still supported
- Clarified exclusion semantics: defaults preserved, `--exclude` adds to them

## Valid Critiques Accepted
- **CLI shape**: Use `--output <dir>` flag instead of second positional arg. Cleaner extension of existing CLI.
- **Machine-readable progress**: Add `--json` flag for line-delimited JSON events on stderr. CI/CD requirement.
- **Child-summary fingerprints**: Make these first-class in the manifest. Parent dirtiness depends on child summary content, not just child directory existence.
- **Don't change .gitignore**: In-place output remains supported. Keep existing gitignore entries.
- **Exclusion semantics**: `--exclude` adds to defaults (`.git` + dot-dirs). Use `--no-default-excludes` to start from scratch.

## Critiques Rejected (with reasoning)
- **"Remove --base-url"**: User explicitly chose `--base-url` in interview as the link strategy for external wikis. Keeping it.
- **"Per-directory sidecar metadata"**: Single manifest file at wiki root is simpler to manage, atomic to write, and easier to inspect than scattered `.wikigen.json` sidecars in every directory.
- **"Add --incremental flag"**: Automatic detection is simpler — if manifest exists, use it. `--full` flag to override. No extra flag needed.

## Interview Refinements Applied
- Git diff primary, filesystem hash fallback for change detection
- `--base-url` flag for GitHub URLs when wiki is external
- `--exclude` flags only (no config file), additive to defaults

## Final Decisions
- **CLI**: `wikigen [flags] <repo>` with `--output <dir>`, `--exclude <pattern>`, `--base-url <url>`, `--full`, `--json`, `--dry-run`, `--clean`
- **Change detection**: Try `git diff` against manifest's last commit hash first; fall back to SHA-256 filesystem hashing if git unavailable or not a repo
- **Manifest**: Single `.wikigen-manifest.json` in wiki root. Stores per-file hashes, per-directory content hash, child-summary hashes, last git commit, schema version.
- **Incremental**: Automatic when manifest exists. `--full` to force full scan.
- **Links**: `--base-url` generates remote URLs; without it, filenames only (no broken relative links)
- **Exclusions**: `.git` + dot-prefixed dirs excluded by default. `--exclude` adds more. `--no-default-excludes` to start fresh.
- **Progress**: Human-readable by default. `--json` for line-delimited JSON events.
- **Output**: `--output` flag; defaults to in-place (repo root) for backwards compat.
- **.gitignore**: No changes (in-place mode still valid).
