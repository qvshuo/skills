---
name: custom-skills-updater
description: Manage manually installed skills (non-ClawHub). Supports checking updates, updating, and listing custom skills from GitHub or local sources.
---

# custom-skills-updater

Manages **manually installed skills not installed from ClawHub**.

Supported types: `github-dir`, `github-file`, `github-readme`, `local`

This skill **checks and updates existing skills only**. It does NOT create new skills.
For skill creation, delegate to `skill-creator`.

---

# Prerequisites

All GitHub operations require an authenticated `gh` CLI session.
Before any GitHub request, run `gh auth status`.
If it fails, prompt: `"Run gh auth login first."` and stop.

---

# Operations

## Check for updates

Scan `REGISTRY.yaml`, detect remote versions, compare with stored versions, report:

```
skill-name ........ up-to-date
skill-name ........ update available
```

## Update skills

Target all outdated skills or a specific skill by name.

1. Run update check
2. Update outdated skill(s)
3. Update `REGISTRY.yaml`

## List installed skills

Read and list all entries in `REGISTRY.yaml`.

---

# Registry

Location: `REGISTRY.yaml` in the same directory as this `SKILL.md`.

If it does not exist:

1. If `REGISTRY.yaml.template` exists, copy it to `REGISTRY.yaml`
2. Otherwise create with:
```yaml
skills: {}
```

**Do not rename the `skills` root key.**

## Format

Map structure keyed by skill name:

```yaml
skills:
  example-dir-skill:
    type: github-dir
    source: example-owner/example-repo@main:skills/example-dir-skill
    version: abc123
    updated: 2026-01-01
  example-readme-skill:
    type: github-readme
    source: example-owner/example-project@main
    version: def456
    updated: 2026-01-02
```

| Field   | Meaning |
|---------|---------|
| key     | skill name |
| type    | `github-dir` / `github-file` / `github-readme` / `local` |
| source  | source location |
| version | commit SHA (`github-dir`) or SHA256 (file-based types) |
| updated | last update date |

---

# Automatic Skill Discovery

Scan `skills/*/SKILL.md`. Only direct subdirectories of `skills/`, no recursion.

If a skill exists but is not in `REGISTRY.yaml`:

1. Notify the user and ask for source type and location
2. Add to registry

If unable to prompt, register as `local` and notify the user to configure later.

---

# Version Detection

Compare remote version against `version` in `REGISTRY.yaml`.

## github-dir

```
gh api "repos/{owner}/{repo}/commits?path={path}&per_page=1" --jq '.[0].sha // empty'
```

## github-file

```
gh api "repos/{owner}/{repo}/contents/{path}?ref={branch}" -H "Accept: application/vnd.github.raw+json" | shasum -a 256
```

## github-readme

Find README filename:

```
gh api "repos/{owner}/{repo}/contents/?ref={branch}" --jq '.[].name' | grep -i '^readme' | head -n 1
```

Take first match, download and hash:

```
gh api "repos/{owner}/{repo}/contents/{readme_filename}?ref={branch}" -H "Accept: application/vnd.github.raw+json" | shasum -a 256
```

## local

Skip entirely.

---

# Update Procedure

Only update when remote version differs from stored `version`.

## github-dir

```
gh api "repos/{owner}/{repo}/tarball/{branch}" > archive.tar.gz
```

Verify the file is valid gzip before extracting. Copy target path to `skills/{name}/`.

## github-file

```
gh api "repos/{owner}/{repo}/contents/{path}?ref={branch}" -H "Accept: application/vnd.github.raw+json" > skills/{name}/SKILL.md
```

## github-readme

1. Download README using the same method as version detection
2. If `skills/skill-creator/SKILL.md` exists, delegate conversion to it
3. Otherwise save README directly as `skills/{name}/SKILL.md`

---

# Error Handling and Safety

Handle `gh api` failures by HTTP status:

| Status | Action |
|--------|--------|
| 401 | `"Authentication expired. Run gh auth login."` Stop all operations. |
| 403 | `"Permission denied or rate limit for {skill-name}."` Skip. |
| 404 | `"Source not found for {skill-name}."` Skip. |
| Other | `"Check failed for {skill-name}."` Skip. |

On any failure: do NOT overwrite local files, do NOT modify registry.

Registry updates: modify only the target entry, do not reorder or remove others, update `version` and `updated` only after success.

**This skill manages manually installed skills only. ClawHub-installed skills are out of scope.**
