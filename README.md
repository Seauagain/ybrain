# ybrain

My toolkit — skills, hooks, memory templates, and project templates.

## Directory Structure

```
skills/       # Claude Code skills (each skill = one dir with SKILL.md)
hooks/        # Shell hook scripts (e.g., ears-trace)
memory/       # Reusable memory templates
templates/    # Project config templates (e.g., CLAUDE.md)
```

## Commit Convention

Every commit that adds or modifies a skill, hook, or template **must** follow this format:

```
<type>(<scope>): <description> [v<version>]
```

- **type**: `feat` | `fix` | `refactor` | `docs`
- **scope**: `skills` | `hooks` | `memory` | `templates`
- **version**: semver of the changed component, e.g., `v1.0`, `v1.1`, `v2.0`

Examples:

```
feat(skills): add brainstorm skill [v1.0]
fix(hooks): ears-trace handle empty file [v1.1]
refactor(templates): simplify CLAUDE.md template [v2.0]
```

This is enforced by a `commit-msg` hook — non-conforming commits will be rejected.
