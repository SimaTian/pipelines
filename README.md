# Pipelines

Copilot agent skills & AI review pipelines for .NET projects.

Adapted from patterns observed in [dotnet/fsharp](https://github.com/dotnet/fsharp)'s production Copilot workflow.

## What's in here

```
.github/
├── copilot-instructions.md              # Base agent behavior rules
└── skills/
    ├── review-council/SKILL.md          # Multi-model panel review (15 agents, 5 dimensions × 3 models)
    ├── hypothesis-driven-debugging/     # Scientific method for debugging
    ├── creating-skills/                 # Meta-skill: how to create new skills
    ├── pr-build-status/                 # CI status retrieval & interpretation
    └── release-notes/                   # Auto-generate changelog entries
```

## How to use

### Option 1: Copy into your repo

Copy the `.github/` directory into any repository. Copilot will automatically pick up:
- `copilot-instructions.md` — loaded for every Copilot interaction in the repo
- `skills/*/SKILL.md` — loaded on-demand when the skill name or trigger keywords are mentioned

### Option 2: Reference as a Copilot Space

Create a [Copilot Space](https://docs.github.com/en/copilot) that includes this repo's skills as reference material.

## Skills

### Review Council

The most powerful skill. Runs a **3-phase multi-model code review**:

1. **Briefing Pack** — auto-gathers PR metadata, linked issues (with all comments), full diff, commit messages
2. **Parallel Assessment** — 15 agents (5 dimensions × 3 models) independently assess:
   - Claims Coverage (are PR goals met?)
   - Test Coverage (happy path, negative path, feature interactions)
   - Code Quality (layer placement, ad-hoc patches, error handling)
   - Code Reuse (copy-paste detection, missed abstractions)
   - Cyclomatic Complexity (nesting depth, method length)
3. **Consolidation** — deduplicate, filter false positives, classify as Behavioral > Quality > Nitpick

**Invoke**: Ask Copilot to "run the review council" on your current branch.

### Hypothesis-Driven Debugging

Enforces scientific method for bug investigation:
- Create minimal reproduction
- Form **≥3 competing hypotheses**
- Verify each systematically
- Document in `HYPOTHESIS.md`
- Always re-run build + tests after changes

**Invoke**: Ask Copilot to "debug this using hypothesis-driven debugging"

### PR Build Status

Retrieves and interprets CI status for a PR. Handles multi-job workflows correctly (ANY fail = overall fail, not just picking one arbitrary job).

### Release Notes

Generates user-facing changelog entries, classifying PRs into Added/Fixed/Changed/Deprecated/Removed/Performance.

## Customization

### Tailoring for your project

1. **`copilot-instructions.md`**: Update build/test commands to match your project
2. **Review Council Dimension 2** (Test Coverage): Adjust test layer taxonomy
3. **Review Council Dimension 4** (Code Reuse): Add project-specific patterns
4. **Models**: Change the 3 models in the review council to match your access

### Adding new skills

Use the `creating-skills` skill, or manually:

```bash
mkdir .github/skills/my-skill
# Create .github/skills/my-skill/SKILL.md with YAML frontmatter
```

## Origin

These skills are distilled from analyzing 149 PRs (94 developer-driven) in dotnet/fsharp over a 2-month period. Key patterns:

- **Review Council** — their multi-model review panel with battle-tested false-positive filters
- **Hypothesis-Driven Debugging** — systematic debugging enforced via skill
- **Copilot-instructions.md** — "no bullshit" rules that produce better AI output
- **Iterative feedback** — Copilot PRs get 4.5× more review rounds than human PRs
