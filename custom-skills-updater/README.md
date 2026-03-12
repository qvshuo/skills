# custom-skills-updater

Manage **manually installed skills** that are **not installed from ClawHub**.

This skill can:

* check for updates
* update outdated skills
* list installed skills

Supported sources:

* `github-dir` — skill stored in a repository directory
* `github-file` — skill stored as a single file
* `github-readme` — skill generated from a repository README
* `local` — locally created skill (no remote source)

This skill **manages existing skills only**.
For generating skills from repositories or READMEs, use `skill-creator`.

## Requirement

Install GitHub CLI and authenticate:

```
gh auth login
```

All remote operations use `gh api`.

## Registry

Installed skills are tracked in:

```
REGISTRY.yaml
```

located in the same directory as `SKILL.md`.

Example:

```yaml
skills:
  example-dir-skill:
    type: github-dir
    source: example-owner/example-repo@main:skills/example-dir-skill
    version: abc123
    updated: 2026-01-01
```

A example file `REGISTRY.example.yaml` is included.

## Typical Workflow

1. Install a skill manually
2. Register it in `REGISTRY.yaml`
3. Use this skill to check or update it

## Scope

Only **manually installed skills** are managed.
Skills installed from **ClawHub** are ignored.

## License

MIT
