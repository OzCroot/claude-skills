---
name: refactor
description: Identify refactor opportunities across a codebase following language-specific best practices with pluggable language configs. Use when user wants to find refactoring opportunities, reduce tech debt, clean up code, improve code structure, or mentions code smells.
---

# Refactor

Identify and execute refactor opportunities through a structured, four-phase workflow. Never refactor without shared understanding.

## Language Detection

1. Auto-detect languages by scanning for file extensions and config files:
   - `*.vue` + `*.ts` → load `vue.md` + `typescript.md`
   - `@theme` or `@import "tailwindcss"` or `@tailwind` in CSS → load `tailwind.md`
   - `*.rs` + `Cargo.toml` → load `rust.md`
2. Load all matching configs from `languages/` in this skill directory
3. If the user provides a language argument (e.g., `/refactor rust`), load only that config
4. If the user provides a path argument (e.g., `/refactor src/modules/messages`), scope the scan to that path

## Project Awareness

Before scanning, check for project-level documentation:
- `.docs/`, `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`
- Read patterns, standards, and architecture docs when they exist
- **Project docs override language configs** — if the project has an established pattern, do not suggest changing it
- If no project docs exist, rely solely on the language configs

## Phase 1: Discover

Scan the codebase using smart scoping:
1. Read project structure — architecture docs, directory tree
2. Sample representative files from each area (do NOT read every file)
3. Go deeper only where something warrants investigation
4. If user provided a path, scope to that subtree

Present findings in these **universal categories**:

| Category | What to look for |
|---|---|
| **Duplication** | Repeated logic that could be consolidated |
| **Structural** | Code in the wrong place, modules to split/merge |
| **Idiom** | Working code that doesn't follow language best practices |
| **Complexity** | Functions/components doing too much |
| **Dead code** | Unused exports, unreachable branches |

For each finding, rate:
- **Impact**: High / Medium / Low — how much of the codebase benefits
- **Risk**: High / Medium / Low — how likely to introduce bugs
- **Effort**: High / Medium / Low — how much work to implement

Present as a summary table grouped by category. Wait for the user to proceed.

## Phase 2: Discuss

For each category that has findings:
1. Walk through the findings as a group
2. Ask probing questions about intent and trade-offs:
   - "This pattern appears in N files — was there a reason they diverged?"
   - "Consolidating these would couple modules X and Y — comfortable with that?"
   - "This uses an older pattern. Migration in progress, or stable?"
3. Listen to the answers — remove items the user wants to keep as-is
4. Summarize agreed actions at the end of each category before moving on

Do NOT proceed to Plan until all categories are discussed and the user confirms the final list.

## Phase 3: Plan

For each agreed refactor, produce a **file-level change manifest**:
- Files to create, modify, or delete
- Description of what changes in each file
- **Order of operations** — dependencies between refactors (do X before Y)
- **Risk notes** — edge cases, things that could break, testing needs
- **Rollback strategy** — each refactor gets its own commit

Present the full plan. Wait for explicit approval before executing.

## Phase 4: Execute

For each refactor in the plan, in order:
1. Implement the changes
2. Commit with a descriptive message
3. **Stop and show the user what changed**
4. Wait for approval before proceeding to the next refactor

If the user requests adjustments, make them before moving on. If the user wants to stop, all completed refactors are already committed and coherent.

## Adding New Languages

Create a new `.md` file in `languages/` with three sections:

```md
## Refactor Patterns
(What to look for — named patterns with detection heuristics)

## Language Idioms
(How idiomatic code should read — prefer X over Y)

## Structural Conventions
(Where things should live — project organization rules)
```

Update the detection rules in Phase 1 to recognize the new language.
