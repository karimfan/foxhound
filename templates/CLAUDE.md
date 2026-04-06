# Codebase Navigation via Wiki

This project has an auto-generated codebase wiki. **Always consult the wiki before reading source files.** It gives you a layered understanding of the codebase — from project-level architecture down to individual files — so you never waste time searching or guessing.

## Wiki location

- **Root**: `{{WIKI_PATH}}` — start here, always
- **Directory summaries**: `{{WIKI_DIR}}/{dir}/SUMMARY.md` (one per directory)
- **Root artifacts** (all in `{{WIKI_DIR}}/`):
  - `deps-graph.md` — which directories depend on which (blast radius)
  - `file-index.md` — every source file with its directory and description
  - `symbol-index.md` — key types, functions, interfaces and where they're defined
  - `recipes.md` — common modification patterns (which files change together for a task)
  - `traces.md` — end-to-end execution flows from entry points through the system
  - `churn.md` — which directories have been actively changing (last 90 days)

## The three-layer approach

Navigate the wiki top-down through three layers. Each layer gives you more detail. Stop as soon as you have what you need — you often don't need to reach Layer 3.

### Layer 1: Project-level understanding

**Read `{{WIKI_PATH}}` first.** Every task starts here. It gives you:
- What the project does and who uses it
- How the major pieces connect (architecture)
- A navigation guide mapping task categories → specific directories
- Links to all root artifacts

**Then consult the root artifact that matches your question:**

| Question | Read this |
|----------|-----------|
| What files do I need to change for this task? | `{{WIKI_DIR}}/recipes.md` |
| Where is a specific file or type defined? | `{{WIKI_DIR}}/file-index.md` or `{{WIKI_DIR}}/symbol-index.md` |
| What else depends on the code I'm about to change? | `{{WIKI_DIR}}/deps-graph.md` |
| How does data flow through the system end-to-end? | `{{WIKI_DIR}}/traces.md` |
| Is this area under active development? | `{{WIKI_DIR}}/churn.md` |

You now have a mental model of the project and know which directories to focus on.

### Layer 2: Directory-level understanding

**Read the `SUMMARY.md` for the 1-3 directories relevant to your task.** Each summary gives you:
- **Overview** — what the directory is responsible for
- **Key files** — every file with a one-line description (so you know which to open)
- **How it works** — internal data flow and call chains
- **Key types and interfaces** — the important exports with who uses them
- **Dependencies** — what it imports and what imports it
- **Boundary** — whether this is a public API, internal implementation, or entry point
- **Testing** — where tests live, what pattern they use (table-driven, fixtures, etc.)
- **When to look here** — task descriptions that lead to this directory
- **Child directories** — what each child owns

If a child directory looks relevant, read its SUMMARY.md too. Keep drilling down the tree until you've identified the specific files you need.

You now know exactly which files to read and why.

### Layer 3: Source code

**Only now open the actual source files** the wiki pointed you to. You should already know:
- Which files to read (from Layer 2's Key files section)
- What each file does (from the one-line descriptions)
- How the files interact (from the How it works section)
- What types and functions to look for (from Key types and interfaces)

Read source files in dependency order: data layer first, then business logic, then API/UI.

## Worked example

Task: "Add a new CLI subcommand that lists active policies"

```
Layer 1 — Project understanding:
  1. Read {{WIKI_PATH}}
     → Architecture: CLI lives in cmd/, policies in internal/policy/
     → Navigation guide: "To modify CLI commands → cmd/"
     → Navigation guide: "Policy management → internal/policy/"
  2. Read {{WIKI_DIR}}/recipes.md
     → Recipe "Add a new CLI subcommand" says: touch commands.go, flags.go, and add a test

Layer 2 — Directory understanding:
  3. Read {{WIKI_DIR}}/cmd/SUMMARY.md
     → Key files: main.go (entry point), commands.go (subcommand registry)
     → Boundary: entry point
     → Testing: table-driven tests in cmd_test.go
  4. Read {{WIKI_DIR}}/internal/policy/SUMMARY.md
     → Key files: manager.go (PolicyManager type), store.go (persistence)
     → Key types: PolicyManager.ListActive() returns []Policy
     → Boundary: public (imported by cmd/ and api/)

Layer 3 — Source code:
  5. Read cmd/commands.go (to see how subcommands are registered)
  6. Read internal/policy/manager.go (to see the ListActive API)
```

Total: 4 wiki reads + 2 source reads = full understanding, no searching.

## Task-specific workflows

### Modifying existing behavior
1. **Layer 1**: Read wiki.md → find the directory → check deps-graph.md for what depends on it
2. **Layer 2**: Read SUMMARY.md → identify the file → check Boundary (is this public API?)
3. **Layer 3**: Read the source file, make changes

### Adding a new feature
1. **Layer 1**: Read wiki.md → check recipes.md for a matching pattern → check traces.md to understand the flow you're extending
2. **Layer 2**: Read SUMMARY.md for each directory the recipe mentions
3. **Layer 3**: Read source files in dependency order, implement

### Debugging / investigating
1. **Layer 1**: Read wiki.md → find the relevant directory from the error/symptom → check churn.md (recent changes may be the cause)
2. **Layer 2**: Read SUMMARY.md → use "How it works" to trace the likely failure path
3. **Layer 3**: Read the specific source files along that path

### Writing tests
1. **Layer 2**: Read SUMMARY.md for the directory → check "Testing" section for patterns, locations, and helpers
2. **Layer 3**: Read an existing test file to see the exact style, then write yours to match

### Understanding blast radius
1. **Layer 1**: Read deps-graph.md → find all directories that import the one you're changing
2. **Layer 2**: Read SUMMARY.md for each dependent directory → check their Boundary sections
3. Scope your changes accordingly

## What NOT to do

- **Don't grep the entire repo** for a keyword. The wiki's navigation guide, file index, and symbol index are faster and more accurate.
- **Don't skip Layer 1.** The root wiki.md has architecture context that everything else assumes you know. Skipping it leads to wrong assumptions.
- **Don't read source speculatively.** If the wiki doesn't point you to a file, you probably don't need it.
- **Don't ignore recipes.md.** When available, it tells you exactly which files change together — derived from actual git history, not guesswork.
- **Don't assume file paths from memory.** Files get moved. The wiki and file-index.md reflect the current structure.
- **Don't forget to check Boundary.** Changing a "public" directory affects its importers. Changing an "internal" directory is safer.

## When the wiki isn't enough

The wiki covers structure, purpose, and relationships. For these, use other tools:
- **Recent changes**: `git log`, `git blame`
- **Runtime behavior**: test output, logs
- **Exact implementation details**: read the source (after the wiki points you to it)
- **If the wiki is missing or stale**: fall back to normal exploration. The wiki may need regeneration via `wikigen`.
