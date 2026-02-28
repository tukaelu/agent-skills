# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

This is an **agent skills collection** distributed via `npx skills add tukaelu/agent-skills`. Skills extend AI coding agents' capabilities by providing structured workflows and domain knowledge.

## Skill Structure

Each skill lives in `skills/<skill-name>/` and must contain a `SKILL.md` file. Optional components:

```
skills/<skill-name>/
├── SKILL.md              # Required: frontmatter + workflow instructions
├── evals/
│   └── evals.json        # Test cases for the skill
└── references/
    └── *.md              # Supplementary docs loaded on demand
```

### SKILL.md Frontmatter

```yaml
---
name: skill-name                  # lowercase, hyphens only, max 64 chars
description: >                    # What it does + when to trigger (max 1024 chars)
  ...
argument-hint: "[arg]"            # Optional: shown in /skill-name autocomplete
allowed-tools:                    # Required if skill uses specific tools
  - Bash(git *)
  - Read
  - Write
disable-model-invocation: true   # Required for skills with side effects (commit, deploy)
model: sonnet                     # Optional: override default model
---
```

**Key rules:**
- `disable-model-invocation: true` is required for any skill that writes files, runs git commands, or takes irreversible actions
- `allowed-tools` must explicitly list every tool the skill uses
- Reserved words `anthropic` and `claude` cannot appear in `name`

## Evals Format (`evals/evals.json`)

```json
{
  "skill_name": "skill-name",
  "evals": [
    {
      "id": 0,
      "prompt": "User prompt that triggers the skill",
      "expected_output": "Description of expected behavior",
      "files": [],
      "assertions": [
        {
          "id": "assertion-id",
          "description": "What to verify",
          "type": "contains_step | output_format | not_contains_step | contains_explanation"
        }
      ]
    }
  ]
}
```

## References Files

Referenced from SKILL.md body via relative links (e.g., `[best-practices.md](references/best-practices.md)`). They are loaded on demand, not upfront — keep SKILL.md concise and delegate details to references. Add a table of contents to reference files over 100 lines.

## Skill Quality Standards

The canonical checklist is at `skills/skill-reviewer/references/best-practices.md`. Key points:

- **SKILL.md body**: Keep under 500 lines; avoid explaining things the agent already knows
- **description**: Must explain what the skill does AND when to trigger it, with specific trigger phrases
- **Workflows**: Break complex operations into numbered steps with explicit conditions and validation
- **Progressive disclosure**: Move large reference content to `references/` files

## Key Skills for Development

- **`skill-composer`**: Orchestrates skill creation/maintenance — calls `skill-creator` then `skill-reviewer`
- **`skill-reviewer`**: Reviews a SKILL.md against `best-practices.md` and interactively fixes issues
- **`committer`**: Creates logically grouped commits (invoke by name only, not general "commit" requests)
- **`git-branch-cleaner`**: Safely deletes merged branches (invoke by name only)

## Working Directory Notes

- `.gitignore` excludes `*-workspace/` directories (used for eval output storage)
- `researcher-workspace/` holds evaluation runs and is git-ignored
