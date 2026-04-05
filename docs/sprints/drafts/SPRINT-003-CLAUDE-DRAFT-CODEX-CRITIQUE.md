# Critique of `SPRINT-003-CLAUDE-DRAFT.md`

## Overall Assessment

Claude's draft is thoughtful, detailed, and well aligned with the core problem statement. It does a strong job articulating why a hierarchical wiki would help onboarding, review, and AI-agent context building. It also proposes a concrete architecture and operational details that make the sprint feel implementable.

That said, the draft overcommits in a few places relative to the sprint intent, the sprint template, and the repository's current conventions. The biggest issue is that it turns an open question about LLM usage into a hard dependency on Claude API integration, which adds cost, credentials, failure modes, and implementation scope that are not clearly required by the intent. In effect, the draft reads more like a product design for an LLM-powered documentation system than a tightly scoped sprint plan for a first version of repository tooling.

## What Claude's Draft Does Well

- The overview is clear and grounded in a real repository need.
- The use cases are strong and concrete.
- The architecture section is more detailed than the template requires, but it is useful and easy to follow.
- The implementation plan is actionable and broken into phases with effort estimates.
- The draft identifies practical operational concerns such as retries, dry runs, cleaning generated files, and progress output.
- Risks and mitigations are handled thoughtfully.

## Main Issues

### 1. It resolves an open question too aggressively

The intent explicitly lists LLM-powered summaries vs static analysis as an open question. Claude's draft chooses Claude API integration as the central design, recommends it as the default, and builds much of the sprint scope around that choice.

Why this is a problem:
- It locks Sprint 003 into an external dependency that the seed did not require.
- It increases scope substantially: prompt design, API client logic, retries, rate limiting, auth handling, dry-run semantics, CLI fallback, and model selection.
- It weakens reproducibility and determinism for generated output.
- It conflicts somewhat with the repository precedent of simple, dependency-light Go tooling.

A stronger sprint plan would treat LLM enrichment as optional or future work, while ensuring the baseline generator is useful without network or API credentials.

### 2. It introduces scope not justified by the intent

Several proposed features feel more like v2 enhancements than Sprint 003 essentials:
- `--use-cli` fallback to `claude`
- `--dry-run`
- `--clean`
- retry and exponential backoff
- model selection questions
- potential `.gitignore` changes
- progress logging details

These are not bad ideas, but they compete with the main sprint objective: generate hierarchical summaries for Leash in a bottom-up fashion. The draft would be stronger if it separated must-have deliverables from optional polish.

### 3. It makes some questionable architectural choices or phrasing

The draft refers to a "bottom-up BFS," which is awkward and potentially confusing. What the sprint really needs is post-order traversal or depth-sorted processing so child summaries exist before parent summaries are generated. That should be described more precisely.

It also proposes excluding all hidden directories in the data flow section, while the intent only explicitly excludes `.claude`, `.codex`, and `docs`. Excluding `.git` is sensible, but broad hidden-directory exclusion should be justified rather than assumed.

### 4. It is slightly off-template in a few places

The sprint template calls for:
- Overview
- Use Cases
- Architecture
- Implementation Plan
- API Endpoints (if applicable)
- Files Summary
- Definition of Done
- Security Considerations
- Dependencies
- References

Claude's draft includes strong extra sections like Risks & Mitigations and Open Questions, but omits the explicit `API Endpoints (if applicable)` section and the `References` section from the template. Since the user explicitly asked for adherence to the sprint template, that is a meaningful miss.

### 5. Some file-path decisions need stronger alignment with the repo

Claude chooses `tools/wikigen.go`, which is plausible, but the draft does not justify that location against the actual repo structure or the precedent cited (`docs/sprints/tracker.go`). Given the prompt's emphasis on familiarizing with project structure, the sprint plan should explain why the tool belongs where it does.

Similarly, the proposal to modify `.gitignore` is framed as "TBD," which feels premature in a planning doc unless the sprint explicitly decides whether generated documentation is intended to be tracked.

## Specific Strengths Worth Keeping

If this draft is revised, these elements are worth preserving:
- The clear articulation of bottom-up summary chaining.
- The emphasis on relative links and navigability.
- The recognition that mixed-language support matters for Leash.
- The validation mindset: link checking, exclusion verification, manual quality review.
- The idea that generated docs should say what code does, not just enumerate files.

## Recommended Revisions

1. Reframe LLM usage as optional or deferred.
   - Make the baseline sprint deliver a deterministic generator using static analysis and heuristics.
   - If desired, add an explicit note that optional LLM enrichment may be explored later.

2. Narrow Sprint 003 scope to the core deliverable.
   - Keep traversal, file classification, summary generation, markdown rendering, and repository rollout.
   - Move `--clean`, `--use-cli`, retry logic, and model-selection concerns out of the main sprint unless clearly justified.

3. Tighten terminology.
   - Replace "bottom-up BFS" with "post-order traversal" or "depth-sorted bottom-up traversal."

4. Conform more closely to the sprint template.
   - Add `API Endpoints (if applicable)` with "Not applicable."
   - Add a `References` section pointing to the root `README.md`, sprint README, and sprint intent.

5. Make repository placement more deliberate.
   - Justify the tool location with reference to the existing project structure and tooling precedent.

## Bottom Line

Claude's draft is high quality in terms of thoughtfulness and execution detail, but it is too ambitious and opinionated about LLM integration for the intent as written. It would be an excellent foundation for a later, richer version of the tool, but for Sprint 003 it should be simplified, made more deterministic, and brought into closer alignment with the sprint template and the repository's lightweight Go tooling style.
