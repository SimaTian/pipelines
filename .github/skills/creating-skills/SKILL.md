---
name: creating-skills
description: "Create custom agent skills. Use when discovering novel tools, receiving task-agnostic tips, or after researching specialized workflows. Teaches structure, YAML optimization for LLM discoverability, and token efficiency."
---

# Creating GitHub Copilot Agent Skills

## Pre-Check: Avoid Duplication

**STOP** and check before creating:

1. List all skills: `ls -la .github/skills/`
2. Read existing skill frontmatter
3. If semantically similar → update existing skill instead
4. If content incomplete → add section to existing skill

**Only create new skill if:** no semantic overlap, distinct problem domain, unique triggers.

## Skill Structure

### Directory

`.github/skills/skill-name/SKILL.md`

Directory name must match `name` in frontmatter. Lowercase, hyphenated.

### SKILL.md Requirements

1. **YAML Frontmatter** (required):
   ```yaml
   ---
   name: your-skill-name
   description: "1-2 sentences. Include trigger keywords for discoverability."
   ---
   ```

2. **Markdown Body**: instructions, examples, procedures

### Description Guidelines

Think SEO for LLMs — include terms users would type:

✅ `"Guide for debugging failing CI workflows. Use when asked to debug build failures, test errors, or pipeline issues."`
❌ `"Helps with debugging"`

### Content Structure

1. Title and Overview
2. When to Use (clear triggers)
3. Prerequisites
4. Step-by-Step Instructions
5. Examples with expected output
6. Troubleshooting
7. References

### Token Efficiency

- Keep instructions focused
- Use bullet points and numbered lists
- Reference external resources rather than duplicating
- Skills load on-demand — clear descriptions prevent unnecessary loading

## Additional Resources

Skills can include scripts (`.ps1`, `.sh`, `.csx`), templates, and reference docs alongside SKILL.md.
