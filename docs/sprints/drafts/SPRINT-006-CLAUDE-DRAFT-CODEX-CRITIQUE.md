# Critique of Claude's Sprint 006 Draft

## Overall Assessment

Claude's draft is directionally strong: it understands the core problem, preserves the human-in-the-loop motivation from the intent, and proposes a practical asynchronous Slack-thread workflow. It also does a good job emphasizing the judgment boundary between static wiki usage and escalating to humans.

That said, the draft has three meaningful weaknesses relative to the intent and current repo trajectory:

1. It overcommits to a **single-file implementation** (`consult/consult.go`) even though this sprint introduces a second durable tool and should establish maintainable structure from the start.
2. It leans too heavily on **Slack email lookup as the primary identity strategy**, while the intent explicitly calls out configurable mapping as an open question and the repo tends to prefer deterministic local configuration.
3. It under-specifies how the new tool fits the repo's emerging **multi-tool architecture and skill-file layout**, especially since Sprint 005 already introduced skill generation patterns.

## What Claude's Draft Does Well

### 1. Strong problem framing

The Overview is clear and persuasive. It explains why docs are insufficient, why human expertise matters, and how this tool complements `wikigen`. That framing matches the intent well.

### 2. Good async interaction model

The decision to avoid blocking for a reply and instead poll a saved Slack thread is a solid v1 choice. It is operationally realistic and easier to compose with agent workflows than a synchronous wait loop.

### 3. Useful command concepts

The proposed `ask`, `check`, `sessions`, and `who` commands are intuitive. In particular, the non-contacting discovery command (`who`) is an important safety valve for both users and agents.

### 4. Message-context thinking is strong

Claude correctly includes file context, optional wiki excerpts, and diff/question payloads in the consultation message. That makes the output likely to be useful to the human recipient.

## Main Gaps

### 1. Tool naming drifts from the intent

The intent names the tool `wikiask` and describes it as tool #2 in a growing suite. Claude renames it to `consult`. The rename is not fatal, but it weakens continuity with the product direction established in the intent (`wikigen`, then `wikiask`) and with the likely naming convention for related tools.

**Why this matters:** naming is architecture here, not cosmetics. The repo is evolving from one tool to a family of tools; consistency matters early.

### 2. Single-file implementation is the wrong abstraction level

Claude explicitly anchors the sprint around `consult/consult.go` as one large file. That mirrors the current `wikigen.go` pattern, but the intent also says the project is expanding into a suite of tools and that this new tool should live in its own subfolder. A second long-lived tool is exactly where the codebase should start moving toward small internal modules rather than another monolith.

**Why this matters:** this sprint is not just “add one more command.” It introduces:
- git analysis
- expert scoring
- identity resolution
- Slack transport
- session persistence
- skill guidance

Those concerns are likely to change independently. A modular layout is a better fit.

### 3. Identity resolution is too optimistic

Claude treats `users.lookupByEmail` as the main path and positions explicit mapping as a future enhancement. The intent frames mapping as an open question and suggests configurability may be necessary. In real repos, git emails often differ from Slack emails, bots commit code, and shared/service accounts pollute history.

**Better default:** repo-local mapping first, email lookup second.

### 4. The repo integration story is too thin

The draft mentions this is tool #2, but it does not really plan for the repo consequences:
- how `README.md` should describe multiple tools
- where tool-specific guidance should live
- whether the new tool writes its own `AGENTS.md` / `CLAUDE.md` or plugs into the existing templates pattern
- whether there is any shared logic or shared conventions between `wikigen` and the new tool

Claude does include a `--update-skills` idea, which is smart, but it is underspecified relative to the existing template-driven setup.

### 5. Session persistence in CWD is awkward

Writing `.consult-session-{id}.json` to the current directory is simple, but probably the wrong default. Agents may invoke the tool from different working directories, and littering repo roots or arbitrary shells with session files is not ideal.

**Better options:**
- persist under the analyzed repo in a dedicated hidden file/directory, or
- persist under a user cache/state directory with explicit request IDs

This should at least be called out as a design choice, not silently assumed.

### 6. Scope is a bit uneven

Claude spends significant plan weight on `sessions` and `--update-skills`, but does not give enough attention to the quality and transparency of the expert-ranking model itself. The ranking logic is the product core: if it identifies the wrong humans, the rest of the tool is much less valuable.

## Specific Suggestions to Strengthen Claude's Draft

### Prefer `wikiask` over `consult`

Unless the team deliberately wants a shorter neutral name, the sprint should keep the intent's `wikiask` name to reinforce the emerging suite identity.

### Split the tool into a few focused files

Even if implementation remains lightweight, the sprint should plan separate files for:
- CLI wiring
- git analysis / ranking
- Slack transport
- identity mapping
- message rendering
- session/request state

That is still small and idiomatic Go, but far easier to evolve.

### Make `who-knows` a first-class command

Claude's `who` command is valuable, but it feels slightly tacked on. The sprint should elevate it as a first-class, independently useful mode that works with no Slack token and no side effects.

### Put configuration at the center

The sprint should define a repo-local config file up front for:
- Slack defaults
- author-to-Slack mappings
- path-specific fallback owners
- optional behavior thresholds

This de-risks the hardest operational part of the tool.

### Clarify wiki integration boundaries

The draft says it may include wiki excerpts, which is good, but it should explicitly define this as optional enrichment rather than a hard dependency. The tool should still work in repos that do not have fresh wiki output available.

## Comparison Against the Intent

### Strong alignment

- Human experts identified from recent code history
- Slack as the v1 communication channel
- Structured message containing code context and uncertainty
- Async reply retrieval rather than building a server
- Skill guidance for agent judgment

### Partial or weak alignment

- Configurable git-author → Slack mapping is not treated as central enough
- New-tool subfolder structure is acknowledged but not used to encourage modular design
- Relationship to existing templates/skill-file patterns is only partially developed
- The intent's multi-tool product direction is described, but not fully reflected in the file plan

## Bottom Line

Claude's draft is a solid product sketch and a credible sprint candidate. Its strongest contributions are the problem framing, asynchronous consultation workflow, and agent-judgment emphasis.

However, I would not adopt it unchanged. I would merge in three corrections before treating it as the basis for the final sprint:

1. Rename the tool back to `wikiask` unless there is a strong reason not to.
2. Replace the single-file plan with a modest but explicit multi-file module layout.
3. Make repo-local identity mapping and configuration a core v1 feature rather than a fallback.

With those changes, the draft would better match both the sprint intent and the codebase's likely next stage of evolution.
