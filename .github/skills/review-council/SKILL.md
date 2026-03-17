---
name: review-council
description: "Multi-agent review council. Invoke for: review council, panel review, multi-model review, deep code review, diverse model review, fellowship review."
---

# Review Council

A multi-model code review panel. You orchestrate 3 phases: build a briefing pack, dispatch parallel assessment agents, and consolidate findings.

## Phase 1: Briefing Pack

Auto-detect the PR from the current branch. Build a self-contained briefing document containing:

1. **PR metadata**: title, body, labels
2. **Linked issues**: for every issue referenced in the PR body or commits, fetch the full issue body and **all comments** — this is where the real requirements and context live
3. **Full diff**: merge-base diff only (`git diff $(git merge-base HEAD <base>)..HEAD`) — shows only what the PR adds
4. **Changed files list**: with change type (added/modified/deleted), test files called out separately
5. **Commit messages**: useful for correlating claims to specific code changes

Use `claude-opus-4.6` for this phase. The output is a single structured briefing document. Every assessment agent in Phase 2 receives this briefing verbatim.

## Phase 2: Parallel Assessment

Dispatch **15 agents in parallel**: each of the 5 dimensions below assessed independently by each of 3 models.

**Models**: `claude-opus-4.6`, `gemini-3-pro-preview`, `gpt-5.2-codex`

Each agent receives:
- The full briefing pack from Phase 1
- Its specific dimension prompt (below)
- Instruction to return findings as a list, each with: location (file:line), description, severity (critical/high/medium/low)

---

### Dimension 1: Claims Coverage

Are the goals met? Cross-reference every claim in the PR description, linked issues, and commit messages against actual code changes. Flag:

- **Orphan claims**: stated in PR/issue but no corresponding code change
- **Orphan changes**: code changed but not mentioned in any claim
- **Partial implementations**: claim says X, code does X-minus-something

**Before flagging**, apply these gates:
- **Execution context**: understand how and where the code runs before judging. Test harnesses, standalone projects, and production code have different constraints.
- **Direction**: classify each behavioral change as regression, improvement, or unclear. Only regressions are findings.

---

### Dimension 2: Test Coverage

Is every feature, fix, or behavioral change covered with tests?

- **Happy path**: does the basic intended usage work and is it tested?
- **Negative path**: invalid input, error conditions, edge cases?
- **Feature interactions**: how does the change interact with other features? Example: a serialization change should test both value types and reference types, nullable and non-nullable.
- **Assertion quality**: tests must assert the claimed behavior, not just "runs without throwing"

Flag behavioral changes that lack corresponding tests. Tests should be in the appropriate layer:
- **Unit tests**: isolated logic, pure functions, utilities
- **Integration tests**: cross-component behavior, database interactions, API calls
- **End-to-end tests**: full workflow correctness
- **Regression tests**: specific bug-fix scenarios with repro from the issue

---

### Dimension 3: Code Quality

Assess structural quality of changes:

- **Logical layer placement**: is the code in the right class/namespace/project?
- **Ad-hoc patches**: `if condition then specialCase else normalPath` that looks like a band-aid. The fix should be at the source, not patched at a consumer.
- **Duplicated logic within the PR diff**: same or near-same code in multiple places
- **Error handling**: not swallowing exceptions, not ignoring Result values, proper use of nullable reference types
- **Respect intentional deviations**: some projects deliberately diverge from conventions. Check whether the deviation is structural before flagging.

---

### Dimension 4: Code Reuse & Patterns

Search the codebase for existing patterns that match new code:

- **Copy-paste-modify**: new code duplicating an existing method with small changes. The difference should be a parameter, a generic type argument, or a delegate.
- **Missed abstractions**: two pieces of code sharing structure but differing in a specific operation — extract it.
- **Existing utilities ignored**: the codebase may already have helpers, extension methods, or base classes that do what the new code reimplements.
- **C#/.NET patterns**: check for proper use of `IDisposable`, `async/await`, `Span<T>`, LINQ, pattern matching, and other modern C# features.

---

### Dimension 5: Cyclomatic Complexity

Assess complexity of added/changed code:

- **Deep nesting**: any nesting beyond 3 levels should be questioned
- **C# offers better tools**: pattern matching with `switch` expressions, guard clauses for early returns, LINQ for collection operations
- **High branch count**: methods with many `if/else if` chains or `switch` arms — consider splitting
- **Method length**: methods over 50 lines should be examined for single-responsibility violations
- **Flatter is better**: prefer guard clauses and early returns over nested `if/else`

---

## Phase 3: Consolidation

Collect all findings from the 15 agents. Then:

1. **Deduplicate**: merge findings pointing at the same location with the same problem. Keep the best description.
2. **Filter false positives**:
   - Wrong context assumed → discard
   - Correct convention flagged → discard
   - Improvement flagged as problem → downgrade to informational
   - "Unexplained" ≠ "wrong" — missing rationale is a doc gap, not a defect
   - Speculation consensus (≥2 models agree but reasoning relies on "could cause" without evidence) → LOW confidence
3. **Classify** into three buckets:
   - **Behavioral**: missing coverage, incorrect logic, claims not met — affects correctness
   - **Quality**: structure, readability, complexity, reuse — works but could be better
   - **Nitpick**: typos, naming, formatting — low priority, only surface if no higher findings
4. **Rank**: prefer findings flagged by more agents (cross-model agreement = higher confidence)
5. **Present**: Behavioral first, then Quality, then Nitpicks. For each: location, dimension, description, how many agents flagged it.

If a model is unavailable at runtime, proceed with remaining models. Minimum viable council = 2 models.
