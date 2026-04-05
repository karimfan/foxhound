# Critique of `SPRINT-004-CLAUDE-DRAFT.md`

## Overall Assessment

Claude's draft is strong on the core product direction. It correctly centers the sprint on the two main intent items: externalized output and incremental regeneration. It also does a good job of explaining why those changes matter operationally, especially for CI/CD and multi-repo usage.

That said, the draft drifts from the provided intent and current codebase in a few important ways:

- it introduces some features that are not clearly justified by the sprint seed
- it treats a few implementation choices as settled when the intent leaves room for simpler options
- it under-specifies a couple of correctness constraints that matter more than feature breadth
- it proposes at least one repository change that appears inconsistent with the current backwards-compatibility requirement

Overall: **good strategic draft, but too expansive and slightly less disciplined than the intent calls for**.

## What Claude Got Right

- The main framing is correct: Sprint 004 is about separating scan input from generated output and adding incremental behavior.
- The draft understands that ancestor regeneration is a required correctness rule, not just an optimization detail.
- The manifest/metadata concept is directionally sound and fits the single-file Go + stdlib constraint.
- The draft keeps backwards compatibility visible by preserving a one-argument invocation path.
- It recognizes that configurable exclusions are necessary to make the tool less Leash-specific.
- It adds useful validation items around repeated runs, changed leaves, deletions, and dry-run behavior.

## Main Issues

### 1. CLI shape is presented too definitively

The intent says the tool must remain backwards-compatible and that output should default to `<repo>`, but it does **not** require the new interface to be positional `wikigen <repo> <wiki>`. Claude’s draft makes that positional two-arg shape central.

Why this matters:

- The current tool already has flags and one positional root argument.
- Adding `--output <dir>` is a lower-friction extension than changing the positional contract.
- Positional `<repo> <wiki>` works, but it is a stronger interface decision than the intent requires.

Recommendation: treat output destination as a flag-first design unless there is a compelling reason to change the CLI grammar.

### 2. It adds `--base-url`, which is likely out of scope

The intent focuses on external output, incremental mode, configurable exclusions, and machine-readable progress. Claude adds `--base-url` and link rewriting for remote source hosting.

This is interesting, but it appears to be scope expansion rather than a requirement.

Why this matters:

- external output does not inherently require remote source links
- relative links inside the mirrored output tree are the more immediate concern
- `--base-url` adds prompt or post-processing complexity that is not necessary to satisfy the sprint intent

Recommendation: defer remote-link generation unless it emerges from a concrete requirement.

### 3. The draft misses the intent’s machine-readable progress requirement

The sprint intent explicitly calls out CI/CD suitability with deterministic behavior, exit codes, and machine-readable progress. Claude includes human-readable stats and progress, but the draft does not clearly elevate machine-readable progress as a first-class deliverable.

Why this matters:

- this is one of the few explicit non-functional requirements in the intent
- CI workflows benefit more from stable event output than from end-of-run summary strings

Recommendation: add an explicit progress mode such as `--progress=json` or equivalent, and mention deterministic event semantics in the Definition of Done.

### 4. The manifest design is reasonable, but too file-centric

Claude’s manifest stores per-file hashes and a per-directory `files_hash`. That is workable, but the draft under-emphasizes the role of **child summary changes** as a direct input to parent regeneration.

The intent is explicit: changing a leaf must regenerate its summary and all ancestors. In practice, parent dirtiness is not only about direct files; it is also about whether a child summary changed.

Why this matters:

- parent correctness depends on child summary content, not merely on child directory existence
- if the manifest focuses too heavily on direct file hashes, the implementation may bolt on propagation later rather than modeling it cleanly

Recommendation: make child-summary fingerprints or equivalent metadata a first-class cached input.

### 5. It proposes `.gitignore` changes that conflict with backwards compatibility

Claude’s `Files Summary` says `.gitignore` should be modified to remove `wiki.md` and `**/SUMMARY.md` entries because output is now external.

That looks questionable given the explicit requirement that running with just `<repo>` must still work and default output to the repo.

Why this matters:

- in-place output remains a supported mode
- generated files may still need to be ignored in repo-output mode
- changing `.gitignore` in that direction could be actively harmful

Recommendation: do not assume `.gitignore` should be changed unless the sprint intentionally drops in-repo output, which the intent does not permit.

### 6. It treats “no changes detected” early exit as a required UX outcome

Claude requires the second run to print `No changes detected` and make zero API calls. Zero API calls is a useful success condition; the exact string is not important.

Why this matters:

- the sprint template should emphasize behavior, not overfit user-visible wording
- machine-readable progress is more important than a specific English sentence

Recommendation: specify deterministic skip behavior and zero summarization calls, but avoid pinning exact phrasing unless needed.

### 7. Configurable exclusions need clearer backwards-compatible semantics

Claude says `--exclude` replaces the hardcoded exclusion list, but the intent is more nuanced: exclusions should be configurable, while existing behavior should remain usable and backwards-compatible.

Why this matters:

- “replace” could imply default exclusions disappear unless re-specified
- the likely safer behavior is “defaults remain, user adds or overrides via flags”

Recommendation: spell out whether `--exclude` augments defaults, replaces defaults, or has separate modes.

## Secondary Issues

- The architecture section is helpful, but the ASCII diagram consumes space without adding much implementation precision.
- The draft’s link-rewriting section introduces prompt/post-processing complexity before the simpler mirrored-output solution is exhausted.
- The “safe to commit” statement about the manifest in Security Considerations is too strong; whether metadata is committed should remain a repository policy decision.
- Deleted-directory handling is a good inclusion, but it should be framed as part of output-root cleanup semantics rather than only manifest maintenance.

## Suggested Improvements

If this draft were revised, I would suggest these changes:

- Reframe the CLI around `--output` instead of making two positional args the primary interface.
- Remove `--base-url` and remote link rewriting from Sprint 004 scope.
- Add explicit machine-readable progress requirements and DoD items.
- Make child-summary fingerprints a first-class part of incremental metadata design.
- Clarify exclusion semantics: preserve current defaults unless explicitly overridden.
- Remove the proposed `.gitignore` change unless there is a separately justified repo policy update.
- Keep validation focused on correctness properties and compatibility, not exact output strings.

## Bottom Line

Claude’s draft is useful and substantially aligned with the sprint’s direction, but it is **a bit too product-expansive and not strict enough about the intent’s operational constraints**. The biggest gaps are:

- unnecessary scope added via `--base-url`
- insufficient emphasis on machine-readable progress
- a questionable `.gitignore` change
- not making child-summary inputs central enough in the incremental model

With those corrected, it would be a solid sprint plan.
