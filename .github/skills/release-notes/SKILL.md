---
name: release-notes
description: "Write release notes entries for merged PRs. Use when asked to write release notes, changelog entries, or document what changed in a release."
---

# Release Notes

Generate release notes entries for PRs.

## Process

1. **Detect scope**: current PR, or range of PRs for a release
2. **Classify each PR**:
   - `Added` — new features, new APIs
   - `Fixed` — bug fixes
   - `Changed` — behavioral changes, breaking changes
   - `Deprecated` — features marked for removal
   - `Removed` — features removed
   - `Performance` — performance improvements
   - `Infrastructure` — CI, build, tooling (usually omitted from user-facing notes)

3. **Write entry**: one line per PR, user-facing language

## Format

```markdown
### Added
- Added `--myOption` CLI switch for X (#PR_NUMBER)

### Fixed
- Fixed crash when Y with Z enabled (#PR_NUMBER, fixes #ISSUE)

### Changed
- Changed default behavior of X to Y (#PR_NUMBER)
```

## Rules

- Use user-facing language, not implementation details
- Reference PR number and fixed issue numbers
- Skip automated/bot PRs
- Skip pure infrastructure changes unless they affect users
- Group related changes under a single bullet if they were split across PRs
- Breaking changes get a `⚠️ Breaking:` prefix
