# Sprint 003 Merge Notes

## Claude Draft Strengths
- Rich architecture diagram and clear data flow
- Thorough implementation plan with actionable tasks per phase
- Good risk identification (hallucinations, rate limits, large files)
- Strong validation phase (link checking, manual review)
- Emphasis on summaries describing what code *does*, not just listing files

## Codex Draft Strengths
- Tighter scope focused on core deliverable
- Better terminology: "post-order traversal" vs "bottom-up BFS"
- Closer template adherence (included API Endpoints, References sections)
- Pragmatic phase sizing and validation approach
- Clean separation of must-have vs nice-to-have features

## Valid Critiques Accepted
- **Terminology**: "post-order traversal" is more precise than "bottom-up BFS" — adopted
- **Template conformance**: Added API Endpoints (N/A) and References sections
- **Scope trimming**: `--use-cli` fallback, model selection flag, and retry/backoff are v2 features — removed from core sprint
- **Tool location justification**: Now explicitly at `docs/wikigen.go` per user decision, justified by proximity to tracker.go

## Critiques Rejected (with reasoning)
- **"LLM should be deferred"**: User explicitly chose LLM via Claude API in interview. The seed says "understand what the intent of these files are" which implies semantic understanding, not just structural parsing. LLM integration stays as core requirement.
- **"Static analysis baseline first"**: While a reasonable engineering choice, the user wants rich descriptions now. Static analysis alone would produce shallow output that doesn't meet the seed's goals.

## Interview Refinements Applied
- Tool lives at `docs/wikigen.go` (user chose this over `tools/` or `cmd/`)
- All generated files gitignored (user chose this — cleanest git history)
- Config files (Dockerfile, Makefile, YAML) included in summaries (user confirmed)
- LLM via Claude API is the summarization method (user chose over CLI or static)

## Final Decisions
- **Location**: `docs/wikigen.go` — single-file Go CLI alongside tracker.go
- **Summarization**: Claude API via direct HTTP, using Haiku for speed/cost
- **Traversal**: Post-order depth-first traversal (deepest dirs first)
- **Output**: `SUMMARY.md` per directory, `wiki.md` at root, both gitignored
- **Scope**: Core traversal + LLM summarization + markdown generation. No `--use-cli`, no `--model` flag, no retry/backoff. Keep `--dry-run` and `--clean` as they're essential for safe operation.
- **Exclusions**: `.claude`, `.codex`, `docs`, `.git` (and other dot-prefixed dirs)
- **Config files**: Included in summaries alongside source code
